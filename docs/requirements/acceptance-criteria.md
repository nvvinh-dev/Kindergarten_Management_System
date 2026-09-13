# Acceptance Criteria — Kindergarten Management System

## 0. Document Information

- **Purpose:** Define clear, testable acceptance criteria for the approved functional requirements and business behavior of the system, so that developers and testers share a common definition of "complete and acceptable."
- **Source of truth:** `CLAUDE.md`, `docs/requirements/functional-requirements.md`, `docs/requirements/non-functional-requirements.md`, `docs/business/user-roles.md`, `docs/business/use-cases.md`, `docs/business/business-rules.md`. No criterion below goes beyond what those documents state or directly imply.
- **Status:** Draft — for review and approval before Database Design, Architecture Design, API Design, and UI Design proceed.
- **Out of scope for this document:** database schema, entity classes, API endpoints, controllers, services, source code, UI implementation details, and any specific authentication, payment, or media storage technology.
- **Conventions:**
  - Each criterion uses Given/When/Then where that structure fits the underlying requirement.
  - A criterion is written only when the source documents provide enough detail to make it testable. Where a detail (status value, field, threshold, verification method, workflow) is not defined, the criterion omits it rather than inventing it, and the gap is listed in §16 instead.
  - This document does not resolve any Open Question already recorded in the source documents.
  - Where a functional requirement uses the verb "manage" without decomposing it, the corresponding criteria are written using only the minimal actions any "manage" capability implies (create and update), without asserting a full Create+Update+Delete scope. Whether delete is included remains an open question (see §16, item 2) and is not tested here.

---

## 1. Authentication

### AC-AUTH-01 — Log In
- **Given:** A user has valid registered credentials associated with a role.
- **When:** The user submits their credentials to log in.
- **Then:** The system authenticates the user and grants access to the functionality permitted for their role.
- **Traceability:** FR-AUTH-01 | BR-AUTH-01 | UC-AUTH-01

### AC-AUTH-02 — Role-Based Access Restriction
- **Given:** An authenticated user with an assigned role.
- **When:** The user attempts an action that is not permitted for their role.
- **Then:** The system denies the action.
- **Traceability:** FR-AUTH-02 | BR-AUTH-02 | UC-AUTH-01 (Exception Flow E2)

---

## 2. User & Permission Management

### AC-USER-01 — Manage User Accounts
- **Given:** An authenticated Admin / Ban giám hiệu.
- **When:** Admin creates or updates a user account.
- **Then:** The system stores the account, associated with a role.
- **Traceability:** FR-USER-01 | BR-USER-01 | UC-USER-01

### AC-USER-02 — Manage Roles / Permissions
- **Given:** An authenticated Admin / Ban giám hiệu.
- **When:** Admin updates the permission configuration for a role.
- **Then:** The role reflects the updated permission configuration.
- **Traceability:** FR-USER-02 | BR-USER-02 | UC-USER-02

---

## 3. Class Management

### AC-CLASS-01 — Assign Child to Class by Age Group
- **Given:** An authenticated Admin and a child with an existing enrollment record.
- **When:** Admin assigns the child to a class appropriate to their age group.
- **Then:** The system records the child's class assignment.
- **Traceability:** FR-CLASS-01 | BR-CLASS-01 | UC-CLASS-01

### AC-CLASS-02 — Class Assignment Requires Enrollment
- **Given:** A child without an existing enrollment record.
- **When:** Admin attempts to assign the child to a class.
- **Then:** The system does not complete the class assignment.
- **Traceability:** FR-CLASS-01 | BR-CLASS-02, BR-STUDENT-02 | UC-CLASS-01 (Preconditions)

---

## 4. Child Management

### AC-STU-01 — Manage Child Enrollment Record
- **Given:** An authenticated Kế toán / Văn phòng user.
- **When:** Kế toán creates or updates a child's enrollment record.
- **Then:** The system stores the enrollment record, available for use by other modules.
- **Traceability:** FR-STU-01 | BR-STUDENT-01 | UC-STU-01

---

## 5. Teacher Management

### AC-TEACHER-01 — Manage Teacher Record
- **Given:** An authenticated Kế toán / Văn phòng user.
- **When:** Kế toán creates or updates a teacher record.
- **Then:** The system stores the teacher record, available for class assignment.
- **Traceability:** FR-TEACHER-01 | BR-TEACHER-01 | UC-TEACHER-01

