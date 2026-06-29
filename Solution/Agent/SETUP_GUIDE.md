# GrantFlow AI — Tool Setup Guide

## Adding Built-In Tools (2 minutes)

The GrantFlow AI agent is configured to use **Analyze Files** and **DeepRAG** for document processing. These tools are pre-registered in your Studio Web environment and need to be wired to the agent canvas.

### Step 1: Open the Agent Canvas

1. Open Studio Web
2. Navigate to `/solution/Agent`
3. The GrantFlow AI agent canvas will open

### Step 2: Add Analyze Files Tool

1. Click **"+ Add Tool"** (or the tool icon) on the canvas
2. Select **"Analyze Files"** from the built-in tools list
3. The tool will appear on the canvas with `toolType: analyze-attachments`
4. Connect it to the agent node

### Step 3: Add DeepRAG Tool

1. Click **"+ Add Tool"** again
2. Select **"DeepRAG"** from the built-in tools list
3. The tool will appear on the canvas with `toolType: deep-rag`
4. Connect it to the agent node

### Step 4: Save and Validate

Click **Save** — the tools are now wired and the agent is ready to run.

---

## What Each Tool Does in GrantFlow AI

### Analyze Files (analyze-attachments)
Used in **Stage 1 — Document Verification**

When a user submits documents (transcript, ID, CV, recommendation letter), the agent calls this tool with:
```json
{
  "attachments": [{ "ID": "...", "FullName": "transcript.pdf" }],
  "analysisTask": "Extract student name, university, degree program, GPA, graduation year. Flag anomalies: blank pages, illegible text, name mismatches, inconsistent letterheads."
}
```
Returns: `{ "analysis": "Student: Amara Osei, University: University of Ghana, GPA: 3.8, Degree: Computer Science..." }`

### DeepRAG (deep-rag)
Used for **deep synthesis** of long documents (50+ page transcripts, detailed CVs)

When a document needs comprehensive analysis beyond extraction, the agent calls DeepRAG with the full document attachment. Returns an array of cited content chunks.

---

## Running Your First Test

### Option A: Quick Text-Only Test (no documents)
Use sample applicant `DEMO-001` from `sample_applicants.json`:

```json
{
  "applicantName": "Amara Osei",
  "email": "amara.osei@university.edu",
  "country": "Ghana",
  "university": "University of Ghana",
  "degreeProgram": "Computer Science",
  "gpa": "3.8/4.0",
  "scholarshipName": "STEM Excellence Award",
  "scholarshipCriteria": "{\"minGpa\": 3.5, \"eligibleCountries\": [\"Any\"], \"eligibleDegrees\": [\"Computer Science\"]}",
  "personalStatement": "Growing up in Accra, I witnessed firsthand how technology could transform communities..."
}
```

**Expected output:** Eligible, Low fraud, Approve, overall score 85-95.

### Option B: Fraud Detection Test
Use `DEMO-003` — GPA 4.8/4.0 (impossible), generic AI essay.
**Expected:** HIGH FRAUD RISK ALERT, Request More Info.

### Option C: Scholarship Match Test  
Use `DEMO-002` — Priya Sharma, GPA below Research Innovation Award minimum.
**Expected:** Not Eligible + 3 matched scholarship recommendations.

---

## Environment Variables (Production Setup)

For production deployment, configure these in Orchestrator:

| Variable | Description |
|----------|-------------|
| `SCHOLARSHIP_DB_URL` | URL to live scholarship database API |
| `EMAIL_SERVICE_URL` | SMTP or SendGrid API endpoint |
| `NOTIFICATION_EMAIL_FROM` | Sender email for applicant notifications |
| `REVIEW_PORTAL_URL` | URL to human review dashboard |
| `AUDIT_LOG_ENDPOINT` | Endpoint for case audit trail logging |

---

## Maestro Case Integration

To integrate with UiPath Maestro Case Management:

1. Create a new Case Management project
2. Define the case lifecycle stages (see `ARCHITECTURE.md` state diagram)
3. Wire the GrantFlow AI agent as a subprocess at each stage
4. Configure human review tasks at the `Human Review` stage
5. Set up escalation rules for High Fraud Risk cases

The agent outputs map directly to Maestro case properties:
- `applicationId` → Case ID
- `eligibilityStatus` → Case Stage property  
- `fraudRiskLevel` → Risk Level property
- `recommendedDecision` → Pending Action property
- `reviewerSummary` → Case Description
