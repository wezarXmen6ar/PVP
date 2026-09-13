# Design: Project Timeline / Visual Summary Component

**Status:** Draft — validated through interactive mockup, pending your review
**Date:** 11 September 2026 (revised 12 September 2026 — see §3.1a)
**Author:** Technical PM, GD of AI (design captured via collaborative mockup session)
**Related:** [BRD — Portfolio & Resource Management System](../../BRD-portfolio-resource-management.md), specifically FR-VIZ-03 (timeline view), FR-VIZ-04 (variance breakdown), FR-CHG-03/04 (change request impact), FR-HLD-05/06/07 (hold recalculation), RULE-10 (hold recalculation), RULE-12 (hold reasons); [PM Tool design](2026-09-11-pm-tool-design.md) §6.1 (developer verification workflow, which consumes the weight/sub-step model defined here)

---

## 1. Purpose

The BRD calls for a stakeholder-facing timeline that makes two things visible without a deck being built: **what a new requirement costs**, and **what pausing a project for a surprise priority costs**. This document specifies the concrete visual and interaction design for that timeline, validated through sixteen rounds of an interactive HTML mockup built live with the user.

The core idea that survived every iteration: **a single vertical bar, built bottom-to-top out of discrete blocks, each block belonging to exactly one phase, stacked in the literal chronological order the user places them** — not grouped by phase type. This is what makes a pause insertable at an arbitrary point without restructuring the whole model, and what makes the bar read as one continuous, buildable timeline rather than a set of parallel phase lanes.

## 2. Scope

**In scope for this document:** the visual/interaction design of the timeline component itself — data model, business rules for date math, and the sandbox interactions (create, place, edit, split, merge, reorder) used to build and manipulate it.

**Out of scope:** wiring this to real project/resource data, authentication, multi-user concurrency, persistence, and the capacity/assignment engine described in the BRD. This is validated as a standalone sandbox; integrating it into the wider PRMS application is a separate follow-on (see §7).

**Confirmed role of this sandbox, settled 12 September 2026:** it is a PM-side tool only, in two modes — the PM's own private planning surface, and a tool the PM drives live (screen-shared or projected) in a stakeholder meeting to demonstrate a proposed change's effect. It never appears on a stakeholder's own screen or under their own access — that role belongs entirely to the separate, read-only [Stakeholder View](2026-09-12-stakeholder-view-design.md), which this document does not define. See that document's §6 for the full reasoning.

## 3. Data Model

### 3.1 Block

The atomic unit of the timeline. Every block belongs to **exactly one phase** — a block cannot span multiple phases (an earlier version of this design allowed that; it was reverted because it made the sandwiching/pause use case impossible to represent cleanly).

| Field | Type | Notes |
|---|---|---|
| `name` | string | Required. Two blocks split by a pause keep the *same* name — they are still one phase, just interrupted. |
| `description` | string | Optional free text. |
| `phase` | enum | See §3.1a — revised 12 September 2026. |
| `days` | integer | Working-day effort. For a `hold` block, this is the pause's duration, not effort. |
| `weight` | percent | See §4.7. Auto-derived from `days` as a share of the project total; a Technical PM can override it, same pattern as a date override (RULE-05). |
| `owner` | string, optional | Free text, not a structured relationship. See §4.9. |
| `subSteps` | list, optional | See §4.8. Each item: `{ name, weight }`, weights summing to 100% of *this block's own* share, not the project's. |
| `reason` | enum, hold only | One of the four BRD hold reasons (RULE-12): resources pulled to higher-priority project; awaiting business owner input; awaiting approval/budget; blocked by external/technical dependency. |
| `groupId` | internal | Set when a block is produced by splitting an existing one; used only to drive the auto-merge rule (§4.4). Not a user-facing field. |

### 3.1a Revision — canonical phase list (12 September 2026)

> Conflicts with the BRD's status enum (FR-PRJ-02/RULE-01) and phase order — see [Shared Definitions §1](shared-definitions.md#1-phase--status-list), pending B1/B2.

Superseding the original six-phase list (`analysis` / `development` / `testing` / `security` / `trial` / `hold`), arrived at by asking what actually earns a phase its own block rather than being folded into another one or left as a sub-step (§4.8): a distinct owner or accountable party, a duration stakeholders want tracked on its own, and not being fine-grained enough to just be a checklist item.

**Revised phase enum:** `requirements` | `analysis` | `development` | `testing` | `security` | `uat` | `deployment` | `trial` | `hold`

