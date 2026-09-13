# Design: PM Tool — Portfolio, Resource Pool & Project Pages

**Status:** Initial draft — validated through conversation and mockups, **not final**. Expect this to change as the timeline sandbox and the real capacity engine get built out and force revisions here.
**Date:** 11 September 2026
**Author:** Technical PM, GD of AI (design captured via collaborative brainstorming session)
**Related:** [BRD — Portfolio & Resource Management System](../../BRD-portfolio-resource-management.md) (FR-PRJ, FR-RES, FR-ASG, RULE-06/07); [Timeline / Visual Summary design](2026-09-11-timeline-visual-design.md)

---

## 1. Purpose

This document specifies the minimum page set for the Technical PM–facing side of PRMS: a portfolio list, a department resource roster, and a per-project management page that ties into the timeline sandbox already designed. It answers the three questions posed to start this session: *what's the minimum feature set, how does it connect to the sandbox, and how are resources and the project team managed.*

## 2. Scope

**In scope:** page inventory, navigation flow, minimum fields per page, and the team-assignment/conflict-warning rule that connects the Resource Pool to the timeline.

**Out of scope:** visual polish (these are structural/functional decisions, not a finished UI design), authentication, and backend/data-layer design. **The read-only stakeholder visualization page (BRD §7.2) was originally out of scope here too, but now has its own document:** [Stakeholder View design](2026-09-12-stakeholder-view-design.md). Also out of scope: full working mockups of the Resource Pool, Portfolio List, and New Project pages — only the Project Detail page's timeline tie-in was mocked interactively; the rest were settled through direct discussion and should be treated as less rigorously validated than that page.

## 3. Page Inventory & Flow

The **Portfolio List is the landing page** — the hub every session starts from. Three things are reachable from it:

```
                         ┌─→ Resource Pool   (manage the department roster)
                         │
Landing Page (list)  ────┼─→ New Project     (create)
                         │
                         └─→ [click a row] → Project Detail (view/edit)
                                                    │
                                          "Edit Timeline →" opens the sandbox
```

Resource Pool is reached as a standing navigation item, not a gate the user must pass through before anything else — you can create or open a project immediately from the landing page, even with an empty pool (§6, item 1).

## 4. Pages

### 4.1 Landing Page (Portfolio List)

One row per project. Minimum columns, settled by direct selection:

- Project name
- Status
- Priority
- Delivery date
- Days remaining
- Owning department
- Business PM

Team avatars and per-person over-allocation flags were deliberately **left off this list** — that detail belongs on the Project Detail page; the list's job is fast scanning, not team visibility.

**Owning department shows only the General Department**, never a sub-department — e.g. a project actually owned by e-Crime or Anti-Fraud both show simply "CID" in this column. Matches BRD FR-PRJ-04 (roll up to the parent General Department for reporting). The specific sub-department, if needed, belongs on the Project Detail page, not this list.

### 4.2 Resource Pool

