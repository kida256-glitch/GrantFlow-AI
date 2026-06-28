# GrantFlow AI — Scholarship & Student Financial Aid Processing Platform

> **UiPath AgentHack 2026 — Track 1: UiPath Maestro Case**

## What It Does

AI-powered scholarship evaluation through 7 autonomous stages:

| Stage | Description |
|-------|-------------|
| 1 | Document Verification (Analyze Files + DeepRAG) |
| 2 | Eligibility Assessment (rules engine) |
| 3 | AI Scoring (academic, leadership, community, essay) |
| 4 | Fraud Detection (Low / Medium / High risk) |
| 5 | Scholarship Matching (alternatives if rejected) |
| 6 | Reviewer Summary (200–300 word AI summary) |
| 7 | Applicant Notification (professional email draft) |

## Quick Start

1. Open in Studio Web at `/solution/Agent`
2. Add built-in tools: **Analyze Files** + **DeepRAG**
3. Run test with DEMO-001 from `data/sample_applicants.json`

## Test Input

```json
{
  "applicantName": "Amara Osei",
  "email": "amara.osei@ug.edu.gh",
  "country": "Ghana",
  "university": "University of Ghana",
  "degreeProgram": "Computer Science",
  "gpa": "3.8/4.0",
  "personalStatement": "Growing up in Accra, I taught myself Python at 14...",
  "scholarshipName": "STEM Excellence Award",
  "scholarshipCriteria": "{\"minGpa\": 3.5, \"eligibleCountries\": [\"Any\"], \"eligibleDegrees\": [\"Computer Science\"]}"
}
```

Expected: Eligible · Score ~85 · Fraud: Low · Decision: Approve

## Stack

- UiPath Agent Builder (Studio Web) — low-code autonomous agent
- Model: anthropic.claude-sonnet-4-6
- UiPath Maestro — case lifecycle tracking
- UiPath Orchestrator — job execution

## Project Structure

```
GrantFlow-AI/
├── agent/
│   ├── agent.json
│   ├── entry-points.json
│   ├── project.uiproj
│   └── resources/
│       ├── analyze-files/resource.json
│       └── deeprag/resource.json
├── data/
│   ├── scholarship_database.json
│   ├── sample_applicants.json
│   └── email_templates.json
└── README.md
```

Built for UiPath AgentHack 2026 — Track 1: Maestro Case | Benjamin Wakida