| Phase | What changed and why |
|---|---|
| **Requirements Gathering** *(new)* | Split out of Analysis. This is where the *business owner*, not GD of AI, is the bottleneck — separating it lets the tool show exactly how long the business took to hand over requirements, distinct from GD of AI's own analysis work. Direct BRD tie-in: this is the same accountability principle behind hold reasons (RULE-12) applied to the front of the project instead of the middle. |
| **Analysis** | Unchanged in meaning, now scoped specifically to GD of AI's own technical analysis and estimation, once requirements are in hand. |
| **Development** | Unchanged. |
| **Testing** | Kept **separate from UAT**, not merged into it — different owner (dev/QA team) and different failure mode (bugs found internally vs. the business rejecting functionality at UAT). |
| **Security testing** | Unchanged. |
| **UAT** *(renamed from "Trial")* | Business-user acceptance testing and sign-off — this is what the original "Testing" block's description ("functional test & business sign-off") was already describing. Comes before Deployment. |
| **Deployment** *(new)* | Release to production, including change-control/approval time — worth tracking on its own since government change-control can genuinely consume real calendar days. |
| **Trial** | Redefined, not removed: now specifically the *post-deployment* limited pilot with real users, distinct from UAT's pre-launch sign-off. Comes after Deployment. |
| **Hold** | Unchanged — cross-cutting interrupt, not a sequential phase. |

**"Design" was considered and deliberately not added as a phase** — it doesn't clearly have a distinct owner or a stakeholder who tracks it apart from Analysis or Development, so it fits better as an optional sub-step (§4.8) inside one of those than as its own top-level block.

**No block is required, and none are enforced in any order.** This was already true of the sandbox and remains true under the revised list — a project can use any subset of these phases, in any sequence, or invent a completely different name for a block (`phase` constrains *color/behavior*, not the block's `name`, which is always free text). Formal ordering rules (e.g. "Analysis must precede Development") are a possible future layer, deliberately not built now — the user's explicit instruction was to prioritize flexibility over structure at this stage.

### 3.2 Timeline

An **ordered list** of blocks. Order is chronological — index 0 is the first thing that happens (rendered at the bottom of the bar), and each subsequent block picks up immediately where the previous one ends. There is no independent "start date" per block; a block's start is always the previous block's end (or the project start date, for index 0).

This is a deliberate simplification: the model has no explicit resource assignment, no per-block owner, and no notion of parallel work. It represents **one continuous thread of time**, which is sufficient for the timeline/impact-visualization use case but is *not* the same model as the BRD's capacity engine (which allocates multiple people across concurrent projects). Reconciling the two is a follow-on question (§7).

### 3.3 Event Log *(added 12 September 2026 — see the [design audit](../../design-audit-2026-09-12.md), recommendation #1)*

The durable record the sandbox itself deliberately lacks. The sandbox's block sequence is a **mutable planning surface** — dragging, splitting, and merging leave no trace, by design, because that's what makes it a good sandbox to think in. But nothing downstream (the [Stakeholder View](2026-09-12-stakeholder-view-design.md), the Portfolio List's variance column, RULE-11's reconciliation) can be built on a surface that erases its own history. The Event Log is the answer: a separate, **append-only** list, distinct from the block sequence, that a project's plan writes to when something is *committed* — not on every drag inside the sandbox, but when a change request is approved or a hold starts/resumes.

| Field | Type | Notes |
|---|---|---|
| `date` | date | When this event happened. |
| `type` | enum | `baseline_set` \| `cr_approved` \| `hold_started` \| `hold_resumed` \| `override_applied` \| `resource_swapped` |
| `description` | string | Free text — what changed. |
| `dayImpact` | integer | Working days added to the project's delivery date by this event. Zero or absent for `hold_started` (the impact isn't known until `hold_resumed` closes it out). Always `0` for `resource_swapped` (see below). For `override_applied`, the difference between the override date and the calculated date it replaced — see below. |
| `cause` | string | Who or what caused this — a requesting department for a CR, a displacing project reference for a hold. For `override_applied`, the PM's required justification (RULE-05). |
| `proofDocument` | file reference, optional | Set on `cr_approved` events. See [PM Tool design §4.6](2026-09-11-pm-tool-design.md) — this is the attached PDF or email evidencing that the approval genuinely happened, even though the approval decision itself was made outside this system. |

