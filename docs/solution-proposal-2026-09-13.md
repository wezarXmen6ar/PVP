# PRMS — Full Ecosystem Solution Proposal

**Status:** Proposal, for your review. Not yet approved. Nothing here is built.
**Date:** 13 September 2026
**Supersedes, if accepted:** the three component specs and the open B1–B8 items — this document proposes a resolution for every one of them, marked where it does.
**Author:** drafted for the Technical PM, GD of AI, from the README mission, the BRD, the three design specs, the shared-definitions file and the design audit.

---

## 0. How to read this

This is one coherent design for the whole system — every page, every account type, every feature, and how they fit. It keeps everything you validated in mockups, changes a few things where the audit showed they worked against the mission, and adds the parts that were missing. Sections are written so that any one of them can later be pointed at and turned into a visual mockup or a build spec on its own.

Where I made a judgment call you might not agree with, it is marked **[choice]** and listed again in §12 so you can accept or overrule each one in a single pass.

---

## 1. The problem, in one paragraph

GD of AI delivers projects for the whole force from one finite pool of people. Business owners add requirements without seeing what they cost. HQ pauses running projects for surprise priorities without seeing what that displaces. Status is rebuilt by hand in PowerPoint for every meeting, so no two versions agree. And when a date slips, there is no record of *why* — so it defaults to being GD of AI's fault. The mission: **make the cost of every project decision visible to the person making it, before they make it, and keep the receipt afterwards.**

## 2. Design principles

Everything below follows from six rules. When two features conflict, these decide.

1. **One record, many views.** There is exactly one durable truth per project — a ledger of what happened. Every screen is a rendering of it. No page keeps its own version of the facts.
2. **Every decision has a receipt.** A change request, a hold, a baseline, a sign-off — each is an event with a cause and, wherever a real approval happened, the document proving it.
3. **Cost is shown before commitment.** A change or a pause is always previewed with its day-impact first. Committing is a separate, deliberate act.
4. **Simulate freely, commit deliberately.** Anyone can play "what if" without risk. Only an explicit commit touches the record, and a commit always produces events.
5. **Attribute cause, never blame.** The system states what happened and who requested it, in the same neutral tone whether the cause was GD of AI, a business owner, or HQ — and it shows external causes as prominently as internal ones.
6. **Flexibility first, rules later.** No mandatory blocks, no forced order, soft warnings not hard blocks. Structure can be tightened once the tool has proven itself.

## 3. Where the ideas come from

None of this is invented from nothing. Each mechanism is borrowed from a system that already does it well, then fitted to your situation.

| Mechanism in PRMS | Borrowed from | What it is there |
|---|---|---|
| Plan of Record + Scenarios (§4.1) | **Git** branches & commits | Work on a branch freely; merge deliberately; history is append-only |
| Baseline & variance (§4.2) | **MS Project / Primavera** | Freeze the approved plan; measure every later change against it |
| Event ledger & activity feed (§4.4) | **Jira / Linear** issue history | Every state change is an immutable, timestamped, attributed entry |
| Replay scrubber (§7.7) | **Google Docs version history** | Drag a slider, see the document as it was on any day |
| Resource swimlane & capacity (§7.6) | **Float / Planview / Smartsheet Resource Mgmt** | People as rows, assignments as bars on a shared calendar |
| Evidence attached to decisions (§4.5) | **DocuSign / procurement systems** | The approval artifact lives *with* the decision, not in a separate folder |
| Verified progress, not elapsed time (§4.6) | **Basecamp hill charts / Linear** | Done means done, confirmed — not "the days have passed" |
| My Work for resources (§7.5) | **Todoist / Trello personal boards** | One person's queue across all projects, nothing else |
| Stakeholder secure links (§7.8) | **Asana / Monday guest views** | Read-only access by link, no account, always current |

## 4. The core model — six concepts everything else uses

### 4.1 Plan of Record, Draft, and Scenario

Every project has one **Plan of Record (PoR)**: its committed blocks, effort, team and dates. The PoR is never edited directly. It changes only when an **event** is written to the ledger (§4.4) — a baseline, an approved change request, a hold, an override, a swap, a re-baseline.

