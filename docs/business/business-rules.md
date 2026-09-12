# Business Rules — Kindergarten Management System

## 0. Document Information

- **Source of truth:** `CLAUDE.md`, `docs/requirements/functional-requirements.md`, `docs/requirements/non-functional-requirements.md`, `docs/business/user-roles.md`, `docs/business/use-cases.md`. No rule below goes beyond what those documents state or directly imply.
- **Status:** Draft — for review and approval before Database Design and API Design proceed.
- **Out of scope for this document:** database schema, API endpoints, source code, UI implementation, technical authorization mechanisms, and infrastructure. Rules here describe how the business operates, not how the system is built.
- **Convention:** A rule is recorded only when explicitly stated or directly implied by the source documents. Where a detail is plausible but not stated (e.g., a verification method, a status list, a CRUD breakdown of "manage"), it is left out of the rule and recorded under Open Questions instead.

---

## 1. BR-AUTH — Authentication & Access

### BR-AUTH-01 — Authentication Required
- **Rule:** Every actor must be authenticated before accessing any role-specific functionality.
- **Applies to:** All roles.
- **Source:** FR-AUTH-01; UC-AUTH-01.

### BR-AUTH-02 — Role-Based Access Restriction
- **Rule:** A user may only access the functionality and data assigned to their own role. Actions belonging to another role are not permitted.
- **Applies to:** All roles.
- **Source:** CLAUDE.md §2; FR-AUTH-02; UC-AUTH-01 (Exception Flow E2); `user-roles.md` AUTH-RULE-02.

---

## 2. BR-USER — User Management

### BR-USER-01 — User Accounts Tied to a Role
- **Rule:** Every user account must be associated with one of the 5 defined roles.
- **Applies to:** Admin / Ban giám hiệu (manages accounts); all roles (hold accounts).
- **Source:** CLAUDE.md §2, §3; FR-USER-01; UC-USER-01.

### BR-USER-02 — Account and Permission Management Restricted to Admin
- **Rule:** Only Admin / Ban giám hiệu may manage user accounts and manage role permissions.
- **Applies to:** Admin / Ban giám hiệu.
- **Source:** FR-USER-01, FR-USER-02; UC-USER-01, UC-USER-02.

---

## 3. BR-CLASS — Class Management

### BR-CLASS-01 — Class Assignment by Age Group
- **Rule:** Children must be organized into classes according to age group.
- **Applies to:** Admin / Ban giám hiệu.
- **Source:** CLAUDE.md §3 ("Phân lớp theo độ tuổi"); FR-CLASS-01; UC-CLASS-01.

### BR-CLASS-02 — Enrollment Required Before Class Assignment
- **Rule:** A child must have an enrollment record before being assigned to a class.
- **Applies to:** Admin / Ban giám hiệu, Kế toán / Văn phòng.
- **Source:** UC-CLASS-01 Preconditions (child has an enrollment record, FR-STU-01).

---

## 4. BR-STUDENT — Child Management

### BR-STUDENT-01 — Enrollment Managed by Kế toán
- **Rule:** Only Kế toán / Văn phòng may create and maintain a child's enrollment record.
- **Applies to:** Kế toán / Văn phòng.
- **Source:** FR-STU-01; UC-STU-01.

### BR-STUDENT-02 — Enrollment Record Required for Class Assignment and Attendance
- **Rule:** A child's enrollment record must exist before the child can be assigned to a class or have check-in attendance recorded.
- **Applies to:** Kế toán / Văn phòng, Admin / Ban giám hiệu, Giáo viên.
- **Source:** FR-STU-01 Outputs ("usable by other modules (class assignment, attendance, health, tuition)"); FR-ATT-01 Preconditions; UC-CLASS-01 Preconditions.

---

## 5. BR-TEACHER — Teacher Management

### BR-TEACHER-01 — Teacher Records Managed by Kế toán
- **Rule:** Only Kế toán / Văn phòng may create and maintain teacher HR records.
- **Applies to:** Kế toán / Văn phòng.
- **Source:** FR-TEACHER-01; UC-TEACHER-01.

### BR-TEACHER-02 — Teacher Record Required for Class Assignment
- **Rule:** A teacher record must exist before it can be used for class assignment.
- **Applies to:** Kế toán / Văn phòng, Admin / Ban giám hiệu.
- **Source:** FR-TEACHER-01 Outputs ("usable for class assignment").

