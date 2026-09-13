# Use Cases — Kindergarten Management System

## 0. Document Information

- **Source of truth:** `CLAUDE.md` §2–§3, `docs/requirements/functional-requirements.md`, and `docs/business/user-roles.md`. No use case below introduces a goal, actor, flow, or rule not traceable to those documents.
- **Status:** Draft — for review and approval before Database Design and API Design proceed.
- **Out of scope for this document:** controllers, API endpoints, database tables, authentication mechanism, UI components, or any other implementation detail. Use cases describe *how an actor interacts with the system from a business perspective*, not how the system is built.
- **Convention used in this document:**
  - A use case is defined only when the source documents describe enough of a coherent actor interaction to support one. A functional area with only a one-line, undetailed feature (e.g., "manage X") still yields a minimal use case, but its flow is kept short rather than invented.
  - Where a functional requirement is a small optional extension of another (e.g., attaching a photo to an incident, exporting a report already generated), it is folded into the main flow of the primary use case as a step or alternative flow, rather than becoming a separate use case with a duplicate flow.
  - Preconditions, flows, and postconditions restate only what the source documents say or directly imply. Where a plausible detail is not stated, it is left out and — if relevant to completing the use case — recorded under Open Questions instead of being guessed.
  - This document does not resolve any Open Question already recorded in `functional-requirements.md` or `user-roles.md`; such items are referenced, not repeated in full or answered.

---

## 1. Authentication

### UC-AUTH-01 — Log In
- **Primary Actor:** Any role (Admin / Ban giám hiệu, Giáo viên, Kế toán / Văn phòng, Y tế, Phụ huynh)
- **Supporting Actors:** None specified.
- **Goal:** Gain access to the functionality permitted for the actor's role.
- **Preconditions:** The actor has a registered account associated with a role.
- **Main Flow:**
  1. The actor submits their credentials to the system.
  2. The system authenticates the actor and identifies their role.
  3. The system grants the actor access to the functionality permitted for that role.
- **Alternative / Exception Flows:**
  - **E1 — Invalid credentials:** The system does not grant access. The specific handling (error message, retry limit, password reset) is not defined — see Open Questions.
  - **E2 — Action outside role:** An authenticated actor attempts an action not permitted for their role; the system denies the action (FR-AUTH-02).
- **Postconditions:** The actor is recognized by the system with their role for the remainder of the session.
- **Related FR IDs:** FR-AUTH-01, FR-AUTH-02

---

## 2. User & Permission Management

### UC-USER-01 — Manage User Accounts
- **Primary Actor:** Admin / Ban giám hiệu
- **Supporting Actors:** None specified.
- **Goal:** Maintain the set of user accounts in the system.
- **Preconditions:** Actor is authenticated as Admin.
- **Main Flow:**
  1. Admin opens the list of user accounts.
  2. Admin creates or updates a user account.
  3. The system stores the account, associated with a role.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** A user account exists and is available for login (UC-AUTH-01).
- **Related FR IDs:** FR-USER-01

### UC-USER-02 — Manage Roles / Permissions
- **Primary Actor:** Admin / Ban giám hiệu
- **Supporting Actors:** None specified.
- **Goal:** Maintain what each role is permitted to do in the system.
- **Preconditions:** Actor is authenticated as Admin.
- **Main Flow:**
  1. Admin opens the permission configuration for a role.
  2. Admin updates the permission configuration.
  3. The system applies the updated configuration to that role.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Role permissions reflect the update, within the bound that no role may be expanded beyond the approved scope (CLAUDE.md §2).
- **Related FR IDs:** FR-USER-02

---

## 3. Class Management

