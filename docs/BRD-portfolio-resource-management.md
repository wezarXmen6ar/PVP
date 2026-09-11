# Business Requirements Document

## Portfolio & Resource Management System (PRMS)

### General Department of Artificial Intelligence — Dubai Police

---

## 1. Document Control

| Field | Value |
|---|---|
| **Document title** | Business Requirements Document — Portfolio & Resource Management System |
| **Working system name** | PRMS (working title — see §19, OQ-01) |
| **Version** | 0.1 — Draft for internal review |
| **Date** | 9 September 2026 |
| **Author** | Technical Project Manager, General Department of Artificial Intelligence |
| **Owning department** | General Department of Artificial Intelligence (GD of AI) |
| **Intended audience** | Director, GD of AI (primary); GD of AI delivery leadership; the team who will later build the system |
| **Status** | Draft — pending internal review |

### 1.1 Revision History

| Version | Date | Author | Summary |
|---|---|---|---|
| 0.1 | 9 Sep 2026 | Technical PM, GD of AI | Initial draft. Scope, problem statement, business and functional requirements, business rules, conceptual data model. |

### 1.2 How to Read This Document

Sections 1–8 are written for business readers and can be read standalone. Sections 9–13 contain the detailed requirements, business rules and data model intended for whoever builds the system. Sections 14–20 cover assumptions, risks, metrics and open items.

---

## 2. Executive Summary

The General Department of Artificial Intelligence is the central delivery department for digitalisation and automation across Dubai Police. Other general departments own the business problems; GD of AI supplies the Business Analysts, Developers, Tech Leads and Technical Project Managers who deliver the solutions.

That model puts GD of AI at the intersection of two groups who do not see each other's constraints. Business owners in the other general departments request new requirements without understanding that each one consumes finite capacity and moves a delivery date. Police HQ halts running projects to insert higher-priority work, then expects the halted projects to land on their original dates. GD of AI absorbs the gap between these expectations and its actual capacity, and currently has no way to make that gap visible.

Today the entire portfolio is tracked in Excel and reported through PowerPoint decks rebuilt by hand for every meeting. The result is status information that is inconsistent between meetings, dependent on whoever prepared it, and unable to answer the one question that matters: *what did this decision actually cost?*

**PRMS is proposed as a single source of truth for the GD of AI project portfolio.** It models the department's people as a finite capacity pool, holds each project's effort estimate in man-days per role, and automatically recalculates delivery dates when scope is added or work is halted. Every change to a date is recorded with its cause and its originator.

The system serves two audiences. GD of AI staff use the full application to manage projects, assignments and change requests. Business owners and HQ stakeholders access a read-only visualization page that answers three questions without a deck being built:

1. **What does this new requirement cost?** — the requested change, the man-days it adds, and the revised delivery date.
2. **What does halting this project cost?** — which project was displaced, by what, for how long, and its revised date.
3. **What is the true state of the portfolio right now?** — live, consistent, and identical for everyone who opens it.

The central value is not project tracking; established tools already do that. The value is **attribution**: converting "GD of AI is behind schedule" into a specific, auditable, and neutral statement of cause. This document defines the requirements for that system.

The first release is a locally-running prototype focused on the capacity and assignment engine, intended for demonstration to the Director of GD of AI. Production deployment concerns are documented separately in §13.2 and are explicitly out of scope for this release.

---

## 3. Business Context

### 3.1 Organisational Structure

Dubai Police is organised into general departments, each large and independently managed. Examples include the General Department of Criminal Investigation (CID), General Department of Police Stations, General Department of Anti-Narcotics, General Department of Forensic Science and Criminology, and the General Department of Punitive and Correctional Establishments.

Each general department contains sub-departments, which contain sections, which may in turn contain sectors. Requesting bodies for projects can sit at any level of this hierarchy — the Department of e-Crime, for example, is a sub-department within CID.

**Relevance to this system:** a project's business owner must be identifiable at the correct organisational level, and the portfolio must be reportable by parent general department. The system therefore models the departmental hierarchy, but only to the depth needed to identify and roll up ownership (§12).

### 3.2 The Role of GD of AI

The General Department of Artificial Intelligence is a central delivery function. It does not own the business processes it automates; it owns the capability to automate them. Its role is to develop and manage the projects that digitalise and automate core police business operations on behalf of the other general departments.

This creates a structural asymmetry that underlies every problem in §4:

- **Demand is distributed.** Any general department, and Police HQ, can generate demand on GD of AI.
- **Supply is centralised and finite.** Developers, BAs, Tech Leads and Technical PMs sit in one pool inside GD of AI.
- **No requesting party sees the whole demand picture.** Each business owner sees only their own project. HQ sees priorities, not capacity.

### 3.3 How Projects Originate

Projects reach GD of AI through two routes:

1. **Direction from Police HQ.** Senior leadership orders a general department to initiate a project. These arrive with implicit high priority and, in some cases, with little notice.
2. **Request from a general department.** A department identifies a need and requests support. These are accepted only after a study establishing the genuine need and expected value.

### 3.4 Current Project Initiation Flow

The established process, using the e-Crime digital complaint channels project as a representative example:

| Step | Actor | Activity |
|---|---|---|
| 1 | Police HQ / General Department | Project is ordered or requested; need and value are established |
| 2 | Owning department (e.g. Dept of e-Crime) | Assigns a business-side Project Manager |
| 3 | Owning department | Presents the project to GD of AI as the central development department |
| 4 | GD of AI | Assigns a Technical Project Manager |
| 5 | GD of AI | Assembles the delivery team (BA, Developers, Tech Lead as required) |
| 6 | GD of AI | Conducts business analysis and produces the delivery plan and timeline |
| 7 | GD of AI | Presents the timeline to the business owner, then to HQ |
| 8 | Business owner + HQ | Approve |
| 9 | GD of AI | Execution begins against the approved plan |

**Step 8 establishes the baseline.** The approved timeline and scope at this point are the reference against which all later variance must be measured and explained. The absence of a durable, system-held baseline is the root cause of the reporting problems described in §4.

---

## 4. Problem Statement

### 4.1 Business-Side Problems (Owning General Departments)

Business owners are experienced police professionals, but they are not project managers. This produces a consistent and predictable set of problems:

| Problem | Consequence for GD of AI |
|---|---|
| **Incomplete requirements at the outset** | Estimates and timelines are built on partial information and are invalidated as the real scope emerges |
| **Scope creep** | New requirements arrive continuously during delivery and are absorbed without formal recognition |
| **No concept of a change request** | A new requirement is experienced by the requester as a clarification, not as a change with a cost |
| **No understanding of requirement weight** | All requests are perceived as similar in size regardless of actual effort |
| **Expectation of unlimited capacity** | GD of AI is perceived as able to deliver anything, at any time, on the original date |
| **Friction when timelines move** | Being told that new scope moves the date is received as an excuse rather than as arithmetic |

