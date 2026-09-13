# Functional Requirements — Kindergarten Management System

## 0. Document Information

- **Source of truth:** The approved project specification, as recorded in `CLAUDE.md` §1–§3 (Project Overview, User Roles, Main Functional Requirements). `CLAUDE.md` and the approved project specification are treated as the same scope source for this document; no requirement below goes beyond what is stated there.
- **Status:** Draft — for review and approval before Business Rules, Use Cases, Database Design, and API Design proceed.
- **Out of scope for this document:** database schema, API endpoints, UI screens, authentication mechanism, payment provider, storage provider, notification technology. These are explicitly unresolved (see `DECISIONS.md`) and must not be decided here.
- **Priority values** (High / Medium / Low) are the author's proposed prioritization for planning purposes only, since the specification does not define priority. They require confirmation and are not a business rule.
- **Convention used in this document**, to keep explicit spec content, reasonable inference, and unresolved detail clearly separate:
  - **Description** paraphrases only what the specification states, with a citation to the source bullet.
  - **Preconditions** are reasonable system preconditions needed for the described flow to make sense (e.g., "a record must already exist to be viewed"). They are inferred, not quoted from the specification, and are not business rules.
  - **Business Rules** are included only when the specification explicitly states or directly implies a constraint (e.g., wording such as "của con" scoping a view to the parent's own child). A plausible-but-unstated constraint is never recorded here.
  - **Open Questions** record details the specification does not define. They are not resolved by this document and must not be treated as decided.

---

## 1. Actors

| Actor | Description (per specification) |
|---|---|
| Admin / Ban giám hiệu | Manages users, permissions, class assignment by age, and views school-wide dashboards/statistics. |
| Giáo viên | Takes daily attendance, handles pickup attendance, records quick health status and incidents, shares activity photos. |
| Kế toán / Văn phòng | Manages tuition, invoices, weekly menu, child enrollment records, teacher HR records, and revenue reporting. |
| Y tế | Tracks children's height and weight, and reviews health/incident history within the specified scope. |
| Phụ huynh | Views their child's attendance history, activity media, notifications, invoices; pays tuition online; manages registered pickup persons. |

Each role may only access functionality appropriate to its own permissions (CLAUDE.md §2). No role's permissions may be expanded beyond what is specified without approval.

---

## 2. Cross-Cutting Rules

- **CC-01 — Role-based access:** Every functional requirement below is restricted to the actor(s) listed for it. A user must not be able to perform actions belonging to another role.
- **CC-02 — No unapproved scope expansion:** Functionality not explicitly listed in the specification (e.g., additional Y tế features) must not be added without approval (CLAUDE.md §3).
- **CC-03 — No invented technical decisions:** Authentication strategy, payment provider, media storage provider, and notification mechanism are pending decisions (see `DECISIONS.md` D20–D22, D25) and are referenced but not resolved by this document.

---

## 3. Module: Authentication & Access (FR-AUTH)

### FR-AUTH-01 — User Login
- **Actor:** All roles
- **Description:** The system must allow a registered user to log in and be identified by their role so that role-appropriate functionality is exposed.
- **Main behavior / expected flow:** User submits credentials → system authenticates the user → system establishes the user's session/identity and role.
- **Outputs / expected result:** User gains access to the functionality permitted for their role.
- **Priority:** High
- **Note:** The need for authentication itself is grounded in `CLAUDE.md` §5 (Authentication & Authorization) and §2 (role-restricted access); this requirement does not invent a mechanism.
- **Open Question:** Authentication mechanism (session, token, provider) is not finalized — see `DECISIONS.md` D20 and `CLAUDE.md` §5. Login/logout/password-reset flow details are not specified.

### FR-AUTH-02 — Role-Based Access Restriction
- **Actor:** All roles
- **Description:** The system must restrict each authenticated user to the functionality assigned to their role.
- **Business Rules:**
  - A role may only access functionality listed for that role in the specification (CLAUDE.md §2).
- **Priority:** High
- **Open Question:** Whether authorization is role-only or role + fine-grained permission (see FR-USER-02) is not finalized.

---

## 4. Module: User & Permission Management (FR-USER)

### FR-USER-01 — Manage User Accounts
- **Actor:** Admin / Ban giám hiệu
- **Description:** Admin manages user accounts in the system (CLAUDE.md §3, "Quản lý người dùng").
- **Main behavior / expected flow:** Admin views, creates, and manages accounts for users of the system.
- **Outputs / expected result:** User accounts exist and are associated with a role.
- **Priority:** High
- **Open Question:** Exact account lifecycle actions supported (e.g., deactivation vs. deletion) are not detailed in the specification.

### FR-USER-02 — Manage Roles / Permissions
- **Actor:** Admin / Ban giám hiệu
- **Description:** Admin manages permissions in the system (CLAUDE.md §3, "Quản lý quyền").
- **Business Rules:**
  - Permissions must be role-based; permissions must not be hard-coded in multiple places (CLAUDE.md §5).
  - No role may be granted functionality beyond the specification without approval.
- **Priority:** High
- **Open Question:** Whether permission management is limited to the 5 fixed roles, or supports finer-grained/custom permission sets per user, is not specified.

---

## 5. Module: Class Management (FR-CLASS)

### FR-CLASS-01 — Manage Classes by Age Group
- **Actor:** Admin / Ban giám hiệu
- **Description:** Admin classifies/organizes children into classes according to age group (CLAUDE.md §3, "Phân lớp theo độ tuổi").
- **Main behavior / expected flow:** Admin defines classes and assigns children to a class based on age.
- **Outputs / expected result:** Each child is associated with one class appropriate to their age group.
- **Priority:** High
- **Open Question:** Age band boundaries, number of classes, and whether teacher-to-class assignment is part of this requirement or a separate unspecified feature are not defined in the specification.

---

## 6. Module: Student / Child Management (FR-STU)

### FR-STU-01 — Manage Child Enrollment Records
- **Actor:** Kế toán / Văn phòng
- **Description:** Kế toán manages children's enrollment records (CLAUDE.md §3, "Quản lý hồ sơ nhập học của trẻ").
- **Main behavior / expected flow:** Kế toán creates and maintains a child's enrollment record when the child joins the school.
- **Outputs / expected result:** An enrollment record exists for each child, usable by other modules (class assignment, attendance, health, tuition).
- **Priority:** High
- **Open Question:** Required enrollment fields (e.g., required documents, parent/guardian relationship model, student status such as active/withdrawn) are not specified in the specification.

---

## 7. Module: Teacher HR Management (FR-TEACHER)

### FR-TEACHER-01 — Manage Teacher Records
- **Actor:** Kế toán / Văn phòng
- **Description:** Kế toán manages teacher HR records (CLAUDE.md §3, "Quản lý hồ sơ giáo viên").
- **Main behavior / expected flow:** Kế toán creates and maintains teacher profile/employment records.
- **Outputs / expected result:** A teacher record exists, usable for class assignment.
- **Priority:** Medium
- **Open Question:** Specific HR fields (contract, qualifications, etc.) are not specified.

---

## 8. Module: Attendance (FR-ATT)

### FR-ATT-01 — Teacher Check-In Attendance
- **Actor:** Giáo viên
- **Description:** Teacher records attendance for students when they arrive at class (CLAUDE.md §3, "Điểm danh học sinh khi đến lớp").
- **Preconditions:** The child has an enrollment record and is assigned to the teacher's class.
- **Main behavior / expected flow:** Teacher marks each student's attendance status upon arrival.
- **Inputs:** Student identity, attendance status, time of check-in.
- **Outputs / expected result:** An attendance record is created for the student for that day.
- **Priority:** High
- **Open Question:** The set of valid attendance statuses (e.g., Present / Absent / Late / Excused) is not finalized — see `DECISIONS.md` D23.

### FR-ATT-02 — Parent View Attendance History
- **Actor:** Phụ huynh
- **Description:** Parent views their child's attendance history (CLAUDE.md §3, "Xem lịch sử điểm danh của con").
- **Preconditions:** Parent is linked to the child's record.
- **Main behavior / expected flow:** Parent requests and views a history of the child's recorded attendance.
- **Outputs / expected result:** Parent sees prior attendance entries for their own child only.
- **Business Rules:** A parent may only view attendance history for their own linked child/children — grounded in the specification's wording "của con" (of their own child) (CLAUDE.md §3).
- **Priority:** High

---

## 9. Module: Pickup Attendance & Registered Pickup Persons (FR-PICKUP)

### FR-PICKUP-01 — Teacher Pickup Attendance
- **Actor:** Giáo viên
- **Description:** Teacher records attendance when a parent (or authorized person) picks up the child (CLAUDE.md §3, "Điểm danh khi phụ huynh đón trẻ").
- **Main behavior / expected flow:** Teacher marks the child as picked up at the time of departure.
- **Outputs / expected result:** A pickup record is created, associated with the child and the day's attendance.
- **Priority:** High

### FR-PICKUP-02 — Support Pre-Registered Pickup Person
- **Actor:** Giáo viên
- **Description:** The system must support a teacher recognizing/handling a pickup by a person the parent has pre-registered (CLAUDE.md §3, "Hỗ trợ người đón trẻ đã được phụ huynh đăng ký trước").
- **Preconditions:** The pickup person has been registered in advance by the child's parent (see FR-PICKUP-03).
- **Main behavior / expected flow:** Teacher checks the person picking up the child against the child's registered pickup person list.
- **Outputs / expected result:** Teacher can confirm whether the person is an authorized, pre-registered pickup person for that child.
- **Priority:** High
- **Open Question:** The verification method (visual/manual by teacher, ID check, code, etc.) is not specified.

### FR-PICKUP-03 — Parent Manage Registered Pickup Persons
- **Actor:** Phụ huynh
- **Description:** Parent registers and manages the list of people authorized to pick up their child (CLAUDE.md §3, "Quản lý / đăng ký người đón trẻ").
- **Main behavior / expected flow:** Parent adds, views, and manages pickup persons associated with their child.
- **Outputs / expected result:** A list of registered pickup persons exists per child, usable by FR-PICKUP-02.
- **Priority:** High
- **Open Question:** Required data fields for a registered pickup person (relationship, phone number, identity/verification document, photo, expiration/validity period) are not finalized — see `DECISIONS.md` D24.

---

## 10. Module: Health Monitoring (FR-HEALTH)

### FR-HEALTH-01 — Teacher Quick Health Status Entry
- **Actor:** Giáo viên
- **Description:** Teacher records a quick note on a child's health status (CLAUDE.md §3, "Ghi nhận nhanh tình trạng sức khỏe").
- **Main behavior / expected flow:** Teacher records a brief health status observation for a child.
- **Outputs / expected result:** A quick health status entry is stored against the child's record.
- **Priority:** Medium
- **Open Question:** The specific fields captured in a "quick" health status (e.g., temperature, mood, appetite) are not defined in the specification.

### FR-HEALTH-02 — Track Height
- **Actor:** Y tế
- **Description:** Y tế tracks a child's height over time (CLAUDE.md §3, "Theo dõi chiều cao").
- **Main behavior / expected flow:** Y tế records height measurements for a child at points in time.
- **Outputs / expected result:** A history of height measurements exists per child.
- **Priority:** Medium
- **Open Question:** Measurement frequency/schedule is not specified.

### FR-HEALTH-03 — Track Weight
- **Actor:** Y tế
- **Description:** Y tế tracks a child's weight over time (CLAUDE.md §3, "Theo dõi cân nặng").
- **Main behavior / expected flow:** Y tế records weight measurements for a child at points in time.
- **Outputs / expected result:** A history of weight measurements exists per child.
- **Priority:** Medium
- **Open Question:** Measurement frequency/schedule is not specified.

### FR-HEALTH-04 — View Health / Incident History
- **Actor:** Y tế
- **Description:** Y tế reviews the history of incidents and health issues related to a child, within the scope defined by the specification (CLAUDE.md §3, "Theo dõi lịch sử sự cố / vấn đề sức khỏe liên quan trong phạm vi specification").
- **Main behavior / expected flow:** Y tế views the recorded incident history (created per FR-INCIDENT-01) and health records for a child.
- **Outputs / expected result:** Y tế can see a chronological history of incidents/health issues for a child.
- **Business Rules:** Health-related functionality must not extend beyond what is specified without approval (CLAUDE.md §3, explicit restriction on Y tế scope).
- **Priority:** Medium

---

## 11. Module: Incident Recording (FR-INCIDENT)

### FR-INCIDENT-01 — Record Incident / Accident
- **Actor:** Giáo viên
- **Description:** Teacher records an incident or accident involving a child (CLAUDE.md §3, "Ghi nhận sự cố / tai nạn").
- **Main behavior / expected flow:** Teacher creates an incident record describing what occurred.
- **Outputs / expected result:** An incident record is created and becomes part of the child's incident history (viewable per FR-HEALTH-04).
- **Priority:** High

### FR-INCIDENT-02 — Attach Incident Photos
- **Actor:** Giáo viên
- **Description:** Teacher may attach photos to an incident record (CLAUDE.md §3, "Có thể đính kèm hình ảnh sự cố").
- **Preconditions:** An incident record has been created (FR-INCIDENT-01).
- **Main behavior / expected flow:** Teacher optionally attaches one or more photos to the incident record.
- **Outputs / expected result:** Photo(s) are associated with the incident record.
- **Priority:** Medium
- **Open Question:** Media storage provider is not finalized (`DECISIONS.md` D22); limits on number/size of photos are not specified.

---

## 12. Module: Activity Media Sharing (FR-MEDIA)

### FR-MEDIA-01 — Teacher Share Activity Photos by Group / Class
- **Actor:** Giáo viên
- **Description:** Teacher shares activity photos with a group or class (CLAUDE.md §3, "Chia sẻ hình ảnh hoạt động theo nhóm / lớp").
- **Main behavior / expected flow:** Teacher uploads/shares one or more activity photos scoped to a specific group or class.
- **Outputs / expected result:** Shared photos become visible to parents of children in that group/class (see FR-MEDIA-02).
- **Priority:** Medium
- **Open Question:** Media storage provider is not finalized (`DECISIONS.md` D22).

### FR-MEDIA-02 — Parent View Activity Photos / Videos
- **Actor:** Phụ huynh
- **Description:** Parent views photos/videos of their child's activities (CLAUDE.md §3, "Xem hình ảnh / video hoạt động").
- **Main behavior / expected flow:** Parent views activity media relevant to their child's class/group.
- **Preconditions:** Parent is linked to the child whose class/group media is being viewed.
- **Outputs / expected result:** Parent sees photos/videos shared for their child's class.
- **Priority:** Medium
- **Open Question:** The specification lists teachers as sharing "hình ảnh" (photos) but parents as viewing "hình ảnh / video" (photos/videos). Whether teachers can upload video, or video comes from another unspecified source, is not clarified.

---

## 13. Module: Tuition, Invoices & Payment (FR-TUITION)

### FR-TUITION-01 — Manage Tuition Fees
- **Actor:** Kế toán / Văn phòng
- **Description:** Kế toán manages tuition fees (CLAUDE.md §3, "Quản lý học phí").
- **Outputs / expected result:** Tuition fee information exists and can be used to generate invoices.
- **Priority:** High
- **Open Question:** Fee structure (per class, per age group, per service) is not specified.

### FR-TUITION-02 — Manage Invoices
- **Actor:** Kế toán / Văn phòng
- **Description:** Kế toán manages invoices (CLAUDE.md §3, "Quản lý hóa đơn").
- **Outputs / expected result:** Invoices exist per child/period and are viewable by parents (FR-TUITION-05) and payable online (FR-TUITION-06).
- **Priority:** High

### FR-TUITION-03 — Revenue Report
- **Actor:** Kế toán / Văn phòng
- **Description:** Kế toán generates revenue reports (CLAUDE.md §3, "Báo cáo doanh thu").
- **Outputs / expected result:** A revenue report is produced, and feeds the Admin dashboard revenue statistics (FR-DASH-04).
- **Priority:** Medium
- **Open Question:** Report period/breakdown granularity is not specified.

### FR-TUITION-04 — Export Report (Excel/PDF)
- **Actor:** Kế toán / Văn phòng
- **Description:** Kế toán exports reports to Excel/PDF (CLAUDE.md §3, "Hỗ trợ xuất báo cáo Excel/PDF").
- **Outputs / expected result:** A report is produced in Excel and/or PDF format.
- **Priority:** Low
- **Open Question:** Which specific reports must support export (revenue report only, or invoices too) is not specified.

### FR-TUITION-05 — Parent View Invoice
- **Actor:** Phụ huynh
- **Description:** Parent views their child's tuition invoice (CLAUDE.md §3, "Xem hóa đơn học phí").
- **Preconditions:** Parent is linked to the child whose invoice is being viewed.
- **Priority:** High

### FR-TUITION-06 — Parent Online Payment
- **Actor:** Phụ huynh
- **Description:** Parent pays tuition online (CLAUDE.md §3, "Thanh toán học phí online").
- **Preconditions:** An unpaid invoice exists for the child (FR-TUITION-02).
- **Outputs / expected result:** Payment status of the invoice is updated upon successful payment.
- **Priority:** High
- **Open Question:** Online payment provider is not selected — see `DECISIONS.md` D21.

---

## 14. Module: Weekly Menu Management (FR-MENU)

### FR-MENU-01 — Manage Weekly Menu
- **Actor:** Kế toán / Văn phòng
- **Description:** Kế toán manages the weekly menu (CLAUDE.md §3, "Quản lý thực đơn tuần").
- **Outputs / expected result:** A weekly menu exists for the school.
- **Priority:** Medium
- **Open Question:** The specification does not state which role(s), if any, view the weekly menu (e.g., parents or teachers) — only that Kế toán manages it.

---

## 15. Module: Notifications (FR-NOTI)

### FR-NOTI-01 — Parent Receive Notifications
- **Actor:** Phụ huynh
- **Description:** Parent receives notifications from the system (CLAUDE.md §3, "Nhận thông báo").
- **Outputs / expected result:** Parent is informed of relevant events via notification.
- **Priority:** Medium
- **Open Question:** Notification triggers/events, delivery channel (in-app, email, push), and whether roles other than Phụ huynh receive notifications are not specified — see `DECISIONS.md` D25.

---

## 16. Module: Admin Dashboard & Statistics (FR-DASH)

### FR-DASH-01 — Overview Dashboard
- **Actor:** Admin / Ban giám hiệu
- **Description:** Admin views a general overview dashboard (CLAUDE.md §3, "Dashboard tổng quan").
- **Priority:** Medium

### FR-DASH-02 — Child Count Statistics
- **Actor:** Admin / Ban giám hiệu
- **Description:** Admin views statistics on the number of children (CLAUDE.md §3, "Thống kê số lượng trẻ").
- **Priority:** Medium

### FR-DASH-03 — Attendance Statistics
- **Actor:** Admin / Ban giám hiệu
- **Description:** Admin views attendance statistics (CLAUDE.md §3, "Thống kê điểm danh").
- **Priority:** Medium

### FR-DASH-04 — Revenue Statistics
- **Actor:** Admin / Ban giám hiệu
- **Description:** Admin views revenue statistics (CLAUDE.md §3, "Thống kê doanh thu"), sourced from FR-TUITION-03.
- **Priority:** Medium
- **Open Question:** Specific breakdowns/time ranges for all dashboard statistics (FR-DASH-02 to 04) are not specified.

---

## 17. Open Questions

The following requirements-related ambiguities need clarification before proceeding to Business Rules / Use Cases / Database Design:

1. **(FR-AUTH-01, FR-AUTH-02)** Authentication mechanism and login/logout/password flows are not finalized (`DECISIONS.md` D20).
2. **(FR-USER-02)** Whether permission management is role-only or supports finer-grained/custom permissions per user.
3. **(FR-CLASS-01)** Age band boundaries/number of classes, and whether teacher-to-class assignment belongs to this requirement.
4. **(FR-STU-01)** Required enrollment fields, parent/guardian relationship model, and student status values.
5. **(FR-ATT-01, FR-ATT-02)** Final list of valid attendance statuses (`DECISIONS.md` D23).
6. **(FR-PICKUP-02, FR-PICKUP-03)** Required data fields for a registered pickup person and the verification method used at pickup (`DECISIONS.md` D24).
7. **(FR-HEALTH-01)** Specific fields captured by "quick health status."
8. **(FR-HEALTH-02, FR-HEALTH-03)** Measurement frequency/schedule for height and weight tracking.
9. **(FR-INCIDENT-02, FR-MEDIA-01)** Media storage provider is not finalized (`DECISIONS.md` D22); upload limits are not specified.
10. **(FR-MEDIA-01, FR-MEDIA-02)** Teacher-side spec mentions only photos; parent-side spec mentions photos and videos. Whether teachers may upload video is unclear.
11. **(FR-TUITION-01)** Tuition fee structure (per class/age group/service) is not specified.
12. **(FR-TUITION-04)** Which specific reports require Excel/PDF export.
13. **(FR-TUITION-06)** Online payment provider is not selected (`DECISIONS.md` D21).
14. **(FR-MENU-01)** Whether any role (e.g., Phụ huynh) views the weekly menu is not stated — the specification only mentions that Kế toán manages it.
15. **(FR-NOTI-01)** Notification triggers, delivery channel/technology, and whether roles other than Phụ huynh receive notifications (`DECISIONS.md` D25).
16. **(FR-DASH-02–04)** Specific statistical breakdowns and time ranges for dashboard metrics.
17. **(FR-MEDIA-02, FR-TUITION-05, FR-NOTI-01)** The specification explicitly scopes attendance history to "của con" (the parent's own child), but does not use equivalent wording for activity media, invoices, or notifications. Whether the same own-child-only scoping applies uniformly to all parent-facing views, or needs separate confirmation per feature, is not stated.