---

## 6. BR-ATTENDANCE — Attendance

### BR-ATTENDANCE-01 — Check-In Attendance Recorded by Teacher
- **Rule:** Only the Giáo viên may record a child's check-in attendance.
- **Applies to:** Giáo viên.
- **Source:** FR-ATT-01; UC-ATT-01.

### BR-ATTENDANCE-02 — Attendance Requires Enrollment and Class Assignment
- **Rule:** A child must have an enrollment record and be assigned to the teacher's class before check-in attendance can be recorded.
- **Applies to:** Giáo viên.
- **Source:** FR-ATT-01 Preconditions.

### BR-ATTENDANCE-03 — Attendance History Restricted to Own Child
- **Rule:** A parent may view attendance history only for their own linked child/children.
- **Applies to:** Phụ huynh.
- **Source:** FR-ATT-02 Business Rule (grounded in "của con"); UC-ATT-02.

---

## 7. BR-PICKUP — Child Pickup

### BR-PICKUP-01 — Pickup Attendance Recorded by Teacher
- **Rule:** Only the Giáo viên may record pickup attendance for a child.
- **Applies to:** Giáo viên.
- **Source:** FR-PICKUP-01; UC-PICKUP-01.

### BR-PICKUP-02 — Non-Parent Pickup Requires Prior Registration
- **Rule:** A person other than the parent may be recognized as authorized to pick up a child only if the parent has registered that person in advance.
- **Applies to:** Giáo viên, Phụ huynh.
- **Source:** FR-PICKUP-02 ("đã được phụ huynh đăng ký trước"); FR-PICKUP-03; UC-PICKUP-01.

### BR-PICKUP-03 — Pickup Person List Managed by Parent
- **Rule:** Only the Phụ huynh may register and manage the list of people authorized to pick up their child.
- **Applies to:** Phụ huynh.
- **Source:** FR-PICKUP-03; UC-PICKUP-02.

---

## 8. BR-HEALTH — Health Management

### BR-HEALTH-01 — Quick Health Status Recorded by Teacher
- **Rule:** Only the Giáo viên may record a quick health status entry for a child.
- **Applies to:** Giáo viên.
- **Source:** FR-HEALTH-01; UC-HEALTH-01.

### BR-HEALTH-02 — Growth Measurements Recorded by Y tế
- **Rule:** Only Y tế may record a child's height and weight measurements.
- **Applies to:** Y tế.
- **Source:** FR-HEALTH-02, FR-HEALTH-03; UC-HEALTH-02.

### BR-HEALTH-03 — Health Records Associated with a Specific Child
- **Rule:** Quick health status entries and growth measurements must be associated with the specific child they describe.
- **Applies to:** Giáo viên, Y tế.
- **Source:** FR-HEALTH-01 Outputs ("stored against the child's record"); FR-HEALTH-02/03 Outputs ("per child").

### BR-HEALTH-04 — Health/Incident History Review Restricted to Y tế
- **Rule:** Only Y tế reviews the history of incidents and health issues for a child, within the scope defined by the specification.
- **Applies to:** Y tế.
- **Source:** FR-HEALTH-04; UC-HEALTH-03.

---

## 9. BR-INCIDENT — Incident Management

### BR-INCIDENT-01 — Incidents Recorded by Teacher
- **Rule:** Only the Giáo viên may create an incident/accident record.
- **Applies to:** Giáo viên.
- **Source:** FR-INCIDENT-01; UC-INCIDENT-01.

### BR-INCIDENT-02 — Incident Records Associated with a Specific Child
- **Rule:** An incident record must be associated with the child involved.
- **Applies to:** Giáo viên.
- **Source:** FR-INCIDENT-01 ("involving a child"); UC-INCIDENT-01.

### BR-INCIDENT-03 — Incident Photo Attachment Is Optional
- **Rule:** A teacher may optionally attach one or more photos to an incident record; attachment is not mandatory.
- **Applies to:** Giáo viên.
- **Source:** FR-INCIDENT-02 ("may attach", "optionally").

### BR-INCIDENT-04 — Incidents Become Part of the Child's Health History
- **Rule:** An incident record becomes part of the child's incident history and is viewable by Y tế.
- **Applies to:** Giáo viên (creates), Y tế (views).
- **Source:** FR-INCIDENT-01 Outputs ("becomes part of the child's incident history (viewable per FR-HEALTH-04)"); UC-HEALTH-03.

