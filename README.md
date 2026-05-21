# AuthentiCheck — AI Learning Authenticity Evaluator

> **Detect whether students truly understand their submitted work — powered by Claude AI.**

AuthentiCheck is a browser-based academic integrity tool that uses adaptive AI-driven viva questioning to evaluate student understanding. Instead of just checking for plagiarism, it goes deeper: it generates targeted oral exam questions from the student's own submission and scores their comprehension across Depth, Accuracy, and Confidence — delivering a final verdict of **Authentic**, **Partial Understanding**, or **Suspicious**.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Features](#2-features)
3. [Requirements & Dependencies](#3-requirements--dependencies)
4. [Folder Structure](#4-folder-structure)
5. [Environment Configuration](#5-environment-configuration-env)
6. [Setup & Run Guide](#6-setup--run-guide)
7. [Application Screens & User Flow](#7-application-screens--user-flow)
8. [API Integration](#8-api-integration)
9. [File Upload Support](#9-file-upload-support)
10. [Scoring System](#10-scoring-system)
11. [Version Control — GitHub Practices](#11-version-control--github-practices)
12. [Known Limitations](#12-known-limitations)
13. [Screen Recording Guide](#13-screen-recording-guide)
14. [License](#14-license)

---

## 1. Project Overview

| Property        | Detail                                              |
|-----------------|-----------------------------------------------------|
| **App Name**    | AuthentiCheck                                       |
| **Type**        | Single-Page Application (SPA) — pure HTML/CSS/JS    |
| **AI Model**    | Claude Sonnet (`claude-sonnet-4-20250514`)           |
| **API**         | Anthropic Messages API (`/v1/messages`)              |
| **Entry Point** | `compet_fixed.html`                                 |
| **No Build Required** | Open directly in any modern browser          |
| **Data Storage**| Browser `localStorage` (session-based, no backend) |

---

## 2. Features

### Core Functionality
- **AI Submission Analysis** — Claude reads and interprets the student's pasted or uploaded assignment, identifying key topics and assessing depth.
- **Adaptive Viva Question Generation** — 4 targeted questions generated per evaluation, calibrated to Easy / Medium / Hard difficulty.
- **Answer Evaluation** — Claude scores each student response and produces a composite authenticity verdict.
- **3-Level Verdict System** — `Authentic` · `Partial Understanding` · `Suspicious`
- **Score Breakdown** — Three sub-scores: **Depth** (conceptual understanding), **Accuracy** (factual correctness), **Confidence** (answer clarity).

### File Upload
- Drag-and-drop or click-to-browse upload zone
- Supports: `.pdf`, `.docx`, `.doc`, `.txt`, `.js`, `.py`, `.java`, `.cpp`, `.c`, `.cs`, `.html`, `.css`, `.ts`, `.md`
- PDF text extraction via **PDF.js** (CDN-loaded on demand)
- Word document extraction via **Mammoth.js** (CDN-loaded on demand)
- Maximum file size: **5 MB**

### Dashboard & History
- Live metrics: Total Evaluated, Average Score, Authentic count, Suspicious count
- Recent evaluations table with score, verdict, and timestamp
- Class distribution bar chart (Authentic / Partial / Suspicious)
- Mini score sparkline chart
- Session history with per-student feedback and sub-scores
- One-click history clear

### Export
- Downloadable **HTML report** per evaluation, including student info, scores, verdict, and AI feedback

### Authentication
- Role selection: **Teacher** or **Student**
- Named login with Teacher ID / Student Roll Number
- Guest / Demo mode
- API key stored in `localStorage` (never transmitted except to Anthropic)
- Auto-login if session key is present

---

## 3. Requirements & Dependencies

### Runtime Requirements

| Requirement        | Detail                                                    |
|--------------------|-----------------------------------------------------------|
| **Browser**        | Chrome 90+, Firefox 88+, Edge 90+, Safari 15+             |
| **Internet**       | Required (API calls + CDN fonts/icons/libraries)          |
| **Anthropic API Key** | Required — `sk-ant-api03-...` format                  |

### No Installation Required
This is a zero-dependency, zero-build project. There is no `package.json`, no Node.js, no Python — just one HTML file.

### External CDN Dependencies

All external resources are loaded via CDN at runtime. No local installation needed.

| Library / Resource         | Version    | Purpose                              | CDN URL                                                                 |
|----------------------------|------------|--------------------------------------|-------------------------------------------------------------------------|
| **Cormorant Garamond**     | Latest     | Serif display font                   | `fonts.googleapis.com`                                                  |
| **Figtree**                | Latest     | UI body font                         | `fonts.googleapis.com`                                                  |
| **Tabler Icons**           | 2.44.0     | Icon set (webfont)                   | `cdn.jsdelivr.net/npm/@tabler/icons-webfont`                            |
| **PDF.js**                 | 3.11.174   | PDF text extraction (lazy-loaded)    | `cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/`                       |
| **Mammoth.js**             | 1.6.0      | DOCX text extraction (lazy-loaded)   | `cdnjs.cloudflare.com/ajax/libs/mammoth/1.6.0/`                         |
| **Anthropic API**          | 2023-06-01 | AI analysis and evaluation           | `https://api.anthropic.com/v1/messages`                                 |

> **Note:** PDF.js and Mammoth.js are only fetched when a user uploads a `.pdf` or `.docx` file respectively. They are not loaded on initial page load.

---

## 4. Folder Structure

Since AuthentiCheck is a single-file application, the recommended project structure when hosting or submitting is:

```
authenticheck/
│
├── compet_fixed.html          # Main application (all HTML, CSS, JS in one file)
├── README.md                  # This documentation file
├── .env.example               # Environment variable template (see Section 5)
├── .gitignore                 # Git ignore rules
│
├── docs/                      # Extended documentation (optional)
│   ├── screenshots/           # UI screenshots for documentation
│   │   ├── landing.png
│   │   ├── login.png
│   │   ├── dashboard.png
│   │   ├── evaluation-step1.png
│   │   ├── evaluation-step2.png
│   │   └── results.png
│   └── demo.mp4               # 5-minute screen recording (see Section 13)
│
└── reports/                   # Sample exported HTML reports (optional)
    └── sample_report.html
```

> **Important:** The entire application lives inside `compet_fixed.html`. There is no `src/`, `dist/`, or `node_modules/` directory. If you are converting this to a multi-file project (e.g., for a React or Vite build), see the modularisation notes at the end of this document.

---

## 5. Environment Configuration (.env)

Because this is a client-side-only application, there is no server-side `.env` file. However, the Anthropic API key is sensitive and should be managed carefully.

### How the API Key Works in This App

The user enters their own API key on the login screen. The key is:
- Stored in the browser's `localStorage` under the key `ac_apikey`
- Sent only to `https://api.anthropic.com` — never to any other server
- Never hardcoded in the source file

### .env.example (for future server-side version)

If you extend this project with a backend proxy (recommended for production), create a `.env` file based on the following template:

```env
# ─────────────────────────────────────────
# AuthentiCheck — Environment Configuration
# ─────────────────────────────────────────

# Anthropic API Key
# Obtain from: https://console.anthropic.com/account/keys
# Format: sk-ant-api03-...
ANTHROPIC_API_KEY=sk-ant-api03-YOUR_KEY_HERE

# Claude Model
# Do not change unless intentionally upgrading
CLAUDE_MODEL=claude-sonnet-4-20250514

# App Settings
APP_PORT=3000
APP_ENV=development   # development | production

# Optional: Rate limiting (for backend proxy)
MAX_REQUESTS_PER_MINUTE=20
```

### .gitignore

Create a `.gitignore` file at the project root with the following contents to prevent secrets from being committed:

```gitignore
# Environment variables — NEVER commit these
.env
.env.local
.env.production
.env*.local

# OS & editor clutter
.DS_Store
Thumbs.db
.vscode/
.idea/

# Logs
*.log
npm-debug.log*

# Node (if you add a backend later)
node_modules/
dist/
build/

# Sensitive reports (optional)
reports/*.html
```

> ⚠️ **Security Warning:** Never hardcode your `ANTHROPIC_API_KEY` directly into `compet_fixed.html` or commit it to a public repository. If you accidentally expose a key, revoke it immediately at [console.anthropic.com](https://console.anthropic.com).

---

## 6. Setup & Run Guide

### Option A — Run Locally (Simplest)

1. **Download** `compet_fixed.html` to your computer.

2. **Open** the file in your browser:
   - Double-click the file, **or**
   - Drag it into an open Chrome/Firefox/Edge window, **or**
   - Right-click → Open with → your browser of choice

3. **Get an Anthropic API Key:**
   - Go to [https://console.anthropic.com](https://console.anthropic.com)
   - Sign up or log in
   - Navigate to **API Keys** → **Create Key**
   - Copy the key (it starts with `sk-ant-api03-`)

4. **Sign in to AuthentiCheck:**
   - On the landing page, click **Get Started** or **Start Evaluating**
   - Select your role (Teacher recommended)
   - Enter your name and ID
   - Paste your Anthropic API key into the **API Key** field
   - Click **Continue to Dashboard**

5. **Run your first evaluation:**
   - Click **New Evaluation** in the sidebar
   - Fill in Student Name, Subject, and Assignment Title
   - Choose difficulty (Easy / Medium / Hard)
   - Upload a file or paste the student's submission
   - Click **Analyze & Generate Viva**
   - Enter the student's answers to the 4 questions
   - Click **Submit Answers** to see the verdict

---

### Option B — Serve via Local HTTP Server (Recommended for PDF support)

Some browsers restrict `FileReader` and fetch APIs when opening HTML from the filesystem. Serving over HTTP resolves this.

**Using Python (no install needed on macOS/Linux):**
```bash
# Navigate to the folder containing compet_fixed.html
cd path/to/authenticheck/

# Python 3
python3 -m http.server 8080

# Then open in browser:
# http://localhost:8080/compet_fixed.html
```

**Using Node.js (if installed):**
```bash
npx serve .
# Then open: http://localhost:3000/compet_fixed.html
```

**Using VS Code Live Server extension:**
- Install the **Live Server** extension by Ritwick Dey
- Right-click `compet_fixed.html` → **Open with Live Server**

---

### Option C — Deploy to a Static Host

The file can be deployed as-is to any static hosting provider:

| Platform        | Steps                                                       |
|-----------------|-------------------------------------------------------------|
| **GitHub Pages** | Push to a repo → Settings → Pages → Deploy from branch    |
| **Netlify**      | Drag the file into [netlify.com/drop](https://app.netlify.com/drop) |
| **Vercel**       | `vercel --prod` from project folder                        |
| **Cloudflare Pages** | Connect repo or drag-and-drop in dashboard             |

> No build step is needed. Simply upload `compet_fixed.html` as the root `index.html`.

---

## 7. Application Screens & User Flow

```
┌─────────────────┐
│  Landing Page   │  → Hero, Features, How It Works, CTA
└────────┬────────┘
         │ "Get Started" or "Start Evaluating"
         ▼
┌─────────────────┐
│   Login Screen  │  → Role selection, Name, ID, Institution, API Key
└────────┬────────┘
         │ "Continue to Dashboard" or "Guest Mode"
         ▼
┌─────────────────────────────────────────────────┐
│                  Dashboard (App)                 │
│  ┌──────────┐  ┌──────────────────────────────┐ │
│  │ Sidebar  │  │       Main Content Area       │ │
│  │          │  │  ┌──────────┐ ┌────────────┐ │ │
│  │Dashboard │  │  │Dashboard │ │ Evaluation │ │ │
│  │Evaluation│  │  │  Page    │ │  Wizard    │ │ │
│  │History   │  │  │          │ │ Step 1→2→3 │ │ │
│  │Stats     │  │  └──────────┘ └────────────┘ │ │
│  └──────────┘  │  ┌──────────┐                 │ │
│                │  │ History  │                 │ │
│                │  │  Page    │                 │ │
│                │  └──────────┘                 │ │
└─────────────────────────────────────────────────┘
```

### Evaluation Wizard — 3 Steps

| Step | Name | What Happens |
|------|------|--------------|
| **Step 1** | Student & Submission | Enter student details, upload or paste submission, choose difficulty |
| **Step 2** | Viva Questions | AI analyzes submission → generates 4 adaptive questions → teacher records student's verbal answers |
| **Step 3** | Results | AI evaluates answers → displays score (0–100), verdict, Depth/Accuracy/Confidence breakdown, and written feedback |

---

## 8. API Integration

AuthentiCheck makes two API calls per evaluation session, both to the Anthropic Messages API.

### API Call 1 — Submission Analysis & Question Generation

**Triggered by:** Clicking "Analyze & Generate Viva" (Step 1 → Step 2)

```
POST https://api.anthropic.com/v1/messages
```

**Headers:**
```
Content-Type: application/json
x-api-key: <user's API key>
anthropic-version: 2023-06-01
anthropic-dangerous-direct-browser-access: true
```

**Request Body:**
```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1000,
  "system": "You are an academic integrity AI evaluator. Respond ONLY in valid JSON with no markdown.",
  "messages": [{
    "role": "user",
    "content": "Analyze this student submission and generate 4 adaptive viva questions at [Difficulty] difficulty..."
  }]
}
```

**Expected Response (JSON):**
```json
{
  "topics": ["Recursion", "Stack Memory", "Base Case"],
  "analysis": "The submission demonstrates surface-level understanding of recursion...",
  "questions": [
    { "id": 1, "question": "Can you explain what happens in memory when a recursive function calls itself?", "difficulty": "Medium", "tests": "Stack Memory" },
    ...
  ]
}
```

---

### API Call 2 — Answer Evaluation & Scoring

**Triggered by:** Clicking "Submit Answers" (Step 2 → Step 3)

**Expected Response (JSON):**
```json
{
  "totalScore": 72,
  "depth": 75,
  "accuracy": 68,
  "confidence": 73,
  "verdict": "Authentic",
  "title": "Strong conceptual understanding",
  "feedback": "The student demonstrated solid understanding of recursive calls and stack frames..."
}
```

### Error Handling

| HTTP Status | Meaning | App Behaviour |
|-------------|---------|---------------|
| `401` | Invalid API key | Red notification: "API Error: ..." |
| `429` | Rate limit exceeded | Red notification with error message |
| `500` | Anthropic server error | Red notification, stays on current step |
| Network failure | No internet / CORS | Red notification: "Error: Failed to fetch" |

---

## 9. File Upload Support

| Format | Extension(s) | Parser | Notes |
|--------|-------------|--------|-------|
| Plain Text | `.txt` | Native `FileReader` | Instant, no CDN load |
| Source Code | `.js` `.py` `.java` `.cpp` `.c` `.cs` `.ts` `.html` `.css` `.md` | Native `FileReader` | Treated as plain text |
| PDF | `.pdf` | PDF.js 3.11.174 | Loaded from CDN on first use; extracts text layer |
| Word Document | `.docx` `.doc` | Mammoth.js 1.6.0 | Loaded from CDN on first use; extracts raw text |

**Size Limit:** 5 MB per file

**Behaviour:** Extracted text is populated into the submission textarea and passed to the AI exactly as if the teacher had pasted it manually.

---

## 10. Scoring System

### Score Ranges & Verdicts

| Score | Verdict | Meaning |
|-------|---------|---------|
| 70 – 100 | ✅ **Authentic** | Student demonstrates genuine understanding of their work |
| 50 – 69 | ⚠️ **Partial Understanding** | Student understands parts but has significant gaps |
| 0 – 49 | 🚨 **Suspicious** | Student cannot adequately explain their own submission |

### Sub-Score Dimensions

| Dimension | What It Measures |
|-----------|-----------------|
| **Depth** | How deeply the student understands the underlying concepts |
| **Accuracy** | Correctness of facts, terminology, and technical claims in answers |
| **Confidence** | Clarity, coherence, and decisiveness of the student's responses |

All sub-scores are on a 0–100 scale. The `totalScore` is a composite determined by Claude based on all three dimensions.

---

## 11. Version Control — GitHub Practices

### Recommended Branch Strategy

```
main              ← stable, production-ready
  └── develop     ← integration branch
        ├── feature/file-upload
        ├── feature/export-report
        ├── fix/cors-api-headers
        └── docs/readme-update
```

### Commit Message Convention

Follow the **Conventional Commits** specification:

```
<type>(<scope>): <short description>

Types:
  feat      — new feature
  fix       — bug fix
  docs      — documentation only
  style     — formatting, no logic change
  refactor  — code restructure, no behaviour change
  perf      — performance improvement
  chore     — build process, tooling, dependencies
```

**Examples:**
```
feat(upload): add PDF text extraction via PDF.js
fix(api): add anthropic-dangerous-direct-browser-access header for CORS
fix(nav): prevent Dashboard button from returning to landing page
docs: add README with setup guide and API documentation
chore: add .gitignore and .env.example
style(login): improve API key field placeholder text
```

### Recommended GitHub Repository Setup

```bash
# 1. Initialise a new repository
git init authenticheck
cd authenticheck

# 2. Add project files
cp /path/to/compet_fixed.html ./index.html
cp /path/to/README.md ./README.md

# 3. Create .gitignore
cat > .gitignore << 'EOF'
.env
.env.local
.DS_Store
node_modules/
*.log
EOF

# 4. Initial commit
git add .
git commit -m "feat: initial commit — AuthentiCheck single-page app"

# 5. Push to GitHub
git remote add origin https://github.com/YOUR_USERNAME/authenticheck.git
git branch -M main
git push -u origin main
```

### Pull Request Template

When contributing, open PRs against `develop` with the following description format:

```markdown
## Summary
Brief description of what this PR does.

## Changes
- [ ] Feature / fix description
- [ ] Tests updated (N/A for front-end only)
- [ ] Documentation updated

## How to Test
1. Open index.html in Chrome
2. Log in with a valid API key
3. ...

## Screenshots (if UI change)
| Before | After |
|--------|-------|
| ...    | ...   |
```

### Tagging Releases

```bash
git tag -a v1.0.0 -m "Release v1.0.0 — initial public release"
git push origin v1.0.0
```

Use **Semantic Versioning**: `MAJOR.MINOR.PATCH`
- `MAJOR` — breaking changes
- `MINOR` — new features, backward-compatible
- `PATCH` — bug fixes

---

## 12. Known Limitations

| Limitation | Detail | Workaround |
|------------|--------|------------|
| **No persistent data** | Session history clears on page refresh | Export report before closing |
| **API key exposed client-side** | Key visible in browser DevTools | Use a backend proxy for production |
| **No real-time collaboration** | Single evaluator per session | Share exported HTML reports |
| **PDF.js requires internet** | Cannot extract PDFs offline | Copy-paste text manually |
| **No multi-language support** | UI and AI prompts in English only | Manual prompt translation |
| **5 MB file limit** | Large codebases may be truncated | Split submission into sections |
| **API costs** | Each evaluation uses ~1,500–2,500 tokens | Monitor usage at console.anthropic.com |

---

## 13. Screen Recording Guide

> As per the assignment requirements, a **5-minute screen recording** is required demonstrating the working application with a voice-over explanation.

### Recommended Recording Script (5 Minutes)

| Time | Section | What to Show & Say |
|------|---------|-------------------|
| **0:00 – 0:30** | Introduction | "This is AuthentiCheck, an AI-powered academic integrity tool. Open the HTML file in Chrome..." |
| **0:30 – 1:00** | Landing Page | Scroll through hero, features section, how-it-works steps, CTA |
| **1:00 – 1:45** | Login & API Key | "Click Get Started. Select Teacher role. Fill in name, ID, institution. Paste your API key from console.anthropic.com. Click Continue." |
| **1:45 – 2:15** | Dashboard | "This is the teacher dashboard. It shows total evaluated, average score, and the class distribution chart. Everything starts at zero." |
| **2:15 – 3:15** | Running Evaluation — Step 1 | "Click New Evaluation. Enter student name, subject, assignment. Choose difficulty — Medium. Upload a file or paste text. Click Analyze." |
| **3:15 – 4:00** | Viva Questions — Step 2 | "Claude has generated 4 targeted questions. Notice the topics detected in the sidebar. Fill in the student's answers. Click Submit." |
| **4:00 – 4:30** | Results — Step 3 | "Here's the result: score 72/100, verdict Authentic. The score ring animates. We can see Depth 75, Accuracy 68, Confidence 73. The feedback explains the reasoning." |
| **4:30 – 5:00** | Export & History | "Click Export Report to download an HTML report. Click Dashboard to see the updated metrics. Click History to review all sessions. Click Clear to reset." |

### Recommended Tools

| Tool | Platform | Cost |
|------|----------|------|
| **OBS Studio** | Windows / macOS / Linux | Free |
| **Loom** | Browser extension | Free (up to 5 min) |
| **QuickTime Player** | macOS | Free (built-in) |
| **Xbox Game Bar** | Windows 10/11 | Free (Win + G) |
| **Screencastify** | Chrome extension | Free (up to 5 min) |

### Tips for a High-Quality Recording
- Set browser zoom to **100%** for consistent appearance
- Use **1920×1080** resolution minimum
- Speak clearly, at a moderate pace — aim for one thought per action
- Have a **real API key** ready before recording (avoid fumbling)
- Prepare a **sample submission** (e.g., a short Python function or essay paragraph) so the AI call succeeds smoothly
- Record a test run first to check audio levels

---

## 14. License

```
MIT License

Copyright (c) 2025 AuthentiCheck

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

## Quick Reference Card

```
┌──────────────────────────────────────────────────────────┐
│               AuthentiCheck — Quick Reference             │
├──────────────────────────────────────────────────────────┤
│  Get API Key   →  console.anthropic.com                  │
│  Open App      →  double-click compet_fixed.html          │
│  Local Server  →  python3 -m http.server 8080             │
│                                                           │
│  Verdict Logic:                                           │
│    70–100  ✅  Authentic                                  │
│    50–69   ⚠️  Partial Understanding                      │
│    0–49    🚨  Suspicious                                 │
│                                                           │
│  File Uploads: PDF, DOCX, TXT, Code (max 5 MB)           │
│  API Model:    claude-sonnet-4-20250514                   │
│  Tokens/eval:  ~1,500 – 2,500                            │
└──────────────────────────────────────────────────────────┘
```

---

*Documentation last updated: May 2025 · AuthentiCheck v1.0.0*