> The Junior TPM role introduced below isn't in the BRD's role list — see [Shared Definitions §2](shared-definitions.md#2-roles), pending B5.

A list of GD of AI delivery staff. Each entry:

| Field | Notes |
|---|---|
| Name | |
| Role | Technical Project Manager, Junior TPM, Business Analyst, Tech Lead, or Developer. Junior TPM is new — not yet in the BRD's role list (FR-RES-02); see §6, item 4. |
| Availability % | Matches BRD RES-03 — accounts for meetings/admin eating into capacity |
| Active / inactive toggle | Lets someone who's left or is on long leave be hidden from assignment pickers without deleting their history on past projects (BRD FR-RES-06) |
| Current workload | Shown as a **timeline swimlane**, not a text list (validated via mockup) — see below |

This page is the **single source of truth** the New Project and Project Detail pages pull from — there is no free-text entry of a team member's name anywhere else in the tool.

**No hard deletion, ever.** Deactivation is the only removal path (BRD FR-RES-06) — confirmed explicitly during this design session: someone who has left the department must still show up in the historical record of which projects they worked on. There is no "delete person" action anywhere in this tool, full stop.

**Workload display (validated via mockup):** the page is a swimlane, not a plain list — one row per person (grouped by role, with the new Junior TPM tier included), and one horizontal bar per active project they're on, all positioned against a shared calendar axis at the top. The same project renders in the same color across every row, so a project's full team is scannable across the page, not just within one person. A Developer whose two bars genuinely overlap in time gets a red hatched marker under the overlapping span — this is the Resource Pool's visual expression of the §5 conflict rule, made visible before you even open a project, not just enforced reactively when assigning. TPMs, Junior TPMs, BAs, and Tech Leads show overlapping bars with no flag — multi-project is expected for those roles.

### 4.3 New Project

> This section's field list conflicts with the BRD on two points — required fields (FR-PRJ-01) and department capture level (FR-PRJ-04) — see [Shared Definitions §5–6](shared-definitions.md#5-department-capture-level), pending B6/B7.

Creation form, expanded from the original minimal five fields after direct review (validated via mockup):

- Name
- **Description** *(added)* — what the project actually is, in a couple of sentences. The original form had no way to say what was even being built, which was a real gap, not a stylistic one.
- **Standalone, or part of another system?** *(added)* — free text for now (e.g. "Standalone," or "a new page inside the Officer Portal"). Some projects are a full system; others are one page or service living inside something that already exists, and that distinction matters for scoping. Deliberately kept as a text box rather than a link to an actual other-system record — that requires the user to first specify what those other systems even are, which hasn't happened yet.
- Owning department (General Department only — see the note below §4.1's table)
- Priority
- Business PM name
- Origin (HQ directive vs. departmental request) — matches BRD FR-PRJ-01
- **Value justification / business case note** *(decided, not yet added to the mockup)* — the studied need and expected value behind the project, most relevant when Origin is a departmental request, since the BRD requires that study before acceptance (§3.4 of the BRD). Out of ten candidate fields discussed in this session, this was the only one kept — the other nine (project reference code, target dates, directive/reference number, sponsor, category, data sensitivity, expected user base) were considered and explicitly rejected, not merely unmentioned.

Team assignment does **not** happen at creation — a new project starts with no team, and is staffed afterward on its Project Detail page. This keeps project creation fast and treats staffing as a distinct, deliberate decision rather than something rushed through a creation wizard.

**Explicitly rejected as top-level New Project fields**, and why: budget/cost and document attachments (both already out of scope per BRD §8.2); the specific sub-department (a project's underlying record could still capture this, but it wasn't added as a field here — flagged as a possible gap, since the list only ever shows the rolled-up General Department, §4.1); project reference codes, target dates, directive numbers, sponsors, categories, data sensitivity, and expected user base (discussed as a candidate list of ten, all but the value-justification field above were turned down).

### 4.4 Project Detail

> The team-assignment picker described below has no allocation-% field, which the BRD's over-allocation rule (RULE-07) requires — see [Shared Definitions §4](shared-definitions.md#4-assignment-allocation-), pending B4.

Follows the "compact preview + button" layout validated in the timeline design session (Option 2 of three mocked layouts):

- **Header:** project name, status badge, priority badge, owning department, business PM
- **Stats row:** current phase, delivery date, days remaining, team size
- **Team & Resources card:** people currently assigned, each with a role tag; an "add person" control that opens a role-grouped dropdown pulling from the Resource Pool (§4.2); assignment is **freeform** — any role, any number of times (e.g. two Developers, zero Tech Leads) — matching BRD FR-ASG-03 rather than a fixed-slot model
- **Recent Activity:** a short feed of change requests and holds affecting this project (ties to BRD FR-RPT-02)
- **Timeline preview:** a compact, non-interactive summary of the project's bar (current phase composition, delivery date, % complete) with an **"Edit Timeline →"** button that opens the full interactive sandbox designed separately

### 4.5 Mid-Project Resource Swap

Rare, but needs to exist without corrupting anything: replacing a Developer (or anyone) partway through a project — someone leaves, is reassigned, or otherwise needs to hand off mid-stream.

**The key realization: this doesn't touch the timeline at all.** The timeline sandbox tracks *work* (phases, days, pauses) — it has no notion of *who* specifically is doing that work. Who's assigned lives entirely in the Project Detail page's Team & Resources card. Because those two are already decoupled, a resource swap is purely an assignment-record change, not a schedule change — nothing about blocks, dates, or delivery gets touched or needs recalculating.

Mechanically, this is exactly what the BRD's conceptual data model already anticipated (§12.1: Assignment has `start date`, `end date`; FR-ASG-13: "retain a history of all assignments, including ended ones") — this session just needed to decide how the *page* exposes it:

- Each person on the Team & Resources card gets a **"Replace"** action alongside add/remove.
- Replacing ends that person's assignment as of a chosen date (defaulting to today) — this record is **kept, not deleted**, consistent with §4.2's no-hard-deletion rule — and opens the same role-grouped picker to choose who takes over, whose assignment starts from that same date.
- The swap writes a `resource_swapped` entry to the project's [Event Log](2026-09-11-timeline-visual-design.md §3.3) — always `dayImpact: 0`, since a swap never touches the schedule — and appears in Recent Activity ("Yousef M. replaced by Lina R. as Developer — Oct 12, 2026"). The Resource Pool's swimlane reflects it accurately: Yousef's bar for this project now ends at the swap date instead of running the full original span; Lina's bar starts there and runs to the project's end.

Not yet decided: whether a reason/note should be required on a swap (parallel to the Hold reason taxonomy) or left as free text. Leaning toward optional free text — a swap isn't a business-rule event like a hold is, just a record-keeping one.

### 4.6 Change Request Workflow *(added 12 September 2026 — resolves [design audit](../../design-audit-2026-09-12.md) recommendation #2, the largest gap it identified)*

The mission's first bullet — *"a new requirement is answered with the man-days it adds and the date it moves, before it is accepted"* — had no design anywhere until now. Three steps:

1. **Raise.** Title, description, requester, requesting department, effort in man-days per role (BRD FR-EST-01). **Note a real inconsistency here, not yet resolved:** the timeline sandbox's block model (Timeline design §3.1/§4.7) only has a single undifferentiated `days` number per block, with no role breakdown — it cannot currently represent "man-days per role" at all. This change request field and the block model it's supposed to feed are built on two different effort models. See the [Shared Definitions](shared-definitions.md) file, effort model entry, and design audit recommendation #3 — this is queued to be resolved, not resolved here.
2. **Preview the impact, before anything is committed.** The calculated day-impact and the resulting revised delivery date are shown — this is the actual moment the mission describes, and matches BRD FR-CHG-10 exactly (view impact without committing it).
3. **Record the decision — made outside this system.** The approval decision itself is not made inside PRMS — it happens in a meeting, over email, wherever the organisation actually makes it. What the PM does here is **record that it happened**, and attach a single piece of proof: a PDF or an email, evidencing the approval. This attachment matters specifically because it will be needed later — when a stakeholder is looking at the [Stakeholder View's replay feature](2026-09-12-stakeholder-view-design.md §4.3) and asks "did we actually agree to this," the proof is one click away.

On recording approval, the change request writes one entry to the project's [Event Log](2026-09-11-timeline-visual-design.md §3.3) (`cr_approved`, with `dayImpact` and the attached `proofDocument`) and appears in Recent Activity. This is the only way a delivery date is allowed to move due to new scope — the sandbox itself does not commit day-impact changes to the Event Log on every drag; only an approved change request (or a hold) does.

**This is a single proof-of-approval attachment per change request, not a document repository.** The BRD explicitly excludes general document management (§8.2) — that exclusion stands. This is narrower: one file, evidencing one decision, kept for later reference.

**Not yet designed:** what a rejected or withdrawn change request looks like on screen (FR-CHG-05/07 already define the states; no UI has been sketched for them), and where exactly on Project Detail the "Raise a change request" action lives.

## 5. The Assignment Conflict Rule

This is the one rule in this document that isn't just a restatement of a page layout — it's where the Resource Pool and the timeline sandbox genuinely depend on each other.

When assigning a Developer to a project, the picker does **not** warn simply because that person is "on another project." It warns only when the other project's actual date range **overlaps** this project's date range:

> A Developer finishing Project A on 20 October can be freely assigned to Project B starting 21 October — no warning, no conflict. The same Developer assigned to a project running 15 October–5 November *while* still on Project A through the 20th *does* get flagged.

This matches BRD RULE-06 / FR-ASG-04 exactly, but makes explicit what "overlapping dates" requires: **each project's real delivery-date math**, which only exists because the timeline sandbox computes it (working days, weekends, pauses all folded in). The conflict check is therefore not a Resource Pool feature in isolation — it reads the computed date range that the timeline produces for every project a person is on.

## 6. Open Questions

1. ~~Should "New Project" be blocked if the Resource Pool has no active people at all?~~ **Resolved: no blockers.** A project can be created — and even staffed with nobody, if that's where things stand — regardless of Resource Pool state. Consistent with the BRD's general principle of soft warnings over hard blocks (RULE-06/07).
2. ~~Where does the conflict check actually run?~~ **Resolved: live recalculation.** Every conflict check walks the other project's blocks from scratch to get its current date range — never a cached/stored value that could go stale. Slower than caching, but always accurate; acceptable at the scale this tool operates at (BRD: ~30–60 active projects).
3. ~~Resource Pool page interactivity was not mocked.~~ **Resolved via mockup: a timeline swimlane**, detailed above. Two people-list variants were compared (click-to-expand chip list vs. always-visible swimlane); the swimlane was preferred.
4. ~~Multiple TPMs, or one TPM per project?~~ **Resolved: one lead TPM, plus any number of Junior TPMs as needed.** This introduces a role tier the BRD doesn't currently have — it only defines a single "Technical Project Manager" role (FR-RES-02). Before implementation, the BRD's role list and RULE-08 (multi-project roles) should be revisited to add Junior TPM explicitly, including whether it behaves like a TPM for the freeform-assignment and multi-project rules, or has its own constraints.

## 6.1 Future Ideas (raised, not yet designed)

### Developer progress reporting, with PM verification

> This idea requires Developer accounts, which directly contradicts BRD §7.1 ("Developers are modelled as a managed resource, not as active system users") — see [Shared Definitions §7](shared-definitions.md#7-developer-system-access), pending B8.

Raised twice, connecting to two rounds of discussion: first as a bare idea ("a screen for developers to update their progress — we'll get into that later"), then, once the timeline design gained a weight/sub-step model (see [Timeline design](2026-09-11-timeline-visual-design.md) §4.7–4.8), the actual mechanism became clear:

- **A block's completion is not time-based** — it's based on which of its sub-steps have been verified as actually done. This directly answers a limitation the timeline spec flagged in its own §4.5: today's % complete in the sandbox is purely "days elapsed," not "work confirmed."
- **Two-step workflow, not self-reported alone:** a Developer marks a sub-step "done" from their own account; a Technical PM then reviews and confirms it before it counts. A developer's own say-so is not sufficient on its own — the PM verifies.
- **This requires Developers to have real accounts and their own limited screen** — a genuine expansion beyond every other design decision in this document, all of which assumed Developers are a managed resource, not a logged-in user of the tool themselves. Not yet decided: what a Developer's screen shows them (presumably: their assigned sub-steps, across whichever projects they're on) and what happens to a sub-step that's rejected rather than confirmed.
- **Not yet decided:** whether a "reported but not confirmed" sub-step counts toward completion at all (see the matching open question in the timeline design, §7 item 7 there) — this is the same unresolved question, not a separate one, since it's really about how this workflow feeds that document's data model.

This is still explicitly **not designed, only scoped** — the user's instruction was to think about it, not build it. It's written here in more detail than the one-liner it started as because the weight/sub-step model it depends on is now real, not because the workflow itself has been finalized.

## 7. Relationship to the Timeline Design

This document assumes the [timeline/visual-summary design](2026-09-11-timeline-visual-design.md) as-is, including its own open questions (holiday calendar, drop-date precision, persistence). Nothing here resolves those; the "Edit Timeline" button is simply a doorway into that already-specified sandbox. Where that document's open questions get resolved — particularly persistence and real project data — will directly affect how the Project Detail page's timeline preview and the assignment conflict rule (§5) actually get implemented.

## 8. Relationship to the Stakeholder View

§4.6's Change Request Workflow is the write side of a pair whose read side is the [Stakeholder View design](2026-09-12-stakeholder-view-design.md) — a change request approved here becomes an Event Log entry there, viewable (proof attachment included) in that document's replay feature. Neither document duplicates the other's content; each covers one direction.
