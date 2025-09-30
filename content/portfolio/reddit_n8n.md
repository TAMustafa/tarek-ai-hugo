---
title: "n8n automation"
date: 2025-09-28
# weight: 1
# aliases: ["/first"]
tags: ["n8n", "Anthropic", "Reddit", "Automation"]
author: "Tarek Mustafa"
draft: false
hidemeta: false
comments: false
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "/images/RedditN8NBanner.png"
    alt: "Image showing a cartoonish image about using a local RAG" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---

# n8n Workflow to find Fintech issues reported on Reddit

In the fast-paced world of financial services, critical user insights and emerging issues are often buried in scattered community discussions across Reddit and other forums. Manually tracking these conversations is time-consuming.
This project demonstrates a scalable solution: an automated **n8n workflow** that continuously scans key financial subreddits, uses AI to extract user-reported issues, and delivers summarized, actionable insights directly to **google sheets**.

## How It Works: From Data Chaos to Structured Insight

![n8n and Reddit Workflow](/images/n8nWorkflow.png)

The system automates the entire intelligence-gathering pipeline, from raw posts to analyzed trends.

1.  **Multi-Source Data Collection**
    The workflow simultaneously queries multiple finance-focused subreddits to capture a broad market view:
    *   **r/FinTech** for innovative financial technology insights and pain points.
    *   **r/Banking** for traditional banking challenges and customer feedback.
    *   **r/PaymentProcessing** for specific operational and technical issues.

2.  **Intelligent Data Processing**
    The raw data is cleaned and refined for analysis:
    *   **Merge & Deduplicate:** Combines results from all sources and removes duplicate posts.
    *   **Relevance Filtering:** Applies smart filters to focus only on the most valuable content.

  > The workflow filters for posts from the last 72 hours that contain keywords like "issue," "problem," or "solution," ensuring the analysis focuses on actionable discussions.

3.  **AI-Powered Analysis & Summarization**
    Each filtered post is sent to **Anthropic's Claude model** for deep analysis, where it:
    *   Assesses the overall relevance and urgency.
    *   Extracts the core problem and any proposed solutions.
    *   Generates a concise, actionable summary.
    *   Helps detect emerging trends across multiple discussions.

4.  **Structured Output & Storage**
    The final, AI-processed insights are formatted using n8n’s output parser and automatically appended to a **Google Sheet**, creating:
    *   A clean, searchable database of user-reported pain points.
    *   An easily shareable resource for product, support, and strategy teams.
    *   A growing historical knowledge base for trend analysis.

## Technical Architecture

**Automation & Orchestration**
*   **n8n:** Serves as the core workflow engine, orchestrating the entire pipeline.

**Data Sources & AI**
*   **Reddit API:** Provides direct access to real-time discussions from targeted subreddits.
*   **Anthropic Claude:** Delivers advanced natural language understanding to distill complex posts into clear insights.

**Storage & Output**
*   **Google Sheets:** Acts as the persistent storage layer, offering a familiar and collaborative interface for end-users to access the results.

## Strategic Impact

*   **Product & Market Intelligence:** Provides a systematic way to discover common user frustrations, feature requests, and unmet needs in the market.
*   **Efficiency & Scale:** Automates a previously manual and tedious process, freeing up human analysts for more important tasks.
*   **Proactive Problem-Solving:** Allows companies to identify and respond to emerging issues before they become widespread, improving customer satisfaction and retention.