---

## 10. BR-MEDIA — Activity Media

### BR-MEDIA-01 — Activity Photos Shared by Teacher, Scoped to Group/Class
- **Rule:** A teacher shares activity photos scoped to a specific group or class.
- **Applies to:** Giáo viên.
- **Source:** FR-MEDIA-01 ("theo nhóm / lớp"); UC-MEDIA-01.

### BR-MEDIA-02 — Parent Views Media Associated with Their Child's Class
- **Rule:** A parent views activity photos/videos associated with their child's class/group.
- **Applies to:** Phụ huynh.
- **Source:** FR-MEDIA-02; UC-MEDIA-02 (Precondition: parent is linked to the child whose class/group media is viewed).
- **Note:** Unlike attendance (BR-ATTENDANCE-03), the specification does not use equivalent wording to explicitly state an exclusivity restriction (that a parent cannot view media from other classes). This document does not assert one — see Open Questions.

---

## 11. BR-TUITION — Tuition & Payment

### BR-TUITION-01 — Tuition Fees Managed by Kế toán
- **Rule:** Only Kế toán / Văn phòng manages tuition fee information.
- **Applies to:** Kế toán / Văn phòng.
- **Source:** FR-TUITION-01; UC-TUITION-01.

### BR-TUITION-02 — Invoices Managed by Kế toán
- **Rule:** Only Kế toán / Văn phòng manages invoices.
- **Applies to:** Kế toán / Văn phòng.
- **Source:** FR-TUITION-02; UC-TUITION-02.

### BR-TUITION-03 — Invoices Are Generated from Tuition Fee Information
- **Rule:** Invoices are generated using tuition fee information.
- **Applies to:** Kế toán / Văn phòng.
- **Source:** FR-TUITION-01 Outputs ("used to generate invoices"); UC-TUITION-02 Preconditions.

### BR-TUITION-04 — Parent Views Invoice Associated with Their Child
- **Rule:** A parent views the tuition invoice associated with their child.
- **Applies to:** Phụ huynh.
- **Source:** FR-TUITION-05; UC-TUITION-04 (Precondition: parent is linked to the child whose invoice is viewed).
- **Note:** As with BR-MEDIA-02, the specification does not use equivalent "của con" wording here, so an exclusivity restriction is not asserted — see Open Questions.

### BR-TUITION-05 — Online Payment Requires an Existing Invoice
- **Rule:** A parent can pay online only against an existing, unpaid invoice.
- **Applies to:** Phụ huynh.
- **Source:** FR-TUITION-06 Preconditions ("An unpaid invoice exists"); UC-TUITION-05.

### BR-TUITION-06 — Successful Payment Updates Invoice Status
- **Rule:** A successful online payment updates the payment status of the corresponding invoice.
- **Applies to:** Phụ huynh; Kế toán / Văn phòng (invoice owner).
- **Source:** FR-TUITION-06 Outputs.

### BR-TUITION-07 — Revenue Reports Generated by Kế toán
- **Rule:** Only Kế toán / Văn phòng generates revenue reports.
- **Applies to:** Kế toán / Văn phòng.
- **Source:** FR-TUITION-03; UC-TUITION-03.

### BR-TUITION-08 — Dashboard Revenue Statistics Reflect Tuition and Payment Data
- **Rule:** The Admin dashboard must reflect revenue statistics based on the system's tuition and payment data.
- **Applies to:** Kế toán / Văn phòng (source); Admin / Ban giám hiệu (consumer).
- **Source:** FR-TUITION-03 Outputs; FR-DASH-04.

---

## 12. BR-MENU — Weekly Menu

### BR-MENU-01 — Weekly Menu Managed by Kế toán
- **Rule:** Only Kế toán / Văn phòng manages the weekly menu.
- **Applies to:** Kế toán / Văn phòng.
- **Source:** FR-MENU-01; UC-MENU-01.

---

## 13. BR-NOTIFICATION — Notifications

### BR-NOTIFICATION-01 — Notifications Directed to Parent
- **Rule:** The specification defines Phụ huynh as the recipient of system notifications.
- **Applies to:** Phụ huynh.
- **Source:** FR-NOTI-01; UC-NOTI-01.

---

## 14. BR-DASHBOARD — Dashboard & Statistics

