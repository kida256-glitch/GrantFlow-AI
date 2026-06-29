# GrantFlow AI — 5-Minute Demo Script
## UiPath AgentHack 2026 | Track 1: Maestro Case

---

## Setup Checklist (Before Demo)
- [ ] Studio Web open at `/solution/Agent`
- [ ] Agent canvas visible with GrantFlow AI loaded
- [ ] `sample_applicants.json` open in a text editor (for copy-paste)
- [ ] Browser tab with UiPath Automation Cloud ready

---

## MINUTE 0:00 – 0:45 | The Hook (Problem Statement)

> *"Every year, scholarship providers receive thousands of applications. Each one is reviewed manually — taking weeks, introducing bias, missing fraud, and worst of all: rejecting qualified students without telling them there are OTHER scholarships they could win. GrantFlow AI changes all of that."*

**Show:** The agent.json canvas in Studio Web.

> *"This is GrantFlow AI — a single autonomous agent built on UiPath that replaces an entire manual review committee. It runs seven specialized stages in sequence, each one powered by AI, each one auditable."*

---

## MINUTE 0:45 – 1:30 | Architecture Overview

**Show:** ARCHITECTURE.md or draw on whiteboard/slide.

> *"The agent receives an application — form data plus uploaded documents. It then runs seven stages automatically:"*

Walk through the pipeline quickly:
1. **Stage 1** — "It reads the documents using UiPath's built-in Analyze Files tool — extracts names, GPAs, checks for forgeries."
2. **Stage 2** — "It checks eligibility against the scholarship rules — minimum GPA, eligible countries, degree programs."
3. **Stage 3** — "It scores the application across four dimensions: academic, leadership, community impact, essay quality."
4. **Stage 4** — "It runs fraud detection — catching impossible GPAs, name mismatches, AI-generated essays."
5. **Stage 5** — "Here's the differentiator: if rejected, it searches our scholarship catalog and recommends alternatives."
6. **Stage 6** — "It writes a professional reviewer summary for the human panel."
7. **Stage 7** — "It drafts the applicant's email — personalized, warm, never cold."

> *"All seven stages. One agent. Fully auditable."*

---

## MINUTE 1:30 – 2:30 | Demo Path 1 — APPROVE (Strong Applicant)

**Input:** Use `DEMO-001` from `sample_applicants.json` — Amara Osei, Ghana, Computer Science, GPA 3.8/4.0.

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
  "personalStatement": "[paste from DEMO-001]"
}
```

**Run the agent. Show the output:**

> *"Application ID generated: GF-UNI-20260626-0042."*
> *"Eligibility: Eligible — GPA 3.8 exceeds the 3.5 minimum, degree matches, country eligible."*
> *"Scores: Academic 92, Leadership 80, Community 85, Essay 88 — Overall: 86."*
> *"Fraud: Low risk — no flags detected."*
> *"Decision: APPROVE — with high confidence."*
> *"And here's the reviewer summary... and the applicant email, ready to send."*

**Point out:** "This took 15 seconds. A human reviewer would take 2-3 days."

---

## MINUTE 2:30 – 3:15 | Demo Path 2 — SCHOLARSHIP MATCH (Rejected + Redirected)

**Input:** Use `DEMO-002` — Priya Sharma, India, Mechanical Engineering, GPA 3.2/4.0, applying for Research Innovation Award (requires 3.7 GPA).

> *"Watch what happens when someone is rejected..."*

**Show output:**
> *"Not Eligible — GPA 3.2 does not meet the 3.7 minimum for Research Innovation Award."*

> *"But here's what makes GrantFlow different — the agent doesn't just say NO."*

> *"It found three alternative scholarships:"*
> - *"Women in Technology Grant — 91% match. Her STEM degree and gender qualify perfectly."*
> - *"STEM Excellence Award — 78% match. GPA is borderline but the degree fits."*
> - *"Emerging Leaders Scholarship — 74% match. Strong leadership evidence in her essay."*

> *"The applicant receives a warm email with these alternatives. She never gets only a rejection."*

---

## MINUTE 3:15 – 3:45 | Demo Path 3 — FRAUD DETECTION (30 seconds)

**Input:** Use `DEMO-003` — GPA 4.8/4.0 (impossible), generic AI-written essay, name mismatch.

**Show output:**
> *"HIGH FRAUD RISK ALERT — three flags detected: Impossible GPA of 4.8 on a 4.0 scale. Generic personal statement with no personal anecdotes — AI-pattern detected. No personal details whatsoever."*

> *"Decision: Request More Info — escalated to human review with a full fraud report."*

> *"The human reviewer sees exactly what the AI flagged and why. They make the final call."*

---

## MINUTE 3:45 – 4:30 | Human-in-the-Loop + Governance

> *"Every output from GrantFlow AI is a RECOMMENDATION — not a verdict."*

**Show the reviewerSummary field:**
> *"The reviewer panel sees this: a structured 250-word summary, score breakdown, fraud assessment, and a confidence level. They can approve, override, or request more information."*

> *"Every decision, every tool call, every flag — fully logged in UiPath Maestro Case Management. Complete audit trail."*

**Key points:**
- AI recommends, humans decide
- Every stage is traceable
- Escalation built into the workflow
- Appeals and overrides are tracked

---

## MINUTE 4:30 – 5:00 | Impact & Close

**Show a quick numbers slide or summarize verbally:**

| Metric | Manual Process | GrantFlow AI |
|--------|---------------|--------------|
| Review time | 2-3 days | 15-30 seconds |
| Consistency | Varies by reviewer | 100% consistent criteria |
| Fraud detection | ~60% | Systematic 7-flag check |
| Rejection w/ guidance | Rarely | Always (scholarship match) |
| Audit trail | Paper-based | Full digital trace |

> *"GrantFlow AI doesn't replace human judgment — it amplifies it. Reviewers spend their time on the edge cases and appeals, not on reading 2,000 identical forms."*

> *"Built on UiPath Automation Cloud. Powered by GPT-5.4. Orchestrated by Maestro. One agent. Seven stages. Zero students left behind."*

**CLOSE:**
> *"This is GrantFlow AI — and this is what agentic automation looks like when it's built for impact."*

---

## Fallback if Live Demo Fails

1. Walk through the `agent.json` canvas and explain each section
2. Show `sample_applicants.json` outputs side-by-side
3. Explain the architecture from `ARCHITECTURE.md`

---

## Q&A Prep

**Q: How do you prevent the AI from being wrong?**
> Human reviewers have final authority. The AI provides recommendations with confidence levels and explicit reasoning. Every override is logged.

**Q: What if documents are in different languages?**
> The Analyze Files tool supports multilingual OCR. The LLM handles translation inline.

**Q: How does this scale to 10,000 applications?**
> UiPath Orchestrator handles parallel agent execution. Each application runs as an independent case. Throughput scales with Orchestrator capacity.

**Q: What about GDPR / data privacy?**
> All document processing happens within your UiPath Automation Cloud tenant. No data leaves your environment.

**Q: Can the matching catalog be customized?**
> Yes — the catalog is defined in `scholarship_database.json` and can be extended or connected to a live database.
