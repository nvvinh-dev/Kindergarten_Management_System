# User Roles & Permissions — Kindergarten Management System

## 0. Document Information

- **Source of truth:** `CLAUDE.md` §2–§3 (User Roles, Main Functional Requirements) and `docs/requirements/functional-requirements.md`. No role or permission below goes beyond what those two documents state.
- **Status:** Draft — for review and approval before Database Design and API Design proceed.
- **Out of scope for this document:** database roles/tables, API-level permissions, JWT/session claims, authorization middleware, or any other implementation mechanism. This document describes *business-level* roles and responsibilities only; how they are technically enforced is a later, separate decision.
- **Convention used in this document:**
  - **Allowed Actions** use the FR-mapped requirement's own wording. An action level (**View / Create / Update / Delete / Manage**) is assigned only when the functional requirement supports it.
  - **"Manage"** is used only where the functional requirement itself describes the activity as "quản lý" (manage) — it is recorded as a single action level, not expanded into Create + Update + Delete, because the FR text does not break that down. The exact CRUD scope of each "Manage" permission is listed under Open Questions.
  - This document does not resolve any Open Question already recorded in `functional-requirements.md`; where relevant, those are referenced, not repeated in full.

---

## 1. Roles Overview

| Role ID | Role Name | Short Description |
|---|---|---|
| ROLE-ADMIN | Admin / Ban giám hiệu | Manages users, permissions, class assignment by age, and views school-wide dashboards/statistics. |
| ROLE-TEACHER | Giáo viên | Takes daily attendance and pickup attendance, records quick health status and incidents, shares activity photos. |
| ROLE-ACCOUNTING | Kế toán / Văn phòng | Manages tuition, invoices, weekly menu, child enrollment records, teacher HR records, and revenue reporting. |
| ROLE-MEDICAL | Y tế | Tracks children's height and weight, and reviews health/incident history within the specified scope. |
| ROLE-PARENT | Phụ huynh | Views their child's attendance history, activity media, notifications, and invoices; pays tuition online; manages registered pickup persons. |

Each role may only access functionality appropriate to its own permissions (CLAUDE.md §2). No role's permissions may be expanded beyond what is specified without approval.

---

## 2. Role: Admin / Ban giám hiệu (ROLE-ADMIN)

**Description:** School leadership/administration role responsible for user and permission management, organizing children into classes by age, and overseeing the school through dashboard statistics.

**Main responsibilities (per CLAUDE.md §3):**
- Manage user accounts.
- Manage permissions.
- Classify/organize children into classes by age group.
- View a general dashboard and statistics (children count, attendance, revenue).

**Allowed functional areas:** User & Permission Management, Class Management, Admin Dashboard & Statistics.

**Allowed actions:**

| FR ID | Requirement | Action Level | Notes |
|---|---|---|---|
| FR-USER-01 | Manage User Accounts | Manage | Exact lifecycle actions (e.g., deactivate vs. delete) not specified — see FR doc Open Question. |
| FR-USER-02 | Manage Roles / Permissions | Manage | Granularity (role-only vs. custom per-user) not specified — see FR doc Open Question. |
| FR-CLASS-01 | Manage Classes by Age Group | Manage | Whether this includes assigning teachers to classes is not specified — see FR doc Open Question. |
| FR-DASH-01 | Overview Dashboard | View | |
| FR-DASH-02 | Child Count Statistics | View | |
| FR-DASH-03 | Attendance Statistics | View | |
| FR-DASH-04 | Revenue Statistics | View | |

**Not supported by the specification:** Direct create/update/delete access to individual student, attendance, health, or tuition records is not stated for this role (see Open Questions §6).

---

## 3. Role: Giáo viên (ROLE-TEACHER)

**Description:** Classroom teacher responsible for daily attendance, pickup handling, quick health observations, incident recording, and sharing activity photos with parents.

**Main responsibilities (per CLAUDE.md §3):**
- Take attendance when children arrive at class.
- Take attendance when parents pick up children.
- Support recognizing a pre-registered pickup person.
- Record a quick health status.
- Record incidents/accidents, optionally with photos.
- Share activity photos by group/class.