---

## 6. Attendance

### AC-ATT-01 — Record Check-In Attendance
- **Given:** A teacher is authenticated and the child has an enrollment record and is assigned to the teacher's class.
- **When:** The teacher records check-in attendance for the child.
- **Then:** The system records the child's check-in attendance.
- **Traceability:** FR-ATT-01 | BR-ATTENDANCE-01 | UC-ATT-01

### AC-ATT-02 — Check-In Attendance Requires Enrollment and Class Assignment
- **Given:** A child without an enrollment record, or not assigned to the teacher's class.
- **When:** The teacher attempts to record check-in attendance for the child.
- **Then:** The system does not record the attendance.
- **Traceability:** FR-ATT-01 | BR-ATTENDANCE-02 | UC-ATT-01 (Preconditions)

### AC-ATT-03 — Parent Views Own Child's Attendance History
- **Given:** A parent linked to a child's record.
- **When:** The parent requests that child's attendance history.
- **Then:** The system displays the attendance history for that child.
- **Traceability:** FR-ATT-02 | BR-ATTENDANCE-03 | UC-ATT-02

### AC-ATT-04 — Parent Cannot View Another Child's Attendance History
- **Given:** A parent linked only to their own child's record.
- **When:** The parent attempts to view attendance history for a child not linked to them.
- **Then:** The system does not display that other child's attendance history.
- **Traceability:** FR-ATT-02 | BR-ATTENDANCE-03 | UC-ATT-02

---

## 7. Child Pickup

### AC-PICKUP-01 — Record Pickup Attendance
- **Given:** A teacher is authenticated and the child was checked in for the day.
- **When:** The teacher records that the child has been picked up.
- **Then:** The system creates a pickup record associated with the child and the day's attendance.
- **Traceability:** FR-PICKUP-01 | BR-PICKUP-01 | UC-PICKUP-01

### AC-PICKUP-02 — Non-Parent Pickup Checked Against Registered List
- **Given:** A person other than the parent arrives to pick up a child, and the parent has registered one or more pickup persons for that child.
- **When:** The teacher checks the person against the child's list of registered pickup persons.
- **Then:** The system allows the teacher to confirm whether the person is on the registered list.
- **Traceability:** FR-PICKUP-02 | BR-PICKUP-02 | UC-PICKUP-01

### AC-PICKUP-03 — Parent Manages Registered Pickup Persons
- **Given:** An authenticated parent linked to a child.
- **When:** The parent adds or updates a registered pickup person for that child.
- **Then:** The system stores the updated list of registered pickup persons, available for verification during pickup.
- **Traceability:** FR-PICKUP-03 | BR-PICKUP-03 | UC-PICKUP-02

---

## 8. Health Monitoring

### AC-HEALTH-01 — Record Quick Health Status
- **Given:** An authenticated Giáo viên.
- **When:** The teacher records a quick health status entry for a child.
- **Then:** The system stores the entry against that child's record.
- **Traceability:** FR-HEALTH-01 | BR-HEALTH-01, BR-HEALTH-03 | UC-HEALTH-01

### AC-HEALTH-02 — Record Height Measurement
- **Given:** An authenticated Y tế user.
- **When:** Y tế records a height measurement for a child.
- **Then:** The system stores the measurement as part of that child's growth history.
- **Traceability:** FR-HEALTH-02 | BR-HEALTH-02, BR-HEALTH-03 | UC-HEALTH-02

### AC-HEALTH-03 — Record Weight Measurement
- **Given:** An authenticated Y tế user.
- **When:** Y tế records a weight measurement for a child.
- **Then:** The system stores the measurement as part of that child's growth history.
- **Traceability:** FR-HEALTH-03 | BR-HEALTH-02, BR-HEALTH-03 | UC-HEALTH-02

### AC-HEALTH-04 — Y tế Reviews Health & Incident History
- **Given:** An authenticated Y tế user and a child with existing incident and/or health records.
- **When:** Y tế selects the child.
- **Then:** The system displays that child's chronological incident and health history.
- **Traceability:** FR-HEALTH-04 | BR-HEALTH-04 | UC-HEALTH-03