### UC-CLASS-01 — Assign Children to Classes by Age Group
- **Primary Actor:** Admin / Ban giám hiệu
- **Supporting Actors:** None specified.
- **Goal:** Organize enrolled children into classes appropriate to their age.
- **Preconditions:** The child has an enrollment record (FR-STU-01).
- **Main Flow:**
  1. Admin reviews children needing a class assignment.
  2. Admin assigns a child to a class matching their age group.
  3. The system records the child's class assignment.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Each assigned child is associated with one class appropriate to their age group.
- **Related FR IDs:** FR-CLASS-01

---

## 4. Child Enrollment

### UC-STU-01 — Manage Child Enrollment Record
- **Primary Actor:** Kế toán / Văn phòng
- **Supporting Actors:** None specified.
- **Goal:** Establish and maintain a child's enrollment record when the child joins the school.
- **Preconditions:** None specified beyond the actor being authenticated as Kế toán.
- **Main Flow:**
  1. Kế toán creates an enrollment record for a child joining the school.
  2. Kế toán updates the record as needed over time.
  3. The system stores the enrollment record for use by other modules (class assignment, attendance, health, tuition).
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** An enrollment record exists for the child.
- **Related FR IDs:** FR-STU-01

---

## 5. Teacher Management

### UC-TEACHER-01 — Manage Teacher Record
- **Primary Actor:** Kế toán / Văn phòng
- **Supporting Actors:** None specified.
- **Goal:** Establish and maintain a teacher's HR record.
- **Preconditions:** None specified beyond the actor being authenticated as Kế toán.
- **Main Flow:**
  1. Kế toán creates a teacher record.
  2. Kế toán updates the record as needed over time.
  3. The system stores the teacher record for use by class assignment.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** A teacher record exists.
- **Related FR IDs:** FR-TEACHER-01

---

## 6. Attendance

### UC-ATT-01 — Record Check-In Attendance
- **Primary Actor:** Giáo viên
- **Supporting Actors:** None specified.
- **Goal:** Record which children are present in class on a given day.
- **Preconditions:** The child has an enrollment record and is assigned to the teacher's class (FR-ATT-01).
- **Main Flow:**
  1. A child arrives at class.
  2. The teacher marks the child's attendance status.
  3. The system creates an attendance record for that child for the day.
- **Alternative / Exception Flows:** None specified. (The valid set of attendance statuses is not finalized — see Open Questions.)
- **Postconditions:** An attendance record exists for the child for that day.
- **Related FR IDs:** FR-ATT-01

### UC-ATT-02 — View Child's Attendance History
- **Primary Actor:** Phụ huynh
- **Supporting Actors:** None specified.
- **Goal:** Review a child's past attendance.
- **Preconditions:** The parent is linked to the child's record; attendance records exist (UC-ATT-01).
- **Main Flow:**
  1. Parent requests their child's attendance history.
  2. The system retrieves and displays prior attendance entries for that child.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Parent sees attendance history limited to their own child (FR-ATT-02 Business Rule — "của con").
- **Related FR IDs:** FR-ATT-02

---

## 7. Pickup

### UC-PICKUP-01 — Record Pickup Attendance
- **Primary Actor:** Giáo viên
- **Supporting Actors:** Phụ huynh (as the registrant of the pickup person being checked — FR-PICKUP-03).
- **Goal:** Record that a child has been picked up, and confirm the person picking up is authorized when they are not the parent.
- **Preconditions:** The child was checked in for the day (UC-ATT-01).
- **Main Flow:**
  1. A person arrives to pick up the child.
  2. If the person is not the parent, the teacher checks them against the child's list of registered pickup persons (FR-PICKUP-03).
  3. The teacher confirms the pickup.
  4. The system creates a pickup record associated with the child and the day's attendance.
- **Alternative / Exception Flows:**
  - **E1 — Person not on the registered list:** Not specified in the source documents what the teacher should do in this case — see Open Questions.
- **Postconditions:** A pickup record exists, associated with the child and the day's attendance.
- **Related FR IDs:** FR-PICKUP-01, FR-PICKUP-02