**Two entry types added 13 September 2026, closing a real gap:** the original four types couldn't account for two things RULE-11 itself requires as terms in its reconciliation formula — a manual date override (RULE-05) and a mid-project resource swap (PM Tool design §4.5, which says a swap is "logged to Recent Activity" without ever saying where that log actually is). Both now write here:
- `override_applied` — a Technical PM overrides a calculated duration or date. Its `dayImpact` is the delta between the override and the calculation it replaced, so it folds into the same additive sum as every other event, and its `cause` field carries the required justification.
- `resource_swapped` — always `dayImpact: 0`. It exists purely so the swap described in PM Tool §4.5 has a durable record at all; per that section's own reasoning, a swap never touches the schedule, so it must never contribute to the date sum.

**Current delivery date is always `baseline_set`'s date plus the sum of every subsequent entry's `dayImpact`.** This is RULE-11 made literal rather than aspirational — there is no code path that can move a delivery date without an entry existing to explain the movement, because the date *is* the sum, not an independently-stored value that a reconciliation rule checks against. This now holds for all four ways a date can legitimately move (change request, hold, override) plus the one event that must never move it (resource swap) — not just the two it originally covered.

This is also, deliberately, the entire data source for the [Stakeholder View](2026-09-12-stakeholder-view-design.md) — including its replay feature, which is only possible because this log exists: replaying a project's history means reconstructing what was true as of any given date, which requires the sequence of what-changed-when to actually be retained somewhere.

## 4. Business Rules

### 4.1 Working days vs. calendar time

A block's `days` value is working-day effort. The project has a working calendar (Monday–Friday; holidays out of scope for the mockup). Every date shown anywhere in the UI — block boundaries, delivery date, month bands — is computed by walking the calendar and skipping weekends, exactly as RULE-03/RULE-04 in the BRD specify.

**The bar itself does not visually stretch to show weekend space** in its normal (compact) state — a 5-day block is 5 units tall regardless of whether a weekend falls inside its span. This was tried the other way (each block rendered with visible weekend "gaps") and reverted: the underlying dates stayed accurate either way, and the visual clutter wasn't worth it. The distinction is preserved as a **separate stat**, not a drawn element (§4.2).

### 4.2 Effort vs. pause — the core hold rule

This was the most important correction made during the session: **a pause does not consume effort, but it does consume time.**

- **Working days (effort)** = sum of `days` across every non-hold block.
- **Paused days** = sum of `days` across hold blocks only.
- **Calendar span / Projected delivery date** = computed from the *full* sequence including holds — a pause pushes every date after it, exactly as far as its own duration, and nothing more.

Concretely: a Development block of 18 working days, interrupted by a 5-day pause, still shows "18" toward total effort — the pause doesn't inflate it — but the delivery date is 5 working days later than it would have been without the pause. This is the mechanism that gives the BRD's "what did this pause cost?" question a precise, honest answer (FR-HLD-05, RULE-10).

### 4.3 Splitting

A block splits into two pieces at an offset when something is inserted into the middle of it. Both pieces keep the original's `name`, `description`, and `phase` — they are presented as the same phase, continued. The two pieces are tagged with a shared internal `groupId`.

A block can be split by:
1. **The "now" marker** (§4.5) — user clicks the ruler to mark current position; if it lands inside a block, a "Split & pause here" action appears.
2. **A hold's start date** — creating a Hold block with an explicit start date automatically finds whatever block occupies that date and splits it there.
3. **Dragging anything onto a point inside an existing block** (§4.6) — same splitting logic, generalized to any block type being placed, not just holds.

Splitting a **Hold** block itself is not allowed — dropping something into the middle of an existing pause instead snaps to whichever edge of that pause is closer.

### 4.4 Auto-merge

Whenever two blocks sharing a `groupId` become **directly adjacent** in the sequence — because the pause between them was deleted, or dragged elsewhere — they silently recombine into a single block (days summed, `groupId` cleared). This is checked on every state change, not just on delete, so it also fires if the separating pause is moved away by drag. The rule is general: it is not specific to holds, only to "two pieces of an originally-split block with nothing between them."

### 4.5 The "now" marker

A manually-placed marker representing where the project currently stands — **not** tied to the system clock. Set by clicking the date ruler. Drives:
- A completion-percentage overlay on each block, purely time-based (days elapsed ÷ days planned for whichever block the marker falls inside; 100% for anything fully before it, untouched for anything after).
- The split-and-insert-pause action described above.

