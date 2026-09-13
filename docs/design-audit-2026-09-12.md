# Design Audit — PRMS
## Do the drafts actually solve the problem?

**Date:** 12 September 2026
**Method:** Re-derived the required solution from the mission (`README.md`) and the BRD's problem statement, objectives and KPIs — independently of what has been designed — then compared that against the two design drafts.
**Verdict:** The drafts solve roughly half the problem well, leave the other half undesigned, and contain two decisions that work *against* the stated mission.

---

## 1. What the problem actually demands

The mission names three mechanisms:

1. *"a new requirement is answered with the man-days it adds and the date it moves, **before it is accepted**"*
2. *"halting a project for higher-priority work shows what it does to the project being halted, **before the decision is taken**"*
3. *"every delivery date that moves has a **recorded cause** attached to it"*

Working only from the problem statement, a system that delivers those three things needs seven parts, in this dependency order:

| # | Required capability | Why the problem demands it | Designed? |
|---|---|---|---|
| 1 | **A baseline, frozen at approval** | Everything else is a comparison *against* something. Without it there is nothing to measure variance from. | ✗ **No** |
| 2 | **A change-request pipeline** with costing and a "presented to the business owner" gate | This is the direct answer to §4.1 — the cost of a request being invisible at the moment it's made | ✗ **No** |
| 3 | **A hold register** with reasons and displacement links | The direct answer to §4.2 — reprioritisation cost invisible to the decision-maker | ✓ **Yes, well** |
| 4 | **A variance ledger** reconciling baseline → current through attributed events | This *is* RULE-11 / OBJ-05 — the BRD calls it the mechanism that makes a date structurally unable to move without a cause | ✗ **No** |
| 5 | **A capacity model** — effort per role, allocation % per assignment | The answer to §4.3 — workload invisible, no forward view of who's free | ◐ **Partly, and diverged** |
| 6 | **A live stakeholder view** rendering all of the above | The answer to §4.2's reporting burden; the thing KPI-01 measures | ✗ **No** |
| 7 | **A PM working UI** to maintain it all | Necessary, but supporting — not itself a mission outcome | ✓ **Yes, thoroughly** |

**We built #7 and #3. We skipped #1, #2, #4 and #6.** Item #7 is the one part of the list that isn't itself a mission outcome.

---

## 2. The two things I think we're doing *wrong*

These aren't omissions to be filled in later. They're choices that conflict with the stated problem.

### 2.1 The sandbox reproduces the exact deficiency the BRD says is the core problem

The BRD's own assessment of Excel (§5):

> *"Excel holds a single mutable view of the present, so each revision destroys the evidence needed to account for it. **This is the specific deficiency PRMS must address**; simply moving the same data into a better tracker would not resolve it."*

Now describe the timeline sandbox honestly: a single mutable sequence of blocks. Drag a block and the dates change. Edit a block's days and the old value is gone. Delete a block and it leaves no trace. Auto-merge silently sums two blocks' days and discards the `groupId` that recorded they were ever separate. "Load ready project" overwrites state outright.

**It is a single mutable view of the present that destroys the evidence needed to account for revisions.** It is a better tracker, and the BRD explicitly says a better tracker doesn't resolve this.

The sandbox is excellent as a *planning and simulation* tool — that's genuinely valuable and worth keeping. The error is that it's currently standing in for the system of record, and it isn't built to be one. Nothing in either draft defines what gets durably recorded when a PM changes the plan.

### 2.2 The tool surfaces GD of AI's failings and hides the external causes

Look at what the drafts make visually prominent:

- Portfolio List: a **stale-data flag** on projects the PM hasn't updated (⏱ 18d stale) — a judgement on PM discipline
- Resource Pool: a red **⚠ overlap** badge against a named developer — a judgement on GD of AI resourcing

Now look at what the drafts make prominent about *external* causes: nothing. There is no count of change requests a department has raised, no total days lost to HQ reprioritisation, no "this department has added 40 days of scope since baseline" anywhere in any view. FR-RPT-04 and FR-RPT-05 (change requests per department; days lost per hold reason) have no screen at all.

The mission says: *"I absorb the difference, and I have no instrument for showing it."* The drafts have produced an instrument that shows the internal cost of absorbing it, and not the external causes creating it. That's backwards, and it also runs against RISK-03's mitigation, which requires the *forward-looking, neutral, cause-attributing* framing to be the prominent one.

---

## 3. Gaps, ranked by how much they cost

### 3.1 Change requests — the mission's first bullet — are entirely undesigned (**critical**)

Ten functional requirements (FR-CHG-01…10), one business rule (RULE-14), one objective (OBJ-02), two KPIs (KPI-04 at 100%, KPI-05 at 100%). Design coverage: **zero screens.** The only appearance of a change request anywhere is as a line of past-tense text in the Project Detail "Recent Activity" feed.

