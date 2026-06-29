# GrantFlow AI — Scholarship & Student Financial Aid Processing Platform

> **UiPath AgentHack 2026 — Track 1: UiPath Maestro Case**

---

## Project Description

GrantFlow AI is an autonomous scholarship application processing platform that helps universities, foundations, and financial aid offices evaluate student applications at scale while keeping humans in final control of award decisions.

**The problem:** Scholarship programs receive hundreds or thousands of applications per cycle. Manual review is slow, inconsistent, and prone to bias. Reviewers must verify documents, check eligibility rules, score essays and extracurriculars, detect fraud, match rejected applicants to alternative funding, and draft applicant communications — often under tight deadlines.

**The solution:** GrantFlow AI runs a 7-stage autonomous pipeline for every application. It verifies uploaded documents, assesses eligibility against scholarship criteria, scores applicants across four dimensions, flags fraud risk, recommends alternative scholarships when appropriate, produces a 200–300 word summary for human reviewers, and drafts a professional notification email. All outputs are structured AI recommendations; a human review panel retains final authority.

| Stage | Description |
|-------|-------------|
| 1 | Document Verification (Analyze Files + DeepRAG) |
| 2 | Eligibility Assessment (rules engine) |
| 3 | AI Scoring (academic, leadership, community, essay) |
| 4 | Fraud Detection (Low / Medium / High risk) |
| 5 | Scholarship Matching (alternatives if rejected or low score) |
| 6 | Reviewer Summary (200–300 word AI summary) |
| 7 | Applicant Notification (professional email draft) |

---

## UiPath Components

| Component | Role in GrantFlow AI |
|-----------|----------------------|
| **UiPath Agent Builder (Studio Web)** | Hosts and configures the GrantFlow AI low-code agent |
| **Low-code Agent** | Executes the 7-stage scholarship evaluation pipeline via system prompt and structured I/O |
| **Analyze Files** (built-in tool) | Extracts and analyzes uploaded application documents (transcripts, CV, ID, recommendation letters) |
| **DeepRAG** (built-in tool) | Deep document synthesis with citations for academic transcript verification |
| **UiPath Maestro** | Tracks scholarship application cases through the review lifecycle (Track 1 requirement) |
| **UiPath Orchestrator** | Runs agent jobs and manages document attachments (`JobAttachment` inputs) |
| **Claude Sonnet 4.6** (`anthropic.claude-sonnet-4-6`) | LLM powering eligibility, scoring, fraud detection, matching, and notification drafting |

**Supporting data files** (reference data for judges and demos):

- `data/scholarship_database.json` — 10-scholarship catalog used for alternative matching
- `data/sample_applicants.json` — Three demo scenarios (Approve, Scholarship Match, High Fraud)
- `data/email_templates.json` — Notification templates for approved, rejected, and follow-up cases

---

## Agent Type

**Low-code Agent only.**

GrantFlow AI is implemented as a UiPath low-code agent (`agent/agent.json`, `"type": "lowCode"`). The solution does **not** use Coded Agents. All pipeline logic is defined through the agent system prompt, input/output schemas, and built-in tools (Analyze Files, DeepRAG) configured in Agent Builder.

---

## Setup Instructions

Follow these steps to configure and run GrantFlow AI for judging.

### Prerequisites

- UiPath Automation Cloud tenant with access to **Studio Web** and **Agent Builder**
- **UiPath Maestro** enabled (Track 1: Maestro Case)
- **UiPath Orchestrator** for job execution and document attachments
- Built-in agent tools available: **Analyze Files** and **DeepRAG**

### Step 1 — Import the project

1. Clone or download this repository: [github.com/kida256-glitch/GrantFlow-AI](https://github.com/kida256-glitch/GrantFlow-AI)
2. In **Studio Web**, open the Agent solution at `/solution/Agent`
3. Import or open the `agent/` folder contents (`project.uiproj`, `agent.json`, `entry-points.json`, and `resources/`)

### Step 2 — Verify agent configuration

1. Open `agent/agent.json` in Agent Builder and confirm:
   - **Model:** `anthropic.claude-sonnet-4-6`
   - **Type:** Low-code
   - **Max iterations:** 25
   - **Temperature:** 0
2. Confirm the two tools are enabled under **Resources**:
   - `Analyze Files/` (`agent/resources/analyze-files/resource.json`)
   - `DeepRAG/` (`agent/resources/deeprag/resource.json`)

### Step 3 — Connect Maestro (Track 1)

1. In **UiPath Maestro**, create or open a case type for scholarship applications
2. Map agent inputs (applicant details, scholarship criteria) and outputs (scores, decision, notification) to case fields
3. Trigger the GrantFlow AI agent when a new application case is created or submitted

### Step 4 — Run a test (no documents)

Use **DEMO-001** from `data/sample_applicants.json` — a strong STEM applicant expected to be **Approved**.

1. In Agent Builder, open the agent test panel
2. Paste the following input:

```json
{
  "applicantName": "Amara Osei",
  "email": "amara.osei@ug.edu.gh",
  "country": "Ghana",
  "university": "University of Ghana",
  "degreeProgram": "Computer Science",
  "gpa": "3.8/4.0",
  "personalStatement": "Growing up in Accra, I taught myself Python at 14 and built a mobile app helping 200 local market traders. I founded Code4Africa, training 180 students in programming. My AI research was presented at the African AI Symposium 2026.",
  "scholarshipName": "STEM Excellence Award",
  "scholarshipCriteria": "{\"minGpa\": 3.5, \"eligibleCountries\": [\"Any\"], \"eligibleDegrees\": [\"Computer Science\"]}"
}
```

3. Run the agent and verify outputs:
   - `eligibilityStatus`: **Eligible**
   - `overallScore`: ~**85**
   - `fraudRiskLevel`: **Low**
   - `recommendedDecision`: **Approve**
   - All **14 output fields** populated

### Step 5 — Run additional demo scenarios

| Demo ID | Scenario | Expected outcome |
|---------|----------|------------------|
| **DEMO-001** | Strong STEM applicant | Eligible · Approve · Low fraud |
| **DEMO-002** | Below GPA cutoff (Priya Sharma) | Not Eligible · Scholarship matches returned |
| **DEMO-003** | Impossible GPA + generic essay (John Smith) | High fraud risk · Reject or manual review |

Full inputs for DEMO-002 and DEMO-003 are in `data/sample_applicants.json`.

### Step 6 — Test with documents (optional)

To exercise **Stage 1 (Document Verification)**, attach PDFs via Orchestrator `JobAttachment` fields:

- `academicTranscript`
- `nationalId`
- `cv`
- `recommendationLetter`

The agent will invoke **Analyze Files** and **DeepRAG** to extract and cross-check document content against the application form.

---

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

---

Built for **UiPath AgentHack 2026 — Track 1: Maestro Case** | Benjamin Wakida
