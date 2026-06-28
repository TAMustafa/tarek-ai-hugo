---
title: "Lead Enrichment with Python and SerpAPI"
date: 2026-04-11
tags: ["Salesforce", "Python", "SerpAPI", "Lead Enrichment"]
author: "Tarek Mustafa"
draft: false
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
  image: "/images/python_salesforce_lead_Banner.webp"
  alt: "Image showing a cartoonish image about using Agentforce to search for lead data"
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# Enriching Salesforce Leads with Place IDs

I built a Python-driven automation that enriches Salesforce Leads using Google Place IDs as a stable identifier for businesses on Google Maps.

The goal was to move beyond manual lead research. If a Lead already has a Place ID, the script can use that identifier to fetch structured business details and sync the useful fields back into Salesforce.

---

## The Three Ingredients

### 1. Salesforce Lead Object Configuration

To prepare Salesforce for enrichment, I added custom fields to the Lead object. This included a Place ID field to serve as the lookup key, along with fields for Google rating, review count, website, and other useful business details.

These fields give the external data a structured landing spot instead of leaving it buried in notes or spreadsheets.

![Lead record with custom fields](/images/LeadBeforeRun.webp)

### 2. Python & Google Maps API Integration

The engine is a Python script that connects Salesforce to Google Maps data through SerpAPI. The logic follows a simple pipeline:

- **Authentication:** Connect to Salesforce using `simple_salesforce`.
- **Query:** Find Leads that have a Place ID and still need enrichment.
- **Enrichment:** Call SerpAPI to fetch business details for each Place ID.
- **Mapping:** Convert API response fields into the Salesforce fields created for the project.

![Python File Image](/images/AutomationPython.webp)

### 3. Automated Update Logic

The script does not just find data; it manages the record update. After fetching details like rating, review count, and website, the automation maps the API response back to Salesforce and performs an update.

That keeps the CRM closer to a living database instead of a static list of partially filled records.

---

## How It Works in the Pipeline

Once the Python script runs in the Colab environment, it performs four steps:

- **Extraction:** Pull Leads from Salesforce that are marked for enrichment.
- **Transformation:** Parse the JSON response and extract the business details that matter.
- **Validation:** Check whether fields like website or rating exist before updating, so empty values do not overwrite useful data.
- **Sync:** Push the processed data back to Salesforce so it appears directly on the Lead page.

![Lead after enrichment](/images/LeadAfterRun.webp)

---

## Key Takeaways

Using SerpAPI and `simple_salesforce` makes the enrichment process repeatable. The most important design choice was using Place ID as the key, because names and addresses can be messy while Place IDs are much more stable.

This gives sellers immediate context on a Lead's online presence without asking them to leave Salesforce and research everything manually.

---

## The Bottom Line

This setup turns lead research into a background process. By combining Salesforce custom fields, Python, and a reliable external identifier, the CRM becomes more useful without adding extra manual work for the sales team.