### UC-PICKUP-02 — Manage Registered Pickup Persons
- **Primary Actor:** Phụ huynh
- **Supporting Actors:** None specified.
- **Goal:** Maintain the list of people authorized to pick up their child.
- **Preconditions:** Parent is linked to the child's record.
- **Main Flow:**
  1. Parent opens the list of registered pickup persons for their child.
  2. Parent adds or updates a pickup person.
  3. The system stores the updated list, available for verification during pickup (UC-PICKUP-01).
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** The child's list of registered pickup persons reflects the update.
- **Related FR IDs:** FR-PICKUP-03

---

## 8. Health Monitoring

### UC-HEALTH-01 — Record Quick Health Status
- **Primary Actor:** Giáo viên
- **Supporting Actors:** None specified.
- **Goal:** Record a brief observation of a child's health status during the day.
- **Preconditions:** None specified beyond the actor being authenticated as Giáo viên.
- **Main Flow:**
  1. Teacher observes something worth noting about a child's health.
  2. Teacher records a brief health status entry for the child.
  3. The system stores the entry against the child's record.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** A quick health status entry is stored for the child. (Specific fields captured are not defined — see Open Questions.)
- **Related FR IDs:** FR-HEALTH-01

### UC-HEALTH-02 — Record Growth Measurement
- **Primary Actor:** Y tế
- **Supporting Actors:** None specified.
- **Goal:** Track a child's height and weight over time.
- **Preconditions:** None specified beyond the actor being authenticated as Y tế.
- **Main Flow:**
  1. Y tế measures a child's height and/or weight.
  2. Y tế records the measurement for the child.
  3. The system stores the measurement as part of that child's growth history.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** A height and/or weight measurement is added to the child's history. (Measurement frequency is not defined — see Open Questions.)
- **Related FR IDs:** FR-HEALTH-02, FR-HEALTH-03

### UC-HEALTH-03 — Review Health & Incident History
- **Primary Actor:** Y tế
- **Supporting Actors:** None specified.
- **Goal:** Review a child's history of incidents and health issues within the approved scope.
- **Preconditions:** Incident and/or health records exist for the child (UC-INCIDENT-01, UC-HEALTH-02).
- **Main Flow:**
  1. Y tế selects a child.
  2. The system displays that child's chronological incident and health history.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Y tế has reviewed the available history. This use case must not be extended with health functionality outside the approved scope without approval (CLAUDE.md §3; FR-HEALTH-04 Business Rule).
- **Related FR IDs:** FR-HEALTH-04

---

## 9. Incident Management

### UC-INCIDENT-01 — Record Incident / Accident
- **Primary Actor:** Giáo viên
- **Supporting Actors:** None specified.
- **Goal:** Document an incident or accident involving a child.
- **Preconditions:** None specified beyond the actor being authenticated as Giáo viên.
- **Main Flow:**
  1. An incident or accident involving a child occurs.
  2. The teacher creates an incident record describing what happened.
  3. The teacher may optionally attach one or more photos to the record.
  4. The system stores the incident record, becoming part of the child's incident history (available to Y tế via UC-HEALTH-03).
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** An incident record exists for the child, with any attached photos.
- **Related FR IDs:** FR-INCIDENT-01, FR-INCIDENT-02

---

## 10. Activity Media

### UC-MEDIA-01 — Share Activity Photos
- **Primary Actor:** Giáo viên
- **Supporting Actors:** None specified.
- **Goal:** Share photos of classroom activities with parents of a group or class.
- **Preconditions:** None specified beyond the actor being authenticated as Giáo viên.
- **Main Flow:**
  1. Teacher selects one or more activity photos to share.
  2. Teacher scopes the sharing to a specific group or class.
  3. The system stores the photos, making them visible to parents of children in that group/class (UC-MEDIA-02).
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** The shared photos are available to the relevant parents.
- **Related FR IDs:** FR-MEDIA-01

