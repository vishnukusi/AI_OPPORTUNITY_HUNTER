Got you. You want **one single copyable block from the very first line to the very last line**, with nothing outside it.


# AI Opportunity Hunter

> An AI-powered automation system that discovers, evaluates, filters, and delivers relevant student opportunities automatically.

## Overview

As a student, discovering hackathons, coding competitions, AI challenges, student programs, research opportunities, and other career-building opportunities can be surprisingly difficult.

Many opportunities are scattered across different platforms, have different data formats, and are easy to miss.

The **AI Opportunity Hunter** solves this problem by collecting opportunities from multiple sources, processing them through an automated pipeline, evaluating them using AI, and sending only relevant new opportunities directly to Telegram.

The system runs automatically every morning at **8:00 AM**, so opportunities come to me instead of me having to search for them manually.

---

## Problem

Students often discover opportunities too late.

A typical process looks like:

Search multiple websites
        ↓
Check dozens of listings
        ↓
Compare opportunities
        ↓
Check relevance
        ↓
Remember what was already seen
        ↓
Save useful opportunities
        ↓
Repeat again tomorrow

This is repetitive, time-consuming, and makes it easy to miss deadlines.

### The Goal

Build an automated system that handles this entire discovery pipeline.

---

# Solution

The AI Opportunity Hunter transforms the process into:

Multiple Sources
        ↓
Data Collection
        ↓
Data Normalization
        ↓
Pre-AI Filtering
        ↓
Duplicate Removal
        ↓
AI Evaluation
        ↓
Relevance & Score Filtering
        ↓
Previously-Seen Check
        ↓
Google Sheets
        ↓
Telegram Notification

The result is a personalized stream of relevant opportunities delivered automatically.

---

# Key Features

## 1. Multi-Source Opportunity Discovery

The system collects opportunities from multiple platforms and ecosystems instead of depending on a single website.

### Current Sources

- Devpost
- Unstop
- Kaggle
- Google Developer ecosystem
- Microsoft student/developer ecosystem

Each source has its own dedicated workflow because every platform exposes data differently.


SOURCE - Devpost
SOURCE - Unstop
SOURCE - Kaggle
SOURCE - Google
SOURCE - Microsoft


This modular approach makes it easy to add more sources later.

---

## 2. Source-Specific Data Extraction

Different platforms return completely different data structures.

For example:

Unstop
→ title
→ organization
→ deadline
→ location
→ URL

Devpost
→ title
→ organization
→ submission period
→ URL

Kaggle
→ competition title
→ organization
→ competition URL

Google / Microsoft
→ program/event/resource
→ official URL

Instead of forcing all sources into one extraction process, each source has its own workflow that converts the raw data into a common structure.

### Standardized Format

json
{
  "title": "...",
  "organization": "...",
  "url": "...",
  "deadline": "...",
  "location": "...",
  "status": "...",
  "source": "..."
}


This allows all sources to be combined later in the pipeline.

---

# 3. Intelligent Pre-AI Filtering

Sending every collected opportunity directly to an AI model would waste tokens and increase unnecessary API usage.

Therefore, the system performs a lightweight filtering stage before the AI.

The filter identifies opportunity-related keywords such as:

* AI
* Machine Learning
* Coding
* Hackathon
* Competition
* Challenge
* Research
* Innovation
* Startup
* Developer
* Student
* Fellowship
* Entrepreneurship

This removes obvious irrelevant listings before they reach the AI.

### Why This Matters

Instead of:

60+ opportunities
↓
AI

The system reduces the dataset first:

60+ opportunities
↓
Cheap local filtering
↓
~20 relevant candidates
↓
AI

This reduces AI usage while preserving potentially valuable opportunities.

---

# 4. Duplicate Detection

The same opportunity can appear on multiple platforms.

For example:

Devpost
→ AI Innovation Hackathon

Unstop
→ AI Innovation Hackathon

The system detects duplicate opportunities based on their title and keeps only one.

Importantly, opportunities are **not** considered duplicates simply because they belong to the same category.

For example:

AI Hackathon A
AI Hackathon B
AI Research Challenge

All three remain because they are different opportunities.

---

# 5. AI-Based Opportunity Evaluation

After filtering and deduplication, the remaining opportunities are evaluated by Gemini.

The AI evaluates each opportunity based on factors such as:

* Relevance to Computer Science
* AI/ML relevance
* Coding and technical value
* Suitability for university students
* Career value
* Learning opportunities
* Project and portfolio value
* Competitions and prizes
* Networking opportunities
* Research or entrepreneurship potential

Each opportunity receives:

Relevance
Score
Priority
Category
Reason

### Example


Opportunity:
AI Innovation Hackathon 2026

Relevant:
true

Score:
9/10

Priority:
HIGH

Category:
AI/ML

Reason:
Strong opportunity to build practical AI skills
and strengthen a technical portfolio.


---

# 6. Structured AI Output

The AI is not allowed to return an arbitrary paragraph.

The workflow uses structured output so every opportunity follows the same format:

json
{
  "title": "...",
  "url": "...",
  "deadline": "...",
  "relevant": true,
  "priority": "HIGH",
  "score": 9,
  "reason": "...",
  "category": "AI/ML"
}


This makes the AI output predictable and easy for the following n8n nodes to process.

---

# 7. Persistent Opportunity History

One of the most important features is the **previously-seen opportunity check**.

Google Sheets acts as a lightweight persistence layer.

Before sending a Telegram notification, the system checks:


Does this URL already exist in my database?


### If It Exists


Already Seen
     ↓
Do Not Notify


### If It Doesn't Exist


New Opportunity
     ↓
