---
title: "Testing AI Agents with Evals"
date: 2026-09-17
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
  image: "/images/agentscript_duplicate_Banner.webp"
  alt: "Two-Tiered Evaluation Harness Architecture for AI Agents"
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# Testing AI Agents with Evals: A Two-Tiered Evaluation Harness

When building production AI agents, traditional software testing practices fall apart.

Standard unit tests rely on deterministic assertions: `assert response == "expected output"`. But LLM agents generate non-deterministic language, adapt reasoning on the fly, and interact with external tools and retrieval pipelines. A minor phrasing difference can break a brittle regex or string match, even if the agent's decision was 100% correct.

On the other hand, relying exclusively on LLM-as-a-judge evaluations for everything creates a new set of problems: high API latency, ballooning token costs, and flaky CI/CD pipelines.

For this project, I built an evaluation harness to solve a practical challenge:

> **How do we build a robust, production-grade test suite for AI agents that catches hallucinations and business rule violations without blowing up CI/CD runtime or burning API budgets?**

The solution is a **Two-Tiered Evaluation Architecture**: enforce deterministic structure first, and only invoke LLM judges when semantic evaluation is truly needed.

---

## What I Built

I created a clean, production-ready evaluation harness modeling an **automated customer support triage agent**. The agent is responsible for evaluating customer return/refund requests against store policies, choosing an action (`issue_refund`, `reject_policy`, or `escalate_to_human`), and generating a customer reply.

The evaluation harness tests execution traces across two distinct layers:

1. **Tier 1: Deterministic Syntax & Business-Rule Guardrails (Pydantic)**  
   Validates JSON structure, field ranges, and strict business invariants instantly in memory with zero API latency and zero cost.

2. **Tier 2: Probabilistic Semantic Evaluation (DeepEval + DeepSeek Judge)**  
   Evaluates nuances like **Faithfulness** (detecting policy hallucinations against retrieval context) and **Answer Relevancy** (catching conversational drift) using LLM-as-a-judge.

3. **Fast-Fail Cost Optimization**  
   If an execution trace fails Tier 1, the harness automatically skips Tier 2, saving token costs and keeping test runs fast.

---

## The Architecture

```
                    ┌───────────────────────────────┐
                    │     Agent Execution Trace     │
                    │   (Action, Amount, Reply)    │
                    └───────────────┬───────────────┘
                                    │
                                    ▼
                    ┌───────────────────────────────┐
                    │            TIER 1             │
                    │   Deterministic Guardrails    │
                    │      (Pydantic Schema)        │
                    └───────┬───────────────┬───────┘
                            │               │
                     [PASS] │               │ [FAIL]
                            ▼               ▼
            ┌───────────────────────┐   ┌───────────────────────┐
            │        TIER 2         │   │   Fast-Fail Skip      │
            │   Semantic Evals      │   │ (Zero Token Burn)     │
            │      (DeepEval)       │   └───────────────────────┘
            └───────┬───────┬───────┘
                    │       │
      Faithfulness  │       │ Answer Relevancy
      (Hallucination)       (Directness & Precision)
```

---

## 1. Tier 1: Deterministic Guardrails with Pydantic

Before spending time or money asking an LLM judge to evaluate text, we should enforce everything that can be validated deterministically.

In the harness, a data contract validates field formats, score boundaries (`0.0` to `1.0`), and three strict business rules:

```python
# Tier 1 Guardrails (Pseudocode)
validate_trace(trace):
    assert trace.action in ["issue_refund", "escalate_to_human", "reject_policy"]
    assert 0.0 <= trace.confidence_score <= 1.0

    # Rule 1: A rejection must never authorize a payout
    if trace.action == "reject_policy" and trace.refund_amount > 0:
        fail("Rejected refunds cannot authorize payouts")

    # Rule 2: An authorized refund requires a traceable order ID
    if trace.action == "issue_refund" and not trace.order_id:
        fail("Cannot issue a refund without a valid order_id")

    # Rule 3: High-confidence actions should not waste human triage resources
    if trace.action == "escalate_to_human" and trace.confidence_score >= 0.85:
        fail("High-confidence cases must not be routed to human escalation")
```