---

## 9. Incident Management

### AC-INCIDENT-01 — Record Incident / Accident
- **Given:** An authenticated Giáo viên.
- **When:** The teacher creates an incident record describing an incident involving a child.
- **Then:** The system stores the incident record, associated with that child.
- **Traceability:** FR-INCIDENT-01 | BR-INCIDENT-01, BR-INCIDENT-02 | UC-INCIDENT-01

### AC-INCIDENT-02 — Attach Photo to Incident Record
- **Given:** An existing incident record.
- **When:** The teacher attaches one or more photos to it.
- **Then:** The system associates the attached photo(s) with the incident record.
- **Traceability:** FR-INCIDENT-02 | BR-INCIDENT-03 | UC-INCIDENT-01

### AC-INCIDENT-03 — Incident Photo Attachment Is Optional
- **Given:** An incident record with no attached photo.
- **When:** The teacher saves the incident record.
- **Then:** The system accepts the incident record as valid without requiring a photo.
- **Traceability:** FR-INCIDENT-02 | BR-INCIDENT-03 | UC-INCIDENT-01

### AC-INCIDENT-04 — Incident Becomes Part of Child's Health History
- **Given:** An incident record has been created for a child.
- **When:** Y tế reviews that child's health/incident history.
- **Then:** The incident record appears within that history.
- **Traceability:** FR-INCIDENT-01 | BR-INCIDENT-04 | UC-HEALTH-03

---

## 10. Activity Media

### AC-MEDIA-01 — Share Activity Photos by Group / Class
- **Given:** An authenticated Giáo viên.
- **When:** The teacher shares one or more activity photos scoped to a specific group or class.
- **Then:** The system stores the photos, scoped to that group/class.
- **Traceability:** FR-MEDIA-01 | BR-MEDIA-01 | UC-MEDIA-01

### AC-MEDIA-02 — Parent Views Activity Media for Their Child's Class
- **Given:** An authenticated parent linked to a child.
- **When:** The parent opens activity media for that child's class.
- **Then:** The system displays the photos/videos shared for that class.
- **Traceability:** FR-MEDIA-02 | BR-MEDIA-02 | UC-MEDIA-02

---

## 11. Tuition & Payment

### AC-TUITION-01 — Manage Tuition Fees
- **Given:** An authenticated Kế toán / Văn phòng user.
- **When:** Kế toán creates or updates tuition fee information.
- **Then:** The system stores the fee information, available for generating invoices.
- **Traceability:** FR-TUITION-01 | BR-TUITION-01 | UC-TUITION-01

### AC-TUITION-02 — Manage Invoices
- **Given:** An authenticated Kế toán / Văn phòng user and existing tuition fee information.
- **When:** Kế toán creates or updates an invoice for a child.
- **Then:** The system stores the invoice.
- **Traceability:** FR-TUITION-02 | BR-TUITION-02, BR-TUITION-03 | UC-TUITION-02

### AC-TUITION-03 — Generate Revenue Report
- **Given:** An authenticated Kế toán / Văn phòng user.
- **When:** Kế toán requests a revenue report.
- **Then:** The system generates a revenue report based on the system's tuition and payment data.
- **Traceability:** FR-TUITION-03 | BR-TUITION-07, BR-TUITION-08 | UC-TUITION-03

### AC-TUITION-04 — Export Revenue Report
- **Given:** A generated revenue report.
- **When:** Kế toán chooses to export the report.
- **Then:** The system produces the report in Excel and/or PDF format.
- **Traceability:** FR-TUITION-04 | — | UC-TUITION-03

### AC-TUITION-05 — Parent Views Invoice
- **Given:** A parent linked to a child with an existing invoice.
- **When:** The parent requests that child's invoice.
- **Then:** The system displays the invoice.
- **Traceability:** FR-TUITION-05 | BR-TUITION-04 | UC-TUITION-04

### AC-TUITION-06 — Parent Pays Tuition Online
- **Given:** An existing, unpaid invoice for the parent's child.
- **When:** The parent submits payment for that invoice.
- **Then:** The system updates the invoice's payment status upon successful payment.
- **Traceability:** FR-TUITION-06 | BR-TUITION-05, BR-TUITION-06 | UC-TUITION-05

---