Before a project is baselined, its plan is a **Draft** — freely editable in the Plan Editor (the sandbox you validated) with no ceremony, because nothing has been promised to anyone yet.

A **Scenario** is a fork. It copies the PoR of one or more projects (and can add hypothetical projects that don't exist yet), and you can then do anything to it — drag blocks, pull people, insert pauses, add scope — with zero effect on any record. A scenario can be saved, named, compared side-by-side against the PoR, and projected in a meeting. It ends one of two ways: **discarded**, or **committed** — and committing translates the scenario's differences into real events, each of which requires its cause and, where relevant, its evidence.

This single distinction fixes the audit's biggest structural finding: the sandbox remains exactly as free as it is today, but it is now explicitly a *scenario*, never the record. The record cannot be mutated silently because there is no way to mutate it except through events.

> The BRD already lists "scenario planning with multiple saved what-if scenarios" as a Phase 4 enhancement (§8.3). This proposal pulls it forward, because it turned out to be the only clean answer to "what is the sandbox?" — not a nice-to-have.

### 4.2 Lifecycle state vs. current phase **[resolves B1, B2]**

These were being conflated. They are two different things, and both are needed.

**Lifecycle state** — where the project is administratively. Five values: **Draft → Active → On Hold → Launched**, plus **Cancelled** from anywhere. Active is entered by baselining. On Hold is *derived*: it means "there is an open hold event," not a status someone sets. Launched is terminal.

**Current phase** — which block the project is actually in right now: Requirements Gathering, Analysis, Development, Testing, Security Testing, UAT, Deployment, or Trial. Derived from block completion (§4.6), not from the calendar.

The Portfolio shows both together: *"Active · Development — 4 days left"* or *"On Hold · Development"* or *"Launched."* The BRD's "Delivered" and "Trial" statuses stop being statuses — Delivered is the Deployment block completing, Trial is a block. The BRD gets amended to this five-state lifecycle.

### 4.3 Blocks with effort by role, duration derived **[resolves B3, B4]**

A block keeps everything you validated — name, description, phase, weight, sub-steps, owner, hold reason, split/merge — and gains one thing: **effort by role** instead of a single day-count.

```
Development block
  effort:  Developer 30 man-days · Tech Lead 5 man-days
  team on project:  2 Developers @ 100% · 1 Tech Lead @ 50%
  → Developer share:  30 ÷ (2 × 1.0 × 0.8 availability) = 18.75 → 19 working days
  → Tech Lead share:   5 ÷ (1 × 0.5 × 0.8)              = 12.5  → 13 working days
  → block duration = 19 working days (the longer of the two)
```

Duration is *derived* (BRD RULE-02), and — as the BRD already allows — a PM can override it with a justification, which writes an `override_applied` event.

Two consequences that make the capacity engine real:
- **Adding a second developer now shortens the block**, as it should.
- **"Can we take this new project, and when?"** becomes computable: given its role effort and who's free, the earliest realistic start date falls out (BRD FR-ASG-12, OBJ-07 — currently impossible).

**Externally-owned blocks are the exception.** Security Testing done by the Security Department, UAT done by the business, Deployment by the database team — GD of AI has no role effort in these. Their duration is simply *entered* as expected working days (lead time), not derived. Which is which is decided by the block's owner (§4.7).

**Assignments stay at project level** — you still add people to the *project*, not to individual blocks, exactly as designed. Each assignment gains an **allocation %** (default 100% for Developers, editable), which is what the general over-allocation rule (BRD RULE-07) needs and what was missing. The Developer date-overlap warning you validated stays as-is on top of it.

### 4.4 The ledger — events **[built out from Timeline §3.3]**

An append-only list per project. Current delivery date is always *baseline date + the sum of every event's day-impact*. Nothing else can move a date. The full event catalogue:

| Event | Day impact | Cause / who | Evidence attached |
|---|---|---|---|
| `baseline_set` | — (sets the reference) | approving authority | **Baseline approval memo** |
| `rebaselined` | archives prior variance | PM, with justification | approval memo |
| `cr_approved` | + effort ÷ capacity, in working days | requesting department, requester | **Approval proof (PDF / email)** |
| `cr_rejected` / `cr_withdrawn` | 0 — recorded so declined scope is visible | requesting department | optional |
| `hold_started` | pending until resumed | reason (RULE-12); displacing project if applicable | **Authorization for the pull** (HQ order / memo) |
| `hold_resumed` | + held working days | — | — |
| `override_applied` | ± delta vs. calculated | PM justification | — |
| `resource_swapped` | 0 | note | optional transfer memo |
| `block_completed` | 0 | who confirmed | **Sign-off for gated blocks**: UAT acceptance, security test report, deployment/change-control approval |
| `lifecycle_changed` | 0 | user | as relevant (e.g. cancellation memo) |
| `lead_pm_changed` | 0 | Director | — |

There is also a **global ledger** for things outside a single project — Resource Pool changes (person added, role changed, availability changed, deactivated), absences, and template/calendar changes — so the department's record is complete, not just each project's.

### 4.5 Evidence — attachments where they matter

The BRD excluded "document management." That exclusion stands: PRMS is not a file repository. But you were right that proof needs to live in the tool, and the audit agreed. The distinction:

**Evidence** is a file attached to *a specific decision or artifact*, so that when someone later asks "did we agree to this?", the answer is one click from the place they're asking. It lives in exactly two kinds of places:

1. **On events** (§4.4): CR approval proof, hold authorization, baseline approval, UAT sign-off, security report, deployment approval, cancellation memo.
2. **On a project's fixed slots**: Requirements document, Analysis/scope document, Value study. A handful of named slots on the project — not a folder.

That's it. No general upload area, no folders, no search across documents. Every file is attached to *the thing it proves*, and the Replay (§7.7) shows it at the moment it happened. The BRD's §8.2 line gets amended to say "document management remains out of scope; *evidence attached to decisions is in scope*."

### 4.6 Progress = verified work, not elapsed time

A block's completion is not "days elapsed ÷ days planned." It is the weighted sum of its **sub-steps confirmed done**. Sub-steps keep `{ name, weight }` and gain an optional **assignee** (a team member) so they can appear in that person's queue.

The workflow, unchanged from what you specified: a resource **reports** a sub-step done from their own account → the PM **confirms** (or returns it with a note) → only then it counts. A block with no sub-steps is a single unit the PM marks complete. Externally-owned blocks are marked complete by the PM with the sign-off evidence attached (§4.4, `block_completed`).

**Two percentages, shown side by side, everywhere:**
- **Planned %** — where the *calendar* says the project should be today (time-based, from the PoR).
- **Verified %** — where the *confirmed work* says it actually is.

The gap between them is the earliest, most honest health signal the tool can give: *"Planned 60% · Verified 40%"* means trouble before any date has visibly slipped. This also cleanly retires the sandbox's manual "now" marker for real projects — on the PoR, "now" is simply today. The manual marker remains inside Scenarios, where it belongs.

**[choice]** A sub-step that is reported but not yet confirmed counts as **0% verified**, but is shown as a separate *"reported, awaiting confirmation"* number so the PM sees the confirmation backlog. Alternative: count it at 50%. I recommend 0% — it keeps "verified" meaning verified.

### 4.7 Owner, slightly firmed up **[alters Timeline §4.9]**

Owner stays optional and stays flexible, with one small structural addition because it now determines how duration is computed (§4.3): an owner is either **GD of AI team** (duration derived from role effort and the project team) or **External party** (a free-text name — "Security Department," "Database Department," "Business Owner" — and duration entered as lead time). Hold keeps reusing its displacing-project link as its owner, as decided. No "Designer" role is added.

### 4.8 Departments — capture the unit, roll up the display **[resolves B7]**

A project's owning department is picked from the department **tree** at whatever level is real (e-Crime, under CID). The Portfolio rolls it up to the General Department, as you decided. The project page shows the full path. Both the BRD (capture the level) and your list decision (display the roll-up) hold.

---

## 5. Account types and permissions

Eight kinds of people touch this system. Five log in. Two use links. One is future.

| Account type | Who | Logs in? |
|---|---|---|
| **Director** | Head of GD of AI | Yes |
| **Lead TPM** | Owns projects | Yes |
| **Junior TPM** | Works on a lead's projects | Yes |
| **Resource Coordinator** | Maintains the Resource Pool | Yes |
| **Resource** | Developer, BA, Tech Lead | Yes — minimal |
| **Business stakeholder** | Business owner, business-side PM | No — secure link |
| **HQ stakeholder** | Police HQ leadership | No — secure link |
| **External block owner** | Security Dept, DB Dept contact | *Future* — optional |

| Capability | Director | Lead TPM | Junior TPM | Coordinator | Resource | Stakeholder (link) |
|---|---|---|---|---|---|---|
| See all projects & portfolio | ✓ | ✓ | ✓ | ✓ (read) | own projects only | own projects (business) / portfolio (HQ) |
| Create project | ✓ | ✓ | — | — | — | — |
| Edit draft plan | — **[choice]** | own | assigned | — | — | — |
| Set / re-set baseline | — | own | — | — | — | — |
| Raise a change request | — | own | assigned | — | BA: draft only | — |
| Preview CR impact | ✓ | ✓ | ✓ | — | — | ✓ (when "presented") |
| Record CR decision + proof | — | own | — | — | — | — |
| Start / resume a hold + proof | — | own | propose only | — | — | — |
| Override a calculated date | — | own | — | — | — | — |
| Assign / swap team | — | own | — | — | — | — |
| Confirm sub-steps | — | own | assigned | — | — | — |
| Report own sub-step done | — | — | — | — | ✓ | — |
| Create & save scenarios | ✓ | ✓ | ✓ | — | — | — |
| Commit a single-project scenario | — | own | — | — | — | — |
| Commit a cross-project scenario | ✓ | — **[choice]** | — | — | — | — |
| Change a project's Lead TPM | ✓ | — | — | — | — | — |
| Manage Resource Pool, absences | — | — | — | ✓ | — | — |
| Admin: calendar, departments, templates, users | ✓ (or Admin) | — | — | — | — | — |
| Reports | ✓ | ✓ | ✓ | ✓ | — | HQ: portfolio report |

Three deliberate calls in that table:

- **Junior TPM can do the daily work but not the promises.** Baselining, recording an approval, starting a hold, overriding a date — these change what's been committed to someone outside the department. Those stay with the Lead. **[choice]** — the alternative is Junior can do all of it and the Lead is notified.
- **Cross-project commits are Director-level.** A surprise HQ project that pulls people from three other PMs' projects is exactly the decision the Director executes in reality. Making the Director the one who commits it keeps the record honest about who decided. **[choice]** — the alternative is "every affected Lead acknowledges."
- **Director is read-everything, decide-strategically, and does not edit plans.** Keeps the roles clean. **[choice]**

Prototype note: all of this is designed now but enforced only in the pilot phase; the local prototype is single-user (BRD NFR-P-02).

---

## 6. The ecosystem at a glance

```
┌──────────────────────────────── PRMS ────────────────────────────────┐
│                                                                       │
│   INBOX          PORTFOLIO            RESOURCE POOL        REPORTS    │
│   (per role)     (landing for PMs)    (people, capacity)   ADMIN      │
│                        │                                              │
│                        ▼                                              │
│              PROJECT WORKSPACE ──── Plan Editor (draft / scenario)    │
│              overview · plan · team · CRs · holds · evidence · history│
│                        │                                              │
│              SCENARIO STUDIO  (cross-project what-if, for meetings)   │
│                        │                                              │
│   MY WORK ◄────────────┼──────────────► REPLAY (project & portfolio) │
│   (resources)          │                                              │
│                        ▼                                              │
│              THE LEDGER — one append-only record per project          │
│                        │                                              │
│                        ▼                                              │
│          STAKEHOLDER PORTAL  (business owner view · HQ view)          │
│          read-only · secure link · replay · evidence · pending CRs    │
└───────────────────────────────────────────────────────────────────────┘
```

Everything above the ledger *writes* to it (through events). Everything below it *reads* from it. Nothing bypasses it.

---

## 7. Pages and modules

Each: who uses it, what it shows, what you can do. Validated mockup patterns are kept and named.

### 7.1 Inbox (home, per role)

The first thing anyone sees after logging in. Not a dashboard — a to-do list generated by the system.

- **Lead / Junior TPM:** sub-steps awaiting your confirmation; CRs raised but with no decision recorded; holds past their expected resume date; projects stale 14+ days; over-allocation warnings acknowledged-pending; scenarios saved but not resolved.
- **Resource:** sub-steps assigned to you, by project; anything returned to you with a note.
- **Coordinator:** people with no availability set; absences ending this week.
- **Director:** cross-project scenarios awaiting commit; projects whose verified % lags planned % by more than a threshold; holds caused by HQ directives this month.

No push notifications (BRD excludes them). This is an in-app queue only.

### 7.2 Portfolio (landing page for PMs and Director)

Everything you validated stays: block-based status with days-left in the current block, sortable columns, status filter chips, "My Projects" toggle, lead-PM avatar, stale flag, summary stats. Three additions:

- **A variance column** — *"+12d vs baseline"* — the single most important number in the system, currently absent.
- **Health** — a neutral three-state indicator derived from the planned-vs-verified gap and open holds. Never red-for-blame; it is "attention needed," and its tooltip says why in one line.
- **A second stats row, external causes** — *days added by approved CRs (30d) · days lost to reprioritisation · CRs pending decision · top requesting department.* Same weight, same styling as the internal stats (stale, over-allocated). This is the audit's #5, and it's what makes the page balanced instead of self-incriminating.

### 7.3 Project Workspace

The "compact preview + button" pattern you chose, extended. An **Overview** page with cards, each card drilling into its own full view:

- **Header:** name, reference code (auto-generated, e.g. PRJ-2026-014 — costs you nothing, resolves B6), lifecycle state + current phase, priority, department path, business PM, lead TPM + juniors.
- **Stats:** delivery date, variance vs baseline, days left, **Planned % · Verified %**, team size.
- **Status note:** a one-paragraph narrative the PM keeps current — what stakeholders read first (BRD FR-PRJ-10).
- **Plan card → Plan Editor.** Compact bar preview; "Edit plan" opens the editor on the draft (if not baselined) or **opens a new scenario** (if baselined — you never edit a PoR directly). A "Baseline this plan" action lives here, requiring the approval evidence.
- **Team & Resources card → Team view.** Add from the Resource Pool by role, with allocation %, date-aware conflict warning as validated, **Replace** (swap) as designed.
- **Change Requests card → CR list.** Each CR with its state (Raised → Assessing → Presented → Approved / Rejected / Withdrawn), its previewed impact, and — for approved ones — the proof. "Present to business" makes it visible on the stakeholder portal with its impact, so the business owner sees the cost *before* deciding. Recording the decision attaches the proof and writes the event.
- **Holds card → Holds list.** Open and past holds, reason, displacing project, expected/actual resume, authorization evidence. "Start a hold" previews the provisional new date before committing. If this project *caused* holds elsewhere, this card also shows *"displaced 3 projects, 15 days total"* (BRD FR-HLD-09).
- **Evidence card.** The project's fixed slots (Requirements, Analysis/scope, Value study) plus every event-attached file, listed chronologically.
- **History card → Ledger view.** The full event list, newest first. This *is* Recent Activity, made complete.

**[choice]** Overview-with-drill-ins rather than tabs — consistent with what you chose when the page had four cards; it now has eight, and I still think drill-in reads better for a status-check page. Easy to flip.

### 7.4 Plan Editor (single-project)

The sandbox exactly as validated — template picker, drag-and-drop with the daily ruler, split/merge, holds by date, owner, sub-steps, weight — with three changes:

1. **It always operates on a Draft or a Scenario, never on a PoR.** The title says which. If you opened it from a baselined project, you are in "Scenario: untitled" and can name, save, discard, or commit.
2. **Blocks carry effort by role** (§4.3); the block form gains role-effort fields for GD-of-AI-owned blocks, or a lead-time field for external ones. Derived duration is shown, overridable.
3. **Commit** (on a scenario) shows the diff — *"+ CR 'SMS notifications' +5d · new hold 5–10 Oct · Development resized 18→22d (override)"* — and walks you through the cause and evidence for each line before writing the events.

The manual "now" marker stays, for scenario work. **Project templates** (§7.10) replace the "load ready project" button: a new draft is pre-filled from the department's standard template (your Analysis 5 / Dev 15 / Security 5 / UAT 5 / Deploy 5).

### 7.5 My Work (resources)

One page. Left: my assignments as a small personal swimlane (which projects, which dates, allocation). Right: my sub-steps, grouped by project, each with **Report done**. Returned items show the PM's note. Read-only access to the plan of each project I'm on. Nothing else — no portfolio, no other people's data, no stakeholder pages.

BA and Tech Lead accounts get one extra each: BAs can **draft** a change request on their projects (the Lead raises it formally); Tech Leads can **propose** role-effort estimates on a draft plan.

### 7.6 Resource Pool & Capacity

The swimlane you validated (people as rows, project bars on a shared calendar, red overlap marker for Developers), plus add/edit person, search, role filters, no hard deletion — all kept. Additions:

- **Absences** (leave, training) as date ranges that reduce capacity (BRD FR-RES-04) — shown as grey gaps on the person's lane.
- **Secondary role** per person (BRD FR-RES-05).
- **Utilisation** per person and per role, computed from allocation % × availability — now possible.
- **A capacity summary strip** — per role: headcount, utilisation, earliest date someone is free.
- **Person profile** — their history of assignments including ended ones (FR-ASG-13); this is where "who worked on what" for someone who has left lives.

### 7.7 Replay — the project movie

Exactly as specified in the Stakeholder View doc, now placed where it belongs: available to PMs in the Project Workspace *and* on the stakeholder portal.

- **Scrub day by day**: the bar redraws as of that day, with the projected delivery date *as it was then*.
- **Play Highlights**: auto-advance stop-by-stop through the ledger — each CR, hold, resume, override, baseline — with a callout: what, who, day-impact, and the evidence link.
- **Compare**: a ghost bar of the baseline behind the current bar, so drift is visible at every frame.
- **Portfolio Replay** **[adds to Stakeholder View §5.3]**: the same engine over several projects side by side — the domino *as it unfolded*. This is how you show HQ "your three surprise projects this quarter cost these five projects 40 days." Project selection is deliberate (choose which to include), as decided.

### 7.8 Scenario Studio — cross-project what-if, for meetings

The "grand scale" sandbox. PM-side only, projected in the room; stakeholders watch, never touch (as decided).

- Start from the live portfolio, or a saved scenario.
- **Add a hypothetical project** — name, role effort, priority, requested start. The system shows the earliest realistic start given current commitments (FR-ASG-12), and what it would take to start sooner.
- **Pull resources** from existing projects into it — the affected projects' bars visibly extend, each with the hold that would be created and its cost in days.
- **Compare** scenarios side by side ("pull from Project A" vs "pull from Project B").
- **Present mode** — large type, no editing chrome, meeting-room legible (BRD NFR-06).
- **Commit** (Director) — writes `hold_started` events with displacing-project links to every affected project, the new project's creation, and requires the authorization evidence once, attached to all of them.

This is the second mission bullet, delivered to the person who makes the decision, before they make it.

### 7.9 Stakeholder Portal (secure link, read-only, no login)

Two views, both pure renderings of the ledger:

**Business owner view (one project):** status note; lifecycle + phase; promised date vs current date with the variance *explained line by line*; **pending change requests with their previewed impact** ("if approved: +5 days → 8 Dec") — the mission's first bullet, in front of the person who decides; the Replay; every evidence file that concerns them (their CR approvals, their UAT sign-off); *last updated* stamp.

**HQ view (portfolio):** the Portfolio's summary stats **including the external-cause row**; the list with variance and health; holds this period and what caused them; **Portfolio Replay** for a selected set of projects; a **"days lost to reprioritisation, by directive"** table — neutral, factual, and the first time anyone will have seen that number.

Neutral tone (FR-VIZ-11) is enforced at the authoring end: the free-text fields on CR and hold forms use a fixed sentence frame (*"[Requester] requested [what]. Impact: [N] working days."*) with the free text inside it — a small guard the audit found missing.

**Export**: any portal page → PDF snapshot; Highlights → a one-page-per-stop storyboard PDF, for rooms where a live screen can't be shown.

Prototype: open links. Pilot/production: controlled access per BRD NFR-P-03.

### 7.10 Reports and Admin

**Reports** (Director, PMs, Coordinator): portfolio totals; variance by project; days added by CRs per department; days lost to holds by reason and by directive; utilisation by role; load per Lead TPM; declined scope (rejected CRs and their cost); estimate confidence overview. All exportable.

**Admin**: working calendar & public holidays (RULE-04 — currently missing, every date is wrong without it); department tree; roles and users; **project templates** (standard block set, editable); the "standalone / part of another system" field graduates later into a **Systems registry** so projects can link to the system they live in — deferred until you provide the list.

---

## 8. How it works — seven walkthroughs

**8.1 A project is born and baselined.** Lead TPM creates it (name, description, department from the tree, priority, business PM, origin, value justification, standalone/embedded note). Reference code is assigned. A Draft plan is pre-filled from the template; the Lead adjusts blocks, enters role effort, assigns the team with allocation %, attaches the Requirements and Analysis documents to their slots. The team's Tech Lead proposes estimates from My Work. When the business and HQ approve the plan, the Lead clicks **Baseline**, attaches the approval memo, and `baseline_set` is written. The project is now Active. The stakeholder link is shared.

**8.2 The business asks for something new.** The business PM emails "can we also send SMS notifications?" The Lead raises a CR (or the BA drafts it): requester, department, Developer 5 man-days. The system previews: +5 working days → 8 Dec. The Lead marks it **Presented** — it appears on the business owner's portal page with that exact line. In the meeting they say yes; the Lead records **Approved**, attaches the email, and `cr_approved` writes +5d. The Portfolio variance column ticks to +5d; the Replay gains a stop.

**8.3 HQ drops a surprise project.** The Director opens **Scenario Studio**, adds "Anti-Fraud Intake — Critical — Developer 40 man-days, BA 10, start 1 Oct." The earliest start with nobody pulled is 15 Nov. The Director drags two Developers from e-Crime Portal into it: e-Crime's bar extends by 12 working days with a hold block; the new project starts 1 Oct. Presented to HQ with both bars on screen. HQ confirms. The Director **commits**: `hold_started` on e-Crime (reason: resources pulled, displacing project: Anti-Fraud Intake, evidence: the HQ order), the new project is created and baselined, the swap events are written. e-Crime's business owner's portal now shows the hold, the cause, and the memo.

**8.4 Work gets done and confirmed.** Yousef (Developer) opens My Work, sees "API integration — e-Crime Portal — 30% of Development," finishes it, clicks **Report done**. The Lead's Inbox gets it; the Lead confirms. Verified % moves. Planned % had moved days ago; the gap closes. When the last Development sub-step is confirmed, the block completes and the current phase becomes Testing.

**8.5 UAT and deployment, externally owned.** The UAT block's owner is "Business Owner," duration entered as 7 working days. When the business signs off, the Lead marks the block complete and attaches the acceptance form — `block_completed` with evidence. Same for Security (report) and Deployment (change-control approval). The Replay shows each artifact at its stop. Launched is set when Trial completes.

**8.6 Someone leaves mid-project.** The Lead opens Team, clicks **Replace** on Yousef, picks Lina from the pool (date-aware conflict check runs), optionally attaches the transfer note. `resource_swapped` (0 days) is written; Lina's sub-steps appear in her My Work; Yousef's profile keeps the history.

**8.7 The Director reviews the quarter.** HQ portfolio view: variance across the board, *"days lost to reprioritisation: 62, from 4 directives"*, *"days added by CRs: 41, top: CID"*. Portfolio Replay over the six affected projects, Highlights mode, projected in the room. No deck was built.

---

## 9. What changes versus the current drafts

| Area | Kept as validated | Altered | Added |
|---|---|---|---|
| Timeline sandbox | template picker, drag/drop, split/merge, holds by date, owner, sub-steps, weight, daily ruler | now explicitly a Draft/Scenario editor, never the record; blocks get effort-by-role; "load ready project" → templates; owner gets internal/external toggle | commit-with-diff; scenario save/compare |
| Portfolio | columns, block status, sort, filters, My Projects, stale flag, stats | — | variance column; health; external-cause stats row |
| Project Detail | compact preview + button pattern; team card; swap | Overview with eight cards, not four | reference code; status note; CRs, Holds, Evidence, History cards; Planned/Verified % |
| Resource Pool | swimlane, add/edit, search, filters, no deletion | — | absences; secondary role; utilisation; capacity strip; person profile |
| New Project | all decided fields | department picked from tree (unit captured, GD displayed) | auto reference code; optional requested start |
| Change requests | 3-step flow with external approval + proof | — | CR states incl. "Presented" visible to stakeholder; rejected CRs recorded |
| Event log | 6 types | — | full catalogue incl. `block_completed`, `rebaselined`, `cr_rejected`, `lifecycle_changed`; global ledger |
| Stakeholder view | replay (scrub + highlights), domino view, read-only link, PM drives sandbox in meetings | — | pending CRs with impact; evidence links; HQ cause tables; export; portfolio replay |
| Roles | 5 roles incl. Junior TPM; no hard deletion | — | account types & full permission matrix; Resource accounts; My Work |
| Not previously anywhere | — | — | Inbox; Scenario Studio; Reports; Admin (calendar, holidays, templates, department tree) |

## 10. Proposed resolutions for B1–B8

| # | Proposal |
|---|---|
| B1 | Lifecycle (Draft/Active/On Hold/Launched/Cancelled) **and** current phase (from blocks). BRD RULE-01/FR-PRJ-02 amended to the five-state lifecycle. |
| B2 | Phase order: Requirements → Analysis → Development → Testing → Security → UAT → Deployment → Trial. BRD amended; "Delivered"/"Trial" become phases, not statuses. |
| B3 | Blocks carry effort by role; duration derived per RULE-02 per block, summed across blocks. External blocks enter lead time. BRD RULE-09 kept (assign to project, not block). |
| B4 | Assignments carry allocation %. RULE-07 computed. |
| B5 | Junior TPM added to BRD FR-RES-02 and RULE-08 (multi-project like TPM). Permissions per §5. |
| B6 | Reference code auto-generated (BRD satisfied, zero user effort). Target start date kept as *optional* "requested start." |
| B7 | Capture department at its real level from a tree; display rolled up. Both BRD and list decision hold. |
| B8 | Resources get minimal accounts (My Work). BRD §7.1 amended. |

## 11. Phasing

The BRD's four phases stand; what changes is the *order inside Phase 1*, because the audit showed the accounting spine must come before more screens.

- **Phase 1a — the record.** Ledger + Plan of Record + baseline + templates + working calendar. Plan Editor on Drafts. Project Workspace (Overview, Plan, Team, Evidence, History). Portfolio with variance.
- **Phase 1b — decisions with receipts.** Change requests (all states, preview, proof) + holds (preview, authorization) + overrides + swaps. Stakeholder portal, business-owner view. Replay — Highlights first, scrub second.
- **Phase 1c — the meeting tools.** Scenarios on single projects → Scenario Studio cross-project → Portfolio Replay → HQ view with cause tables → export.
- **Phase 1d — the people.** Resource Pool full (absences, utilisation, capacity strip). Resource accounts + My Work + verification workflow + Planned/Verified %. Inbox.
- **Phase 2/3/4** as the BRD: pilot with real data and access control; production hardening; refinements.

Everything in 1a–1c can be demonstrated to the Director with seeded data on one machine.

## 12. Decisions for you to confirm or overrule

1. Scenarios as the sandbox model (fork → commit/discard) — the central structural change.
2. Five-state lifecycle + phase from blocks, replacing the BRD's six statuses.
3. Effort by role on blocks; external blocks by lead time; assignment stays project-level.
4. Reported-but-unconfirmed sub-steps count 0% (shown separately) vs 50%.
5. Junior TPM cannot baseline / record approvals / start holds / override / swap — Lead only.
6. Cross-project scenario commits are Director-only.
7. Director does not edit plans.
8. Project Workspace as Overview + drill-in cards, not tabs.
9. Evidence: fixed project slots + event attachments only; BRD §8.2 amended accordingly.
10. Neutral-tone sentence frame on CR/hold free text.
11. Reference code auto-generated; requested start date optional.
12. Phase 1 reordered as §11.

## 13. Still deliberately out

Timesheets and hours; budgets; general document management; push notifications; task/sprint tracking; inter-project *technical* dependencies (shared-resource cascades are covered by scenarios; "B needs A's API" is not); vendor capacity; Arabic/RTL and SSO until production; mobile; the Systems registry until you provide the systems.