This is explicitly a planning tool, not a live status feed — the user places it to model a scenario ("we are here, and a pause happens now"), and it does not advance on its own.

**Reconciling this with weight-based completion (§4.7–4.8):** these serve two different purposes and are not in conflict. The marker's time-based % is for *scenario planning* inside the sandbox — "if we're here, what does a pause cost." The weight/sub-step model, combined with the developer-reported-then-PM-confirmed verification workflow (PM Tool design §6.1), produces the *actual, real* completion percentage once that workflow exists — based on verified work, not elapsed time. The sandbox marker does not disappear or get replaced; it answers a different question than the real one does.

### 4.6 Placement and dragging

Every block — whether newly created or already on the timeline — is drag-and-drop placeable at an arbitrary point:

- **New blocks** are created via a template picker (§5.1), land in a "ready to place" tray, and are dragged from there onto the bar.
- **Existing blocks** can be picked directly off the bar and dropped elsewhere; dropping outside the bar (or anywhere invalid) restores the block to its original position — full sandbox safety, nothing is lost by an aborted drag.
- While any drag is in progress, the ruler temporarily switches from its normal compact view (only block-boundary dates shown) to **every working day labeled**, so the drop point can be judged precisely. It reverts to compact the instant the drag ends.
- A live indicator line follows the cursor during the drag, labeled with the exact date it would land on.

**Known limitation, accepted as out of scope:** the pixel-to-date conversion during drop is not perfectly precise — dropping "on" a specific date can land a day or two off. This was identified and explicitly deferred by the user: *"the date is not accurate... but that is fine, this is just a mockup, don't fix it."* If this component moves toward production, this is the first thing to tighten (§7).

### 4.7 Block weight

> The `days`-only block model this section builds on conflicts with the BRD's man-days-per-role effort model — see [Shared Definitions §3](shared-definitions.md#3-effort-model), pending B3.

Every block carries a `weight` — its share of the project, as a percentage. By default this is **derived automatically** from `days` (a block's day-count ÷ the project's total day-count), never entered independently. This was a deliberate choice over letting weight be freely set: an independent weight field would be a second source of truth that can silently disagree with the schedule (e.g. a block claiming 40% weight while actually being 10% of the days), and validating the two against each other is exactly the kind of complexity the user was trying to avoid by asking for flexibility.

A Technical PM can override the derived weight, following the same pattern already established for calculated dates (RULE-05: calculated, but overridable with the override visibly distinguished from the calculated value). An override is for the rare case where a block's actual importance genuinely doesn't track its day-count — e.g. a short Security Testing block that carries disproportionate risk.

Weights across all of a project's top-level blocks always sum to 100%.

### 4.8 Sub-steps

Any block — not just Development, per explicit correction during this session ("we should add this flexibility for all blocks") — can optionally be broken into sub-steps. Each sub-step is `{ name, weight }`, where weight is that step's share of **its parent block's** weight, not the project's — sub-step weights sum to 100% *within their block*, independently of every other block's internal breakdown.

There is no limit on how many sub-steps a block can have, and having zero is the normal case — most blocks stay a single, unbroken unit. Sub-steps exist specifically for blocks complex enough that tracking real progress inside them matters — Development being the obvious common case, but not the only one.

Sub-steps are the unit the developer-verification workflow (PM Tool design §6.1) reports progress against. A sub-step's own completion is not time-based at all — see that document for the reported/confirmed workflow. This spec defines the *structure* (weighted breakdown); it does not define *who marks a sub-step done*, which is a PM Tool concern, not a timeline-rendering one.

### 4.9 Owner