## 12. Weekly Menu

### AC-MENU-01 — Manage Weekly Menu
- **Given:** An authenticated Kế toán / Văn phòng user.
- **When:** Kế toán creates or updates the weekly menu.
- **Then:** The system stores the weekly menu.
- **Traceability:** FR-MENU-01 | BR-MENU-01 | UC-MENU-01

---

## 13. Notifications

### AC-NOTI-01 — Parent Views Notification
- **Given:** A notification intended for a parent exists in the system.
- **When:** The parent accesses their notifications.
- **Then:** The parent is able to view that notification.
- **Traceability:** FR-NOTI-01 | BR-NOTIFICATION-01 | UC-NOTI-01

---

## 14. Admin Dashboard

### AC-DASH-01 — View Overview Dashboard
- **Given:** An authenticated Admin.
- **When:** Admin opens the dashboard.
- **Then:** The system displays a general overview.
- **Traceability:** FR-DASH-01 | BR-DASHBOARD-01 | UC-DASH-01

### AC-DASH-02 — View Child Count Statistics
- **Given:** An authenticated Admin.
- **When:** Admin views the dashboard.
- **Then:** The system displays statistics on the number of children.
- **Traceability:** FR-DASH-02 | BR-DASHBOARD-01, BR-DASHBOARD-02 | UC-DASH-01

### AC-DASH-03 — View Attendance Statistics
- **Given:** An authenticated Admin.
- **When:** Admin views the dashboard.
- **Then:** The system displays attendance statistics.
- **Traceability:** FR-DASH-03 | BR-DASHBOARD-01, BR-DASHBOARD-02 | UC-DASH-01

### AC-DASH-04 — View Revenue Statistics
- **Given:** An authenticated Admin.
- **When:** Admin views the dashboard.
- **Then:** The system displays revenue statistics based on the system's tuition and payment data.
- **Traceability:** FR-DASH-04 | BR-DASHBOARD-01, BR-DASHBOARD-02, BR-TUITION-08 | UC-DASH-01

### AC-DASH-05 — Dashboard Access Restricted to Admin
- **Given:** An authenticated user who is not Admin.
- **When:** The user attempts to access the dashboard.
- **Then:** The system denies access.
- **Traceability:** FR-DASH-01 to FR-DASH-04 | BR-DASHBOARD-01, BR-AUTH-02 | UC-DASH-01

---

## 15. Non-Functional Acceptance Criteria

Only NFRs verifiable as observable behavior, without numeric targets, are included.

### AC-NFR-01 — Backend Validates Input Independently of Frontend
- **Given:** Input is submitted to the system, with or without frontend validation applied.
- **When:** The backend processes the request.
- **Then:** The backend validates the input itself, regardless of what the frontend already checked.
- **Traceability:** NFR-SEC-02 | — | —

### AC-NFR-02 — No Internal Error Details Exposed to Users
- **Given:** An internal error occurs while processing a request.
- **When:** The system responds to the user.
- **Then:** The response does not contain internal exception details or stack traces, and the error is handled consistently.
- **Traceability:** NFR-SEC-03, NFR-DATA-02 | — | —

### AC-NFR-03 — Responsive UI Across Device Sizes
- **Given:** A user accesses the system from a common desktop or mobile screen size.
- **When:** The user views a page.
- **Then:** The UI remains usable at that screen size.
- **Traceability:** NFR-USE-01, NFR-COMPAT-01 | — | —

### AC-NFR-04 — Loading, Error, and Empty States Are Presented
- **Given:** A feature is loading data, has encountered an error, or has no data to show.
- **When:** The user views that feature.
- **Then:** The system presents the corresponding loading, error, or empty state.
- **Traceability:** NFR-USE-02 | — | —

### AC-NFR-05 — Data Access Restricted to Authorized Scope
- **Given:** An authenticated user.
- **When:** The user attempts to access data or functionality outside the scope permitted for their role.
- **Then:** The system prevents that access.
- **Traceability:** NFR-SEC-01, NFR-SEC-05 | BR-AUTH-02 | UC-AUTH-01

---

## 16. Open Questions / Deferred Criteria

Acceptance criteria could not be completed for the following, because an existing Open Question in the source documents must be resolved first. These are not resolved here.