The common root is that **the cost of a request is invisible at the moment the request is made.** Nothing in the current process converts "please also add X" into "that is 12 developer-days and moves your go-live from 4 March to 22 March." Because the cost is invisible, it is not weighed, and the requester cannot make an informed decision about whether the change is worth its price.

### 4.2 Police HQ Stakeholder Problems

| Problem | Consequence |
|---|---|
| **Status reporting is rebuilt manually for every meeting** | Presentations consume significant preparation time on each request |
| **Reported information is author-dependent** | The same project can be represented differently by different preparers, undermining trust in all reporting |
| **Reporting is a point-in-time snapshot** | Information is already ageing when presented and cannot be interrogated |
| **Running projects are halted for surprise high-priority work** | The displaced project stops, but the expectation of its original delivery date does not |
| **The consequence of a halt is not visible to the decision-maker** | Prioritisation decisions are made without sight of what they cost elsewhere in the portfolio |

The critical point is the last one. Reprioritisation is a legitimate command decision and the system must not obstruct it. But at present the decision is made **without the trade-off being visible**, and the resulting delay to the displaced project is later attributed to GD of AI rather than to the reprioritisation that caused it.

### 4.3 Internal Capacity Problems (GD of AI)

| Problem | Consequence |
|---|---|
| **No consolidated view of who is working on what** | Assignment decisions rely on individual recollection |
| **No visibility of workload distribution** | Load is unevenly spread; some staff are over-committed while others are underused |
| **No forward view of when resources become free** | Cannot answer "when could we realistically start this?" |
| **No portfolio-level effect analysis** | Cannot show what adding a project does to everything already committed |
| **No record of why a date changed** | Variance cannot be explained after the fact, so it defaults to being GD of AI's fault |

### 4.4 Cost of Taking No Action

- Preparation effort for status reporting continues to be consumed on every request, indefinitely.
- Delivery dates continue to slip for reasons that are real but undocumented, progressively eroding the credibility of GD of AI's estimates.
- Prioritisation and scope decisions continue to be made without visibility of their cost, producing outcomes that no party would have chosen with full information.
- Staff over-allocation remains invisible until it appears as missed dates or attrition.
- Every new project increases the coordination load on Technical PMs superlinearly, because there is no system holding the state.

---

## 5. Current State (As-Is)

| Aspect | Current state |
|---|---|
| **Project tracking** | Microsoft Excel workbooks, maintained manually |
| **Resource allocation** | Held informally; no consolidated record of assignments |
| **Effort estimation** | Produced per project during analysis; not retained in a comparable structure |
| **Timeline management** | Dates maintained manually; revisions overwrite prior values |
| **Baseline retention** | None — the originally approved timeline is not durably retained for comparison |
| **Change requests** | No formal capture, costing, or approval record |
| **Hold / halt records** | Not formally recorded; cause of delay is retained only in individuals' memory |
| **Status reporting** | Microsoft PowerPoint, rebuilt by hand for each meeting |
| **Stakeholder access** | None — stakeholders receive prepared presentations only |
| **Audit history** | None |

**Assessment.** The current tooling can record what is planned but cannot explain what changed or why. Excel holds a single mutable view of the present, so each revision destroys the evidence needed to account for it. This is the specific deficiency PRMS must address; simply moving the same data into a better tracker would not resolve it.

---

## 6. Objectives and Success Criteria

| # | Objective | Success criterion | Addresses |
|---|---|---|---|
| **OBJ-01** | Establish a single source of truth for the GD of AI portfolio | All active projects are recorded in PRMS; status presentations are generated from it rather than assembled by hand | §4.2 |
| **OBJ-02** | Make the cost of a new requirement visible at the moment it is requested | Every change request records added effort in man-days and the resulting revised delivery date before acceptance | §4.1 |
| **OBJ-03** | Make the cost of reprioritisation visible to the decision-maker | Every hold records its reason, its duration, the displacing project where applicable, and the revised delivery date of the displaced project | §4.2 |
| **OBJ-04** | Provide an accurate view of departmental capacity and workload | Utilisation is visible per person and per role; over-allocation is flagged as it occurs | §4.3 |
| **OBJ-05** | Make every timeline change attributable | 100% of variance between baseline and current dates is accounted for by a recorded change request or hold record | §4.1, §4.2, §4.3 |
| **OBJ-06** | Eliminate manual preparation of routine status reporting | Stakeholders self-serve current status from the visualization page; no deck is built for routine status meetings | §4.2 |
| **OBJ-07** | Support informed decisions on new demand before commitment | The earliest realistic start date for a proposed project can be determined from current capacity | §4.3 |

---

## 7. Stakeholders and User Roles

### 7.1 System Users (GD of AI)

| Role | System access | Primary use |
|---|---|---|
| **Director / Department management, GD of AI** | Full read; portfolio-level | Portfolio oversight, capacity planning, resourcing decisions, escalation |
| **Technical Project Manager (TPM)** | Full read/write on own projects | Maintains project data, estimates, assignments, change requests and holds. Primary data owner. |
| **Business Analyst (BA)** | Read; write on assigned projects | Requirements and analysis effort; raises change requests arising from analysis |
| **Tech Lead** | Read; write on assigned projects | Technical estimation input; assigned to projects where required |
| **Developer** | Read own assignments | Visibility of own allocation and workload |
| **Resource Manager / Dept coordinator** | Full read/write on resource pool | Maintains the staff roster, roles, availability and leave |

> **Note on the Developer role.** Developers are modelled as a managed resource, not as active system users. Read access to their own assignments is included for transparency but is not a core requirement of the first release.

### 7.2 Read-Only Consumers (External to GD of AI)

| Stakeholder | Access | Interest |
|---|---|---|
| **Police HQ senior leadership** | Read-only visualization page | Portfolio status; consequence of prioritisation decisions; delivery confidence |
| **Business owner — General Department management** | Read-only visualization page | Status of their own projects; effect of their own change requests on their dates |
| **Business-side Project Manager (owning department)** | Read-only visualization page | Day-to-day status of their project; pending change requests and impacts |

These parties **do not hold accounts and do not enter data.** They consume a presentation-oriented view. This is a deliberate scoping decision: it removes access administration, permission modelling and data-integrity risk from the first release, while still delivering the transparency that resolves §4.1 and §4.2.

### 7.3 Stakeholder Interest Summary

| Group | What they currently lack | What PRMS gives them |
|---|---|---|
| HQ leadership | Consistent, current, interrogable portfolio status | A live view identical for every viewer, with causes of variance attached |
| Business owners | Understanding of what their requests cost | The man-day cost and revised date of each request, before they commit to it |
| GD of AI management | Visibility of capacity and its limits | Utilisation, bottleneck roles, and realistic start dates for new demand |
| Technical PMs | A system that holds project state and history | Automated recalculation and a defensible, auditable record of every change |