### UC-MEDIA-02 — View Activity Photos / Videos
- **Primary Actor:** Phụ huynh
- **Supporting Actors:** None specified.
- **Goal:** View photos/videos of their child's classroom activities.
- **Preconditions:** Parent is linked to the child whose class/group media is being viewed.
- **Main Flow:**
  1. Parent opens the activity media for their child's class.
  2. The system displays the photos/videos shared for that class.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Parent has viewed the available media for their child's class. (Whether this is strictly limited to the parent's own child's class, as it is for attendance, is not explicitly confirmed — see Open Questions.)
- **Related FR IDs:** FR-MEDIA-02

---

## 11. Tuition & Invoices

### UC-TUITION-01 — Manage Tuition Fees
- **Primary Actor:** Kế toán / Văn phòng
- **Supporting Actors:** None specified.
- **Goal:** Maintain tuition fee information used to generate invoices.
- **Preconditions:** None specified beyond the actor being authenticated as Kế toán.
- **Main Flow:**
  1. Kế toán opens tuition fee information.
  2. Kế toán creates or updates a tuition fee.
  3. The system stores the fee information for use in invoicing (UC-TUITION-02).
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Tuition fee information is available for generating invoices. (Fee structure is not defined — see Open Questions.)
- **Related FR IDs:** FR-TUITION-01

### UC-TUITION-02 — Manage Invoices
- **Primary Actor:** Kế toán / Văn phòng
- **Supporting Actors:** None specified.
- **Goal:** Maintain invoices for children's tuition.
- **Preconditions:** Tuition fee information exists (UC-TUITION-01).
- **Main Flow:**
  1. Kế toán creates or updates an invoice for a child.
  2. The system stores the invoice, making it available to the parent (UC-TUITION-04) and payable online (UC-TUITION-05).
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** An invoice exists for the child.
- **Related FR IDs:** FR-TUITION-02

### UC-TUITION-03 — Generate Revenue Report
- **Primary Actor:** Kế toán / Văn phòng
- **Supporting Actors:** None specified.
- **Goal:** Produce a report of revenue, optionally exported for sharing outside the system.
- **Preconditions:** None specified beyond the actor being authenticated as Kế toán.
- **Main Flow:**
  1. Kế toán requests a revenue report.
  2. The system generates the report.
  3. Kế toán may optionally export the report to Excel or PDF.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** A revenue report exists and may feed the Admin dashboard (UC-DASH-01). (Report period/breakdown and exact export scope are not defined — see Open Questions.)
- **Related FR IDs:** FR-TUITION-03, FR-TUITION-04

