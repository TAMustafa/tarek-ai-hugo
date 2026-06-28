---
title: "Reddit Insight Automation with n8n"
date: 2025-09-28
tags: ["n8n", "Anthropic", "Reddit", "Automation"]
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
  image: "/images/RedditN8NBanner.webp"
  alt: "Image showing an n8n workflow for finding fintech issues on Reddit" # alt text
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# Finding Fintech Issues on Reddit with n8n

In financial services, useful customer signals are often buried inside community discussions. People talk about payment issues, banking frustrations, failed onboarding, app bugs, and support problems long before those signals show up in a formal report.

For this project, I built an automated **n8n workflow** that scans finance-related subreddits, filters for relevant posts, uses AI to summarize the issue, and sends the results to **Google Sheets**.

---

## How It Works

![n8n and Reddit Workflow](/images/n8nWorkflow.webp)

The system automates the pipeline from raw Reddit posts to a structured list of user-reported issues.

### 1. Multi-Source Data Collection

The workflow queries multiple finance-focused subreddits to capture a broader market view:

- **r/FinTech:** Financial technology discussions and product pain points.
- **r/Banking:** Traditional banking challenges and customer feedback.
- **r/PaymentProcessing:** More specific payment and operational issues.

### 2. Filtering and Deduplication

The raw data is cleaned before it reaches the AI step:

- **Merge and deduplicate:** Combine results from all sources and remove duplicate posts.
- **Relevance filtering:** Keep recent posts that contain terms like "issue," "problem," or "solution."
- **Time window:** Focus on the last 72 hours so the output stays current.

### 3. AI-Powered Analysis

Each filtered post is sent to **Anthropic Claude** for analysis. The prompt asks the model to:

- Assess relevance and urgency.
- Extract the core problem.
- Capture any suggested solution.
- Generate a short, actionable summary.

### 4. Structured Output

The final insights are formatted with n8n's output parser and appended to a **Google Sheet**. This creates a clean and searchable list of pain points that can be shared with product, support, or strategy teams.

---

## Technical Architecture

**Automation & Orchestration**

- **n8n:** Serves as the core workflow engine and orchestrates the pipeline.

**Data Sources & AI**

- **Reddit API:** Provides access to discussions from targeted subreddits.
- **Anthropic Claude:** Turns long posts into structured summaries and issue labels.

**Storage & Output**

- **Google Sheets:** Acts as the storage layer and gives non-technical users an easy way to review the results.

---

## Key Takeaways

- **Product intelligence:** The workflow provides a repeatable way to discover user frustrations and feature requests.
- **Efficiency:** Analysts no longer need to manually scan several communities every day.
- **Early signal detection:** Repeated complaints can surface before they become larger support or product issues.

## The Bottom Line

This workflow turns public community discussion into structured research. It is not a replacement for direct customer feedback, but it is a useful early-warning layer for fintech product and support teams.