Missing entirely: raising a CR, estimating its effort, calculating its date impact, the "view impact without committing it" preview (FR-CHG-10 — *the* mission mechanic), the six-state workflow, the record of when it was presented to the business owner and what they decided.

**And worse than missing:** the sandbox provides an alternative route that bypasses it. A PM can drag a 10-day block onto the bar and the delivery date moves — with no requester, no cost presentation, no approval, no record that this was new scope rather than baseline scope. We have designed a mechanism that lets scope creep in *unattributed*, which is the precise behaviour §4.1 exists to stop.

### 3.2 No baseline, therefore no variance, therefore no attribution (**critical**)

BR-04, FR-PRJ-05, FR-PRJ-07, FR-EST-03, RULE-13, FR-VIZ-03 all depend on a frozen baseline. None of it is designed. Consequences that follow automatically:

- RULE-11 (`baseline + Σ changes + Σ holds = current`) is unenforceable — there's no baseline term and no event terms
- OBJ-05 and KPI-03 ("100% of variance attributable") are not just unmet, they're unmeasurable
- FR-VIZ-03's baseline-bar-vs-current-bar comparison has no data to draw
- The Portfolio List shows delivery date and days-left but **no variance column** — so the single most important number in the whole system (how far has this moved, and why) appears nowhere

### 3.3 The stakeholder view — the deliverable the mission is actually for — is out of scope and never picked up (**critical**)

Twelve requirements (FR-VIZ-01…12), OBJ-06, KPI-01 (*"zero prep time"* — the headline metric), KPI-08. The PM Tool draft lists it as out of scope in §2 and nothing else covers it.

This is worth stating plainly: the mission's audience is the **business owner and HQ** — they are the people making decisions without seeing cost. The PM already knows the cost. We have designed, in detail, the instrument for the person who needs the least convincing, and have not designed the view for the two audiences the mission names.

### 3.4 The effort model has quietly diverged from the BRD (**high**)

The BRD locks in **man-days per role** (FR-EST-01: 60 dev-days + 15 BA-days + 10 TPM-days), and RULE-02 derives duration from `effort ÷ assigned capacity`.

The timeline design uses a block with **one undifferentiated `days` number and no role dimension at all** (`owner` is explicitly free text, not a resource link). These are not compatible models. What breaks:

- RULE-02 / FR-SCH-01 can't be computed — duration is entered, not derived
- Adding a second developer to a project changes nothing, because block days are fixed
- FR-ASG-12 (earliest realistic start date from capacity) is uncomputable → **OBJ-07 unmet**
- FR-ASG-08 utilisation has no effort input → the Resource Pool's swimlane bars are drawn from project *date ranges*, not from actual load, so "utilisation" isn't really being measured

The timeline draft flags this as open question §7.2, but understates it as "undecided." It isn't a loose end — one of the two models has to change.

### 3.5 Effort estimation has no home in the UI (**high**)

Six requirements (FR-EST-01…06). Where does a PM enter role effort? New Project doesn't ask. Project Detail doesn't show it. The sandbox asks for block days, which is a different quantity. There is no screen for the estimate that FR-EST-03 says must be baselined and FR-EST-04 says accumulates approved change-request effort.

### 3.6 Allocation % was designed away (**high**)

FR-ASG-01 gives assignments an allocation percentage; RULE-07 warns when a person's total exceeds 100% of availability; FR-ASG-08 computes utilisation from it. Our Project Detail design is "pick a person from a role dropdown" — no percentage. So RULE-07 can't be evaluated at all.

I previously raised this to you as "the over-allocation check only covers Developers." That framing was too soft. It's not that we chose to implement the narrow Developer rule and skip the general one — it's that **we removed the field the general rule needs.**

### 3.7 Project status vs. block list left unreconciled (**medium**)

FR-PRJ-02's status set (Analysis, In Progress, On Hold, Delivered, Trial, Launched) and RULE-01's transition table were effectively replaced by "current block" from the 9-phase list. Fallout never written back: "In Progress" and "Delivered" no longer exist; "Launched" appears in the Portfolio List but isn't in the phase enum; FR-PRJ-06 (record every status transition) is undesigned; RULE-01's transition rules are now meaningless.

### 3.8 Load-bearing items parked as "later" (**medium**)

- **Persistence** — nothing saves. KPI-02 (100% of the portfolio recorded in PRMS) is unachievable by construction.
- **FR-VIZ-12 export** — undesigned. Needed precisely when a live view can't be shown, which in an HQ meeting room is often.
- **Junior TPM** — introduced in the PM Tool draft, still not written back into the BRD's FR-RES-02/RULE-08. Flagged twice, actioned zero times.
- **Public holidays** (RULE-04) — deferred; every calculated date is wrong by however many holidays fall inside it.

