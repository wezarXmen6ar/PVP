# Design: Project Timeline / Visual Summary Component

**Status:** Draft — validated through interactive mockup, pending your review
**Date:** 11 September 2026
**Author:** Technical PM, GD of AI (design captured via collaborative mockup session)
**Related:** [BRD — Portfolio & Resource Management System](../../BRD-portfolio-resource-management.md), specifically FR-VIZ-03 (timeline view), FR-VIZ-04 (variance breakdown), FR-CHG-03/04 (change request impact), FR-HLD-05/06/07 (hold recalculation), RULE-10 (hold recalculation), RULE-12 (hold reasons)

---

## 1. Purpose

The BRD calls for a stakeholder-facing timeline that makes two things visible without a deck being built: **what a new requirement costs**, and **what pausing a project for a surprise priority costs**. This document specifies the concrete visual and interaction design for that timeline, validated through sixteen rounds of an interactive HTML mockup built live with the user.

The core idea that survived every iteration: **a single vertical bar, built bottom-to-top out of discrete blocks, each block belonging to exactly one phase, stacked in the literal chronological order the user places them** — not grouped by phase type. This is what makes a pause insertable at an arbitrary point without restructuring the whole model, and what makes the bar read as one continuous, buildable timeline rather than a set of parallel phase lanes.

## 2. Scope

**In scope for this document:** the visual/interaction design of the timeline component itself — data model, business rules for date math, and the sandbox interactions (create, place, edit, split, merge, reorder) used to build and manipulate it.

**Out of scope:** wiring this to real project/resource data, authentication, multi-user concurrency, persistence, and the capacity/assignment engine described in the BRD. This is validated as a standalone sandbox; integrating it into the wider PRMS application is a separate follow-on (see §7).

## 3. Data Model

### 3.1 Block

The atomic unit of the timeline. Every block belongs to **exactly one phase** — a block cannot span multiple phases (an earlier version of this design allowed that; it was reverted because it made the sandwiching/pause use case impossible to represent cleanly).

| Field | Type | Notes |
|---|---|---|
| `name` | string | Required. Two blocks split by a pause keep the *same* name — they are still one phase, just interrupted. |
| `description` | string | Optional free text. |
| `phase` | enum | `analysis` \| `development` \| `testing` \| `security` \| `trial` \| `hold` |
| `days` | integer | Working-day effort. For a `hold` block, this is the pause's duration, not effort. |
| `reason` | enum, hold only | One of the four BRD hold reasons (RULE-12): resources pulled to higher-priority project; awaiting business owner input; awaiting approval/budget; blocked by external/technical dependency. |
| `groupId` | internal | Set when a block is produced by splitting an existing one; used only to drive the auto-merge rule (§4.4). Not a user-facing field. |

### 3.2 Timeline

An **ordered list** of blocks. Order is chronological — index 0 is the first thing that happens (rendered at the bottom of the bar), and each subsequent block picks up immediately where the previous one ends. There is no independent "start date" per block; a block's start is always the previous block's end (or the project start date, for index 0).

This is a deliberate simplification: the model has no explicit resource assignment, no per-block owner, and no notion of parallel work. It represents **one continuous thread of time**, which is sufficient for the timeline/impact-visualization use case but is *not* the same model as the BRD's capacity engine (which allocates multiple people across concurrent projects). Reconciling the two is a follow-on question (§7).

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

### 4.6 Placement and dragging

Every block — whether newly created or already on the timeline — is drag-and-drop placeable at an arbitrary point:

- **New blocks** are created via a template picker (§5.1), land in a "ready to place" tray, and are dragged from there onto the bar.
- **Existing blocks** can be picked directly off the bar and dropped elsewhere; dropping outside the bar (or anywhere invalid) restores the block to its original position — full sandbox safety, nothing is lost by an aborted drag.
- While any drag is in progress, the ruler temporarily switches from its normal compact view (only block-boundary dates shown) to **every working day labeled**, so the drop point can be judged precisely. It reverts to compact the instant the drag ends.
- A live indicator line follows the cursor during the drag, labeled with the exact date it would land on.

**Known limitation, accepted as out of scope:** the pixel-to-date conversion during drop is not perfectly precise — dropping "on" a specific date can land a day or two off. This was identified and explicitly deferred by the user: *"the date is not accurate... but that is fine, this is just a mockup, don't fix it."* If this component moves toward production, this is the first thing to tighten (§7).

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

- **Phase colors:** Analysis (light blue-violet), Development (blue), Testing (orange), Security testing (pink), Trial (green) — consistent across the bar, the legend, the template picker, and the side list.
- **Hold / pause:** a diagonal grey hatch, never a solid color — visually distinct as "no work," not as a seventh phase color.
- **Month bands:** alternating faint background stripes behind the bar, labeled with real month names (not "Month 1/2/3") — driven off an actual project start date.
- **The "now" marker:** a dashed red line, distinct from the blue dashed drag-indicator line, so the two concepts (current position vs. drop target) are never visually confused.
- **Linked/grouped blocks:** hovering one half of a split block outlines its other half, so the "these are still one phase" relationship is discoverable without reading a tooltip.

## 7. Open Questions for the Next Phase

These are not blocking the sandbox design, but need answers before this becomes a real, data-backed component:

1. **Drop-date precision** — deferred by explicit user instruction (§4.6); needs a real fix before production use.
2. **Relationship to the capacity/assignment engine** — this model is single-threaded (one sequence of blocks); the BRD's resourcing model is multi-project, multi-person. How a real project's timeline block maps to actual assigned Developers/BAs/Tech Leads is undecided.
3. **Persistence** — this is currently an in-memory sandbox with no save/load beyond a single "load ready project" seed button. Real usage needs the block sequence to be the durable record referenced by BR-04/FR-EST-03 (baseline retention).
4. **Multi-project portfolio view** — this design is one project's timeline. The BRD's portfolio dashboard (FR-VIZ-02/07) needs many of these shown at once; whether that's many small versions of this same bar or a different visualization is unresolved.
5. **Holiday calendar** — only weekends are excluded from working-day math right now; RULE-04 calls for configurable public holidays too.

## 8. What This Design Validates

Returning to the BRD's stated purpose: this design demonstrates, concretely and interactively, that a stakeholder can be shown — not told — what a pause costs (§4.2) and see it happen in front of them (§5.3, §4.6) rather than receiving a static slide. That was the central ask behind the whole BRD, and this mockup is the first artifact that makes it tangible rather than aspirational.