---

## 8. Scope

### 8.1 In Scope — First Release (MVP)

The first release centres on the **capacity and assignment engine**, because every other capability depends on the data it holds.

1. **Project registry and lifecycle management** — record projects, their business owner, priority, dates and status through the defined lifecycle
2. **Resource pool** — the GD of AI staff roster with roles and availability
3. **Effort estimation** — man-days per role, per project, with a retained approved baseline
4. **Assignment engine** — allocate resources to projects; calculate workload; raise over-allocation warnings
5. **Schedule calculation** — derive duration and delivery dates from effort, assigned resources and a working calendar
6. **Change request management** — capture new requirements, cost them in man-days, calculate and record the resulting date impact
7. **Hold management** — record holds with reason codes, link displaced projects to displacing projects, and recalculate downstream dates automatically
8. **Stakeholder visualization page** — read-only view covering portfolio status, change request impact, and hold impact
9. **Audit history** — a durable record of every status, date, scope and assignment change with cause and timestamp

### 8.2 Out of Scope — First Release

| Excluded | Rationale |
|---|---|
| Task-level or sprint-level work tracking | PRMS operates at portfolio and project level. It is not a replacement for a development tracker. |
| Time sheets / actual hours capture | Effort is planned and estimated, not actually recorded, in this release. Estimate-versus-actual is a later phase (§20). |
| Per-phase resource allocation | Resources are assigned for the whole project duration (RULE-09). Per-phase allocation is a later refinement. |
| Financial and budget management | Cost tracking is not part of the current problem statement. |
| Document management | Requirements documents, designs and deliverables remain in existing repositories. |
| Integration with any external system | Local prototype; no integration surface in this release. |
| User accounts for business owners and HQ | Read-only visualization page only, per §7.2. |
| Authentication, SSO and role-based access control | Local single-user prototype (§13.1). Required for production (§13.2). |
| Arabic language and RTL interface | Required for production (§13.2); not required for the internal prototype. |
| Vendor and contractor resource management | The pool is GD of AI internal staff only in this release. |
| Automated notifications and alerting | Warnings are surfaced in the interface, not pushed. |

### 8.3 Deferred to Later Phases

Recorded here so they are not re-litigated as scope gaps during the first release: estimate-versus-actual tracking to improve estimation accuracy over time; per-phase resource modelling; scenario planning with multiple saved what-if scenarios; skill-based resource matching; vendor capacity; dependency modelling between projects; and integration with existing Dubai Police systems. See §20.

---

## 9. Business Requirements

Business requirements state *what the business needs*, in business language. Each is traceable to an objective and is decomposed into functional requirements in §10.

| ID | Requirement | Objective |
|---|---|---|
| **BR-01** | The department must maintain a single, current, authoritative record of every project in its portfolio. | OBJ-01 |
| **BR-02** | The department must maintain a record of its delivery staff, their roles, and their availability, as a finite capacity pool. | OBJ-04 |
| **BR-03** | Each project must carry an estimate of the effort it requires, expressed in man-days for each role it consumes. | OBJ-02, OBJ-04 |
| **BR-04** | Each project must retain the scope and delivery dates approved at the point of business and HQ sign-off, unchanged, as its baseline. | OBJ-05 |
| **BR-05** | The system must calculate a project's delivery date from its effort estimate, its assigned resources and the working calendar, and must allow a Technical PM to override that calculation with justification. | OBJ-02, OBJ-07 |
| **BR-06** | The department must be able to see the current workload of every member of staff and identify who is over-allocated. | OBJ-04 |
| **BR-07** | The department must be alerted when an assignment breaches an allocation rule, but must retain the ability to proceed. | OBJ-04 |
| **BR-08** | Every new requirement raised after baseline approval must be recorded as a change request, costed in man-days, and its effect on the delivery date calculated and presented before it is accepted. | OBJ-02 |
| **BR-09** | Every suspension of work on a project must be recorded with a reason, a duration, and — where the cause is reprioritisation — a link to the project that displaced it. | OBJ-03 |
| **BR-10** | When a project is held, its delivery date must be recalculated automatically to reflect the suspension. | OBJ-03, OBJ-05 |
| **BR-11** | The difference between a project's baseline dates and its current dates must at all times be fully explained by its recorded change requests and holds. | OBJ-05 |
| **BR-12** | Business owners and HQ stakeholders must be able to view current portfolio status, and the cause and effect of every timeline change, without a report being prepared for them. | OBJ-01, OBJ-06 |
| **BR-13** | The department must be able to determine the earliest date at which a proposed new project could realistically start, given current commitments. | OBJ-07 |
| **BR-14** | Every material change to a project must be recorded in an audit history showing what changed, when, and why. | OBJ-05 |

---

## 10. Functional Requirements

Priority is stated as **M** (must have — first release), **S** (should have — first release if capacity allows), or **C** (could have — first release candidate, otherwise deferred).

### 10.1 Project Registry and Lifecycle — `FR-PRJ`

| ID | Requirement | Pri |
|---|---|---|
| FR-PRJ-01 | The system shall allow a project to be created with: name, reference code, description, owning department, business-side PM name, origin (HQ directive or departmental request), priority, and target start date. | M |
| FR-PRJ-02 | The system shall assign every project exactly one status from: **Analysis, In Progress, On Hold, Delivered, Trial, Launched** (see RULE-01). | M |
| FR-PRJ-03 | The system shall record a project's priority as one of: Critical, High, Medium, Low. | M |
| FR-PRJ-04 | The system shall identify the owning department at its correct organisational level and roll it up to its parent general department for reporting. | M |
| FR-PRJ-05 | The system shall record, for every project, its baseline start date, baseline delivery date, current planned start date and current planned delivery date. | M |
| FR-PRJ-06 | The system shall record every status transition with the date, the user, and — where applicable — the reason. | M |
| FR-PRJ-07 | The system shall calculate and display, for every project, the variance in working days between baseline and current delivery date. | M |
| FR-PRJ-08 | The system shall permit a project to be cancelled, retaining its record and history, and release its assigned resources. | S |
| FR-PRJ-09 | The system shall flag a project whose data has not been updated within a configurable period (default 14 days) as stale. | S |
| FR-PRJ-10 | The system shall record a free-text current-status narrative per project, for stakeholder reporting. | S |

### 10.2 Resource Pool — `FR-RES`