### UC-TUITION-04 — View Tuition Invoice
- **Primary Actor:** Phụ huynh
- **Supporting Actors:** None specified.
- **Goal:** Review the tuition invoice for their child.
- **Preconditions:** Parent is linked to the child whose invoice is being viewed; an invoice exists (UC-TUITION-02).
- **Main Flow:**
  1. Parent requests their child's invoice.
  2. The system displays the invoice.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Parent has viewed the invoice. (Whether viewing is strictly limited to the parent's own child, as it is for attendance, is not explicitly confirmed — see Open Questions.)
- **Related FR IDs:** FR-TUITION-05

### UC-TUITION-05 — Pay Tuition Online
- **Primary Actor:** Phụ huynh
- **Supporting Actors:** None specified.
- **Goal:** Settle a tuition invoice online.
- **Preconditions:** An unpaid invoice exists for the child (UC-TUITION-02).
- **Main Flow:**
  1. Parent selects an unpaid invoice.
  2. Parent submits payment for the invoice.
  3. The system updates the invoice's payment status upon successful payment.
- **Alternative / Exception Flows:**
  - **E1 — Payment not successful:** Not specified in the source documents how a failed or declined payment is handled — see Open Questions.
- **Postconditions:** The invoice's payment status reflects the outcome. (Payment provider is not selected — see FR doc Open Question.)
- **Related FR IDs:** FR-TUITION-06

---

## 12. Weekly Menu

### UC-MENU-01 — Manage Weekly Menu
- **Primary Actor:** Kế toán / Văn phòng
- **Supporting Actors:** None specified.
- **Goal:** Maintain the school's weekly menu.
- **Preconditions:** None specified beyond the actor being authenticated as Kế toán.
- **Main Flow:**
  1. Kế toán creates or updates the weekly menu.
  2. The system stores the weekly menu.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** A weekly menu exists for the school.
- **Related FR IDs:** FR-MENU-01

> No "view weekly menu" use case is defined: the specification does not name any role as a viewer of the weekly menu (carried from `functional-requirements.md` Open Question #14 — not resolved here).

---

## 13. Notifications

### UC-NOTI-01 — Receive Notification
- **Primary Actor:** Phụ huynh
- **Supporting Actors:** None specified.
- **Goal:** Be informed of relevant events through the system.
- **Preconditions:** None specified.
- **Main Flow:**
  1. A relevant event occurs (trigger not specified).
  2. The system delivers a notification to the parent.
  3. Parent views the notification.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Parent has been informed of the event. (Whether a notification can be marked read/dismissed, and what events trigger one, are not defined — see Open Questions.)
- **Related FR IDs:** FR-NOTI-01

---

## 14. Dashboard & Reports

### UC-DASH-01 — View Dashboard & Statistics
- **Primary Actor:** Admin / Ban giám hiệu
- **Supporting Actors:** None specified.
- **Goal:** Get a school-wide overview of children, attendance, and revenue.
- **Preconditions:** None specified beyond the actor being authenticated as Admin.
- **Main Flow:**
  1. Admin opens the dashboard.
  2. The system displays the overview, child count statistics, attendance statistics, and revenue statistics.
- **Alternative / Exception Flows:** None specified.
- **Postconditions:** Admin has viewed the current statistics. (Specific breakdowns/time ranges are not defined — see Open Questions.)
- **Related FR IDs:** FR-DASH-01, FR-DASH-02, FR-DASH-03, FR-DASH-04

---

## 15. Use Case Summary Table

| UC ID | Use Case Name | Primary Actor | Related FR IDs |
|---|---|---|---|
| UC-AUTH-01 | Log In | All roles | FR-AUTH-01, FR-AUTH-02 |
| UC-USER-01 | Manage User Accounts | Admin / Ban giám hiệu | FR-USER-01 |
| UC-USER-02 | Manage Roles / Permissions | Admin / Ban giám hiệu | FR-USER-02 |
| UC-CLASS-01 | Assign Children to Classes by Age Group | Admin / Ban giám hiệu | FR-CLASS-01 |
| UC-STU-01 | Manage Child Enrollment Record | Kế toán / Văn phòng | FR-STU-01 |
| UC-TEACHER-01 | Manage Teacher Record | Kế toán / Văn phòng | FR-TEACHER-01 |
| UC-ATT-01 | Record Check-In Attendance | Giáo viên | FR-ATT-01 |
| UC-ATT-02 | View Child's Attendance History | Phụ huynh | FR-ATT-02 |
| UC-PICKUP-01 | Record Pickup Attendance | Giáo viên | FR-PICKUP-01, FR-PICKUP-02 |
| UC-PICKUP-02 | Manage Registered Pickup Persons | Phụ huynh | FR-PICKUP-03 |
| UC-HEALTH-01 | Record Quick Health Status | Giáo viên | FR-HEALTH-01 |
| UC-HEALTH-02 | Record Growth Measurement | Y tế | FR-HEALTH-02, FR-HEALTH-03 |
| UC-HEALTH-03 | Review Health & Incident History | Y tế | FR-HEALTH-04 |
| UC-INCIDENT-01 | Record Incident / Accident | Giáo viên | FR-INCIDENT-01, FR-INCIDENT-02 |
| UC-MEDIA-01 | Share Activity Photos | Giáo viên | FR-MEDIA-01 |
| UC-MEDIA-02 | View Activity Photos / Videos | Phụ huynh | FR-MEDIA-02 |
| UC-TUITION-01 | Manage Tuition Fees | Kế toán / Văn phòng | FR-TUITION-01 |
| UC-TUITION-02 | Manage Invoices | Kế toán / Văn phòng | FR-TUITION-02 |
| UC-TUITION-03 | Generate Revenue Report | Kế toán / Văn phòng | FR-TUITION-03, FR-TUITION-04 |
| UC-TUITION-04 | View Tuition Invoice | Phụ huynh | FR-TUITION-05 |
| UC-TUITION-05 | Pay Tuition Online | Phụ huynh | FR-TUITION-06 |
| UC-MENU-01 | Manage Weekly Menu | Kế toán / Văn phòng | FR-MENU-01 |
| UC-NOTI-01 | Receive Notification | Phụ huynh | FR-NOTI-01 |
| UC-DASH-01 | View Dashboard & Statistics | Admin / Ban giám hiệu | FR-DASH-01, FR-DASH-02, FR-DASH-03, FR-DASH-04 |

---

## 16. Actor → Use Case Matrix

| Actor | Use Cases |
|---|---|
| Admin / Ban giám hiệu | UC-AUTH-01, UC-USER-01, UC-USER-02, UC-CLASS-01, UC-DASH-01 |
| Giáo viên | UC-AUTH-01, UC-ATT-01, UC-PICKUP-01, UC-HEALTH-01, UC-INCIDENT-01, UC-MEDIA-01 |
| Kế toán / Văn phòng | UC-AUTH-01, UC-STU-01, UC-TEACHER-01, UC-TUITION-01, UC-TUITION-02, UC-TUITION-03, UC-MENU-01 |
| Y tế | UC-AUTH-01, UC-HEALTH-02, UC-HEALTH-03 |
| Phụ huynh | UC-AUTH-01, UC-ATT-02, UC-PICKUP-02, UC-MEDIA-02, UC-TUITION-04, UC-TUITION-05, UC-NOTI-01 |

---

## 17. Open Questions

Only items necessary for use-case analysis and not already resolved by the source documents:

1. **(UC-AUTH-01)** How a failed login (invalid credentials) is handled — error messaging, retry limits, lockout, password reset — is not defined.
2. **(UC-PICKUP-01)** What the teacher should do when the person picking up a child is not on the registered pickup list is not defined.
3. **(UC-TUITION-05)** How a failed or declined online payment is handled is not defined.
4. **(UC-NOTI-01)** Whether a parent can mark a notification as read/dismiss it, or can only view it, is not defined. The events that trigger a notification are also not defined (carried from `functional-requirements.md` Open Question #15).
5. **(UC-MENU-01)** No role is defined in the specification as a viewer of the weekly menu, so no corresponding "view" use case could be defined (carried from `functional-requirements.md` Open Question #14).
6. **(UC-MEDIA-02, UC-TUITION-04)** Whether viewing is strictly limited to the parent's own child, as it explicitly is for attendance (UC-ATT-02), is not confirmed for activity media or invoices (carried from `functional-requirements.md` Open Question #17 and `user-roles.md` Open Question #7).
7. **(UC-HEALTH-03)** Whether the history Y tế reviews includes the teacher's "quick health status" entries (UC-HEALTH-01) is not explicitly linked in the source documents (carried from `user-roles.md` Open Question #6).
8. **(UC-USER-01, UC-USER-02)** The exact account and permission lifecycle actions (e.g., deactivate vs. delete an account; scope of a permission change) are not detailed enough to break these into more specific use cases (carried from `functional-requirements.md` and `user-roles.md` Open Questions on "Manage" scope).