Every block has an optional `owner` — free text, not a structured relationship to the Resource Pool, a department, or another project, though it will often name one of those informally ("Security Department," "Database Department," "Business Owner," a specific Developer's name). Deliberately kept unstructured for now, per the user's explicit preference for flexibility over enforced structure at this stage; formalizing it into a real relationship (so it could be queried or reported on) is future work, not required now.

**Hold is the one case where this is not a second field.** A Hold block whose reason is "resources pulled to a higher-priority project" already links to the project that displaced it (§3.1, `reason`). That link *is* the Hold's owner — it is not duplicated into a separate owner field alongside it. For the other three hold reasons, `owner` is free text same as any other block, since there's no existing structured link to reuse.

"Design" work needing an owner was the case that prompted this: it might be a Resource Pool Developer, might need a "Designer" role that doesn't exist, or might need no owner at all because the design already exists in another system. Resolved by not forcing the issue — `owner` stays free text and skippable, and no new Resource Pool role was added.

## 5. Interactions

### 5.1 Creating a block: template picker

Rather than a phase dropdown, the entry point is a grid of six colored boxes — one per phase, using the same colors as the bar and legend — plus a Hold/Pause box using a grey hatch pattern instead of a solid color (signaling "no work happening" rather than a phase). Clicking a template reveals the detail form (name, description, days, and for Hold, reason + optional start date) directly beneath it, with a "change" link to back out and pick differently.

### 5.2 Editing

Clicking any placed block — on the bar itself, or its row in the side list — opens the same form pre-filled, with the matching template box highlighted. Saving updates the block in place; the position in the sequence doesn't change (repositioning is a drag, not an edit).

### 5.3 Inserting a pause by exact date

The Hold template's detail form includes an optional start date. If set, creation bypasses the tray entirely: the system finds which working day that date corresponds to, splits whatever block occupies that point (§4.3), and inserts the pause immediately — this is the direct path for "I know a surprise project starts October 5th."

### 5.4 Reordering

Two mechanisms coexist: drag-and-drop (arbitrary repositioning, precise, uses the daily ruler) and ▲▼ buttons in the side list (adjacent-swap only, coarse but fast for small nudges). Both remain in the design — the buttons weren't judged redundant once dragging worked correctly.

## 6. Visual Language

- **Phase colors:** Analysis (light blue-violet), Development (blue), Testing (orange), Security testing (pink), Trial (green) — consistent across the bar, the legend, the template picker, and the side list. **Not yet assigned:** colors for the three new phases (Requirements Gathering, UAT, Deployment) added in §3.1a — deferred until the next mockup pass on this component.
- **Hold / pause:** a diagonal grey hatch, never a solid color — visually distinct as "no work," not as a seventh phase color.
- **Month bands:** alternating faint background stripes behind the bar, labeled with real month names (not "Month 1/2/3") — driven off an actual project start date.
- **The "now" marker:** a dashed red line, distinct from the blue dashed drag-indicator line, so the two concepts (current position vs. drop target) are never visually confused.
- **Linked/grouped blocks:** hovering one half of a split block outlines its other half, so the "these are still one phase" relationship is discoverable without reading a tooltip.

## 7. Open Questions for the Next Phase

These are not blocking the sandbox design, but need answers before this becomes a real, data-backed component:

1. **Drop-date precision** — deferred by explicit user instruction (§4.6); needs a real fix before production use.
2. **Relationship to the capacity/assignment engine** — this model is single-threaded (one sequence of blocks); the BRD's resourcing model is multi-project, multi-person. How a real project's timeline block maps to actual assigned Developers/BAs/Tech Leads is undecided.
3. **Persistence** — this is currently an in-memory sandbox with no save/load beyond a single "load ready project" seed button. **Partially addressed 12 September 2026:** §3.3's Event Log defines *what* the durable record looks like (referenced by BR-04/FR-EST-03), but not *where or how* it's actually stored — no backend/database decision has been made.
4. **Multi-project portfolio view** — this design is one project's timeline. The BRD's portfolio dashboard (FR-VIZ-02/07) needs many of these shown at once; whether that's many small versions of this same bar or a different visualization is unresolved.
5. **Holiday calendar** — only weekends are excluded from working-day math right now; RULE-04 calls for configurable public holidays too.
6. **Colors for the three new phases** (Requirements Gathering, UAT, Deployment) — added 12 September 2026 (§3.1a), not yet assigned.
7. **Partial credit for "reported but not yet PM-confirmed" sub-steps** — once the verification workflow (PM Tool design §6.1) exists, does a sub-step a developer has marked done but the PM hasn't confirmed count toward the block's weighted completion at all, count partially, or count as zero until confirmed? Not decided — flagged here because it directly affects how this component's weight/sub-step data (§4.7–4.8) gets consumed.
8. **Sub-step ordering and per-step days** — sub-steps currently have only `name` and `weight`, no day-count or sequence of their own. Whether they eventually need their own duration (for scheduling, not just weighting) is open.

## 8. What This Design Validates

Returning to the BRD's stated purpose: this design demonstrates, concretely and interactively, that a stakeholder can be shown — not told — what a pause costs (§4.2) and see it happen in front of them (§5.3, §4.6) rather than receiving a static slide. That was the central ask behind the whole BRD, and this mockup is the first artifact that makes it tangible rather than aspirational.