_(In production, this is enforced in `schemas.py` using Pydantic's `@model_validator`.)_

### Why this matters:

- Runs in **under 10 milliseconds**.
- Catches catastrophic bugs (e.g., an agent rejecting a claim but authorizing a $50 payout).
- Requires **zero external API calls**.

---

## 2. Tier 2: Probabilistic Semantic Evaluation with DeepEval

Syntax validation alone cannot tell you if an agent lied to a customer or ignored their question. That is where Tier 2 comes in.

Using an LLM judge (configured via DeepEval with DeepSeek's fast, low-cost `deepseek-chat`), we evaluate two semantic dimensions against a quality threshold:

```python
# Tier 2 Semantic Judge (Pseudocode)
evaluate_semantics(customer_input, agent_reply, policy_context):
    # Metric A: Faithfulness — Did the agent invent policies outside the retrieved context?
    faithfulness_score = llm_judge.check_grounding(agent_reply, against=policy_context)
    assert faithfulness_score >= 0.70

    # Metric B: Relevancy — Did the agent directly answer the customer's actual question?
    relevancy_score = llm_judge.check_relevancy(agent_reply, to=customer_input)
    assert relevancy_score >= 0.70
```

- **Faithfulness (Hallucination Detection):** Verifies that every claim in the customer reply is strictly grounded in the retrieved store policy, catching fabricated return windows.
- **Answer Relevancy (Directness & Precision):** Ensures the agent directly answers the customer's query rather than dodging the question or pivoting to unrelated topics.

---

## 3. The Fast-Fail Cost Optimization Pattern

Running LLM evaluations on invalid traces wastes money and slows down developer iteration. The harness places a gatekeeper between the two tiers:

```python
# Fast-Fail Optimization (Pseudocode)
for trace in traces:
    if not passes_tier1_guardrails(trace):
        # Stop immediately — don't waste LLM tokens on structurally broken outputs
        skip_trace("Failed Tier 1: Skipping LLM judge to save API cost & time")
        continue

    run_tier2_semantic_evals(trace)
```

If an agent trace produces an illegal payout or missing field, it is rejected at the schema level and never touches the LLM judge.

---

## Testing Scenarios: What the Harness Catches

To validate the harness, I built a dataset of realistic agent execution traces in `dataset/traces.json`:

| Trace ID                                   | Scenario                                                                          | Tier 1 (Pydantic)           | Tier 2 (DeepEval)                            | Result           |
| :----------------------------------------- | :-------------------------------------------------------------------------------- | :-------------------------- | :------------------------------------------- | :--------------- |
| `pass_valid_refund`                        | Valid refund under 30-day damaged goods policy                                    | **PASS**                    | **PASS** (Faithfulness: 1.0, Relevancy: 1.0) | Passed           |
| `pass_valid_rejection`                     | Valid rejection for 6-month-old return                                            | **PASS**                    | **PASS** (Faithfulness: 1.0, Relevancy: 1.0) | Passed           |
| `pass_valid_escalation`                    | Complex duplicate charge / fraud dispute routed to humans at low confidence       | **PASS**                    | **PASS** (Faithfulness: 1.0, Relevancy: 1.0) | Passed           |
| `pass_goodwill_partial_refund`             | Discretionary goodwill credit for late delivery with package damage               | **PASS**                    | **PASS** (Faithfulness: 1.0, Relevancy: 1.0) | Passed           |
| `fail_pydantic_schema_business_rule`       | Agent rejects return but issues a $30 payout (Rule 1)                             | **FAIL** (Rule 1 Violation) | **SKIPPED** (Fast-Fail)                      | Caught at Tier 1 |
| `fail_pydantic_refund_missing_order_id`    | Agent issues refund without order ID, breaking audit traceability (Rule 2)        | **FAIL** (Rule 2 Violation) | **SKIPPED** (Fast-Fail)                      | Caught at Tier 1 |
| `fail_pydantic_high_confidence_escalation` | Agent unnecessarily routes clear-cut policy to humans at 0.91 confidence (Rule 3) | **FAIL** (Rule 3 Violation) | **SKIPPED** (Fast-Fail)                      | Caught at Tier 1 |
| `fail_deepeval_hallucination`              | Agent invents a fake "180-day worldwide return" policy                            | **PASS** (Syntax valid)     | **FAIL** (Faithfulness: 0.0)                 | Caught at Tier 2 |
| `fail_deepeval_relevancy`                  | Customer asks about opened toner; agent talks about shipping                      | **PASS** (Syntax valid)     | **FAIL** (Relevancy: 0.0)                    | Caught at Tier 2 |

---

## Running the Evals

Because the harness integrates natively with Pytest, you can run the entire evaluation suite using either standard `pytest` or the `deepeval` CLI:

```bash
# Standard Pytest (ideal for headless CI/CD pipelines)
pytest test_evals.py -v

# Or via DeepEval CLI (with rich terminal UI and score tables)
deepeval test run test_evals.py
```

### The Output:

DeepEval outputs a rich CLI summary highlighting exact metric scores, pass/fail status, token costs, and LLM judge reasoning:

```text
================================================================================
Test case: fail_deepeval_hallucination
Metric: Faithfulness | Score: 0.0 (threshold=0.7) | Status: FAILED
Reason: The score is 0.00 because the actual output directly contradicts the
retrieval context by claiming that international items can be returned within 180 days,
whereas the retrieval context explicitly states that international sales cannot be returned.
================================================================================
✓ Evaluation completed (time taken: 12.4s | token cost: $0.00057 USD)
```

The entire test suite ran for **less than $0.0006**, providing full visibility into both structural and semantic compliance.

---

## Why This Approach Works

1. **Separation of Concerns:**  
   Code should validate rules; LLMs should validate language. Using Python and Pydantic for business invariants keeps rules testable, transparent, and deterministic.
2. **Cost & Latency Efficiency:**  
   By gating LLM judge calls behind fast-fail guardrails and choosing efficient models like DeepSeek, running evals on every pull request becomes completely viable.
3. **Actionable Failure Explanations:**  
   Instead of a generic test failure, the harness tells you whether the issue was a broken contract, a hallucinated policy, or an evasive response.

---

## Key Takeaways

- **Don't use string matches for LLM outputs:** Use semantic metrics like Faithfulness and Answer Relevancy to test probabilistic behavior.
- **Don't use LLMs for things Python can check:** Enforce business invariants and schema contracts in code.
- **Fast-fail early:** Save token costs and reduce CI time by skipping LLM judges when deterministic guardrails fail.

---

## The Bottom Line

Evaluating AI agents doesn't have to be a choice between brittle regexes and expensive, slow LLM pipelines. By combining deterministic Pydantic guardrails with targeted DeepEval semantic checks, you get a fast, reliable, and cost-effective harness ready for production CI/CD.
