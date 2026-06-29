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

GrantFlow AI is implemented as a UiPath low-code agent (`Solution/Agent/agent.json`, `"type": "lowCode"`). The solution does **not** use Coded Agents. All pipeline logic is defined through the agent system prompt, input/output schemas, and built-in tools (Analyze Files, DeepRAG) configured in Agent Builder.

---

## Setup Instructions

Follow these steps once to import and configure GrantFlow AI on your UiPath tenant.

### Prerequisites

- UiPath **Automation Cloud** account with **Studio Web** and **Agent Builder** enabled
- LLM access to **`anthropic.claude-sonnet-4-6`** on your tenant
- Built-in agent tools: **Analyze Files** and **DeepRAG**
- **UiPath Orchestrator** (required for document attachments and published runs)
- **UiPath Maestro** (Track 1 — case lifecycle integration)

### Step 1 — Get the repository

```bash
git clone https://github.com/kida256-glitch/GrantFlow-AI.git
cd GrantFlow-AI
```

All runnable agent files are under **`Solution/`**. Demo data is in **`Solution/Agent/sample_applicants.json`**.

### Step 2 — Import the solution into Studio Web

1. Log in to [cloud.uipath.com](https://cloud.uipath.com)
2. Open **Studio Web** → **Solutions**
3. Click **Import solution** and upload the **`Solution`** folder (or zip the `Solution` folder first, then upload)
4. After import, open the solution and select the **Agent** project
5. Agent Builder opens the GrantFlow AI canvas (`Solution/Agent/agent.json`)

> **Already on this tenant?** If the solution is deployed, go directly to **`/solution/Agent`** in Studio Web.

### Step 3 — Wire the built-in tools

The agent needs two tools on the canvas before it can verify documents:

1. On the Agent Builder canvas, click **+ Add Tool**
2. Add **Analyze Files** (`toolType: analyze-attachments`) and connect it to the agent
3. Click **+ Add Tool** again and add **DeepRAG** (`toolType: deep-rag`)
4. Click **Save**

Tool configs are pre-defined in `Solution/Agent/resources/`. See `Solution/Agent/SETUP_GUIDE.md` for screenshots-level detail.

### Step 4 — Validate (optional)

If you use the UiPath CLI locally:

```bash
uip agent validate /solution/Agent --output json
```

Expected: `"Status": "Valid"` with **2 resources** (Analyze Files + DeepRAG).

### Step 5 — Connect Maestro (Track 1)

1. In **UiPath Maestro**, create a case type for scholarship applications
2. Map agent **inputs** (applicant fields, `scholarshipCriteria`) and **outputs** (scores, `recommendedDecision`, `reviewerSummary`, etc.) to case properties
3. Trigger the published agent when a case moves to the evaluation stage

Output mapping reference:

| Agent output | Maestro case property |
|--------------|----------------------|
| `applicationId` | Case ID |
| `eligibilityStatus` | Case stage / status |
| `fraudRiskLevel` | Risk level |
| `recommendedDecision` | Pending action |
| `reviewerSummary` | Case description |

---

## How to Run the Agent

Use this process every time you evaluate an application.

### 1. Open the agent

1. Go to **Studio Web** → open the **GrantFlow AI** solution → **Agent** project
2. Confirm **Analyze Files** and **DeepRAG** appear on the canvas
3. Click **Run** (test panel) for a quick test, or **Publish** first for Orchestrator/Maestro runs

### 2. Provide application input

Paste a JSON object with the **9 required fields**. Optional fields: `phoneNumber` and four document attachments.

**Required fields:**

| Field | Example |
|-------|---------|
| `applicantName` | `"Amara Osei"` |
| `email` | `"amara.osei@ug.edu.gh"` |
| `country` | `"Ghana"` |
| `university` | `"University of Ghana"` |
| `degreeProgram` | `"Computer Science"` |
| `gpa` | `"3.8/4.0"` |
| `personalStatement` | Essay text |
| `scholarshipName` | `"STEM Excellence Award"` |
| `scholarshipCriteria` | JSON string, e.g. `"{\"minGpa\": 3.5, ...}"` |

**Quick test — copy DEMO-001** from `Solution/Agent/sample_applicants.json`:

```json
{
  "applicantName": "Amara Osei",
  "email": "amara.osei@university.edu",
  "phoneNumber": "+233-20-123-4567",
  "country": "Ghana",
  "university": "University of Ghana",
  "degreeProgram": "Computer Science",
  "gpa": "3.8/4.0",
  "scholarshipName": "STEM Excellence Award",
  "scholarshipCriteria": "{\"minGpa\": 3.5, \"eligibleCountries\": [\"Any\"], \"eligibleDegrees\": [\"Computer Science\", \"Engineering\", \"Mathematics\", \"Data Science\"], \"level\": [\"undergraduate\", \"graduate\"]}",
  "personalStatement": "Growing up in Accra, I witnessed firsthand how technology could transform communities..."
}
```

### 3. Execute the agent

1. Click **Run** in Agent Builder (Studio Web test panel)
2. The agent autonomously runs all **7 pipeline stages** (up to 25 iterations)
3. If documents are attached, the agent calls **Analyze Files** and **DeepRAG** in Stage 1; if not, Stage 1 is skipped and evaluation continues from Stage 2
4. Wait for the run to complete — typical runtime is under 60 seconds

### 4. Review the output

The agent returns **14 output fields**. Confirm all are populated:

| Field | DEMO-001 expected value |
|-------|-------------------------|
| `applicationId` | `GF-UNI-YYYYMMDD-XXXX` format |
| `eligibilityStatus` | `Eligible` |
| `eligibilityReason` | Reason citing criteria met |
| `academicScore` | ~90–100 |
| `leadershipScore` | 0–100 |
| `communityImpactScore` | 0–100 |
| `essayQualityScore` | 0–100 |
| `overallScore` | ~85–95 |
| `fraudRiskLevel` | `Low` |
| `fraudRiskReason` | Clean or flagged |
| `matchedScholarships` | `[]` (empty — not triggered when eligible) |
| `reviewerSummary` | 200–300 word panel summary |
| `recommendedDecision` | `Approve` |
| `notificationMessage` | Full email draft |

> All outputs are **AI recommendations**. A human reviewer makes the final award decision.

### 5. Run additional demo scenarios

Open `Solution/Agent/sample_applicants.json` and paste each `input` block into the Run panel:

| Demo ID | Scenario | Expected outcome |
|---------|----------|------------------|
| **DEMO-001** | Strong STEM applicant (Amara Osei) | Eligible · Approve · Low fraud |
| **DEMO-002** | Below GPA cutoff (Priya Sharma) | Not Eligible · `matchedScholarships` populated |
| **DEMO-003** | Impossible GPA + generic essay (John Smith) | High fraud · Reject or Request More Info |

For a live presentation walkthrough, see `Solution/Agent/DEMO_SCRIPT.md`.

### 6. Run with documents (Stage 1)

To test document verification, run via **Orchestrator** (not the Studio Web test panel alone):

1. **Publish** the agent from Studio Web (version `1.0.0`)
2. Start a job in Orchestrator with the text fields above **plus** PDF attachments for any of:
   - `academicTranscript`
   - `nationalId`
   - `cv`
   - `recommendationLetter`
3. Attachments must be Orchestrator **`JobAttachment`** objects with a valid `ID`
4. The agent extracts document data, cross-checks against the form, and flags inconsistencies in `fraudRiskReason`

### 7. Publish for production (optional)

1. In Studio Web, click **Publish** on the Agent project
2. Set version `1.0.0` and publish to **Orchestrator**
3. Run from **Orchestrator → Automations → Processes**, or trigger from a **Maestro** case

---

## Project Structure

```
GrantFlow-AI/
├── Solution/                          ← UiPath solution (start here for judging)
│   ├── Solution.uipx                  ← Solution manifest — open this in Studio Web
│   ├── SolutionStorage.json
│   └── Agent/
│       ├── agent.json                 ← Main low-code agent configuration
│       ├── entry-points.json
│       ├── project.uiproj
│       ├── sample_applicants.json     ← Demo test cases (DEMO-001, 002, 003)
│       ├── scholarship_database.json
│       ├── email_templates.json
│       ├── SETUP_GUIDE.md             ← Tool wiring and first-run guide
│       ├── DEMO_SCRIPT.md             ← 5-minute hackathon demo script
│       ├── ARCHITECTURE.md            ← System architecture diagrams
│       └── resources/
│           ├── Analyze Files/resource.json
│           └── DeepRAG/resource.json
├── data/                              ← Reference data (mirrors Solution/Agent/)
│   ├── scholarship_database.json
│   ├── sample_applicants.json
│   └── email_templates.json
├── agent/                             ← Legacy agent export (same config as Solution/Agent/)
└── README.md
```

### Additional documentation (in `Solution/Agent/`)

| File | Purpose |
|------|---------|
| `SETUP_GUIDE.md` | Step-by-step tool setup and environment configuration |
| `DEMO_SCRIPT.md` | 5-minute demo script for hackathon presentation |
| `ARCHITECTURE.md` | Pipeline architecture and Maestro case lifecycle |

---

Built for **UiPath AgentHack 2026 — Track 1: Maestro Case** | Benjamin Wakida