### BR-DASHBOARD-01 — Dashboard Restricted to Admin
- **Rule:** Only Admin / Ban giám hiệu views the school-wide dashboard and statistics.
- **Applies to:** Admin / Ban giám hiệu.
- **Source:** FR-DASH-01 to FR-DASH-04; UC-DASH-01.

### BR-DASHBOARD-02 — Dashboard Represents Defined Statistics Only
- **Rule:** The dashboard represents an overview, child count statistics, attendance statistics, and revenue statistics — no other metrics are defined.
- **Applies to:** Admin / Ban giám hiệu.
- **Source:** FR-DASH-01, FR-DASH-02, FR-DASH-03, FR-DASH-04.

---

## 15. Open Questions

These are unresolved business decisions carried from `functional-requirements.md`, `user-roles.md`, and `use-cases.md`. They are not resolved by this document.

1. **Authentication strategy** — The authentication mechanism is not finalized (FR-AUTH-01 Open Question; `DECISIONS.md` D20).
2. **Permission granularity** — Whether "Manage" (BR-USER-02, BR-STUDENT-01, BR-TEACHER-01, BR-CLASS-01, BR-TUITION-01/02, BR-MENU-01, BR-PICKUP-03) means full Create + Update + Delete, or a narrower set of actions, is not defined (`user-roles.md` Open Question #1).
3. **Class age groups and teacher assignment** — Age band boundaries and whether teacher-to-class assignment is part of class management are not defined (FR-CLASS-01 Open Question).
4. **Child enrollment fields** — Required enrollment fields, the parent/guardian relationship model, and student status values are not defined (FR-STU-01 Open Question).
5. **Attendance statuses** — The valid set of attendance statuses is not defined (FR-ATT-01 Open Question; `DECISIONS.md` D23).
6. **Pickup-person verification** — Required data fields for a registered pickup person and the verification method used at pickup are not defined (FR-PICKUP-02/03 Open Questions; `DECISIONS.md` D24).
7. **Health status fields** — The specific fields captured in a "quick" health status are not defined (FR-HEALTH-01 Open Question).
8. **Health measurement schedule** — The frequency/schedule for height and weight measurements is not defined (FR-HEALTH-02/03 Open Questions).
9. **Health/incident history linkage** — Whether the teacher's "quick health status" entries are included in the history Y tế reviews (FR-HEALTH-04) is not explicitly linked (`user-roles.md` Open Question #6).
10. **Media storage/access scope** — The media storage provider is not selected, and whether a teacher may share video (not just photos) is unclear (FR-MEDIA-01/02 Open Questions; `DECISIONS.md` D22).
11. **Parent access scope (media and invoices)** — Whether the own-child-only restriction explicit for attendance also applies to activity media and invoices is not confirmed (FR-ATT-02 vs. FR-MEDIA-02/FR-TUITION-05; FR doc Open Question #17; `user-roles.md` Open Question #7).
12. **Tuition fee structure** — The structure of tuition fees (per class, per age group, per service) is not defined (FR-TUITION-01 Open Question).
13. **Payment provider** — The online payment provider is not selected (FR-TUITION-06 Open Question; `DECISIONS.md` D21).
14. **Invoice/payment detail** — Invoice status values, payment deadlines, and refund policy are not defined anywhere in the source documents.
15. **Weekly menu visibility** — No role is defined as a viewer of the weekly menu (FR-MENU-01 Open Question).
16. **Notification triggers/channels** — The events that trigger a notification, the delivery channel, and whether a notification can be marked read/dismissed are not defined (FR-NOTI-01 Open Question; `DECISIONS.md` D25).
17. **Dashboard breakdowns** — Specific statistical breakdowns and time ranges for dashboard metrics are not defined (FR-DASH Open Question).
18. **Admin record-level access** — Whether Admin has direct access to individual student/attendance/health/tuition records beyond aggregate dashboards is not defined (`user-roles.md` Open Question #3).
19. **Kế toán visibility into attendance/health** — Whether Kế toán has any view access to individual attendance or health records is not defined (`user-roles.md` Open Question #2).
20. **Teacher action scoping** — Whether a teacher's actions (pickup, health, incident, media) are scoped to their assigned class, as explicitly stated for check-in attendance, is not defined (`user-roles.md` Open Question #4).
21. **Y tế visibility into attendance/class data** — Whether Y tế can view attendance or class-assignment data is not defined (`user-roles.md` Open Question #5).
22. **Exception-flow handling** — Handling of a failed login, an unrecognized pickup person, and a failed/declined online payment are not defined (`use-cases.md` Open Questions #1–3).

---

## 16. Traceability

| Business Rule | Source | Related Use Cases |
|---|---|---|
| BR-AUTH-01 | FR-AUTH-01 | UC-AUTH-01 |
| BR-AUTH-02 | CLAUDE.md §2; FR-AUTH-02 | UC-AUTH-01 |
| BR-USER-01 | CLAUDE.md §2, §3; FR-USER-01 | UC-USER-01 |
| BR-USER-02 | FR-USER-01, FR-USER-02 | UC-USER-01, UC-USER-02 |
| BR-CLASS-01 | CLAUDE.md §3; FR-CLASS-01 | UC-CLASS-01 |
| BR-CLASS-02 | FR-CLASS-01; FR-STU-01 | UC-CLASS-01 |
| BR-STUDENT-01 | FR-STU-01 | UC-STU-01 |
| BR-STUDENT-02 | FR-STU-01; FR-ATT-01 | UC-STU-01, UC-CLASS-01, UC-ATT-01 |
| BR-TEACHER-01 | FR-TEACHER-01 | UC-TEACHER-01 |
| BR-TEACHER-02 | FR-TEACHER-01 | UC-TEACHER-01, UC-CLASS-01 |
| BR-ATTENDANCE-01 | FR-ATT-01 | UC-ATT-01 |
| BR-ATTENDANCE-02 | FR-ATT-01 | UC-ATT-01 |
| BR-ATTENDANCE-03 | FR-ATT-02 | UC-ATT-02 |
| BR-PICKUP-01 | FR-PICKUP-01 | UC-PICKUP-01 |
| BR-PICKUP-02 | FR-PICKUP-02, FR-PICKUP-03 | UC-PICKUP-01, UC-PICKUP-02 |
| BR-PICKUP-03 | FR-PICKUP-03 | UC-PICKUP-02 |
| BR-HEALTH-01 | FR-HEALTH-01 | UC-HEALTH-01 |
| BR-HEALTH-02 | FR-HEALTH-02, FR-HEALTH-03 | UC-HEALTH-02 |
| BR-HEALTH-03 | FR-HEALTH-01, FR-HEALTH-02, FR-HEALTH-03 | UC-HEALTH-01, UC-HEALTH-02 |
| BR-HEALTH-04 | FR-HEALTH-04 | UC-HEALTH-03 |
| BR-INCIDENT-01 | FR-INCIDENT-01 | UC-INCIDENT-01 |
| BR-INCIDENT-02 | FR-INCIDENT-01 | UC-INCIDENT-01 |
| BR-INCIDENT-03 | FR-INCIDENT-02 | UC-INCIDENT-01 |
| BR-INCIDENT-04 | FR-INCIDENT-01; FR-HEALTH-04 | UC-INCIDENT-01, UC-HEALTH-03 |
| BR-MEDIA-01 | FR-MEDIA-01 | UC-MEDIA-01 |
| BR-MEDIA-02 | FR-MEDIA-02 | UC-MEDIA-02 |
| BR-TUITION-01 | FR-TUITION-01 | UC-TUITION-01 |
| BR-TUITION-02 | FR-TUITION-02 | UC-TUITION-02 |
| BR-TUITION-03 | FR-TUITION-01 | UC-TUITION-02 |
| BR-TUITION-04 | FR-TUITION-05 | UC-TUITION-04 |
| BR-TUITION-05 | FR-TUITION-06 | UC-TUITION-05 |
| BR-TUITION-06 | FR-TUITION-06 | UC-TUITION-05 |
| BR-TUITION-07 | FR-TUITION-03 | UC-TUITION-03 |
| BR-TUITION-08 | FR-TUITION-03; FR-DASH-04 | UC-TUITION-03, UC-DASH-01 |
| BR-MENU-01 | FR-MENU-01 | UC-MENU-01 |
| BR-NOTIFICATION-01 | FR-NOTI-01 | UC-NOTI-01 |
| BR-DASHBOARD-01 | FR-DASH-01–04 | UC-DASH-01 |
| BR-DASHBOARD-02 | FR-DASH-01–04 | UC-DASH-01 |
