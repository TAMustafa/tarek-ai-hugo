---
title: "Testing AI Agents with Evals"
date: 2026-09-16
tags: ["Pydantic", "DeepEval", "Agent Testing"]
author: "Tarek Mustafa"
draft: false
description: "Why testing AI agents requires more than standard unit tests, and how to build a cost-effective two-tiered evaluation harness using Pydantic and DeepEval."
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
  image: "/images/Evals_Banner.webp"
  alt: "Two-Tiered Evaluation Harness Architecture for AI Agents"
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# Testing AI Agents with Evals

AI agents are probabilistic. The wording can change between runs, the agent may take different paths through its tools, and a slightly different response doesn't necessarily mean the agent made a wrong decision. This mmakes it challenging for traditional software testing to produce reliable and actionable results and findings.

Using an LLM to judge every test isn't the answer either. It adds latency, cost, and another source of variability to the test suite.

So the aim of this project was to solve a practical problem:

> **How do we build a reliable evaluation suite for AI agents without turning every test into an expensive LLM call?**

The approach I built is a **two-tiered evaluation harness**:

**deterministic checks first, semantic evaluation second.**

---

## What I Built

I built the test around an **automated customer support triage agent**.

The agent evaluates return and refund requests against store policies and decides whether to:

- `issue_refund`
- `reject_policy`
- `escalate_to_human`

It also generates the response to the customer.
The evaluation checks each execution trace in two stages:

**Tier 1 — Deterministic Guardrails**

Pydantic **(what does pydantic mean in this context)** validates the output structure and enforces business rules such as:

- Valid actions only
- Confidence scores between `0.0` and `1.0`
- Rejected refunds cannot authorize a payout
- Refunds require an order ID

**Tier 2 — Semantic Evaluation**

DeepEval **(link and explanation what it is)** evaluates things that are harder to express as deterministic rules:

- **Faithfulness** — is the response grounded in the retrieved policy context?
- **Answer Relevancy** — does the response actually address the customer's question?

If Tier 1 fails, Tier 2 is skipped.

> That gives us a simple **fast-fail mechanism**: don't spend LLM tokens evaluating an output that has already failed a basic business rule.

---

## The Architecture

The idea is as follows:

![Agent Excecution](/images/AgentExcecution.webp)

**Use code for deterministic rules and LLMs for semantic evaluation.**

---

## Tier 1: Deterministic Guardrails

Before calling an LLM judge, I validate everything that can be checked with normal code.
A simplified version looks like this:

```python
validate_trace(trace):

    assert trace.action in [
        "issue_refund",
        "escalate_to_human",
        "reject_policy"
    ]

    assert 0.0 <= trace.confidence_score <= 1.0

    # A rejection must never authorize a payout
    if trace.action == "reject_policy" and trace.refund_amount > 0:
        fail("Rejected refunds cannot authorize payouts")

    # A refund requires an order ID
    if trace.action == "issue_refund" and not trace.order_id:
        fail("Cannot issue a refund without a valid order_id")

    # High-confidence cases should not be escalated
    if trace.action == "escalate_to_human" \
            and trace.confidence_score >= 0.85:
        fail("High-confidence cases must not be escalated")
```

These checks are **deterministic, fast, and require no external API calls**.
More importantly, the rules are explicit and easy to understand.
If the business rule says a rejected refund cannot contain a payout, **Python should enforce that rule — not another LLM.**

---

## Tier 2: Semantic Evaluation

A structurally valid response can still be wrong.
For example, the agent could return a perfectly valid refund decision but tell the customer that the store offers a 180-day return policy when the retrieved policy says otherwise.
This is where LLM-based evaluation becomes useful.
With DeepEval, I evaluate the response for **Faithfulness** and **Answer Relevancy**:

```python
faithfulness_score = llm_judge.check_grounding(
    agent_reply,
    against=policy_context
)

assert faithfulness_score >= 0.70

relevancy_score = llm_judge.check_relevancy(
    agent_reply,
    to=customer_input
)

assert relevancy_score >= 0.70
```

The important distinction is that Faithfulness evaluates whether the response is supported by the supplied context. It isn't a general-purpose fact checker.
This makes it particularly useful for RAG-based agents where the question is:
**"Did the agent stay within the information it retrieved?"**

---

## The Fast-Fail Pattern

The two tiers are connected by a simple gate:

```python
for trace in traces:

    if not passes_tier1_guardrails(trace):
        skip_trace("Tier 1 failed")
        continue

    run_tier2_semantic_evals(trace)
```

This means an invalid trace never reaches the LLM judge.

For example:

![Agent Evaluation](/images/AgentEvaluation.webp)

This setup has practical benefits when the evaluation dataset grows.

---

## What the Evaluation Catches

I tested the setup with both valid and intentionally broken traces.

| Scenario                                  | Tier 1   | Tier 2   | Result |
| ----------------------------------------- | -------- | -------- | ------ |
| Valid refund                              | PASS     | PASS     | Passed |
| Valid rejection                           | PASS     | PASS     | Passed |
| Valid escalation                          | PASS     | PASS     | Passed |
| Rejection with $30 payout                 | **FAIL** | Skipped  | Tier 1 |
| Refund without order ID                   | **FAIL** | Skipped  | Tier 1 |
| High-confidence escalation                | **FAIL** | Skipped  | Tier 1 |
| Invented 180-day return policy            | PASS     | **FAIL** | Tier 2 |
| Response doesn't answer customer question | PASS     | **FAIL** | Tier 2 |

This is the part I find most useful.

The evaluation suite doesn't just tell me that something failed. It tells me **what type of failure occurred**.

---

## Running the Evals

Because DeepEval integrates with Pytest, the evaluation suite can run as part of a normal CI/CD pipeline.

```bash
deepeval test run test_evals.py
```

A semantic failure can then provide both a score and an explanation, for example:

```text
Metric: Faithfulness
Score: 0.0
Threshold: 0.7
Status: FAILED

Reason:
The response claims that international purchases can be
returned within 180 days, but the retrieved policy states
that international sales cannot be returned.
```

That's considerably more useful than a simple:

```text
AssertionError: test failed
```

---

## Why This Approach Works

The approach separates responsibilities:

Code handles deterministic rules such as schemas, required fields, and business logic.
LLM evaluations handle semantics such as grounding and relevancy.
Fast-fail avoids unnecessary LLM calls when deterministic checks already fail.

## Key Takeaways

- **Don't use string matching for everything.** LLM outputs are probabilistic, so semantic evaluation is often more appropriate.
- **Don't use LLMs for things Python can check.** Keep business rules deterministic.
- **Fail fast.** Skip expensive semantic evaluations when basic validation already fails.
- **Test more than the final answer.** For AI agents, the execution trace, tool calls, decisions, and retrieved context can all become part of the evaluation.
