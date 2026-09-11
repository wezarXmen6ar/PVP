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

**Out of scope:** visual polish (these are structural/functional decisions, not a finished UI design), the read-only stakeholder visualization page (BRD §7.2), authentication, and backend/data-layer design. Also out of scope: full working mockups of the Resource Pool, Portfolio List, and New Project pages — only the Project Detail page's timeline tie-in was mocked interactively; the rest were settled through direct discussion and should be treated as less rigorously validated than that page.

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

Resource Pool is reached as a standing navigation item, not a gate the user must pass through before anything else — you can create or open a project immediately from the landing page. Whether project creation should be *blocked* until the pool has at least some people in it is flagged as an open question (§6).

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

### 4.2 Resource Pool

A list of GD of AI delivery staff. Each entry:

| Field | Notes |
|---|---|
| Name | |
| Role | Technical Project Manager, Business Analyst, Tech Lead, or Developer (BRD FR-RES-02) |
| Availability % | Matches BRD RES-03 — accounts for meetings/admin eating into capacity |
| Active / inactive toggle | Lets someone who's left or is on long leave be hidden from assignment pickers without deleting their history on past projects (BRD FR-RES-06) |
| Current workload | Shown directly on this page — which active projects this person is already on, so before opening any project you can see who's free |

This page is the **single source of truth** the New Project and Project Detail pages pull from — there is no free-text entry of a team member's name anywhere else in the tool.

### 4.3 New Project

Minimum creation form:

- Name
- Owning department
- Priority
- Business PM name
- Origin (HQ directive vs. departmental request) — matches BRD FR-PRJ-01

Team assignment does **not** happen at creation — a new project starts with no team, and is staffed afterward on its Project Detail page. This keeps project creation fast and treats staffing as a distinct, deliberate decision rather than something rushed through a creation wizard.

### 4.4 Project Detail

Follows the "compact preview + button" layout validated in the timeline design session (Option 2 of three mocked layouts):

- **Header:** project name, status badge, priority badge, owning department, business PM
- **Stats row:** current phase, delivery date, days remaining, team size
- **Team & Resources card:** people currently assigned, each with a role tag; an "add person" control that opens a role-grouped dropdown pulling from the Resource Pool (§4.2); assignment is **freeform** — any role, any number of times (e.g. two Developers, zero Tech Leads) — matching BRD FR-ASG-03 rather than a fixed-slot model
- **Recent Activity:** a short feed of change requests and holds affecting this project (ties to BRD FR-RPT-02)
- **Timeline preview:** a compact, non-interactive summary of the project's bar (current phase composition, delivery date, % complete) with an **"Edit Timeline →"** button that opens the full interactive sandbox designed separately

## 5. The Assignment Conflict Rule

This is the one rule in this document that isn't just a restatement of a page layout — it's where the Resource Pool and the timeline sandbox genuinely depend on each other.

When assigning a Developer to a project, the picker does **not** warn simply because that person is "on another project." It warns only when the other project's actual date range **overlaps** this project's date range:

> A Developer finishing Project A on 20 October can be freely assigned to Project B starting 21 October — no warning, no conflict. The same Developer assigned to a project running 15 October–5 November *while* still on Project A through the 20th *does* get flagged.

This matches BRD RULE-06 / FR-ASG-04 exactly, but makes explicit what "overlapping dates" requires: **each project's real delivery-date math**, which only exists because the timeline sandbox computes it (working days, weekends, pauses all folded in). The conflict check is therefore not a Resource Pool feature in isolation — it reads the computed date range that the timeline produces for every project a person is on.

## 6. Open Questions

1. **Should "New Project" be blocked (or just warned) if the Resource Pool has no active people at all?** Not settled — leaning toward "allow it, warn softly," consistent with the BRD's general principle of warnings over hard blocks, but not confirmed.
2. **Where does the conflict check actually run** — client-side against cached project date ranges, or does it require a live recalculation? Depends on decisions not yet made about the data/backend layer, explicitly out of scope here (§2).
3. **Resource Pool page interactivity** was not mocked — only discussed. Before implementation, it likely deserves the same interactive-mockup treatment the timeline got, particularly the "current workload shown per person" view, which has real layout implications (a mini timeline per person? a simple project-count badge? undecided).
4. **Multiple TPMs, or one TPM per project?** BRD language implies one Technical PM per project, but this was never explicitly re-confirmed during this session's freeform-assignment discussion.

## 7. Relationship to the Timeline Design

This document assumes the [timeline/visual-summary design](2026-09-11-timeline-visual-design.md) as-is, including its own open questions (holiday calendar, drop-date precision, persistence). Nothing here resolves those; the "Edit Timeline" button is simply a doorway into that already-specified sandbox. Where that document's open questions get resolved — particularly persistence and real project data — will directly affect how the Project Detail page's timeline preview and the assignment conflict rule (§5) actually get implemented.
