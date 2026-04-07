---
title: "Basic lead enrichment in Salesforce"
date: 2025-12-26
tags: ["Salesforce", "Agentforce", "Agentbuilder", "Automation, Lead Enrichment"]
author: "Tarek Mustafa"
draft: false
hidemeta: false
comments: false
canonicalURL: "https://canonical.url/to/page"
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
  image: "/images/SalesforceLeadBanner.webp"
  alt: "Image showing a cartoonish image about using a Salesforce for Lead Data Enhancement"
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# Using Automation for basic Lead enrichment in Salesforce

To gain more hands-on experience with Salesforce, I used a Developer Edition org to build an AI-driven automation that triggers whenever a new Lead is created. Using **Agentforce**, the system automatically finds and populates the **phone number, email, and website** for that lead.

---

## The Three Ingredients of the Automation

### 1. The Instructions (Flex Prompt Template)
To guide the AI, I created a **Flex Prompt Template**. It takes the company name and location as inputs and instructs the AI to return the corresponding phone number, email, and website in a clean JSON format.

![Flex Prompt Template](/images/FlexPromptTemplate.webp)

### 2. The Interpreter (APEX Class)
The AI returns the retrieved data as a single string, which Salesforce cannot natively map to individual fields. To bridge this gap, I used **Google Gemini** to help write an **Apex Class** that parses the JSON string and assigns the data to the correct Lead fields.

![APEX Class](/images/LeadAPEX.webp)

### The Coordinator (Record Trigger Flow)
This is the "behind-the-scenes" manager. It monitors the system for new Leads, triggers the prompt template, coordinates the data processing through the Apex class, and finally updates the Lead record.

![Record Trigger Flow](/images/LeadFlow.webp)

---

## How it Works in Salesforce

Imagine a researcher creates a new Lead with only a company name street and city, leaving other fields empty and hits **save**.

Immediately, the system starts working:

- **The Trigger**: Salesforce detects the new Lead creation.
- **The Search**: The AI performs a targeted web search based on the provided company name and location.
- **The Delivery**: The AI returns its findings in a structured JSON format.
- **The Update**: The Apex class parses that data and maps it to the empty fields.
- **The Result**: The record is updated instantly.

By the time the page reloads, the additional information have been added to the Lead record. No extra clicks, no manual searching, and no copy-pasting from Google.

![Enriched Lead](/images/SFLead.webp)

---

### Key Takeaways

**Good Enough" Beats Perfect**
The AI is accurate about 75% of the time. However, even when it isn't perfect, having a starting point is far more efficient than staring at blank fields and starting from scratch.

**Simple Foundations Scale Easily**
I started with three core fields to prove the concept. The beauty of this setup is that it can easily be expanded to include social media profiles, Google ratings, or industry descriptions.

---

### The Bottom Line

Tasks that were once tedious are now automatic, scalable, and easy to maintain. This automation allows researchers to focus on high-value strategy rather than hunting for phone numbers and URLs.

That is the real value of Agentforce: it isn't about AI doing everything, but about AI taking care of the *boring things*.