1. **Login failure handling (Authentication)** — No criterion is defined for invalid credentials, retries, or lockout.
2. **Permission granularity (User & Permission Management)** — No criterion verifies a specific CRUD breakdown of "Manage," since the source does not define one.
3. **Class age boundaries / teacher-to-class assignment (Class Management)** — No criterion verifies specific age bands or teacher assignment, since neither is defined.
4. **Enrollment field validation (Child Management)** — No criterion verifies specific required fields, since they are not defined.
5. **Attendance status values (Attendance)** — No criterion verifies specific status values (e.g., Present/Absent/Late), since none are defined.
6. **Pickup-person verification method and unrecognized-person handling (Child Pickup)** — No criterion defines what happens when a pickup person is not on the registered list, or the exact verification method, since neither is defined.
7. **Quick health status fields (Health Monitoring)** — No criterion verifies specific fields captured, since none are defined.
8. **Height/weight measurement schedule (Health Monitoring)** — No criterion verifies a measurement frequency, since none is defined.
9. **Health/incident history linkage (Health Monitoring)** — No criterion verifies whether quick health status entries appear in the history Y tế reviews, since this link is not confirmed.
10. **Media access scope and video upload (Activity Media)** — No criterion verifies teacher video upload or resolves the parent media access scope, since these details are not defined; no criterion asserts an own-child-only restriction on media viewing.
11. **Parent access scope for invoices (Tuition & Payment)** — No criterion asserts an own-child-only restriction on invoice viewing, since none is stated (unlike attendance).
12. **Tuition fee structure (Tuition & Payment)** — No criterion verifies a specific fee structure, since none is defined.
13. **Report export scope (Tuition & Payment)** — No criterion verifies which reports besides the revenue report support export, since this is not defined.
14. **Payment provider and failed-payment handling (Tuition & Payment)** — No criterion defines behavior for a failed or declined payment, since this is not defined.
15. **Weekly menu visibility (Weekly Menu)** — No criterion for viewing the weekly menu is defined, since no viewer role is named.
16. **Notification triggers, channels, and read/unread behavior (Notifications)** — No criterion verifies specific triggers, delivery channels, or read/dismiss behavior, since none are defined.
17. **Dashboard breakdowns and time ranges (Admin Dashboard)** — No criterion verifies specific breakdowns or time ranges, since none are defined.
18. **Admin record-level access (cross-cutting)** — No criterion verifies whether Admin can access individual student/attendance/health/tuition records, since this is not defined.
19. **Kế toán visibility into attendance/health (cross-cutting)** — No criterion verifies such access, since it is not defined.
20. **Teacher action scoping beyond attendance (cross-cutting)** — No criterion verifies that pickup, health, incident, or media actions are scoped to the teacher's assigned class, since only check-in attendance states this explicitly.
21. **Y tế visibility into attendance/class data (cross-cutting)** — No criterion verifies such access, since it is not defined.
22. **Other exception-flow handling (cross-cutting)** — Beyond what is covered above, no criteria are defined for failure/exception paths not explicitly described in the source documents.

---

## 17. Traceability

