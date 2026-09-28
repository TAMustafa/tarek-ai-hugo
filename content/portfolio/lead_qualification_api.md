---
title: "Building a Deterministic Lead Qualification API for Salesforce"
date: 2026-09-28
tags: ["FastAPI", "Salesforce", "Postgres", "SerpAPI"]
author: "Tarek Mustafa"
draft: false
description: "How I built a production-ready API that resolves, deduplicates, enriches, and qualifies restaurant and grocery leads before writing them to Salesforce."
hidemeta: false
comments: false
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
  image: "/images/lead_qualification_API_banner.webp"
  alt: "Two-Tiered Evaluation Harness Architecture for AI Agents"
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# Building a Deterministic Lead Qualification API for Salesforce

Lead qualification sounds simple: receive a business name and address, look up the company, and create a Lead in Salesforce.

The difficult part starts when the same restaurant arrives from two sources, the address is written differently, a search returns several similar businesses, or two requests reach the API at the same time.

A system that guesses incorrectly can enrich the wrong company. A system that checks for duplicates too late can create two Salesforce records. A system without an audit trail cannot explain why a lead was accepted or rejected.

So the aim of this project was to solve a practical problem:

> **How do we qualify restaurant and grocery leads quickly without sacrificing deterministic behavior, duplicate protection, or auditability?**

The result is a modular Python API that resolves the business, checks for existing records, enriches unique leads, applies explicit qualification rules, and only then writes to Salesforce.

No LLM is used for matching, deduplication, or qualification.

---

## What I Built

I built a versioned API around a single main endpoint:

```text
POST /v1/leads/qualify
```

The minimum input is deliberately small:

```json
{
  "source": "GRID",
  "source_lead_id": "grid-12345",
  "business_name": "Example Restaurant",
  "address": "Palmpolstraat 1, 1327 CA Almere"
}
```

The API can also accept a known Google Place ID and an international phone number. If the Place ID is missing, the service searches for the most likely business using its name and address.

Every request ends in one of four machine-readable decisions:

- `QUALIFIED`
- `REJECTED`
- `REVIEW_REQUIRED`
- `DUPLICATE`

The response includes reason codes, resolution confidence, normalized enrichment, duplicate details, Salesforce identifiers, and processing metadata.

The same workflow is available for CSV files containing up to 100 leads. This made it possible to test one restaurant in the local interface and then use the same API for a small batch without building a separate import process.

---

## The Architecture

I kept the first implementation as a **modular monolith**. The workflow has several distinct responsibilities, but they do not need separate services or deployment pipelines yet.

![Lead emrichment API Flow](/images/enrichment_api_flow.webp)

The code is separated into modules for place resolution, deduplication, enrichment, address handling, qualification, Salesforce mapping, Salesforce persistence, and PostgreSQL state.

This structure keeps the workflow easy to follow while allowing individual adapters to be replaced later. For example, BigQuery is treated as the existing-partner source behind a small lookup interface. A lower-latency duplicate index can replace it without rewriting the qualification service.

---

## Resolve Carefully or Ask for Review

When a Place ID is supplied, the API uses it directly. When it is missing, the resolver searches by business name and address.

The important design decision is that the resolver does not simply select the first search result.

Candidates receive a deterministic score based on normalized name and address similarity. A match must clear both a minimum score and a minimum margin over the next candidate. If the evidence is weak or two candidates are too close, the API returns `REVIEW_REQUIRED`.

```python
if best_score < minimum_score:
    return REVIEW_REQUIRED

if best_score - second_best_score < minimum_margin:
    return REVIEW_REQUIRED

return best_candidate
```

This is more conservative than guessing, but that is intentional. A missed automatic match creates a manual task. A wrong automatic match can attach the phone number, website, rating, and cuisine of an entirely different business to Salesforce.

---

## Deduplicate Before Enrichment

The API checks for duplicates before requesting detailed enrichment.

It queries:

- Salesforce Leads
- Salesforce Accounts
- The existing-partner source

Google Place ID is the strongest duplicate key. Normalized phone, address, and business name provide secondary evidence.

If an existing entity is found, the API returns `DUPLICATE` with its system, object type, record ID, and matching reason. It does not make an unnecessary enrichment call and does not update the existing record.

This ordering reduces provider usage and prevents new enrichment from hiding the more important fact that the business already exists.

---

## Why PostgreSQL Is Still Needed

I initially questioned whether the API needed its own database. Salesforce already stores the final Lead, and external providers already store the business data.

The problem is the short period before Salesforce contains the new record.

Imagine two API replicas receiving the same restaurant at the same time:

1. Both check Salesforce.
2. Neither finds an existing Lead.
3. Both qualify the business.
4. Both create a Salesforce record.

PostgreSQL closes that race window. It stores the immutable internal `lead_id`, idempotency fingerprints, advisory locks, identity claims, write reservations, decisions, and audit metadata.

Before the external Salesforce write, the API commits a durable reservation for the Place ID and secondary identity signals. Salesforce is then updated using a stable external key. If a timeout occurs after the remote write, a retry can recover the previous outcome instead of creating a second Lead.

---

## Canonical Enrichment Instead of Provider Objects

The current test setup uses SerpAPI's Google Local results. The provider adapter converts the response into an internal Pydantic model rather than passing provider-specific JSON to Salesforce.

The canonical model includes fields such as:

- Operational status
- Business and place types
- Primary and secondary cuisine
- Phone and website
- Rating and review count
- Latest observed review date
- Delivery, takeaway, and dine-in signals
- Payment and service options
- Opening hours
- Price range
- Address and coordinates