| ID | Requirement | Pri |
|---|---|---|
| FR-RES-01 | The system shall maintain a roster of GD of AI delivery staff with: name, employee reference, primary role, and active/inactive state. | M |
| FR-RES-02 | The system shall support the roles: **Technical Project Manager, Business Analyst, Tech Lead, Developer**. | M |
| FR-RES-03 | The system shall record an availability factor per person (default 80%), representing the proportion of working time available for project delivery after meetings, support and administration. | M |
| FR-RES-04 | The system shall support recording planned absence (leave, training, secondment) as date ranges that reduce available capacity in that period. | S |
| FR-RES-05 | The system shall support a person holding a secondary role, allowing them to be assigned in either capacity. | C |
| FR-RES-06 | The system shall prevent deletion of a person who holds historical assignments, permitting deactivation instead. | M |

### 10.3 Effort Estimation — `FR-EST`

| ID | Requirement | Pri |
|---|---|---|
| FR-EST-01 | The system shall record, per project, an estimated effort in man-days for each role the project consumes. | M |
| FR-EST-02 | The system shall calculate a project's total weight as the sum of estimated man-days across all roles. | M |
| FR-EST-03 | The system shall retain the effort estimate approved at baseline, separately from and unaffected by later revisions. | M |
| FR-EST-04 | The system shall maintain a current effort estimate equal to the baseline estimate plus the effort of all approved change requests. | M |
| FR-EST-05 | The system shall record an optional confidence level per estimate (High, Medium, Low) to indicate estimation reliability. | S |
| FR-EST-06 | The system shall retain a revision history of effort estimates showing what changed, when, and why. | M |

### 10.4 Assignment Engine and Allocation Rules — `FR-ASG`

| ID | Requirement | Pri |
|---|---|---|
| FR-ASG-01 | The system shall allow one or more people to be assigned to a project, each with a role and an allocation percentage (default 100%). | M |
| FR-ASG-02 | The system shall assign a resource for the whole duration of the project, from its planned start date to its planned delivery date (RULE-09). | M |
| FR-ASG-03 | The system shall allow multiple resources of the same role to be assigned to one project. | M |
| FR-ASG-04 | The system shall raise a **warning, not a block**, when a Developer is assigned to more than one project in In Progress status over overlapping dates (RULE-06). | M |
| FR-ASG-05 | The system shall permit a Technical PM to proceed past any allocation warning, recording an acknowledgement with the user and timestamp. | M |
| FR-ASG-06 | The system shall permit Technical Project Managers, Business Analysts and Tech Leads to be assigned to multiple concurrent projects without warning, subject to FR-ASG-07. | M |
| FR-ASG-07 | The system shall raise a warning when any person's total allocation across concurrent projects exceeds 100% of their available capacity (RULE-07). | M |
| FR-ASG-08 | The system shall calculate and display each person's current utilisation as committed allocation against available capacity. | M |
| FR-ASG-09 | The system shall display a capacity view showing, per person and per role, current utilisation and the date on which each becomes available. | M |
| FR-ASG-10 | The system shall release a project's assignments when the project reaches Launched or is cancelled. | M |
| FR-ASG-11 | The system shall release or retain assignments when a project enters On Hold, according to the resource disposition of the hold reason (RULE-12). | M |
| FR-ASG-12 | The system shall identify, for a proposed project with a stated effort estimate, the earliest date at which sufficient capacity exists to start it. | S |
| FR-ASG-13 | The system shall retain a history of all assignments, including ended ones. | M |

### 10.5 Schedule Calculation — `FR-SCH`

| ID | Requirement | Pri |
|---|---|---|
| FR-SCH-01 | The system shall calculate a project's duration in working days from its current effort estimate and its assigned resources (RULE-02). | M |
| FR-SCH-02 | The system shall calculate a project's planned delivery date as its planned start date advanced by its calculated duration across working days only (RULE-03). | M |
| FR-SCH-03 | The system shall maintain a configurable working calendar defining the working week and non-working public holidays (RULE-04). | M |
| FR-SCH-04 | The system shall allow a Technical PM to override a calculated duration or delivery date, requiring a justification, and shall record the override and its reason (RULE-05). | M |
| FR-SCH-05 | The system shall visually distinguish a calculated date from a manually overridden one. | S |
| FR-SCH-06 | The system shall recalculate a project's delivery date whenever its effort estimate, assigned resources, or hold state changes. | M |

### 10.6 Change Request Management — `FR-CHG`

| ID | Requirement | Pri |
|---|---|---|
| FR-CHG-01 | The system shall allow a change request to be raised against a project with: title, description, requester name, requesting department, and date raised. | M |
| FR-CHG-02 | The system shall require an effort estimate in man-days per role for every change request before it can be assessed. | M |
| FR-CHG-03 | The system shall calculate, for a change request under assessment, the resulting revised delivery date and the delta in working days from the current delivery date. | M |
| FR-CHG-04 | The system shall present the calculated impact — added man-days, delta in working days, current date and revised date — as the primary output of the change request record. | M |
| FR-CHG-05 | The system shall support change request states: **Raised, Under Assessment, Presented to Business, Approved, Rejected, Withdrawn**. | M |
| FR-CHG-06 | The system shall apply a change request's effort to the project's current estimate and commit the revised delivery date only when the change request is Approved. | M |
| FR-CHG-07 | The system shall retain rejected and withdrawn change requests, with their assessed impact, as a record of scope that was considered and declined. | M |
| FR-CHG-08 | The system shall record the date on which a change request's impact was presented to the business owner, and the date of their decision. | M |
| FR-CHG-09 | The system shall display, per project, the cumulative effect of all approved change requests on the delivery date. | M |
| FR-CHG-10 | The system shall allow a change request's impact to be viewed without committing it, so it can be presented before a decision is taken. | M |

### 10.7 Hold Management — `FR-HLD`

| ID | Requirement | Pri |
|---|---|---|
| FR-HLD-01 | The system shall allow a project to be placed On Hold with a mandatory reason selected from: **Resources pulled to higher-priority project; Awaiting business owner input; Awaiting approval / budget; Blocked by external or technical dependency**. | M |
| FR-HLD-02 | The system shall require, where the reason is *Resources pulled to higher-priority project*, a link to the project that caused the displacement. | M |
| FR-HLD-03 | The system shall record, for every hold: start date, expected resume date where known, actual resume date, reason, linked displacing project where applicable, the person who recorded it, and the authority who directed it. | M |
| FR-HLD-04 | The system shall record a free-text note against every hold. | M |
| FR-HLD-05 | The system shall recalculate a held project's delivery date on resume, advancing it by the number of working days the project was held (RULE-10). | M |
| FR-HLD-06 | The system shall project a provisional revised delivery date during an open hold, based on the expected resume date where one is recorded. | M |
| FR-HLD-07 | The system shall release the project's assignments to the available pool where the hold reason has a resource disposition of *released*, and retain them where it is *retained* (RULE-12). | M |
| FR-HLD-08 | The system shall support multiple sequential holds on one project, each retained individually in history. | M |
| FR-HLD-09 | The system shall display, for a project that displaced others, the list of projects it displaced and the total delay it caused. | M |
| FR-HLD-10 | The system shall return a project to its prior status on resume. | M |