**Allowed functional areas:** Attendance, Pickup Attendance & Registered Pickup Persons, Health Monitoring (quick status only), Incident Recording, Activity Media Sharing.

**Allowed actions:**

| FR ID | Requirement | Action Level | Notes |
|---|---|---|---|
| FR-ATT-01 | Teacher Check-In Attendance | Create | Attendance status values not finalized — see FR doc Open Question. |
| FR-PICKUP-01 | Teacher Pickup Attendance | Create | |
| FR-PICKUP-02 | Support Pre-Registered Pickup Person | View | Read-only verification against the parent-registered list (FR-PICKUP-03); teacher does not create/modify that list. |
| FR-HEALTH-01 | Teacher Quick Health Status Entry | Create | Fields captured not specified — see FR doc Open Question. |
| FR-INCIDENT-01 | Record Incident / Accident | Create | |
| FR-INCIDENT-02 | Attach Incident Photos | Create / Update | Optional attachment to an existing incident record. |
| FR-MEDIA-01 | Share Activity Photos by Group / Class | Create | |

---

## 4. Role: Kế toán / Văn phòng (ROLE-ACCOUNTING)

**Description:** Office/accounting role responsible for tuition, invoicing, the weekly menu, child enrollment records, teacher HR records, and revenue reporting.

**Main responsibilities (per CLAUDE.md §3):**
- Manage tuition.
- Manage invoices.
- Manage the weekly menu.
- Manage children's enrollment records.
- Manage teacher records.
- Produce revenue reports.
- Support Excel/PDF export.

**Allowed functional areas:** Student/Child Enrollment, Teacher HR Management, Tuition & Invoices, Weekly Menu Management.

**Allowed actions:**

| FR ID | Requirement | Action Level | Notes |
|---|---|---|---|
| FR-STU-01 | Manage Child Enrollment Records | Manage | Required fields not specified — see FR doc Open Question. |
| FR-TEACHER-01 | Manage Teacher Records | Manage | |
| FR-TUITION-01 | Manage Tuition Fees | Manage | Fee structure not specified — see FR doc Open Question. |
| FR-TUITION-02 | Manage Invoices | Manage | |
| FR-TUITION-03 | Revenue Report | Create / View | "Generate" a report; breakdown granularity not specified. |
| FR-TUITION-04 | Export Report (Excel/PDF) | Create | Scope of which reports are exportable not specified. |
| FR-MENU-01 | Manage Weekly Menu | Manage | |

**Not supported by the specification:** Whether this role has any view access to individual attendance or health records (e.g., for billing purposes) is not stated (see Open Questions §3).

---

## 5. Role: Y tế (ROLE-MEDICAL)

**Description:** Medical/health-monitoring role responsible for tracking physical growth measurements and reviewing incident/health history within the project's defined scope.

**Main responsibilities (per CLAUDE.md §3):**
- Track height.
- Track weight.
- Track history of incidents/health issues related to a child, within the specification's scope.

**Allowed functional areas:** Health Monitoring (growth tracking and history review).

**Allowed actions:**

| FR ID | Requirement | Action Level | Notes |
|---|---|---|---|
| FR-HEALTH-02 | Track Height | Create / View | Measurement frequency not specified. |
| FR-HEALTH-03 | Track Weight | Create / View | Measurement frequency not specified. |
| FR-HEALTH-04 | View Health / Incident History | View | Explicitly restricted from extending beyond the specified scope without approval (CLAUDE.md §3; FR-HEALTH-04 Business Rule). |

**Not supported by the specification:** Any functionality beyond height/weight tracking and health/incident history review (e.g., attendance data, class assignment) is explicitly out of scope for this role unless separately approved (CLAUDE.md §3).

---

## 6. Role: Phụ huynh (ROLE-PARENT)

**Description:** Parent/guardian role that views information about their own child and manages who is authorized to pick the child up, and can pay tuition online.

