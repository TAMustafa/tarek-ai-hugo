---
title: "Lead data enhancement"
date: 2025-12-26
tags: ["Salesforce", "Agentforce", "Agentbuilder", "Automation"]
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

# Lead data enhancement using Salesforce AI

In order to get more hands on experience with Salesforce, I used the developer edition and built an AI automation that triggers when a new lead is created. The automation uses Agentforce to find and fill in the phone number, email, and website for that lead.

---

## The Three Ingredients of the Automation

### 1. The Instructions (Flex Prompt Template)
To tell the AI what to do, I used a flex prompt template providing the company name and location and ask to provide the phone, email, and website in a JSON format.

![Flex Prompt Template](/images/FlexPromptTemplate.webp)

### 2. The Interpreter (APEX Class)
The AI provides the retrieved data in a single string which is not useful for adding the records to Salesforce. In order sperate the JSON data and add it to the right fields, I used **Google Gemini** to write an APEX Class that does the job.

![APEX Class](/images/LeadAPEX.webp)

### The Coordinator (Record Trigger Flow)
This is the behind-the-scenes manager that watches for new leads being created and then triggers the prompt template, organizes the data and finally updates the lead record.

![Record Trigger Flow](/images/LeadFlow.webp)

---

## How it Works in Salesforce

Imagine a researcher creates a new Lead with only a company name street and city, leaving other fields empty and hits **save**.

Immediately, the system starts working:

- **The Trigger**: Salesforce notices a new Lead has arrived.
- **The Search**: The AI performs a web search based on the company name and city.
- **The Delivery**: The AI returns its findings in a structured "JSON" format.
- **The Update**: The Apex class sorts that data and fills in the empty fields.
- **The Result**: The page reloads with the information filled in.

By the time the page reloads, the information is there. No extra clicks, no manual searching, and no copy-pasting from Google.

![Process Flow](/images/LeadProcess.webp)

---

### What I Learned

**Good enough beats perfect**
The AI gets it right about 75% of the time. But even when it's slightly off, having something to start with beats staring at blank fields.

**Simple foundations scale easily**
In this example, I started with three fields to make it work. The next step is to add more information like e.g. social accounts, google rating and so on.

---

### The Bottom Line

Work that used to be tedious becomes automatic and is easy to maintain and to scale. This automation allows lead researchers to focus on interesting tasks instead of hunting for phone numbers, urls and emails.

That's the real value: Not AI doing everything, but AI taking care of the boring things.