### 10.8 Stakeholder Visualization Page — `FR-VIZ`

The visualization page is read-only, requires no account, and is designed to be shown on screen in a meeting without preparation.

| ID | Requirement | Pri |
|---|---|---|
| FR-VIZ-01 | The system shall provide a read-only view of the portfolio requiring no data entry and no login. | M |
| FR-VIZ-02 | The system shall display a portfolio overview: all projects with owning department, status, priority, current delivery date, and variance from baseline. | M |
| FR-VIZ-03 | The system shall display a timeline (Gantt-style) view of the portfolio showing each project's baseline bar and current bar, so slippage is visible as a comparison. | M |
| FR-VIZ-04 | The system shall display, per project, a variance breakdown attributing the total slippage to its component causes — each approved change request and each hold, with its contribution in working days. | M |
| FR-VIZ-05 | The system shall display a change request impact view showing the requested change, its man-day cost, the current delivery date and the revised delivery date. | M |
| FR-VIZ-06 | The system shall display a hold impact view showing which project was displaced, by which project, for how long, and the resulting revised delivery date. | M |
| FR-VIZ-07 | The system shall display a capacity summary showing departmental utilisation by role and identifying bottleneck roles. | M |
| FR-VIZ-08 | The system shall allow the portfolio view to be filtered by owning general department, status and priority. | M |
| FR-VIZ-09 | The system shall present a single-project view suitable for a business owner reviewing only their own project. | M |
| FR-VIZ-10 | The system shall display the date and time at which the data shown was last updated. | M |
| FR-VIZ-11 | The system shall present all variance information in neutral, factual language, stating cause without attributing fault (see RISK-03). | M |
| FR-VIZ-12 | The system shall support export of the current view to a static format suitable for distribution where a live view cannot be shown. | S |

### 10.9 Reporting and Audit — `FR-RPT`

| ID | Requirement | Pri |
|---|---|---|
| FR-RPT-01 | The system shall maintain an audit history recording every change to project status, dates, effort estimate and assignments, with timestamp, user and reason. | M |
| FR-RPT-02 | The system shall provide a per-project history showing the full sequence of events from baseline to current state. | M |
| FR-RPT-03 | The system shall report portfolio-level totals: active projects, total committed man-days, utilisation by role, projects on hold, and aggregate slippage. | M |
| FR-RPT-04 | The system shall report the number of change requests and total man-days added per owning department, over a selected period. | S |
| FR-RPT-05 | The system shall report total working days lost to holds, broken down by hold reason. | S |
| FR-RPT-06 | The system shall reconcile, per project, baseline delivery date plus all recorded impacts against current delivery date, and flag any unexplained residual variance (RULE-11). | M |

---

## 11. Business Rules

These rules govern system behaviour and must be implemented exactly as stated.

### RULE-01 — Project Lifecycle

A project occupies exactly one status at a time, from this set:

| Status | Meaning | Resources committed |
|---|---|---|
| **Analysis** | Requirements and analysis work in progress; estimate being produced | Yes — BA, TPM, and Tech Lead where assigned |
| **In Progress** | Approved and under active delivery | Yes — full assigned team |
| **On Hold** | Work suspended; requires a reason (RULE-12) | Per hold reason disposition |
| **Delivered** | Development complete; handed to the business owner | Partially — support-level only |
| **Trial** | Live pilot with a limited user group, prior to full rollout | Partially — support and fixes |
| **Launched** | Full production rollout across the owning department; project closed | No — released |

Permitted transitions:

- Analysis → In Progress, On Hold
- In Progress → On Hold, Delivered
- On Hold → the status held from (RULE-10)
- Delivered → Trial, In Progress *(where trial or acceptance issues require further development)*
- Trial → Launched, In Progress *(where the pilot identifies required changes)*
- Launched → *(terminal)*

Any project may be cancelled from any non-terminal status (FR-PRJ-08).

### RULE-02 — Duration Calculation

For each role consumed by a project:

```
role_capacity_per_day = Σ (assignment.allocation_percentage × resource.availability_factor)
                        for all resources assigned in that role

role_duration_days    = role_effort_man_days ÷ role_capacity_per_day
```

The project's calculated duration is the **maximum** of its role durations, since roles run concurrently across the whole project (RULE-09).

**Limitation, explicitly accepted:** this model does not represent diminishing returns from adding people to a project. Doubling the developers on a project halves its calculated duration, which is not true in practice. This is a deliberate simplification for the first release and is mitigated by RULE-05, which lets the Technical PM override the calculated figure with their professional judgement. See ASM-06.

### RULE-03 — Date Calculation

Delivery dates are calculated across **working days only**, per the working calendar (RULE-04):

```
planned_delivery_date = advance(planned_start_date, ceiling(calculated_duration_days))
```

Where `advance` counts only working days and skips weekends and public holidays.

### RULE-04 — Working Calendar

The working calendar is configurable and defines the working week and the list of non-working public holidays. The default working week is Monday to Friday, with Saturday and Sunday non-working. UAE public holidays are maintained as a configurable list. See ASM-03 — this default must be confirmed against GD of AI's actual working pattern.

### RULE-05 — Manual Override of Calculated Dates

A Technical PM may override a calculated duration or delivery date. An override:

- requires a written justification;
- is recorded with the user and timestamp;
- is displayed distinctly from a calculated value (FR-SCH-05);
- persists until explicitly cleared, and is not silently replaced by recalculation.

Where an override is in force, subsequent impacts from change requests and holds are applied **relative to the overridden date**, not to the calculated one.

### RULE-06 — Developer Exclusivity (Soft)

A Developer is expected to work on one project at a time. Where a Developer is assigned to more than one project in **In Progress** status with overlapping dates, the system raises an over-allocation warning.

**This is a warning, never a block.** The assignment proceeds if the Technical PM acknowledges it (FR-ASG-05). The acknowledgement is recorded.

### RULE-07 — Aggregate Over-Allocation (All Roles)

For any person, the sum of allocation percentages across all concurrently assigned projects, adjusted by their availability factor, must not exceed 100%. Where it does, the system raises an over-allocation warning. This applies to every role, including those exempt from RULE-06.

As with RULE-06, this is a warning and is overridable with a recorded acknowledgement.

### RULE-08 — Multi-Project Roles

Technical Project Managers, Business Analysts and Tech Leads may be assigned to multiple concurrent projects without triggering RULE-06. They remain subject to RULE-07.

### RULE-09 — Whole-Project Assignment