| Acceptance Criterion | Requirement | Business Rule | Use Case |
|---|---|---|---|
| AC-AUTH-01 | FR-AUTH-01 | BR-AUTH-01 | UC-AUTH-01 |
| AC-AUTH-02 | FR-AUTH-02 | BR-AUTH-02 | UC-AUTH-01 |
| AC-USER-01 | FR-USER-01 | BR-USER-01 | UC-USER-01 |
| AC-USER-02 | FR-USER-02 | BR-USER-02 | UC-USER-02 |
| AC-CLASS-01 | FR-CLASS-01 | BR-CLASS-01 | UC-CLASS-01 |
| AC-CLASS-02 | FR-CLASS-01 | BR-CLASS-02, BR-STUDENT-02 | UC-CLASS-01 |
| AC-STU-01 | FR-STU-01 | BR-STUDENT-01 | UC-STU-01 |
| AC-TEACHER-01 | FR-TEACHER-01 | BR-TEACHER-01 | UC-TEACHER-01 |
| AC-ATT-01 | FR-ATT-01 | BR-ATTENDANCE-01 | UC-ATT-01 |
| AC-ATT-02 | FR-ATT-01 | BR-ATTENDANCE-02 | UC-ATT-01 |
| AC-ATT-03 | FR-ATT-02 | BR-ATTENDANCE-03 | UC-ATT-02 |
| AC-ATT-04 | FR-ATT-02 | BR-ATTENDANCE-03 | UC-ATT-02 |
| AC-PICKUP-01 | FR-PICKUP-01 | BR-PICKUP-01 | UC-PICKUP-01 |
| AC-PICKUP-02 | FR-PICKUP-02 | BR-PICKUP-02 | UC-PICKUP-01 |
| AC-PICKUP-03 | FR-PICKUP-03 | BR-PICKUP-03 | UC-PICKUP-02 |
| AC-HEALTH-01 | FR-HEALTH-01 | BR-HEALTH-01, BR-HEALTH-03 | UC-HEALTH-01 |
| AC-HEALTH-02 | FR-HEALTH-02 | BR-HEALTH-02, BR-HEALTH-03 | UC-HEALTH-02 |
| AC-HEALTH-03 | FR-HEALTH-03 | BR-HEALTH-02, BR-HEALTH-03 | UC-HEALTH-02 |
| AC-HEALTH-04 | FR-HEALTH-04 | BR-HEALTH-04 | UC-HEALTH-03 |
| AC-INCIDENT-01 | FR-INCIDENT-01 | BR-INCIDENT-01, BR-INCIDENT-02 | UC-INCIDENT-01 |
| AC-INCIDENT-02 | FR-INCIDENT-02 | BR-INCIDENT-03 | UC-INCIDENT-01 |
| AC-INCIDENT-03 | FR-INCIDENT-02 | BR-INCIDENT-03 | UC-INCIDENT-01 |
| AC-INCIDENT-04 | FR-INCIDENT-01 | BR-INCIDENT-04 | UC-HEALTH-03 |
| AC-MEDIA-01 | FR-MEDIA-01 | BR-MEDIA-01 | UC-MEDIA-01 |
| AC-MEDIA-02 | FR-MEDIA-02 | BR-MEDIA-02 | UC-MEDIA-02 |
| AC-TUITION-01 | FR-TUITION-01 | BR-TUITION-01 | UC-TUITION-01 |
| AC-TUITION-02 | FR-TUITION-02 | BR-TUITION-02, BR-TUITION-03 | UC-TUITION-02 |
| AC-TUITION-03 | FR-TUITION-03 | BR-TUITION-07, BR-TUITION-08 | UC-TUITION-03 |
| AC-TUITION-04 | FR-TUITION-04 | — | UC-TUITION-03 |
| AC-TUITION-05 | FR-TUITION-05 | BR-TUITION-04 | UC-TUITION-04 |
| AC-TUITION-06 | FR-TUITION-06 | BR-TUITION-05, BR-TUITION-06 | UC-TUITION-05 |
| AC-MENU-01 | FR-MENU-01 | BR-MENU-01 | UC-MENU-01 |
| AC-NOTI-01 | FR-NOTI-01 | BR-NOTIFICATION-01 | UC-NOTI-01 |
| AC-DASH-01 | FR-DASH-01 | BR-DASHBOARD-01 | UC-DASH-01 |
| AC-DASH-02 | FR-DASH-02 | BR-DASHBOARD-01, BR-DASHBOARD-02 | UC-DASH-01 |
| AC-DASH-03 | FR-DASH-03 | BR-DASHBOARD-01, BR-DASHBOARD-02 | UC-DASH-01 |
| AC-DASH-04 | FR-DASH-04 | BR-DASHBOARD-01, BR-DASHBOARD-02, BR-TUITION-08 | UC-DASH-01 |
| AC-DASH-05 | FR-DASH-01–04 | BR-DASHBOARD-01, BR-AUTH-02 | UC-DASH-01 |
| AC-NFR-01 | NFR-SEC-02 | — | — |
| AC-NFR-02 | NFR-SEC-03, NFR-DATA-02 | — | — |
| AC-NFR-03 | NFR-USE-01, NFR-COMPAT-01 | — | — |
| AC-NFR-04 | NFR-USE-02 | — | — |
| AC-NFR-05 | NFR-SEC-01, NFR-SEC-05 | BR-AUTH-02 | UC-AUTH-01 |