Add to Google Sheets
     ↓
Send Telegram Notification


This prevents the same opportunity from being sent every morning.

---

# 8. Automated Telegram Notifications

Relevant new opportunities are delivered directly to Telegram.

Example:


HIGH PRIORITY OPPORTUNITY

AI Innovation Hackathon 2026

Score: 9/10
Category: AI/ML

Deadline: August 29, 2026

Why it matters:
Strong opportunity to build a practical AI
project and strengthen your portfolio.

Source:
Unstop

Link:
https://...
`

This means I don't have to manually check multiple websites every day.

---

# 9. Fully Automated Daily Execution

The workflow is scheduled to execute automatically every morning at:


08:00 AM


The complete pipeline runs without manual intervention:


08:00 AM
   ↓
Fetch Sources
   ↓
Process Opportunities
   ↓
AI Evaluation
   ↓
Check History
   ↓
Send New Opportunities
   ↓
Finish


The system is designed around **one scheduled execution per day** rather than repeatedly polling websites throughout the day.

This keeps API and AI usage under control.

---

# 10. Token-Conscious Architecture

A major design goal was to avoid unnecessary AI usage.

Instead of sending every raw opportunity to Gemini, the workflow follows:


Raw Data
   ↓
Source Processing
   ↓
Pre-AI Filtering
   ↓
Duplicate Removal
   ↓
AI Evaluation


Only the reduced set of meaningful opportunities reaches the AI.

The system also uses **one AI evaluation stage for the batch** rather than making a separate AI request for every opportunity.

This makes the workflow more efficient and cost-conscious.



Architecture


                         ┌─────────────────┐
                         │    DEVPOST      │
                         └────────┬────────┘
                                  │
                         ┌────────▼────────┐
                         │     UNSTOP      │
                         └────────┬────────┘
                                  │
                         ┌────────▼────────┐
                         │     KAGGLE      │
                         └────────┬────────┘
                                  │
                         ┌────────▼────────┐
                         │     GOOGLE      │
                         └────────┬────────┘
                                  │
                         ┌────────▼────────┐
                         │    MICROSOFT    │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │      MERGE      │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ PRE-AI FILTER   │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ DEDUPLICATION   │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │     GEMINI      │
                         │ AI EVALUATION   │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ SCORE / RELEVANT│
                         │     FILTER      │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ GOOGLE SHEETS   │
                         │ HISTORY CHECK   │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │    TELEGRAM     │
                         │   NOTIFICATION  │
                         └─────────────────┘




# Tech Stack

| Technology       | Purpose                             |
| ---------------- | ----------------------------------- |
| n8n              | Workflow automation                 |
| Gemini API       | AI opportunity evaluation           |
| REST APIs        | Data collection                     |
| JavaScript       | Data transformation and filtering   |
| Google Sheets    | Opportunity history and persistence |
| Telegram Bot API | Notifications                       |



# Project Structure


AI-Opportunity-Hunter/
│
├── README.md
│
├── workflows/
│   ├── main-opportunity-hunter.json
│   ├── source-devpost.json
│   ├── source-unstop.json
│   ├── source-kaggle.json
│   ├── source-google.json
│   └── source-microsoft.json
│
├── screenshots/
│   ├── workflow.png
│   ├── telegram-alert.png
│   └── google-sheets.png
│
└── .gitignore




# Learning Outcomes

Building this project helped me move beyond simply learning individual tools and start thinking about how complete automation systems are designed.

### Key Concepts Explored

* Workflow automation
* REST APIs
* API authentication
* JSON data processing
* JavaScript in n8n
* Data normalization
* Conditional workflows
* Branching and merging
* AI integration
* Prompt engineering
* Structured AI outputs
* Deduplication
* Persistent data storage
* Scheduled automation
* Telegram API integration
* Cost and token optimization
* Modular workflow architecture

---

# Challenges Solved

## Different API Structures

Every source returned data differently.

**Solution:** Build independent source workflows and normalize their outputs.

## Duplicate Opportunities

The same opportunity could appear across multiple platforms.

**Solution:** Normalize titles and perform duplicate detection.

## AI Token Usage

Sending every raw opportunity to the AI would be inefficient.

**Solution:** Perform cheap filtering and deduplication before the AI stage.

## Repeated Notifications

Running the workflow repeatedly could send the same opportunity again.

**Solution:** Maintain opportunity history in Google Sheets and check URLs before notifying.

## AI Output Reliability

AI responses can vary in structure.

**Solution:** Use structured output parsing and explicit output requirements.



# Future Improvements

This is currently the first version.

Planned improvements include:

* Add more reliable opportunity sources
* Improve eligibility detection
* Detect India-specific eligibility
* Detect online/in-person opportunities
* Track application deadlines
* Add deadline urgency scoring
* Add personalized opportunity ranking
* Add email notifications
* Add a dashboard for opportunity analytics
* Track whether an opportunity has been applied to
* Improve duplicate detection using URL + title similarity
* Add retry and error handling for failed APIs
* Add monitoring for source/API failures

---

# Why I Built This

This project started from a simple personal problem:

**I didn't want to discover opportunities only after they were over.**

Instead of manually checking websites every day, I decided to build an automated system that could do the discovery, filtering, and evaluation for me.

It started as an experiment while learning n8n.

It became a practical project in:

**Automation + APIs + AI + Data Processing + Notifications**

And it is still evolving.





Interested in:

* Artificial Intelligence
* Machine Learning
* AI Automation
* Data Structures & Algorithms
* Building practical software systems



## If you find this project interesting

Feel free to explore the workflow, suggest improvements, or build your own version.