**Main responsibilities (per CLAUDE.md §3):**
- View their child's attendance history.
- View activity photos/videos.
- Receive notifications.
- View tuition invoices.
- Pay tuition online.
- Register/manage people authorized to pick up their child.

**Allowed functional areas:** Attendance history (own child), Registered Pickup Persons, Activity Media (viewing), Notifications, Tuition & Invoices (viewing and payment).

**Allowed actions:**

| FR ID | Requirement | Action Level | Notes |
|---|---|---|---|
| FR-ATT-02 | Parent View Attendance History | View | Explicitly limited to the parent's own child ("của con") — FR-ATT-02 Business Rule. |
| FR-PICKUP-03 | Manage Registered Pickup Persons | Manage | Required data fields not specified — see FR doc Open Question. |
| FR-MEDIA-02 | View Activity Photos / Videos | View | Scoping to own child's class assumed as a reasonable precondition in the FR doc, not stated as an explicit rule — see FR doc Open Question #17. |
| FR-NOTI-01 | Parent Receive Notifications | View | Triggers/delivery channel not specified. |
| FR-TUITION-05 | Parent View Invoice | View | Scoping to own child assumed as a reasonable precondition in the FR doc, not stated as an explicit rule — see FR doc Open Question #17. |
| FR-TUITION-06 | Parent Online Payment | Create | Initiates a payment transaction; payment provider not selected. |

---

## 7. Role-Permission Matrix

| FR ID | Requirement | ROLE-ADMIN | ROLE-TEACHER | ROLE-ACCOUNTING | ROLE-MEDICAL | ROLE-PARENT |
|---|---|---|---|---|---|---|
| FR-AUTH-01 | User Login | ✓ (baseline, all roles) | ✓ | ✓ | ✓ | ✓ |
| FR-AUTH-02 | Role-Based Access Restriction | ✓ (baseline, all roles) | ✓ | ✓ | ✓ | ✓ |
| FR-USER-01 | Manage User Accounts | Manage | – | – | – | – |
| FR-USER-02 | Manage Roles / Permissions | Manage | – | – | – | – |
| FR-CLASS-01 | Manage Classes by Age Group | Manage | – | – | – | – |
| FR-STU-01 | Manage Child Enrollment Records | – | – | Manage | – | – |
| FR-TEACHER-01 | Manage Teacher Records | – | – | Manage | – | – |
| FR-ATT-01 | Teacher Check-In Attendance | – | Create | – | – | – |
| FR-ATT-02 | Parent View Attendance History | – | – | – | – | View (own child) |
| FR-PICKUP-01 | Teacher Pickup Attendance | – | Create | – | – | – |
| FR-PICKUP-02 | Support Pre-Registered Pickup Person | – | View (verify) | – | – | – |
| FR-PICKUP-03 | Parent Manage Registered Pickup Persons | – | – | – | – | Manage |
| FR-HEALTH-01 | Teacher Quick Health Status Entry | – | Create | – | – | – |
| FR-HEALTH-02 | Track Height | – | – | – | Create / View | – |
| FR-HEALTH-03 | Track Weight | – | – | – | Create / View | – |
| FR-HEALTH-04 | View Health / Incident History | – | – | – | View | – |
| FR-INCIDENT-01 | Record Incident / Accident | – | Create | – | – | – |
| FR-INCIDENT-02 | Attach Incident Photos | – | Create / Update | – | – | – |
| FR-MEDIA-01 | Share Activity Photos by Group / Class | – | Create | – | – | – |
| FR-MEDIA-02 | Parent View Activity Photos / Videos | – | – | – | – | View |
| FR-TUITION-01 | Manage Tuition Fees | – | – | Manage | – | – |
| FR-TUITION-02 | Manage Invoices | – | – | Manage | – | – |
| FR-TUITION-03 | Revenue Report | – | – | Create / View | – | – |
| FR-TUITION-04 | Export Report (Excel/PDF) | – | – | Create | – | – |
| FR-TUITION-05 | Parent View Invoice | – | – | – | – | View |
| FR-TUITION-06 | Parent Online Payment | – | – | – | – | Create |
| FR-MENU-01 | Manage Weekly Menu | – | – | Manage | – | – |
| FR-NOTI-01 | Parent Receive Notifications | – | – | – | – | View |
| FR-DASH-01 | Overview Dashboard | View | – | – | – | – |
| FR-DASH-02 | Child Count Statistics | View | – | – | – | – |
| FR-DASH-03 | Attendance Statistics | View | – | – | – | – |
| FR-DASH-04 | Revenue Statistics | View | – | – | – | – |