A resource assigned to a project is committed for the project's entire duration, from planned start to planned delivery date. The system does not model per-phase resource demand in this release.

**Known consequence:** this overstates commitment. A Developer assigned to a project in Analysis status is shown as committed although no development work has begun. This is accepted for the first release as the simpler and more conservative model; per-phase allocation is deferred (§8.3). See ASM-05.

### RULE-10 — Hold Recalculation

On resume:

```
held_working_days     = working_days_between(hold.start_date, hold.actual_resume_date)
new_delivery_date     = advance(current_delivery_date, held_working_days)
```

The project returns to the status it held immediately before the hold. A hold record contributes its `held_working_days` to the project's variance breakdown (FR-VIZ-04).

While a hold is open, the system projects a provisional revised delivery date using the expected resume date where one has been recorded (FR-HLD-06). This figure is presented as provisional.

### RULE-11 — Variance Reconciliation

For every project, at all times:

```
baseline_delivery_date
  + Σ (working-day impact of every approved change request)
  + Σ (held working days of every hold)
  + Σ (working-day effect of every recorded manual override)
  = current_delivery_date
```

Any residual difference is **unexplained variance** and must be flagged (FR-RPT-06). This rule is the mechanism that delivers OBJ-05: it makes it structurally impossible for a date to move without a recorded cause.

### RULE-12 — Hold Reasons and Resource Disposition

| Reason | Displacing project link | Resource disposition | Attribution |
|---|---|---|---|
| **Resources pulled to higher-priority project** | **Required** | Released to pool | Reprioritisation decision |
| **Awaiting business owner input** | Not applicable | Retained | Business owner |
| **Awaiting approval / budget** | Not applicable | Released to pool | Approval authority |
| **Blocked by external or technical dependency** | Not applicable | Retained | External dependency |

*Resource disposition* determines whether the project's assignments are freed for other work during the hold (FR-ASG-11, FR-HLD-07). Where resources are released, they become available capacity immediately; where retained, they remain committed and unavailable.

*Attribution* records where the cause of the delay originated. It is used for factual reporting only and is presented neutrally (FR-VIZ-11).

### RULE-13 — Baseline Establishment and Re-Baselining

A project's baseline is established when the timeline is approved by the business owner and HQ (§3.4, step 8). At that moment the current effort estimate and dates are copied to the baseline fields and become immutable.

A project may be **re-baselined** only by explicit action, requiring a justification. Re-baselining archives the previous baseline together with all variance accumulated against it, and starts a new baseline period. This exists so that a project which has changed beyond recognition does not carry meaningless variance forever — but it must be a deliberate, visible act, not a means of erasing history.

### RULE-14 — Change Request Impact Application

A change request's effort is applied to the project's current estimate, and its date impact committed, **only on approval** (FR-CHG-06). Prior to approval the impact is calculated and displayed but not applied, so it can be presented to the business owner as a projection (FR-CHG-10).

---

## 12. Conceptual Data Model

### 12.1 Entities

| Entity | Purpose | Key attributes |
|---|---|---|
| **Department** | Organisational unit that owns projects | Name, level (General Department / Sub-department / Section / Sector), parent department |
| **Project** | A unit of delivery work | Name, reference code, description, owning department, business PM name, origin, priority, status, baseline start/delivery dates, current planned start/delivery dates, status narrative, last updated |
| **Resource** | A member of GD of AI delivery staff | Name, employee reference, primary role, availability factor, active state |
| **Role** | A capacity type | Name (TPM, BA, Tech Lead, Developer), multi-project permitted flag |
| **EffortEstimate** | Estimated effort for a project in one role | Project, role, baseline man-days, current man-days, confidence |
| **Assignment** | Commitment of a resource to a project | Project, resource, role, allocation percentage, start date, end date, active state, override acknowledgement |
| **ChangeRequest** | A requirement raised after baseline | Project, title, description, requester, requesting department, date raised, state, effort per role, calculated day impact, date presented, date decided |
| **HoldRecord** | A suspension of work | Project, reason, displacing project, start date, expected resume date, actual resume date, held working days, resource disposition, directing authority, note |
| **StatusHistory** | Record of a status transition | Project, from status, to status, date, user, reason |
| **AuditEntry** | Record of any material change | Entity type, entity reference, field, previous value, new value, timestamp, user, reason |
| **WorkingCalendar** | Definition of working time | Working days of week, public holiday dates |
| **Absence** | Planned unavailability of a resource | Resource, type, start date, end date |

### 12.2 Relationships

```
Department ──(parent)──> Department                    self-referencing hierarchy
Department ──(owns)────> Project                       one department owns many projects

Project ───(estimated by)──> EffortEstimate ──(for)──> Role
Project ───(staffed by)────> Assignment     ──(of)───> Resource ──(has)──> Role
Project ───(changed by)────> ChangeRequest  ──(adds effort per)──> Role
Project ───(suspended by)──> HoldRecord
HoldRecord ──(displaced by)──> Project                 the displacing project
Project ───(tracked by)────> StatusHistory

Resource ──(unavailable during)──> Absence
```

### 12.3 Derived Values

These are calculated, never stored as independent editable values:

| Value | Derivation |
|---|---|
| Project total weight | Σ current man-days across all roles (FR-EST-02) |
| Project calculated duration | RULE-02 |
| Project planned delivery date | RULE-03, unless overridden per RULE-05 |
| Delivery variance | Current delivery date − baseline delivery date, in working days |
| Variance breakdown | Σ change request impacts + Σ hold impacts + Σ override effects (RULE-11) |
| Resource utilisation | Σ allocation percentages of active assignments ÷ availability factor |
| Role capacity | Σ availability across all active resources in that role |
| Earliest available start | First date on which unallocated capacity meets a proposed project's role requirements (FR-ASG-12) |

---

## 13. Non-Functional Requirements

### 13.1 First Release — Local Prototype

| ID | Requirement |
|---|---|
| NFR-01 | The system shall run entirely on a single local machine, with no external service dependency. |
| NFR-02 | The system shall persist data locally and retain it across restarts. |
| NFR-03 | The system shall be startable by a single documented command. |
| NFR-04 | The system shall support the stated scale without perceptible degradation: 60 concurrent active projects, 80 staff, and three years of accumulated history. |
| NFR-05 | Any portfolio view shall render within two seconds at the stated scale. |
| NFR-06 | The visualization page shall be legible when projected in a meeting room — adequate type size, sufficient contrast, and no reliance on fine detail or hover interaction to convey primary information. |
| NFR-07 | The system shall support seeding with representative sample data for demonstration purposes. |
| NFR-08 | The system shall permit the complete data set to be exported and re-imported, so a demonstration state can be preserved and restored. |
| NFR-09 | The interface shall be in English for the first release. |
| NFR-10 | Data entry for a routine project update shall require no more than a small number of fields, to keep the maintenance burden low enough to be sustained (see RISK-01). |

