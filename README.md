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

> **Important:** The `Solution/` folder is the agent **source project** — it does not run on its own from your local machine. You must **import it into Studio Web** on [cloud.uipath.com](https://cloud.uipath.com) and run it there. No local Python, Node.js, or CLI is required for judging.

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

**GrantFlow AI runs in UiPath Automation Cloud — not from local files.**

The agent executes inside **Studio Web → Agent Builder** on your UiPath tenant. The files in `Solution/` are imported into the cloud; clicking **Run** in Studio Web is how judges evaluate applications. Typical runtime is **30–60 seconds** per application.

### Where it runs

| Environment | Use case |
|-------------|----------|
| **Studio Web → Run panel** | Primary method for hackathon judging and demos (text-only input) |
| **Orchestrator** | Document attachments (`JobAttachment` PDFs) and production/Maestro triggers |
| **Local filesystem** | ❌ Does **not** execute the agent — import `Solution/` into Studio Web first |

### Step-by-step — Run in Studio Web (judging procedure)

Follow this exact sequence every time you test or demo the agent:

**Step A — Open the agent**

1. Log in to [cloud.uipath.com](https://cloud.uipath.com)
2. Open **Studio Web**
3. Navigate to your imported **GrantFlow AI** solution → **Agent** project  
   (or go directly to **`/solution/Agent`** if already deployed on your tenant)
4. Confirm **Analyze Files** and **DeepRAG** are visible on the Agent Builder canvas
5. If tools are missing, add them via **+ Add Tool** → **Save** (see Setup Step 3 above)

**Step B — Open the Run panel**

1. Click **Run** in Agent Builder (top toolbar or test panel)
2. A JSON input editor opens — this is where you paste application data

**Step C — Paste test input**

Copy the `input` object from `Solution/Agent/sample_applicants.json` (start with **DEMO-001**), or paste this:

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
  "personalStatement": "Growing up in Accra, I witnessed firsthand how technology could transform communities — or leave them behind. At 14, I taught myself Python by borrowing my neighbor's laptop. By 16, I had built a mobile app that helped 200 local traders track inventory. That experience ignited a passion that has driven every academic and professional decision since. At the University of Ghana, I maintain a 3.8 GPA while leading our 45-member Computer Science Society, organizing three national hackathons that attracted over 600 participants. I also founded Code4Africa, a volunteer program that has trained 180 high school students in basic programming across three regions. My research on low-bandwidth AI models for rural connectivity was presented at the African AI Symposium 2025. The STEM Excellence Award would enable me to pursue my master's research in edge computing for healthcare delivery in remote communities."
}
```

**Required fields (9):** `applicantName`, `email`, `country`, `university`, `degreeProgram`, `gpa`, `personalStatement`, `scholarshipName`, `scholarshipCriteria`  
**Optional:** `phoneNumber`, `academicTranscript`, `nationalId`, `cv`, `recommendationLetter`

**Step D — Execute**

1. Click **Run** / **Start** in the test panel
2. The agent autonomously executes all **7 pipeline stages** (up to 25 LLM iterations)
3. With no documents attached, **Stage 1 is skipped** and evaluation starts at Stage 2 (eligibility)
4. Wait for the run to finish — progress appears in the Studio Web run panel (~30–60 seconds)

**Step E — Review output in Studio Web**

When the run completes, Studio Web displays **14 output fields**. Verify all are populated:

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
| `matchedScholarships` | `[]` (empty when eligible) |
| `reviewerSummary` | 200–300 word panel summary |
| `recommendedDecision` | `Approve` |
| `notificationMessage` | Full email draft |

> All outputs are **AI recommendations**. A human reviewer makes the final award decision.

### Additional demo scenarios (Studio Web Run panel)

Open `Solution/Agent/sample_applicants.json`, copy each applicant's `input` block, paste into the Run panel, and click **Run**:

| Demo ID | Scenario | Expected outcome |
|---------|----------|------------------|
| **DEMO-001** | Strong STEM applicant (Amara Osei) | Eligible · Approve · Low fraud |
| **DEMO-002** | Below GPA cutoff (Priya Sharma) | Not Eligible · `matchedScholarships` populated |
| **DEMO-003** | Impossible GPA + generic essay (John Smith) | High fraud · Reject or Request More Info |

For a live presentation walkthrough, see `Solution/Agent/DEMO_SCRIPT.md`.

### Run with documents — Orchestrator only

Document verification (Stage 1) requires **Orchestrator** — the Studio Web Run panel alone cannot attach PDFs:

1. **Publish** the agent from Studio Web (version `1.0.0`)
2. Start a job in **Orchestrator** with the text fields above **plus** PDF attachments for any of:
   - `academicTranscript`
   - `nationalId`
   - `cv`
   - `recommendationLetter`
3. Attachments must be Orchestrator **`JobAttachment`** objects with a valid `ID`
4. The agent calls **Analyze Files** and **DeepRAG**, then cross-checks documents against the form

### Production / Maestro (optional)

1. **Publish** from Studio Web → **Orchestrator**
2. Trigger from **Maestro** cases or **Orchestrator → Automations → Processes**
3. Or use UiPath CLI after login: `uip agent run start <releaseKey> -i '<json>'`

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