---

## 8. Cross-Cutting Authorization Rules

Rules recorded here are explicitly supported by `CLAUDE.md` and/or `functional-requirements.md` — none are new inventions:

- **AUTH-RULE-01 — Authentication required:** Every role must be authenticated (logged in) before accessing any role-specific functionality (FR-AUTH-01).
- **AUTH-RULE-02 — Role-based restriction:** A user may only access the functional areas and actions listed for their own role; no role may perform actions belonging to another role (CLAUDE.md §2; FR-AUTH-02).
- **AUTH-RULE-03 — No unapproved scope expansion:** A role's permissions must not be expanded beyond what is specified without approval. This is stated generally (CLAUDE.md §2) and explicitly for Y tế (CLAUDE.md §3: health functionality must not extend beyond the specified scope without approval; FR-HEALTH-04 Business Rule).
- **AUTH-RULE-04 — Own-child data scoping (attendance):** A parent may view attendance history only for their own linked child/children — explicitly grounded in the specification's "của con" wording (FR-ATT-02 Business Rule). The specification does not use equivalent wording for media, invoices, or notifications, so this scoping is not confirmed to extend to those areas (see FR doc Open Question #17, referenced in Open Questions below).
- **AUTH-RULE-05 — Single consistent permission source (implementation-facing note):** Permissions must be defined role-based and not hard-coded in multiple places (CLAUDE.md §5). This is a constraint on how authorization will later be implemented, not a role responsibility itself, and is included here only for traceability.

---

## 9. Open Questions

The following role/permission details are not defined by the specification or the functional requirements and are not resolved by this document:

1. **(All "Manage" permissions)** Whether "Manage" (FR-USER-01, FR-USER-02, FR-CLASS-01, FR-STU-01, FR-TEACHER-01, FR-TUITION-01, FR-TUITION-02, FR-MENU-01, FR-PICKUP-03) includes full Create + Update + Delete, or only a subset of those actions, is not specified.
2. **(ROLE-ACCOUNTING)** Whether Kế toán has any view access to individual attendance or health records (e.g., to support billing decisions) is not specified.
3. **(ROLE-ADMIN)** Whether Admin has direct create/update/delete access to individual student, attendance, health, or tuition records, or is limited to user/permission/class-age management plus aggregate dashboards (FR-DASH), is not specified.
4. **(ROLE-TEACHER)** FR-ATT-01 states a precondition that the child must be assigned to the teacher's class; whether this same class-scoping applies to a teacher's other actions (FR-PICKUP-01/02, FR-HEALTH-01, FR-INCIDENT-01/02, FR-MEDIA-01) is not explicitly stated.
5. **(ROLE-MEDICAL)** Whether Y tế can view attendance or class-assignment data, or only health/incident data, is not specified.
6. **(ROLE-MEDICAL / ROLE-TEACHER)** Whether the "quick health status" entries created by teachers (FR-HEALTH-01) are included within the "health/incident history" that Y tế reviews (FR-HEALTH-04) is not explicitly linked in the functional requirements.
7. **(ROLE-PARENT)** Whether the own-child-only data scoping explicit for attendance (FR-ATT-02) extends uniformly to activity media, invoices, and notifications is unresolved — carried over from `functional-requirements.md` Open Question #17 and not resolved here.
8. **(ROLE-PARENT / other roles)** Whether roles other than Phụ huynh receive notifications is not specified — carried over from `functional-requirements.md` Open Question #15.
9. **(ROLE-ADMIN)** Whether Admin has any oversight/view access into the Health Monitoring module is not specified.
