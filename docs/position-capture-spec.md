# Position Capture & Best-Practice Discovery — Product Spec (v0.1)

> Working name: **PositionMap**. Seed spec for brainstorming and build planning. Open questions are marked **❓** — resolve these in the brainstorming session before planning.

---

## 1. Problem

Every business, from a 5-person plumbing shop to a Fortune 500 department, runs on knowledge that lives in people's heads.

- **Documented process ≠ real process.** In one case, a Deloitte-mapped quote process had ~7 steps. Process mining found **20 steps and 7 loops**, and 61% of requests looped back to the start. A 60-person accounting firm said it had 6 steps; it had 14. *(Source: Voss / Veric Agents on Greg Isenberg's podcast, Oct 2026.)*
- **Key-person risk.** "This person's been here 20 years and just handles it." That holds at every company size.
- **Discovery is manual and expensive.** Forward-deployed engineers spend 2–3 weeks interviewing staff before they build anything. SMBs can't afford that, and PE roll-ups can't scale it.
- **Nobody knows which way of doing the job works best.** Five people in the same role do it five ways. One of them gets better results, and no one knows why.

## 2. Product in one line

The owner defines the positions. An AI interviewer captures how each person actually does the job, probing until nothing is left out. The system then compares people in the same role against their results, to find the best way to do the job and what should be deleted, delegated, coded, or handed to an agent.

## 3. Goals (outcomes for the customer)

| # | Outcome | Question it answers | Output |
|---|---|---|---|
| G1 | **Transferable positions** | Could someone else do this job next Monday? | Position playbook |
| G2 | **Best-practice discovery** | Who does it best, and what do they do differently? | Variance report + standard procedure |
| G3 | **Delegation** | What could a junior do? | Delegation list with instructions |
| G4 | **Automation** | What should no human be doing? | Automation backlog (MacTutor upsell) |
| G5 | **Training gaps** | What is each person missing vs the best practice? | Per-person gap list (owner/manager only) |

**Non-goals (v1):** screen recording, process mining of system logs (Phase 2), replacing the customer's systems of record, HR performance management.

## 4. Users

| Persona | Does | Sees |
|---|---|---|
| **Owner / Executive** | Defines org blueprint, invites staff, approves playbooks | Everything incl. variance, automation, gaps |
| **Manager** | Reviews playbooks for their team | Their team's playbooks, variance, gaps |
| **Staff member** | Completes interviews, reviews own write-up | Only their own playbook + the approved standard procedure |
| **Consultant (MacTutor / partner agency)** | Runs engagements across multiple client orgs | Multi-tenant view, automation backlog, benchmark library |

## 5. System layers

```
Layer 1  ORG BLUEPRINT      Owner defines positions + responsibilities (or picks an industry template)
              ↓
Layer 2  INDIVIDUAL CAPTURE AI interviewer → structured procedures/steps per person
              ↓
Layer 3  VARIANCE ENGINE    Align steps across people in same position → find differences
              ↓
Layer 4  OUTCOME LINKAGE    Join differences to performance data → candidate best practices
              ↓
Layer 5  OUTPUTS            Playbooks · Standard procedures · Step classification · Automation backlog · Gaps
```

---

## 6. Data model (Supabase / Postgres)

Multi-tenant on `org_id` with RLS. A consultant account can manage many orgs.

| Table | Key fields | Notes |
|---|---|---|
| `orgs` | id, name, industry, size_band, consultant_id | Tenant |
| `positions` | id, org_id, name, template_position_id, reports_to_position_id | "Service Technician", "CSR" |
| `responsibilities` | id, position_id, title, description, source (template/owner/interview) | Owner seeds them, interviews refine them |
| `people` | id, org_id, name, email, external_ids (jsonb: servicetitan_id, crm_user_id…) | external_ids link to outcome data |
| `person_positions` | person_id, position_id, start_date | A person can hold several positions |
| `interview_sessions` | id, person_id, position_id, status, mode (voice/chat), consent_at, started_at, completed_at, transcript_ref | Resumable; consent is mandatory |
| `procedures` | id, org_id, position_id, person_id (nullable = standard), name, trigger, frequency, est_minutes, canonical_procedure_id | Per-person version, plus one canonical standard |
| `steps` | id, procedure_id, seq, action, tool_id, input, output, handoff_to_position_id, decision_rule, est_minutes, wait_time, judgment_level, exception_of_step_id, loop_to_step_id, loop_frequency_pct, confidence, evidence (transcript quote ref) | The atomic unit. Loops and exceptions are first-class |
| `step_alignments` | canonical_step_id, step_id, match_type (same/variant/extra/missing), similarity | Output of the variance engine |
| `tools` | id, org_id, name, category, vendor (ServiceTitan, QuickBooks, Gmail, spreadsheet, paper) | Systems inventory |
| `policies` | id, org_id, position_id, rule, source_person_id, conflicts_with_policy_id | e.g. "discounts >10% need owner approval" |
| `outcome_metrics` | id, org_id, position_id, name, unit, direction (higher/lower better), source | close_rate, avg_ticket, callback_rate, booking_rate |
| `outcome_values` | metric_id, person_id, period_start, period_end, value, n (sample size), source_ref | Imported or entered |
| `step_classifications` | step_id, bucket, rationale, est_savings_min_per_month, reviewer_id | See §8 |
| `findings` | id, org_id, position_id, type (best_practice/gap/conflict/automation), summary, evidence (jsonb), confidence, status | What reports render |

**❓** Separate per-person procedures + one canonical version (as above), or one procedure with per-person step variants?

---

## 7. The interviewer

### 7.1 Principles
1. **Probe until exhausted.** Never accept a one-line answer for a step. Ask why, ask what else, ask whether they're sure.
2. **Concrete over abstract.** "Walk me through the last time you did this" beats "how do you usually do this."
3. **Exceptions are the knowledge.** The happy path is cheap. The value is in "what happens when…"
4. **Capture structure, not just text.** Every answer maps to the step schema (§7.4). The interviewer knows which fields are still empty and asks for them.
5. **Quantify.** How often, how long, how many times does it loop?
6. **Neutral, non-threatening tone.** The framing is "help us capture your expertise," never "justify your job."

### 7.2 Session flow (20–30 min sessions, resumable)

| Phase | Goal | Example prompts |
|---|---|---|
| 0. Consent | Recording + data-use consent (FL all-party consent) | Scripted, logged with timestamp |
| 1. Role overview | Confirm and extend owner-seeded responsibilities | "Your owner listed these 6 responsibilities. What's missing? What would break if you were out a week?" |
| 2. Rhythm | Find all procedures | "Walk me through yesterday. What happens only weekly? Monthly? Seasonally?" |
| 3. Procedure deep-dive (repeat) | Fill the step schema | "What kicks this off? First thing you do? Which screen? Then what? Who gets it next?" |
| 4. Exceptions & loops | Find branches and rework | "What if the customer doesn't answer? How often does it come back to you? Out of 10, how many?" |
| 5. Policies & decisions | Capture the decision rules | "How do you decide X? Who can override? Is that written anywhere?" |
| 6. Tools | Systems inventory | "Which software, logins, spreadsheets, sticky notes, texts?" |
| 7. Friction | Automation candidates | "What's repetitive? What do you re-type? What do you wait on?" |
| 8. Read-back | Validate | Agent summarizes the procedure step by step; person confirms or corrects |

### 7.3 Probing ladder (applied per step)

| Trigger in answer | Probe |
|---|---|
| Vague verb ("I handle it", "I follow up") | "Walk me through exactly what you do. What's the first action?" |
| Step without a reason | "Why is that step there? What goes wrong if you skip it?" (catches theater) |
| Step without a tool | "Where do you do that — which system, or is it paper/phone?" |
| Step without a handoff | "When you're done, who gets it or what happens next?" |
| Decision ("depends", "if it's big") | "What's the rule? What's the threshold? Who taught you that?" |
| Waiting ("then I wait") | "How long, typically? What do you do if it's late?" |
| Happy path only | "What's the most common thing that goes wrong here?" |
| Unquantified frequency | "Out of 10 times, how many?" |
| Answer feels complete | "Is there anything else, even something small you do out of habit?" |
| Contradiction with an earlier answer or with a colleague | "Earlier you said X; another person in this role said Y. Which is right, or do both happen?" |

**Stop rule:** a step is complete when all required schema fields are filled, or the person explicitly says they don't know (recorded as `confidence=low`). **❓** Max probes per step before moving on, to avoid fatigue.

### 7.4 Step schema (interviewer must fill)

```json
{
  "action": "Call customer to confirm appointment",
  "trigger": "Appointment is tomorrow",
  "tool": "Housecall Pro + personal cell",
  "input": "Tomorrow's schedule",
  "output": "Confirmed / rescheduled / no answer",
  "handoff_to": "Dispatch (if rescheduled)",
  "decision_rule": "If no answer twice, text; if no reply by 4pm, flag to dispatch",
  "frequency": "Daily, ~15 calls",
  "est_minutes": 2,
  "wait_time": "Up to 4 hours for reply",
  "judgment_level": "low | medium | high",
  "exceptions": [{"condition": "Customer wants different day", "path": "loop to Dispatch scheduling", "frequency_pct": 20}],
  "why": "Reduces no-shows",
  "confidence": "high | medium | low",
  "evidence": ["transcript:session_12@00:14:32"]
}
```

### 7.5 Cross-interview awareness
When interviewing person N in a position, the agent has the (anonymized) aligned procedure from persons 1..N-1 and probes for steps others mentioned that this person hasn't: "Some people in this role also do X — do you?" **❓** Does this bias answers? Possibly only use it after the person's free description.

---

## 8. Step classification (5 buckets)

Adapted from the Veric Agents framework, with **Delegate** added.

| Bucket | Rule of thumb | Example |
|---|---|---|
| **Delete** | No one can say why it exists; output unused; duplicate | Re-entering job notes into a spreadsheet nobody reads |
| **Code** | Deterministic if-X-then-Y, no judgment, digital inputs | Send confirmation text the day before an appointment |
| **Agent** | Judgment needed but learnable from history/rules; digital | Draft quote follow-up email tailored to job type |
| **Delegate** | Human needed, but rules are clear enough for a junior | Collecting permit documents |
| **Human (keep)** | High judgment, high risk, relationship, money movement | Pricing a complex repair; approving refunds; payments |

Scoring inputs: judgment_level, input digitization, frequency × est_minutes (size of prize), error cost, loop frequency. The AI proposes the bucket, and the consultant/owner confirms it. **❓** Weighted score vs rules vs LLM judgment with rationale?

---

## 9. Variance engine (Layer 3)

1. **Normalize:** map each person's steps to canonical actions (embedding similarity + LLM adjudication).
2. **Align:** build a canonical procedure as the union of steps, ordered. Mark each person's steps as `same / variant / extra / missing`.
3. **Detect:**
   - Extra steps only some people do
   - Missing steps
   - Order differences
   - Policy conflicts (different decision rules for the same situation)
   - Tool differences (one person uses a spreadsheet, others use the system)
4. **Render:** person × step matrix (the "who does what" grid).

## 10. Outcome linkage (Layer 4)

| Position | Example metrics | Typical source |
|---|---|---|
| Sales / Estimator | Close rate, avg deal size, cycle time | CRM, ServiceTitan |
| Technician | Avg ticket, callback rate, membership conversion, jobs/day | ServiceTitan, Housecall Pro, Jobber |
| CSR / Booking | Booking rate, calls handled, no-show rate | Phone system, FSM |
| Dispatch | On-time arrival %, drive time, jobs/tech/day | FSM |
| Accounting | Days to invoice, DSO, invoice error rate | QuickBooks |

**Logic:** for each step variance, compare the outcome of people who do it vs those who don't. Output a **finding** with effect size, sample size, and confidence.

**Guardrails (must ship in v1):**
- Always show n (sample size). Fewer than 3 people per group → label "anecdotal."
- Language is "pattern worth testing," never "proven."
- Suggest a 60-day adoption test and re-measure (closes the loop, and drives recurring revenue).
- Flag confounders the owner can annotate (territory, tenure, lead source).

**❓** v1 outcome data: manual entry / CSV upload first, connectors (ServiceTitan, QuickBooks, HubSpot) later?

---

## 11. Outputs

| Output | Audience | Content |
|---|---|---|
| Position playbook | Staff + owner | Responsibilities, procedures, steps, tools, policies, exceptions |
| Standard procedure | Staff + owner | Canonical best-practice version per procedure |
| Variance report | Owner/manager | Person × step grid, differences, linked outcomes |
| Best-practice findings | Owner/manager | "Rep 2 adds X, Y, drops Z; close rate 31% vs 20%; n=5; test it" |
| Training gaps | Owner/manager | Per person: steps missing vs standard |
| Delegation list | Owner | Steps to hand to a junior, with instructions |
| Automation backlog | Owner + consultant | Code/Agent steps ranked by minutes/month saved; feeds the MacTutor proposal |
| Systems inventory | Owner + consultant | Every tool, who uses it, for what |
| Process map | Everyone | Cross-position flow with handoffs, loops, loop frequencies |

Export: Markdown/PDF playbooks; push to Trainual/Notion/Google Docs (later).

## 12. Industry template: Plumbing (seed)

| Position | Core responsibilities | Typical tools | Outcome metrics |
|---|---|---|---|
| **Owner** | Pricing, hiring, key accounts, cash | QuickBooks, FSM | Revenue, margin |
| **CSR / Booking** | Answer calls, qualify, book, confirm, reschedule, membership upsell on call | Phone system, FSM | Booking rate, call answer rate |
| **Dispatcher** | Assign jobs, manage board, reroute, emergencies, tech communication | FSM board, maps, texts | On-time %, jobs/tech/day |
| **Service Technician** | Diagnose, present options, perform repair, sell memberships, document job, collect payment | FSM mobile app, pricebook, card reader | Avg ticket, callback rate, membership sales |
| **Install / Project Lead** | Water heater/repipe installs, permits, inspections | FSM, permit portal | Install margin, inspection pass rate |
| **Estimator / Comfort Advisor** | Site visits, proposals, follow-up | FSM, proposal tool | Close rate, avg sale |
| **Accounting** | Invoicing, collections, payroll, AP, job costing | QuickBooks, FSM | Days to invoice, DSO |
| **Service Manager** | Tech performance, callbacks, training, warranty | FSM reports | Callback rate, tech KPIs |

Next templates: HVAC, Electrical, B2B Sales team, Accounting firm, Law firm.

---

## 13. Privacy, consent & trust

- **Florida § 934.03 (all-party consent):** spoken and written consent at the start of every voice session; log the timestamp; allow chat-only mode.
- **Visibility:** staff never see other people's names in variance data, gaps, or the automation backlog. The owner controls sharing.
- **Framing:** "capture your expertise / make your job transferable so you can take vacation and grow," not "audit."
- **Data:** per-org isolation (RLS); transcripts retained per org policy; deletion on request.
- **❓** Should staff see the final standard procedure, with credit for their contributions ("built from Rep 2's method")?

## 14. MVP scope (proposal)

| In | Out (later) |
|---|---|
| Org blueprint + plumbing/HVAC templates | Connectors (ServiceTitan, QuickBooks, HubSpot) |
| Chat interviewer with probing ladder + step schema | Voice interviewer (fast follow) |
| Read-back validation | Process mining of system logs |
| Variance grid (same position) | Cross-company benchmark library |
| Manual/CSV outcome entry + findings with guardrails | Auto re-interview on role change |
| 5-bucket classification (AI-proposed, human-confirmed) | Trainual/Notion export |
| Playbook + automation backlog export (Markdown/PDF) | White-label for agencies |

**Stack:** React + Supabase (Postgres, RLS, Auth, Edge Functions), LLM interviewer via agent loop with structured tool calls (`upsert_step`, `flag_exception`, `record_policy`, `mark_complete`).

## 15. Business model hooks

- **Done-for-you diagnostic:** $3.5K–$7.5K by company size, credited toward automation build.
- **Hybrid:** setup + $299–$499/mo for a living manual (re-interviews, new hires, adoption tests).
- **Self-serve SaaS:** $199–$499/mo flat by size band.
- **White-label for agencies/FDE teams/PE roll-ups:** per-seat + per-assessment.
- **Data moat:** anonymized cross-company role benchmarks ("top-quartile plumbing CSRs do these 6 things").

## 16. Open questions for brainstorming

1. Who is the first buyer: SMB owner direct, MacTutor consulting engagements, or agencies/PE?
2. Chat-first or voice-first for v1? Techs in trucks may only do voice.
3. How does the interviewer avoid fatigue but still exhaust the process (session length, probe caps, split sessions)?
4. Step normalization: how reliably can an LLM align "call customer" vs "confirm appointment by phone"?
5. Minimum viable outcome data: what's the least the owner must provide to make findings credible?
6. Should cross-interview probing (§7.5) happen before or after free description?
7. How is a "canonical/standard procedure" approved — by owner, by vote, by outcome data?
8. Re-interview triggers: role change, departure notice, quarterly refresh?
9. How do we handle people who hold multiple positions (common in small shops)?
10. Product name and positioning: "make every position transferable" vs "find your best way of working" vs "AI process discovery."

## 17. Evidence & references

- Greg Isenberg, *Masterclass: How FDEs make $1M/yr deploying AI agents* (Startup Ideas Podcast, Oct 2026), with Voss of Veric Agents — process mapping method, documented-vs-real quote process, 4-bucket step sorting, invoice KPIs (17→7 steps, 24→6 days, $31→$6 per invoice).
- Research report: *SME interview knowledge capture tools* (Oct 3, 2026) — competitor landscape (Ontora, Klarity, Audity, Tacit, Sensay, Nuggetz), demand signals, pricing benchmarks, Florida recording-consent risk.
