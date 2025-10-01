---
title: "Resume Checker"
date: 2025-09-29
# weight: 1
# aliases: ["/first"]
tags: ["Pydantic-ai", "FastAPI", "React", "Docker"]
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
    image: "/images/ChromeExtensionBanner.png"
    alt: "Image showing a cartoonish image about a Resume Checker Chrome extension"
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---

# Resume Checker - Chrome Extension

**Bridging the gap between your resume and the job requirements.**

In today's competitive job market, it's difficult to know if your resume truly meets the needs of a specific job description. I built a Chrome extension to solve this  pain point. This tool gives job seekers instant, actionable feedback by identifying exactly how well their provided resume matches a chosen job description in order to make improvements and increase their chances of landing an interview.

## What It Does

![Resume Checker Chrome Extension](/images/resumeChecker.png)

The Resume Checker provides a dual interface for flexibility:

**Chrome Extension:** Analyze any job description directly from your browser. Simply paste your resume text and get immediate feedback without leaving the tab you're in.

The system provides a clear, visual breakdown, including:
*   **Compatibility Score:** A precise score (0-100) rating your resume's alignment with the job description.
*   **Match & Missing Keywords:** Highlights the skills and experience that align perfectly, as well as critical keywords you're missing.
*   **Improvement Suggestions:** Data-driven recommendations on how to bridge the gaps identified in your resume.

> **Instant Insight** A strategic, data-backed method for resume tailoring. Stop guessing what the job requires—get instant insights into how well your CV matches the job description in seconds.

## Technical Deep Dive

### **Frontend & Interfaces**
*   **Chrome Extension:** Built with **Vanilla JavaScript, HTML, & CSS** using **Manifest V3** for modern security and performance.

### **Backend API & AI Core**
*   **Framework:** **FastAPI (Python)** to serve high-performance, auto-documented endpoints with built-in validation.
*   **AI:** Leverages **pydantic-ai multi agents** to perform a intelligent similarity analysis between the resume and job description.
*   **Performance & Security:** Implements **caching** for a faster user experience and secure file upload handling for PDF processing.

### **Deployment**
*   **Containerization:** The entire application is **packaged with Docker** for consistent environments and easy deployment.


## Project Motivation: Built from Experience

The idea for this tool wasn't born in a vacuum, it came from my own job search. Like many candidates, I spent hours trying to manually adjust my CV for every new job description, essentially guessing what skills and keywords mattered most. I realized there had to be a better way to increase my chances of landing an interview.

I wanted instant, concrete insight into how well my resume actually aligned with a job's requirements. This tool transforms the manual, tedious process of tailoring a CV into a quick, data-backed step, giving every job seeker the power to immediately see and fix the gaps between their experience and the role they want. It’s a solution built by a job seeker, for job seekers.

> If you are interested, here is the github link to the [Resume Checker](https://github.com/TAMustafa/ResumeCheckerAPI)