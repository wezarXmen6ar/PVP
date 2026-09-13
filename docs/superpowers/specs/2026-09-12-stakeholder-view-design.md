# Design: Stakeholder View

**Status:** Initial draft — not final, direction agreed in conversation, not yet mocked up
**Date:** 12 September 2026
**Author:** Technical PM, GD of AI
**Related:** [BRD](../../BRD-portfolio-resource-management.md) FR-VIZ-01…12, OBJ-06, KPI-01; [Timeline design](2026-09-11-timeline-visual-design.md) §3.3 (Event Log — this document's entire data source); [PM Tool design](2026-09-11-pm-tool-design.md) §4.6 (Change Request Workflow, which is where events shown here get created); [Design Audit](../../design-audit-2026-09-12.md) recommendations #2 and #4, which this document resolves

---

## 1. Purpose

This was flagged in the [design audit](../../design-audit-2026-09-12.md) as the one deliverable the mission is actually for, and the [PM Tool design](2026-09-11-pm-tool-design.md) explicitly excluded it (§2) rather than designing it. This document gives it a real design.

Two distinct scales, both read-only, both driven entirely from the Event Log — no separate logic, no separate truth:

1. **Project scale — Replay.** Click into one project and watch its actual history happen, day by day, from kickoff to today.
2. **Portfolio scale — Domino view.** See a chosen set of projects' timelines side by side, so the cascading effect of a decision (pause this, to prioritize that) is visible across projects at once, not just within one.

## 2. Scope

**In scope:** the replay interaction (project scale) and the multi-project side-by-side view (portfolio scale), both as read-only, no-login pages (FR-VIZ-01), and how both render the Event Log.

**Out of scope, deliberately:** any interactivity. This document contains no drag, no editing, no "what-if" manipulation — that is the sandbox's job, and the sandbox stays on the PM's own side of the tool (§6). Also out of scope: authentication, persistence/backend, and the CR/Hold *authoring* workflow (that's [PM Tool design §4.6](2026-09-11-pm-tool-design.md)) — this document only *reads* what that workflow produces.

## 3. Data Source

Everything on every page in this document is a rendering of the **Event Log** (Timeline design §3.3) for the relevant project(s) — plus the events' associated proof-of-approval attachments, where present. There is no independent state here: no number is calculated or invented at render time that isn't already sitting in a log entry.

**Correcting an overclaim from the first draft of this section:** this architecture guarantees the page can't *fabricate* a figure, but it does not, by itself, guarantee FR-VIZ-11's neutral tone. `description` and `cause` are free text a PM types when raising a change request or recording a hold — the architecture constrains *what data can appear*, not *how a human phrases it*. A PM could type something blaming, and the page would faithfully render it. Genuinely enforcing FR-VIZ-11 would need something at the authoring end — e.g. a required phrasing convention or template on those free-text fields in [PM Tool design §4.6](2026-09-11-pm-tool-design.md) and the Hold form — which does not exist yet. Flagged as a new open question (§7).

## 4. Project Scale — Replay

### 4.1 What it shows

A single project's timeline, rendered exactly like the sandbox's stacked bar, but **non-interactive** — no dragging, no editing. Above it, a scrubber spanning from the project's actual start date to today. The portion of the bar beyond today (the still-planned future) renders dimmed/greyed, distinct from the solid-colored past.

### 4.2 Two ways to move through it

- **Manual scrub, day by day.** Drag the scrubber and the bar redraws to show exactly what the plan looked like on that specific day — how many days of Development had happened, whether a hold was in effect, what the projected delivery date was *as of that day* (which may differ from today's projected date, since CRs and holds since then have moved it).
- **Play Highlights.** A single button that auto-advances, but only *jumps between Event Log entries* — skipping the uneventful stretches and landing on each CR, hold, and hold-resume in turn, with a brief callout at each stop describing what happened and its day-impact. This is the "watch the delay happen" mode — a 40-day project might have only four or five highlight-worthy moments in its whole history, and this mode shows exactly those, in seconds.

Both modes are genuinely needed, not redundant: scrubbing answers "what did things look like on this specific date"; Play Highlights answers "walk me through why we're late" without requiring the viewer to know which date to look for.

### 4.3 What gets called out at each stop

At minimum, matching what the Event Log records (Timeline design §3.3): the event type, its description, its day-impact, and its cause/requester. Where a Change Request event has a proof-of-approval attachment (PM Tool design §4.6), a link to it is shown here too — this is explicitly *why* that attachment exists: so that when a stakeholder is watching the replay and asks "did we actually agree to this," the answer is one click away, not "let me go check my email."

## 5. Portfolio Scale — Domino View

### 5.1 What it shows

Several projects' bars, side by side, sharing one calendar axis — the same visual language as the single-project bar, just placed next to each other rather than one at a time. Built specifically to make a cross-project cascade legible: "we paused Project A to prioritize Project B" only reads as a *domino effect* if both bars are visible together, with Project A's hold block visibly linking to Project B (reusing the existing displacing-project link, Timeline design §4.9).

### 5.2 Project selection is deliberate, not automatic

The PM chooses which projects to show together — this is not a permanent view of the entire 30–60-project portfolio at once, which would be unreadable at that scale. A likely default: whichever project is being held, plus its displacing project, plus anything the PM chooses to add for context. Automatically surfacing "everything affected by this decision" is a reasonable future enhancement, not decided now.

### 5.3 Relationship to the single-project replay

The domino view does not (yet) support the same scrub/highlight-playback interaction as §4 — it shows current, real state across projects, not history. Whether it eventually needs its own timeline dimension (replay the domino effect itself, across projects, over time) is an open question (§7).

## 6. Relationship to the Sandbox

Resolved directly in conversation, worth stating plainly since it settles [design audit recommendation #6](../../design-audit-2026-09-12.md): **the interactive sandbox is not part of this document and never appears on a stakeholder's own screen.** It lives entirely on the PM's side (reached via "Edit Timeline" from Project Detail, per the existing design). Its role as a stakeholder-facing tool is that the **PM drives it live, screen-shared or projected, in a meeting** — the stakeholder watches, never touches it themselves. This is why it doesn't conflict with FR-VIZ-01's "no login" requirement: the stakeholder never has their own access to the interactive tool at all, only to this read-only document's pages.

## 7. Open Questions

1. **Smooth day-by-day scrub is real engineering, not just a UI toggle.** It requires reconstructing the exact plan state as of any arbitrary date from the Event Log, on demand. Play Highlights (jumping only between recorded events) is comparatively simple — it just steps through existing rows. Whether both ship together or Highlights ships first is a sequencing decision, not made here.
2. **Does the replay's "future" portion (beyond today, dimmed) show the plan as currently projected, or as it was projected at the point being scrubbed to?** These differ whenever something after that point in history changed the projection. Leaning toward "as currently projected" for the dimmed future (it's clearly marked as not-yet-happened either way), but not confirmed.
3. **Domino view project selection UX** — manual add/remove, confirmed as the starting approach (§5.2), but no interaction has been designed for it yet.
4. **Does the domino view need its own time dimension?** (§5.3) — showing the cascade *as it unfolded* across projects, not just as it stands now.
5. **Export** (FR-VIZ-12) — neither the replay nor the domino view has been considered for static export yet, needed for whenever a live view can't be shown in a meeting room.
6. **How is FR-VIZ-11's neutral tone actually enforced?** (§3, corrected 13 September 2026) — the Event Log architecture stops fabricated figures, not blaming language typed into a free-text field. A phrasing convention or template on the authoring forms is the likely answer; not designed.