Cuisine classification is also deterministic. Explicit provider types are mapped through a configurable taxonomy. For example, `italian_restaurant` can produce `Italian` as the primary cuisine, while an additional `pizza_restaurant` signal can produce `Pizza` as the secondary cuisine.

The API does not infer `Pasta` merely because a restaurant is Italian. Secondary cuisine is only added when the provider returns supporting evidence.

This canonical layer keeps Salesforce independent from the exact SerpAPI or Google response format and makes provider changes easier to test.

---

## Checking for Residential Premises

A restaurant lead can have a valid-looking address while still pointing to a residential property.

For complete Dutch addresses, I added an optional check against the Kadaster BAG data exposed through PDOK. The checker resolves the exact address and inspects the registered use of the premises.

- Residential-only use returns `REJECTED`.
- Explicit non-residential use can continue.
- Mixed use, incomplete addresses, ambiguity, or provider failure returns `REVIEW_REQUIRED`.

This distinction matters because BAG describes the registered use of a property. It is useful evidence, but it is not proof of current commercial activity. The API therefore avoids treating uncertain data as a confident rejection.

---

## Deterministic Qualification Rules

Once enrichment is complete, normal Python rules make the qualification decision.

A lead can be rejected when:

- The business is permanently closed.
- The registered premises are residential-only.
- The business type is outside the configured restaurant and grocery categories.
- Rating or review count falls below the configured threshold.

A lead requires review when critical evidence is missing or uncertain, such as operational status, business type, reputation data, or premises classification.

```python
if business_status == "CLOSED_PERMANENTLY":
    reject("BUSINESS_CLOSED_PERMANENTLY")

if premises_status == "RESIDENTIAL":
    reject("BAG_RESIDENTIAL_PREMISES")

if rating is None or review_count is None:
    review("REPUTATION_DATA_MISSING")

if rating < minimum_rating:
    reject("RATING_BELOW_THRESHOLD")
```

Each result contains reason codes and a rules version. This makes the decision explainable and allows a future reviewer to identify which rules produced it.

---

## Salesforce Without Hard-Coded Transformations

The final canonical data is mapped to Salesforce through JSON configuration.

Each mapping defines:

- The Salesforce field API name
- The canonical source value
- The expected data type
- Maximum length
- Required status
- Allowed picklist values
- Optional value translations

```json
"Primary_Cuisine__c": {
  "source": "enrichment.primary_cuisine",
  "type": "string",
  "max_length": 100
}
```

The application validates the mapping before writing. When persistence is enabled, startup also checks the live Salesforce schema for field names, types, sizes, permissions, picklist values, and external-ID configuration.

This makes customization straightforward. A user can add or remove a Salesforce field in the mapping without changing the orchestration code. If a new provider value is required, it must first be added deliberately to the canonical model and covered by contract tests.

---

## A Simple Local Test Interface

I added a small local-only web interface for development.

The user enters a restaurant name and address and can see the workflow progress through validation, identity, resolution, deduplication, enrichment, qualification, mapping, and persistence.

The page is intentionally simple. It hides optional processing details and CSV import until they are needed, while the API response remains fully detailed for technical users.

The development interface uses an existing Salesforce CLI login, pins the selected development org locally, and refuses to switch silently to another org. Credentials never reach the browser.

---

## Testing the Workflow

The test suite covers the individual rules and the complete workflow.

It includes:

- Unit tests for normalization, cuisine classification, matching, mapping, and qualification
- Contract tests for provider responses and Salesforce field mappings
- PostgreSQL integration tests for idempotency, concurrency, reservations, and recovery
- Demo integration tests for duplicate prevention and Salesforce writes
- Opt-in live tests protected by an expected Salesforce development-org ID

![Lead emrichment API Test Interface](/images/lead_enrichment_api_test.webp)
![Lead creation in Salesforce](/images/lead_enrichment_api_salesforce.webp)

At the time of writing, the project contains 109 passing tests. The Salesforce metadata also passes a dry-run deployment against the development org.

The live tests use synthetic provider data and only create records with unique test identities. Cleanup verifies those identities before deleting anything.

---

## Why This Approach Works

The design separates responsibilities clearly:

- FastAPI and Pydantic validate the API boundary.
- PostgreSQL owns identity, idempotency, coordination, and audit metadata.
- Provider adapters handle external response formats.
- Canonical models isolate business logic from vendors.
- Python rules make deterministic qualification decisions.
- Salesforce mappings validate the final CRM contract.
- Durable reservations and external-ID upserts protect the write boundary.

The system fails safely. Uncertain matches become review tasks. Provider failures do not become false duplicate clearances. Mapping errors do not produce malformed Salesforce writes.

---

## Key Takeaways

**Check duplicates before enrichment.** It reduces external calls and avoids doing work for a business that already exists.

**Do not guess when identity is uncertain.** `REVIEW_REQUIRED` is a useful outcome, not a failure.

**Idempotency alone does not prevent every duplicate.** Concurrent requests also need shared locks, durable reservations, and an idempotent destination write.

**Normalize provider data before mapping it to a CRM.** Salesforce should receive a stable business schema rather than a vendor response.

**Use code for deterministic business rules.** An LLM is unnecessary for matching thresholds, duplicate keys, cuisine taxonomies, or qualification criteria.

**Keep the first architecture simple.** A modular monolith with clear interfaces is easier to operate and can still support horizontal scaling.

**Treat external data as evidence.** Missing, mixed, or ambiguous data should lead to review instead of a confident but unsupported decision.

> If you are interested, the source code and setup instructions are available in this [GitHub repository](https://github.com/TAMustafa/lead-enrichment-api).
