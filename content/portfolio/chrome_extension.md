---
title: "Resume Checker"
date: 2025-09-29
tags: ["Pydantic-ai", "FastAPI", "React", "Docker"]
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
  image: "/images/ChromeExtensionBanner.webp"
  alt: "Image showing a cartoonish image about a Resume Checker Chrome extension"
  caption: "" # display caption under cover
  relative: false # when using page bundles set this to true
  hidden: false # only hide on current single page
---

# Resume Checker - Chrome Extension

In today's job market, it is difficult to know whether your resume really matches a specific job description. I built a Chrome extension to make that comparison faster and more practical.

The tool gives job seekers immediate feedback by comparing their resume against a job description, highlighting matching keywords, missing keywords, and concrete improvement areas.

---

## What I Built

![Resume Checker Chrome Extension](/images/resumeChecker.webp)

The Resume Checker lets users analyze a job description directly from the browser. They paste their resume text, run the check, and get feedback without leaving the page they are viewing.

The system provides a clear, visual breakdown, including:

- **Compatibility score:** A 0-100 score estimating how closely the resume aligns with the job description.
- **Matched and missing keywords:** Skills and requirements that appear in the resume, plus important terms that are missing.
- **Improvement suggestions:** Practical recommendations for closing the gaps.

The point is not to let AI rewrite the resume blindly. The point is to show the candidate where the gaps are so they can tailor their application more intentionally.

---

## Technical Deep Dive

### Frontend

- **Chrome Extension:** Built with **vanilla JavaScript, HTML, and CSS** using **Manifest V3**.

### Backend API and AI Core

- **Framework:** **FastAPI (Python)** to serve high-performance, auto-documented endpoints with built-in validation.
- **AI:** Uses **pydantic-ai** agents to compare the resume and job description and return structured feedback.
- **Performance and handling:** Includes caching for repeated checks and upload handling for PDF processing.

### Deployment

- **Containerization:** The application is packaged with **Docker** for consistent local and deployment environments.

---

## Project Motivation: Built from Experience

The idea for this tool came from my own job search. Like many candidates, I spent hours adjusting my CV for every new job description and guessing which skills or keywords mattered most.

I wanted instant, concrete insight into how well my resume actually aligned with a role. This tool turns the manual process of tailoring a CV into a quick review step, helping job seekers see and fix the gaps between their experience and the role they want.

> If you are interested, here is the GitHub link to the [Resume Checker](https://github.com/TAMustafa/ResumeCheckerAPI).