---

## 4. What the drafts got right

Worth being precise about, because it's substantial and shouldn't get lost in the above:

- **Hold mechanics are genuinely thorough.** Reason taxonomy with resource disposition, mandatory displacing-project link, auto-recalculation on resume, provisional date during an open hold, multiple sequential holds retained individually. FR-HLD-01/02/03/05/07 and RULE-12 are well covered. **OBJ-03 is properly served** — the mission's second bullet is the one that works.
- **"A pause consumes time but not effort"** was a real insight, correctly separated into three distinct numbers (working days / paused days / calendar span). This is the kind of distinction that makes the hold cost defensible in a meeting.
- **Working-day date math** — solid, and correctly kept accurate even when weekends aren't drawn.
- **Mid-project resource swap** — correctly grounded in the BRD's existing Assignment start/end dates (FR-ASG-13) rather than inventing a parallel mechanism.
- **No hard deletion** — correct, matches FR-RES-06, and the reasoning (people who left must stay in the record) is right.
- **Resource Pool swimlane** — a good, legible answer to FR-ASG-09.
- **Split / auto-merge** — elegant, and the right call to make it general rather than hold-specific.

---

## 5. What I'd recommend

The pattern in the gaps is consistent: **we designed the mechanics of *changing a plan* and not the mechanics of *accounting for the change*.** Every critical gap (baseline, change requests, variance ledger, stakeholder view) is on the accounting side. The mission is almost entirely about the accounting side.

Suggested reordering of remaining design work:

1. **Define the record before more UI.** What is durably stored when a plan changes — the event log, with each event carrying its day-impact and cause. This is RULE-11 made real, and it's the spine everything else hangs from. Until it exists, more screens just add more ways to mutate state silently.
2. **Design the change-request flow**, including the "show the impact before it's committed" moment. This is the mission's first bullet and currently has nothing.
3. **Resolve the effort-model conflict** — either blocks gain a role/effort dimension, or the BRD's man-day model is formally amended. Both are legitimate; leaving them contradictory is not.
4. **Design the stakeholder view.** It's the only deliverable that moves KPI-01, and it's the one the Director will judge this by.
5. Rebalance what's shown: if the tool flags stale data and over-allocation, it should equally flag change-request load per department and days lost to reprioritisation. Same neutrality, both directions.
6. Then return to the sandbox and decide what it *is* — simulation surface, or system of record. It can be the former cleanly; it can't be the latter as built.

None of this invalidates the work done. The hold half is solid and the PM's working surface is well understood. But the drafts currently describe a good project-planning tool, and the mission asks for an accountability instrument — and those differ most in exactly the places we haven't designed yet.

---

## 6. Resolution Log (updated 12 September 2026)

| # | Recommendation | Status |
|---|---|---|
| 1 | Define the record (event log) before more UI | ✅ **Actioned** — [Timeline design §3.3](superpowers/specs/2026-09-11-timeline-visual-design.md) |
| 2 | Design the change-request flow, impact-before-commit | ✅ **Actioned** — [PM Tool design §4.6](superpowers/specs/2026-09-11-pm-tool-design.md). Refined beyond the original recommendation: approval is recorded, not decided, in-system, with a required proof-of-approval attachment (PDF/email) — this was the user's own correction, not something the audit anticipated. |
| 3 | Resolve the effort-model conflict (blocks vs. man-days-per-role) | ⏳ **Still open.** A resolution direction was sketched visually (block gains an `effort` map by role, duration derived per RULE-02) but not yet written into the timeline design's data model. |
| 4 | Design the stakeholder view | ✅ **Actioned** — new document, [Stakeholder View design](superpowers/specs/2026-09-12-stakeholder-view-design.md). Expanded well beyond the original recommendation: a project-scale replay (scrub *and* highlight-playback modes) and a portfolio-scale multi-project domino view, both driven from the Event Log. |
| 5 | Rebalance what's flagged (internal vs. external cause) | ⏳ **Still open.** Shown only as a mockup comparison; the actual "CRs raised per department" / "days lost to reprioritisation" stats have not been added to the Portfolio List design. |
| 6 | Decide what the sandbox is — simulation or record | ✅ **Actioned.** Confirmed as simulation-only, PM-side — see [Timeline design §2](superpowers/specs/2026-09-11-timeline-visual-design.md) and [Stakeholder View design §6](superpowers/specs/2026-09-12-stakeholder-view-design.md). It additionally now has a named second purpose: the tool a PM drives live in stakeholder meetings, never exposed under the stakeholder's own access.
