---
type: project
created: 2026-04-01
updated: 2026-04-01
status: active
tags: []
reviewed:
repo: git@github.com:Bhavin-0/InboxIQ.git
---

# InboxIQ

## Goal
InboxIQ is an AI-powered email productivity system that helps users quickly understand and act on emails by generating concise summaries and extracting key insights from raw email content.

---

## Architecture

The system follows a layered architecture:

Client → Controller Layer → Service Layer → AI API (Gemini) → Response

Controller handles HTTP requests and validation
Service processes business logic and interacts with AI APIs
External AI service (Gemini) generates summaries and insights

Future extension:

Chrome Extension (Gmail integration)
Async processing for large emails


## Tech Stack

- Language: Java
- Framework: spring boot 
- Database:
- Tools: Postman, Lambok, Maven

---

## Features

- Generate concise summary for long texts 
- Extracts key points from email content
- REST API for AI-Powered Text processing
- Scalable backend design for future integration
-

---

## Implementation Steps

1.
2.
3.

---

## Problems Faced

- Not able to connect with AI model gemini 1.5

---

## Lessons Learned

- Building fallback between models 
- Retry on faliure

# 🐛 Debugging Log — InboxIQ

---

## Issue #1 — Gemini API 500 Error (Actual 404)

### 🧠 Problem
- API call from backend was returning **500 Internal Server Error** in Postman

---

### 🔍 Root Cause
- The actual error was **404 Not Found from Gemini API**
- Wrong model endpoint was used:
  - ❌ `gemini-1.5-flash-latest`
  - ✅ `gemini-1.5-flash`

---

### ⚠️ Why it was confusing
- Backend wrapped external API error into:
  
  IllegalStateException
---

## Related Notes
- [[ ]]