### 13.2 Production Readiness — Later Phase

Recorded here so the gap between the prototype and a deployable system is explicit and not discovered late. **None of these are in scope for the first release.**

| ID | Requirement |
|---|---|
| NFR-P-01 | Authentication integrated with Dubai Police Active Directory / single sign-on. |
| NFR-P-02 | Role-based access control aligned to §7.1, restricting write access to a user's own projects. |
| NFR-P-03 | Controlled access for external stakeholders to the visualization page, replacing open read access. |
| NFR-P-04 | Compliance with Dubai Police information security and data classification policy. |
| NFR-P-05 | Deployment within the approved Dubai Police hosting environment. |
| NFR-P-06 | Backup and recovery to the standard required for a departmental system of record. |
| NFR-P-07 | Full Arabic language support including right-to-left interface layout. |
| NFR-P-08 | Availability and support commitments appropriate to a system relied upon for HQ reporting. |
| NFR-P-09 | Tamper-evident audit log meeting the department's record-keeping requirements. |

---

## 14. Assumptions

| ID | Assumption | If wrong |
|---|---|---|
| ASM-01 | Technical Project Managers will keep project data current; the system's output is only as good as its input. | The system's reporting becomes untrustworthy and reverts to being a parallel Excel. See RISK-01. |
| ASM-02 | Effort estimates in man-days can be produced with sufficient consistency across projects to make capacity arithmetic meaningful. | Capacity figures mislead. Mitigated by estimate confidence levels (FR-EST-05) and, later, estimate-versus-actual tracking (§20). |
| ASM-03 | The default working week is Monday to Friday. This must be confirmed against GD of AI's actual working pattern before build. | Every calculated date is wrong. The calendar is configurable (RULE-04), so this is correctable, but it must be set correctly from the outset. |
| ASM-04 | A default availability factor of 80% reasonably represents productive project time after meetings, support and administration. | Durations are systematically optimistic or pessimistic. The factor is configurable per person (FR-RES-03). |
| ASM-05 | Whole-project resource assignment (RULE-09) is acceptable for the first release, despite overstating commitment during analysis phases. | Capacity appears more constrained than it is, and the department under-commits. Per-phase allocation is the deferred remedy (§8.3). |
| ASM-06 | Linear duration scaling with headcount (RULE-02) is acceptable given the manual override in RULE-05. | Calculated durations for larger teams are optimistic. Technical PM override is the primary control. |
| ASM-07 | Business owners and HQ stakeholders will engage with a self-service visualization view rather than continuing to request prepared presentations. | OBJ-06 is not met even if the system works. Mitigated by making the view directly presentable in meetings (NFR-06). |
| ASM-08 | Making the cause of delay visible is organisationally acceptable and will be received as factual reporting. | The system is resisted for political rather than functional reasons. See RISK-03 — this is the most significant non-technical risk. |
| ASM-09 | The first release is a prototype for internal demonstration and will not hold operational data requiring formal security classification. | Production security requirements (§13.2) become immediate rather than deferred. |
| ASM-10 | Projects are sufficiently independent that inter-project dependencies need not be modelled in the first release. | Delivery dates ignore real sequencing constraints. Dependency modelling is deferred (§20). |

---

## 15. Constraints

| ID | Constraint |
|---|---|
| CON-01 | The first release must run locally on a single machine, without departmental infrastructure provisioning. |
| CON-02 | No integration with existing Dubai Police systems is available for the first release. |
| CON-03 | GD of AI delivery capacity is finite and already committed; effort spent building PRMS competes with the project portfolio it manages. This argues strongly for a tightly scoped first release. |
| CON-04 | Business owners and HQ stakeholders will not enter data into the system under any circumstances; all input is from GD of AI staff. |
| CON-05 | Any production deployment will require Dubai Police information security review and approval. |
| CON-06 | The system must accommodate reprioritisation by HQ as a legitimate and frequent event. It must record and cost such decisions; it must never obstruct them. |

---

## 16. Risks and Mitigations

| ID | Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| **RISK-01** | **Data currency.** Technical PMs do not keep project data up to date, and the system's output becomes untrustworthy. This is the single most common failure mode for systems of this type. | High | High | Keep required fields to a minimum (NFR-10); assign clear data ownership to the TPM; display staleness indicators (FR-PRJ-09) so out-of-date projects are visibly flagged rather than silently wrong; establish a weekly update cadence; ensure TPMs get direct value from the system so maintaining it is self-interested rather than administrative. |
| **RISK-02** | **Estimation quality.** Poor man-day estimates produce confident-looking but wrong dates, which is worse than no dates. | High | Medium | Record estimate confidence (FR-EST-05); present calculated dates as calculated rather than committed; retain estimate history (FR-EST-06); introduce estimate-versus-actual tracking in a later phase to improve calibration over time. |
| **RISK-03** | **Political sensitivity of attribution.** The system makes visible that a delay was caused by a specific reprioritisation decision or a specific business owner's change requests. This is the system's core purpose, but it can be perceived as blame. | High | Medium | Present all variance in neutral, factual language (FR-VIZ-11); frame the visualization around forward-looking decision support — *what will this cost* — rather than retrospective fault-finding; position attribution as protecting the decision-maker's ability to decide with full information; secure the Director's sponsorship for this framing before wider rollout. |
| **RISK-04** | **Adoption failure.** The system becomes a second place to record what is already in Excel, and is abandoned. | High | Medium | Make PRMS the sole source for status reporting so the alternative disappears; ensure the first release delivers immediate visible value to the TPMs who maintain it; deliberately restrict first-release scope so it is usable quickly. |
| **RISK-05** | **Scope expansion of PRMS itself.** The system acquires task tracking, time sheets, budgets and document management, and is never completed — an ironic instance of the problem it exists to solve. | Medium | Medium | Hold the boundaries in §8.2 firmly; treat additions as change requests against this BRD; retain §8.3 as the register for deferred requests so they are visibly recorded rather than argued. |
| **RISK-06** | **Model oversimplification.** Whole-project allocation (RULE-09) and linear duration scaling (RULE-02) produce figures that experienced PMs recognise as wrong, undermining confidence in the whole system. | Medium | Medium | Manual override with recorded justification (RULE-05); state the limitations openly in the interface rather than presenting calculated figures as authoritative; treat per-phase allocation as the first enhancement after the first release. |
| **RISK-07** | **Prototype-to-production gap.** A local prototype is approved by the Director and immediately expected in production, where §13.2 requirements apply. | Medium | High | Document the production gap explicitly (§13.2); present the prototype as a validation of the model, not as a deployable system; include the production readiness effort in any subsequent proposal. |
| **RISK-08** | **Single point of knowledge.** The system depends on one person who understands both the model and the build. | Medium | Medium | This BRD; documented business rules in §11 sufficient for another team to implement; avoid undocumented behaviour. |

