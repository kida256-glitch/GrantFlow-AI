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

Follow these steps to configure and run GrantFlow AI for judging.

### Prerequisites

- UiPath Automation Cloud tenant with access to **Studio Web** and **Agent Builder**
- **UiPath Maestro** enabled (Track 1: Maestro Case)
- **UiPath Orchestrator** for job execution and document attachments
- Built-in agent tools available: **Analyze Files** and **DeepRAG**

### Step 1 — Clone the repository

```bash
git clone https://github.com/kida256-glitch/GrantFlow-AI.git
cd GrantFlow-AI
```

The runnable UiPath solution lives in the **`Solution/`** folder. Reference data and legacy agent exports are also available under `data/` and `agent/`.

### Step 2 — Open the solution in Studio Web

1. Log in to [cloud.uipath.com](https://cloud.uipath.com) and open **Studio Web**
2. **Import** or **Open** the solution from the `Solution/` folder:
   - Solution file: `Solution/Solution.uipx`
   - Agent project: `Solution/Agent/project.uiproj`
3. In the solution explorer, open the **Agent** project — this loads `Solution/Agent/agent.json` in Agent Builder

> If the solution is already deployed on your tenant, you can also navigate directly to **`/solution/Agent`** in Studio Web.

### Step 3 — Verify agent configuration

1. Open `Solution/Agent/agent.json` in Agent Builder and confirm:
   - **Model:** `anthropic.claude-sonnet-4-6`
   - **Type:** Low-code
   - **Max iterations:** 25
   - **Temperature:** 0
2. Confirm the two built-in tools are enabled under **Resources**:
   - **Analyze Files** (`Solution/Agent/resources/Analyze Files/resource.json`)
   - **DeepRAG** (`Solution/Agent/resources/DeepRAG/resource.json`)

If the tools are not yet on the canvas, click **+ Add Tool** and add **Analyze Files** and **DeepRAG** from the built-in tools list, then **Save**. See `Solution/Agent/SETUP_GUIDE.md` for detailed tool wiring steps.

### Step 4 — Connect Maestro (Track 1)

1. In **UiPath Maestro**, create or open a case type for scholarship applications
2. Map agent inputs (applicant details, scholarship criteria) and outputs (scores, decision, notification) to case fields
3. Trigger the GrantFlow AI agent when a new application case is created or submitted

### Step 5 — Run a test (no documents)

Use **DEMO-001** from `Solution/Agent/sample_applicants.json` (or `data/sample_applicants.json`) — a strong STEM applicant expected to be **Approved**.

1. In Agent Builder, click **Run** (or open the agent test panel)
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

### Step 6 — Run additional demo scenarios

| Demo ID | Scenario | Expected outcome |
|---------|----------|------------------|
| **DEMO-001** | Strong STEM applicant | Eligible · Approve · Low fraud |
| **DEMO-002** | Below GPA cutoff (Priya Sharma) | Not Eligible · Scholarship matches returned |
| **DEMO-003** | Impossible GPA + generic essay (John Smith) | High fraud risk · Reject or manual review |

Full inputs for DEMO-002 and DEMO-003 are in `Solution/Agent/sample_applicants.json`. For a guided 5-minute walkthrough, see `Solution/Agent/DEMO_SCRIPT.md`.

### Step 7 — Test with documents (optional)

To exercise **Stage 1 (Document Verification)**, attach PDFs via Orchestrator `JobAttachment` fields:

- `academicTranscript`
- `nationalId`
- `cv`
- `recommendationLetter`

The agent will invoke **Analyze Files** and **DeepRAG** to extract and cross-check document content against the application form.

### Step 8 — Publish to Orchestrator (optional)

1. In Studio Web, click **Publish** on the Agent project
2. Set version `1.0.0` and publish to **Orchestrator**
3. Run the process from Orchestrator → **Automations** → **Processes**, or trigger it from a Maestro case

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