---

## 17. Success Metrics

| ID | Metric | Baseline | Target |
|---|---|---|---|
| **KPI-01** | Preparation time for routine HQ status reporting | Manual deck rebuilt per meeting | Zero — status presented live from PRMS |
| **KPI-02** | Proportion of active portfolio recorded in PRMS | 0% | 100% of active projects |
| **KPI-03** | Proportion of delivery-date variance with a recorded, attributable cause | Not measurable | 100% (enforced by RULE-11) |
| **KPI-04** | Proportion of post-baseline requirements captured as formal change requests | Effectively 0% | 100% |
| **KPI-05** | Proportion of change requests whose impact was presented to the business owner before acceptance | Not measured | 100% |
| **KPI-06** | Proportion of holds recorded with a reason and, where applicable, a displacing project | Not recorded | 100% |
| **KPI-07** | Over-allocation incidents visible at the time of assignment rather than discovered later | 0% | 100% flagged at assignment |
| **KPI-08** | Consistency of reported status between consecutive stakeholder meetings | Author-dependent | Single source; no discrepancy |
| **KPI-09** | Projects delivered within their current (re-calculated) delivery date | Not measured | Measured from first release; improvement target set after one reporting period |

> KPI-09 is deliberately measured against the *current* date rather than the baseline. Measuring against a baseline that legitimate change requests and directed holds have moved would penalise GD of AI for decisions taken elsewhere — which is precisely the distortion this system exists to correct.

---

## 18. Glossary

| Term | Definition |
|---|---|
| **Baseline** | The scope, effort estimate and delivery dates approved by the business owner and HQ at the point of sign-off. Immutable thereafter except by explicit re-baselining (RULE-13). |
| **BA** | Business Analyst |
| **Business owner** | The general department, sub-department or section that owns the business process being digitalised and on whose behalf the project is delivered. |
| **Change request** | A requirement raised after the baseline is approved, requiring effort estimation, impact calculation and a decision before it is accepted. |
| **Displacing project** | A project whose initiation or prioritisation caused resources to be withdrawn from another project, placing it on hold. |
| **GD of AI** | General Department of Artificial Intelligence — the central delivery department for digitalisation across Dubai Police. |
| **General Department** | A top-level organisational unit of Dubai Police (e.g. CID, Anti-Narcotics, Forensics). |
| **Higher-ups / HQ** | Police Headquarters senior leadership, who direct priorities and receive portfolio reporting. |
| **Hold** | A recorded suspension of work on a project, with a mandatory reason (RULE-12). |
| **Launched** | Full production rollout across the owning department; the project's terminal status. |
| **Man-day** | One person working for one working day. The unit of effort estimation throughout this system. |
| **Over-allocation** | A resource committed beyond their available capacity (RULE-07), or a Developer committed to more than one active project (RULE-06). |
| **PRMS** | Portfolio & Resource Management System — the working name for the system defined by this document. |
| **Re-baselining** | Explicitly replacing a project's baseline, archiving the prior baseline and its accumulated variance (RULE-13). |
| **Scope creep** | The accumulation of additional requirements after baseline approval without formal recognition of their cost. |
| **Trial** | A live pilot of a delivered solution with a limited user group, preceding full rollout. |
| **TPM** | Technical Project Manager — the GD of AI role accountable for delivery and for maintaining the project's record in PRMS. |
| **Variance** | The difference in working days between a project's baseline delivery date and its current delivery date. |
| **Weight** | A project's total estimated effort, being the sum of its man-days across all roles. |

---

## 19. Open Questions

| ID | Question | Why it matters | Needed by |
|---|---|---|---|
| **OQ-01** | What should the system be formally named? "PRMS" is a working title only. | Naming affects how the system is positioned to the Director and to stakeholders. | Before demonstration |
| **OQ-02** | What is GD of AI's actual working week and public holiday schedule? | Directly determines every calculated date (RULE-04, ASM-03). | Before build |
| **OQ-03** | Is 80% a realistic default availability factor for GD of AI delivery staff? | Systematically shifts all calculated durations (ASM-04). | Before build |
| **OQ-04** | Is the baseline established at HQ approval (§3.4 step 8), or at business owner approval, where these differ? | Determines the reference point for all variance reporting (RULE-13). | Before build |
| **OQ-05** | Who is authorised to approve a change request on the business side, and is HQ approval required for changes above a certain size? | Determines whether the change request workflow needs an approval hierarchy beyond FR-CHG-05. | Before build |
| **OQ-06** | Should the read-only visualization page be reachable by stakeholders directly, or presented on screen by GD of AI in meetings? | Affects hosting and access approach even for the prototype (FR-VIZ-01, NFR-01). | Before build |
| **OQ-07** | Are Technical PMs and BAs also subject to a practical limit on concurrent projects, beyond the aggregate rule (RULE-07)? | May require a per-role concurrent project ceiling in addition to percentage-based allocation. | Before build |
| **OQ-08** | Are there existing Dubai Police project governance standards or templates that PRMS status reporting should conform to? | Alignment improves acceptance at HQ and may constrain the visualization design. | Before demonstration |

---

## 20. Appendix — Phased Roadmap

### Phase 1 — Prototype (this BRD, first release)

Local, single-machine system delivering the capacity and assignment engine, change request and hold management with automatic date recalculation, and the read-only stakeholder visualization page. Seeded with representative data. Purpose: **validate the model and demonstrate it to the Director of GD of AI.**

### Phase 2 — Departmental Pilot

Deploy within GD of AI with real portfolio data. Add authentication and role-based access (NFR-P-01, NFR-P-02). Run a full reporting cycle against real HQ meetings. Purpose: **prove the model against reality and establish the data discipline of RISK-01.**

### Phase 3 — Production

Full production readiness per §13.2: security review, approved hosting, Arabic and RTL support, backup and recovery, controlled stakeholder access. Purpose: **make PRMS the department's system of record.**

### Phase 4 — Model Refinement

Enhancements deliberately deferred from Phase 1, in expected order of value:

1. **Per-phase resource allocation** — resolves the overstatement in RULE-09 and materially improves capacity accuracy
2. **Estimate-versus-actual tracking** — closes the loop on RISK-02 and improves estimation over time
3. **Scenario planning** — multiple saved what-if scenarios, allowing options to be compared before a prioritisation decision is taken
4. **Inter-project dependency modelling** — resolves ASM-10
5. **Skill-based resource matching** — beyond role-based assignment
6. **Vendor and contractor capacity** — extends the resource pool beyond GD of AI staff
7. **Integration with existing Dubai Police systems** — removes duplicate data entry

---

*End of document.*
