# Database Design — Kindergarten Management System

## 0. Document Information

- **Stage:** Database Design (design only — no EF Core entities, no migrations, no source code).
- **Source of truth:** `CLAUDE.md`, `docs/requirements/functional-requirements.md`, `docs/requirements/non-functional-requirements.md`, `docs/business/user-roles.md`, `docs/business/use-cases.md`, `docs/business/business-rules.md`, `docs/requirements/acceptance-criteria.md`.
- **Status:** Draft — for review before EF Core implementation or migrations begin.
- **Target platform:** PostgreSQL (per `CLAUDE.md` §4, §8; `DECISIONS.md` D15/D16), accessed only through the backend (no direct frontend DB access).
- **Scope discipline:** Designed for a university project (~2.5 months). No microservices, no event sourcing/CQRS, no generic EAV structures, no general-purpose audit-log system. Audit-style columns (e.g., "recorded by teacher") are added only where an existing Business Rule explicitly restricts who may perform an action on child-safety-relevant data (attendance, pickup, health, incident, media) — not as a blanket pattern.
- **Convention:** Every table/column is traced to a Functional Requirement, Business Rule, or Use Case. Where a detail is an explicit Open Question (status values, required fields, verification method, provider, structure), the schema either omits the detail or includes a clearly-marked minimal/provisional placeholder — it never invents a resolution.

---

## 1. Inspection Summary

- **Repository structure:** `backend/`, `frontend/`, `database/`, `docs/{requirements,business,database,architecture,api,ui,testing}`, `scripts/`, `tests/` — all currently empty except the `docs/requirements` and `docs/business` files produced in earlier stages. No source code, no migrations, no existing schema exist yet.
- **Git status:** On branch `develop`, up to date with `origin/develop`. Only untracked `docs/` files are pending (nothing staged or committed by this task).
- **PROGRESS.md / TASKS.md:** Confirm the project is still in the documentation/analysis phase; database implementation has not started, and both files instruct that no migration should be created before this design is reviewed.
- **README.md:** Empty; not authoritative for anything (per instruction, `CLAUDE.md` and the approved requirements govern, not README content).

---

## 2. Requirement / Business Rule → Entity Mapping

| Requirement / Business Rule | Entity / Relationship / Constraint |
|---|---|
| FR-AUTH-01, BR-AUTH-01 | `users` table; every actor must exist as a `users` row before using the system. |
| FR-AUTH-02, BR-AUTH-02 | `users.role_id` → `roles`; access restriction enforced by role (app-layer), schema supplies the role association. |
| FR-USER-01, BR-USER-01 | `users` (Admin manages this table). |
| FR-USER-02, BR-USER-02 | `roles` lookup; no separate permission table (see Open Question #2). |
| FR-CLASS-01, BR-CLASS-01 | `classes` table; `children.class_id` FK. |
| BR-CLASS-02, BR-STUDENT-02 | `children.class_id` is nullable; enforced at app layer that a child record must exist — representing that enrollment has occurred — before `class_id` is set. What exactly constitutes a complete enrollment record is Open Question #4. |
| FR-STU-01, BR-STUDENT-01 | `children` table (Kế toán manages it). |
| FR-TEACHER-01, BR-TEACHER-01 | `teachers` table (1:1 with `users`, Kế toán manages it). |
| FR-ATT-01, BR-ATTENDANCE-01, BR-ATTENDANCE-02 | `attendances` table; FK to `children` and `teachers`; requires `children.class_id` set (precondition, enforced at app layer). |
| FR-ATT-02, BR-ATTENDANCE-03 | `child_guardians` join table (defines "own child" for scoping); query-level filter, not a separate column. |
| FR-PICKUP-01, BR-PICKUP-01 | `pickups` table; FK to `attendances` and `teachers`. |
| FR-PICKUP-02, BR-PICKUP-02 | `pickups.pickup_person_id` (nullable FK to `registered_pickup_persons`). |
| FR-PICKUP-03, BR-PICKUP-03 | `registered_pickup_persons` table; FK to `children`; managed via `child_guardians` linkage for authorization. |
| FR-HEALTH-01, BR-HEALTH-01, BR-HEALTH-03 | `quick_health_statuses` table; FK to `children`, `teachers`. |
| FR-HEALTH-02/03, BR-HEALTH-02, BR-HEALTH-03 | `growth_measurements` table; FK to `children`, `users` (Y tế). |
| FR-HEALTH-04, BR-HEALTH-04 | Query over `incidents` + `growth_measurements` (+ `quick_health_statuses`, pending Open Question #9) by `child_id`; no separate "history" table needed. |
| FR-INCIDENT-01, BR-INCIDENT-01, BR-INCIDENT-02 | `incidents` table; FK to `children`, `teachers`. |
| FR-INCIDENT-02, BR-INCIDENT-03 | `incident_photos` table; FK to `incidents`; attachment optional (no NOT NULL requirement forcing a photo). |
| BR-INCIDENT-04 | Incident visible to Y tế via the same `incidents` table (no duplication into a separate "history" entity). |
| FR-MEDIA-01, BR-MEDIA-01 | `activity_photos` table; FK to `classes`, `teachers`. |
| FR-MEDIA-02, BR-MEDIA-02 | Query over `activity_photos` filtered by the parent's linked child's `class_id`; no exclusivity constraint added (Open Question #10/#11). |
| FR-TUITION-01, BR-TUITION-01 | `tuition_fees` table. |
| FR-TUITION-02, BR-TUITION-02, BR-TUITION-03 | `invoices` table; FK to `children`, nullable FK to `tuition_fees`. |
| FR-TUITION-03/04, BR-TUITION-07, BR-TUITION-08 | Computed on demand from `invoices` + `payments` (no stored report table — see §5 rationale). |
| FR-TUITION-05, BR-TUITION-04 | Query over `invoices` filtered by the parent's linked child. |
| FR-TUITION-06, BR-TUITION-05, BR-TUITION-06 | `payments` table; FK to `invoices`; `invoices.status` transitions on successful payment. |
| FR-MENU-01, BR-MENU-01 | `weekly_menus` + `menu_entries` tables. |
| FR-NOTI-01, BR-NOTIFICATION-01 | `notifications` table; FK to `users` (recipient constrained to Parent role at app layer). |
| FR-DASH-01–04, BR-DASHBOARD-01/02 | Computed on demand from `children`, `attendances`, `invoices`/`payments` (no stored dashboard entity). |

---

## 3. Open Questions Affecting Database Design

For each item, **A** = design can proceed without deciding it (no schema impact), **B** = a provisional/minimal structure is included now and can be safely extended later, **C** = must be resolved before the affected part of the schema can be finalized/implemented.

| # | Open Question | Classification | Notes |
|---|---|---|---|
| 1 | Authentication strategy | **B/C** | `users` table (identity, role, a generic credential reference) can be designed now (B); the exact credential/token storage shape depends on the chosen mechanism (C) — deferred, not modeled. |
| 2 | Permission granularity (role-only vs. custom) | **A** | Schema uses `users.role_id` only. If finer-grained permissions are confirmed later, this is an additive table, not a redesign. |
| 3 | Class age boundaries; teacher-to-class assignment | **B** | `classes` includes nullable `min_age_months`/`max_age_months` and a nullable single `homeroom_teacher_id`; exact boundary values and multi-teacher scenarios are not decided. |
| 4 | Child enrollment required fields; student status values | **B** | `children` holds only the minimally evidenced fields; no status column is added (see §5 rationale) — deferred. |
| 5 | Attendance status values | **B** | `attendances.status` is an unconstrained text field; no CHECK constraint on specific values. |
| 6 | Pickup-person required fields; verification method | **B** | `registered_pickup_persons` holds only `full_name`; verification is an app-layer process, not a schema concern. |
| 7 | Quick health status fields | **B** | `quick_health_statuses.notes` is free text; no structured fields invented. |
| 8 | Height/weight measurement schedule | **A** | Schedule is an operational concern; schema just timestamps each measurement. |
| 9 | Whether Y tế's history view includes teacher's quick health status | **A** | Both tables already carry `child_id`; this is a query-scope decision, not a schema change. |
| 10 | Media storage/access scope; teacher video upload | **B** | `activity_photos` is photo-only and storage-agnostic (`file_reference` text); video would be an additive column/table later. |
| 11 | Parent access scope for media and invoices (own-child-only?) | **A** | `child_guardians` supports either resolution via a query filter; no schema change either way. |
| 12 | Tuition fee structure | **B** | `tuition_fees` is a flat catalog (`description`, `amount`); structural changes (per class/age/service) are additive. |
| 13 | Payment provider / payment data shape | **B/C** | `payments` holds only generic fields (`amount`, `paid_at`, nullable `external_reference`); provider-specific fields are deferred (C). |
| 14 | Invoice status values beyond paid/unpaid | **B** | `invoices.status` is constrained only to the two values evidenced by FR-TUITION-06 ("unpaid" / "paid"); further values are additive. |
| 15 | Weekly menu visibility (who can view) | **A** | Visibility is an authorization concern; the menu is stored regardless of who may read it. |
| 16 | Notification triggers/channels/read-unread | **B** | `notifications` holds only `recipient_user_id`, `message`, `created_at`; no trigger FK or read-status column is added. |
| 17 | Dashboard breakdowns/time ranges | **A** | Dashboard values are computed at query time; no stored entity. |
| 18–21 | Admin/Kế toán/Y tế/Teacher cross-role visibility scoping | **A** | All are authorization/query-filter questions; the underlying FKs already exist to support any resolution. |
| 22 | Exception-flow handling (login failure, unrecognized pickup, failed payment) | **A** | No dedicated log/error tables are introduced; this is application error-handling behavior, not schema. |

---

## 4. Conceptual Data Model

### Role
- **Purpose:** Fixed set of the 5 system roles.
- **Key:** id
- **Attributes:** name
- **Relationships:** one Role → many User
- **Business Rules:** BR-USER-01 (every account tied to a role)
- **Open Questions:** #2 (permission granularity)

### User
- **Purpose:** An authenticated account for any of the 5 roles.
- **Key:** id
- **Attributes:** full name, login identifier, credential reference, active flag
- **Relationships:** many User → one Role; one User → 0..1 Teacher (when role = Teacher); many User (Parent role) ↔ many Child via ChildGuardian; one User (Parent) → many Notification
- **Business Rules:** BR-AUTH-01, BR-USER-01
- **Open Questions:** #1 (authentication strategy)

### Teacher
- **Purpose:** HR profile for a Giáo viên, managed by Kế toán.
- **Key:** id
- **Attributes:** (minimal — linked to User; further HR fields not yet defined)
- **Relationships:** one User → one Teacher; one Teacher → many Class (as homeroom teacher); one Teacher → many Attendance/Pickup/QuickHealthStatus/Incident/ActivityPhoto (as recorder/uploader)
- **Business Rules:** BR-TEACHER-01, BR-TEACHER-02
- **Open Questions:** none blocking; HR field detail is not specified but not required for the relational skeleton.

### Child
- **Purpose:** The child's main record/profile — the central entity most other modules attach to. For now this record is used, provisionally, as the minimal representation of the child having been enrolled; the requirements do not confirm that this record is definitively equivalent to "the enrollment record" (Open Question #4 on enrollment fields and student status remains open).
- **Key:** id
- **Attributes:** full name, date of birth, enrollment date
- **Relationships:** one Class → many Child; many Child ↔ many User (Parent) via ChildGuardian; one Child → many Attendance, RegisteredPickupPerson, QuickHealthStatus, GrowthMeasurement, Incident, Invoice
- **Business Rules:** BR-STUDENT-01, BR-STUDENT-02, BR-CLASS-02
- **Open Questions:** #4 (required fields, status values)

### ChildGuardian (join)
- **Purpose:** Links a child to the parent(s)/guardian(s) authorized to act on their behalf.
- **Key:** (child, guardian user) pair
- **Attributes:** (relationship type not modeled — Open Question)
- **Relationships:** many-to-many bridge between Child and User (Parent role)
- **Business Rules:** BR-ATTENDANCE-03, BR-PICKUP-03, BR-MEDIA-02, BR-TUITION-04 (all "own child" scoping rules resolve through this table)
- **Open Questions:** #11 (exact exclusivity scope), enrollment/guardian relationship model (FR-STU-01 Open Question)

### Class
- **Purpose:** An age-based classroom grouping.
- **Key:** id
- **Attributes:** name, optional age-band bounds
- **Relationships:** one Class → many Child; one Teacher → many Class (homeroom); one Class → many ActivityPhoto
- **Business Rules:** BR-CLASS-01, BR-MEDIA-01
- **Open Questions:** #3 (age boundaries, teacher assignment cardinality)

### Attendance
- **Purpose:** A child's daily check-in record.
- **Key:** id
- **Attributes:** date, check-in time, status
- **Relationships:** many Attendance → one Child; many Attendance → one Teacher (recorder); one Attendance → 0..1 Pickup
- **Business Rules:** BR-ATTENDANCE-01, BR-ATTENDANCE-02
- **Open Questions:** #5 (status values)

### Pickup
- **Purpose:** Records that a child was picked up, extending that day's Attendance.
- **Key:** id
- **Attributes:** pickup time
- **Relationships:** one Attendance → one Pickup; many Pickup → one Teacher (recorder); many Pickup → 0..1 RegisteredPickupPerson
- **Business Rules:** BR-PICKUP-01, BR-PICKUP-02
- **Open Questions:** #6 (verification method — not modeled, app-layer only)

### RegisteredPickupPerson
- **Purpose:** A person a parent has pre-authorized to pick up their child.
- **Key:** id
- **Attributes:** full name (minimal)
- **Relationships:** many RegisteredPickupPerson → one Child
- **Business Rules:** BR-PICKUP-02, BR-PICKUP-03
- **Open Questions:** #6 (required fields)

### QuickHealthStatus
- **Purpose:** A teacher's brief health observation for a child.
- **Key:** id
- **Attributes:** recorded time, free-text notes
- **Relationships:** many QuickHealthStatus → one Child; many → one Teacher (recorder)
- **Business Rules:** BR-HEALTH-01, BR-HEALTH-03
- **Open Questions:** #7 (specific fields), #9 (linkage into Y tế's history view)

### GrowthMeasurement
- **Purpose:** A Y tế-recorded height and/or weight measurement.
- **Key:** id
- **Attributes:** measured time, height (nullable), weight (nullable)
- **Relationships:** many GrowthMeasurement → one Child; many → one User (Y tế)
- **Business Rules:** BR-HEALTH-02, BR-HEALTH-03
- **Open Questions:** #8 (measurement schedule)

### Incident
- **Purpose:** A teacher-recorded incident/accident involving a child.
- **Key:** id
- **Attributes:** occurred time, description
- **Relationships:** many Incident → one Child; many → one Teacher (recorder); one Incident → many IncidentPhoto
- **Business Rules:** BR-INCIDENT-01, BR-INCIDENT-02, BR-INCIDENT-04
- **Open Questions:** none blocking.

### IncidentPhoto
- **Purpose:** An optional photo attached to an incident.
- **Key:** id
- **Attributes:** file reference (storage-agnostic)
- **Relationships:** many IncidentPhoto → one Incident
- **Business Rules:** BR-INCIDENT-03 (attachment optional)
- **Open Questions:** media storage provider (`DECISIONS.md` D22)

### ActivityPhoto
- **Purpose:** A teacher-shared activity photo, scoped to a class.
- **Key:** id
- **Attributes:** uploaded time, file reference (storage-agnostic)
- **Relationships:** many ActivityPhoto → one Class; many → one Teacher (uploader)
- **Business Rules:** BR-MEDIA-01, BR-MEDIA-02
- **Open Questions:** #10 (video upload, storage provider)

### TuitionFee
- **Purpose:** Tuition fee catalog information.
- **Key:** id
- **Attributes:** description, amount
- **Relationships:** one TuitionFee → many Invoice (provisional)
- **Business Rules:** BR-TUITION-01, BR-TUITION-03
- **Open Questions:** #12 (fee structure)

### Invoice
- **Purpose:** A tuition invoice for a specific child.
- **Key:** id
- **Attributes:** amount, issued time, status (paid/unpaid — minimal confirmed set)
- **Relationships:** many Invoice → one Child; many Invoice → 0..1 TuitionFee; one Invoice → many Payment
- **Business Rules:** BR-TUITION-02, BR-TUITION-03, BR-TUITION-05, BR-TUITION-06
- **Open Questions:** #14 (further status values)

### Payment
- **Purpose:** A payment made against an invoice.
- **Key:** id
- **Attributes:** amount, paid time, optional external reference
- **Relationships:** many Payment → one Invoice
- **Business Rules:** BR-TUITION-05, BR-TUITION-06
- **Open Questions:** #13 (payment provider, failed-payment handling)

### WeeklyMenu / MenuEntry
- **Purpose:** The school's weekly menu.
- **Key:** id (each)
- **Attributes:** week start date; per-day description
- **Relationships:** one WeeklyMenu → many MenuEntry
- **Business Rules:** BR-MENU-01
- **Open Questions:** #15 (who views it); exact meal structure not defined

### Notification
- **Purpose:** A message delivered to a parent.
- **Key:** id
- **Attributes:** message, created time
- **Relationships:** many Notification → one User (Parent)
- **Business Rules:** BR-NOTIFICATION-01
- **Open Questions:** #16 (triggers, channel, read/unread)

---

## 5. Logical Data Model (PostgreSQL)

Notes applicable to all tables: primary keys are `uuid` (via `gen_random_uuid()`) except `roles`, which is a small fixed lookup (`smallint`). Timestamps are `timestamptz`. No generic audit-log table is introduced; "recorded by" columns exist only where a Business Rule explicitly names who may perform that action on child-safety-relevant data.

### `roles`
- PK: `id` (smallint)
- Fields: `name` text **not null**
- Unique: `name`
- Seed rows: the 5 confirmed roles (Admin, Giáo viên, Kế toán/Văn phòng, Y tế, Phụ huynh) — no others (BR-USER-01, CLAUDE.md §2).

### `users`
- PK: `id`
- FK: `role_id` → `roles.id` **not null**
- Fields: `full_name` text **not null**; `login_identifier` text **not null**; `credential_reference` text **not null** (placeholder pending Open Question #1); `is_active` boolean **not null default true**; `created_at` timestamptz **not null default now()**
- Unique: `login_identifier`
- Index: `role_id`

### `teachers`
- PK: `id`
- FK: `user_id` → `users.id` **not null**
- Unique: `user_id` (enforces 1:1 with `users`)
- Fields: `created_at` timestamptz **not null default now()** (no further HR fields — Open Question, additive later)

### `children`
- PK: `id`
- FK: `class_id` → `classes.id` **nullable** (a child may be enrolled before class assignment; BR-CLASS-02)
- Fields: `full_name` text **not null**; `date_of_birth` date **not null**; `enrollment_date` date **not null**
- Index: `class_id`
- (No status column — Open Question #4, intentionally omitted rather than invented.)

### `child_guardians`
- PK: composite (`child_id`, `guardian_user_id`)
- FK: `child_id` → `children.id` **not null**; `guardian_user_id` → `users.id` **not null**
- Constraint intent (app-enforced, not a DB CHECK): `guardian_user_id` must reference a `users` row with role = Phụ huynh.
- Index: `guardian_user_id` (for "find this parent's children" lookups)

### `classes`
- PK: `id`
- FK: `homeroom_teacher_id` → `teachers.id` **nullable** (Open Question #3)
- Fields: `name` text **not null**; `min_age_months` integer **nullable**; `max_age_months` integer **nullable**
- Unique: `name`
- Check: if both age bounds are set, `min_age_months <= max_age_months` (basic integrity, not a business-value invention)
- Index: `homeroom_teacher_id`

### `attendances`
- PK: `id`
- FK: `child_id` → `children.id` **not null**; `recorded_by_teacher_id` → `teachers.id` **not null**
- Fields: `attendance_date` date **not null**; `check_in_time` timestamptz **not null**; `status` text **not null** (values undefined — Open Question #5, no CHECK constraint applied)
- Unique: (`child_id`, `attendance_date`) — one check-in record per child per day
- Index: `child_id`, `attendance_date`

### `pickups`
- PK: `id`
- FK: `attendance_id` → `attendances.id` **not null**; `recorded_by_teacher_id` → `teachers.id` **not null**; `pickup_person_id` → `registered_pickup_persons.id` **nullable**
- Fields: `pickup_time` timestamptz **not null**
- Unique: `attendance_id` — one pickup per attendance record
- Note: `pickup_person_id` is nullable because the requirements only explicitly require checking a *non-parent* pickup person against the registered list (FR-PICKUP-02). No business meaning is assigned to a null value here; the pickup verification method and unrecognized-person handling remain open (Open Question #6).

### `registered_pickup_persons`
- PK: `id`
- FK: `child_id` → `children.id` **not null**
- Fields: `full_name` text **not null** (further fields — phone, relationship, ID, photo, validity — Open Question #6, intentionally omitted)
- Index: `child_id`

### `quick_health_statuses`
- PK: `id`
- FK: `child_id` → `children.id` **not null**; `recorded_by_teacher_id` → `teachers.id` **not null**
- Fields: `recorded_at` timestamptz **not null**; `notes` text **not null**
- Index: `child_id`

### `growth_measurements`
- PK: `id`
- FK: `child_id` → `children.id` **not null**; `recorded_by_user_id` → `users.id` **not null** (Y tế; no separate profile table — asymmetric with `teachers` because no FR requires a Y tế HR record, unlike FR-TEACHER-01)
- Fields: `measured_at` timestamptz **not null**; `height_cm` numeric **nullable**; `weight_kg` numeric **nullable**
- Check: `height_cm is not null or weight_kg is not null`
- Index: `child_id`

### `incidents`
- PK: `id`
- FK: `child_id` → `children.id` **not null**; `recorded_by_teacher_id` → `teachers.id` **not null**
- Fields: `occurred_at` timestamptz **not null**; `description` text **not null**
- Index: `child_id`

### `incident_photos`
- PK: `id`
- FK: `incident_id` → `incidents.id` **not null**
- Fields: `file_reference` text **not null** (storage-agnostic; provider undecided per `DECISIONS.md` D22)
- Index: `incident_id`

### `activity_photos`
- PK: `id`
- FK: `class_id` → `classes.id` **not null**; `uploaded_by_teacher_id` → `teachers.id` **not null**
- Fields: `uploaded_at` timestamptz **not null**; `file_reference` text **not null**
- Index: `class_id`

### `tuition_fees`
- PK: `id`
- Fields: `description` text **not null**; `amount` numeric(12,2) **not null**
- Check: `amount > 0`

### `invoices`
- PK: `id`
- FK: `child_id` → `children.id` **not null**; `tuition_fee_id` → `tuition_fees.id` **nullable** (Open Question #12)
- Fields: `amount` numeric(12,2) **not null**; `issued_at` timestamptz **not null**; `status` text **not null**
- Check: `amount > 0`; `status in ('unpaid', 'paid')` — the only two values evidenced by FR-TUITION-06; extending this list is a future, additive decision (Open Question #14)
- Index: `child_id`, `status`

### `payments`
- PK: `id`
- FK: `invoice_id` → `invoices.id` **not null**
- Fields: `amount` numeric(12,2) **not null**; `paid_at` timestamptz **not null**; `external_reference` text **nullable**
- Check: `amount > 0`
- Index: `invoice_id`

### `weekly_menus`
- PK: `id`
- Fields: `week_start_date` date **not null**
- Unique: `week_start_date`

### `menu_entries`
- PK: `id`
- FK: `weekly_menu_id` → `weekly_menus.id` **not null**
- Fields: `day_of_week` smallint **not null**; `description` text **not null**
- Unique: (`weekly_menu_id`, `day_of_week`)

### `notifications`
- PK: `id`
- FK: `recipient_user_id` → `users.id` **not null** (intended to be a Phụ huynh account; enforced at app layer)
- Fields: `message` text **not null**; `created_at` timestamptz **not null default now()**
- Index: `recipient_user_id`
- (No `is_read` column, no trigger-source FK — Open Question #16, intentionally omitted.)

---

## 6. Key Relationships (as requested)

| Relationship | Modeled as | Notes |
|---|---|---|
| User ↔ Role | `users.role_id` → `roles.id` (many-to-one) | Confirmed. |
| Child ↔ Enrollment | `children` represents the child's main record/profile, used provisionally as the minimal stand-in for the enrollment record | No separate `enrollments` table — nothing in the requirements implies enrollment history/periods distinct from the child record. Treating `children` as the enrollment record is a provisional modeling choice, not a resolved business rule; Open Question #4 (enrollment fields, student status) remains open. |
| Child ↔ Class | `children.class_id` → `classes.id` (many-to-one, nullable) | Confirmed; age-band details open (#3). |
| Teacher ↔ Class | `classes.homeroom_teacher_id` → `teachers.id` (many-to-one, nullable) | Provisional — cardinality and assignment workflow open (#3). |
| Child ↔ Attendance | `attendances.child_id` (one-to-many) | Confirmed; one row per child per day. |
| Child ↔ Pickup Person | `registered_pickup_persons.child_id` (one-to-many) | Confirmed. |
| Child ↔ Health Records | `quick_health_statuses.child_id`, `growth_measurements.child_id` (one-to-many each) | Confirmed. |
| Child ↔ Incident | `incidents.child_id` (one-to-many) | Confirmed. |
| Incident ↔ Photos | `incident_photos.incident_id` (one-to-many, optional) | Confirmed. |
| Class/Group ↔ Activity Media | `activity_photos.class_id` (one-to-many) | Provisional modeling assumption: "Group" is treated as synonymous with "Class," and no `groups` table is introduced, purely to keep the design simple. The requirements do not define whether Group and Class are separate concepts — this remains an open question. |
| Child ↔ Tuition Invoice | `invoices.child_id` (one-to-many) | Confirmed. |
| Tuition Fee ↔ Invoice | `invoices.tuition_fee_id` (many-to-one, nullable) | Provisional single-fee reference pending fee-structure resolution (#12). |
| Invoice ↔ Payment | `payments.invoice_id` (one-to-many) | Confirmed. |
| Weekly Menu | `weekly_menus` + `menu_entries` (one-to-many) | Minimal provisional structure (#15 and meal structure not defined). |
| Parent ↔ Child | `child_guardians` (many-to-many) | Relationship-type attribute not modeled (open). |
| Notifications ↔ Parent | `notifications.recipient_user_id` → `users.id` (many-to-one) | Confirmed recipient is Phụ huynh; enforced at app layer. |

---

## 7. ERD

See `docs/database/erd.puml` (PlantUML, consistent with `DECISIONS.md` D32's choice of PlantUML for project diagrams).

---

## 8. Consistency Review

1. **Every persisted business concept has a place in the model** — yes: all 32 FRs and all 38 Business Rules trace to a table or an explicitly-scoped query in §2.
2. **Every important relationship required by the Business Rules is represented** — yes: role, class, guardian, teacher-recorder, and invoice/payment relationships are all modeled (§6).
3. **No table exists only because of an invented feature** — confirmed: no Group, Permission, AuditLog, LoginAttempt, or ReportSnapshot table was added; reports/dashboards are computed, not stored.
4. **No invented field resolves an Open Question** — confirmed: attendance/invoice statuses are either unconstrained text or limited to the literal values evidenced by the FR text; enrollment/pickup-person/health fields are kept minimal; no payment provider fields were added.
5. **No Business Rule is contradicted by the schema** — confirmed: e.g., `child_guardians` supports (not overrides) BR-ATTENDANCE-03's own-child scoping; `invoices.status` check includes exactly the two values BR-TUITION-05/06 evidence.
6. **Parent access relationships are not over-restricted** — confirmed: no exclusivity CHECK/constraint was added for media or invoices, consistent with Open Question #11 remaining open.
7. **Teacher scope is not expanded beyond what is confirmed** — confirmed: teacher-recorder FKs exist only for the modules where a Business Rule names the teacher as sole recorder (attendance, pickup, quick health status, incident, activity photo); no class-wide "teacher owns all data in class" constraint was added, consistent with Open Question #20 remaining open.
8. **Admin access is not assumed beyond what is documented** — confirmed: no table grants Admin direct row-level access beyond what `users`/`classes` management already implies; dashboard data is computed, not gated by a special Admin-only table.
9. **Schema remains reasonably simple** — 20 tables, no advanced patterns, consistent with a 2.5-month student project.
10. **PostgreSQL compatibility** — all types (`uuid`, `timestamptz`, `numeric`, `smallint`, `text`, `date`) and constructs (CHECK, composite unique/PK) are native PostgreSQL.
11. **Referential integrity considered** — every FK is identified with its nullability; no orphaned-reference paths were left ungoverned.
12. **Nullable/non-nullable decisions justified by requirements** — e.g., `children.class_id` nullable because enrollment can precede class assignment (BR-CLASS-02); `pickups.pickup_person_id` nullable because the requirements only explicitly require checking a *non-parent* pickup person against the registered list (FR-PICKUP-02) — no business meaning is assigned to the null case itself.

No issues were found that required revising the model before reporting.

---

## 9. Output Summary

- **Proposed table count:** 20 (`roles`, `users`, `teachers`, `children`, `child_guardians`, `classes`, `attendances`, `pickups`, `registered_pickup_persons`, `quick_health_statuses`, `growth_measurements`, `incidents`, `incident_photos`, `activity_photos`, `tuition_fees`, `invoices`, `payments`, `weekly_menus`, `menu_entries`, `notifications`).
- **Main entities:** User/Role, Teacher, Child, Class, Attendance/Pickup, Health (QuickHealthStatus, GrowthMeasurement), Incident, ActivityPhoto, Tuition (TuitionFee, Invoice, Payment), WeeklyMenu, Notification.
- **Main relationships:** User↔Role, User↔Teacher (1:1), Child↔Class, Teacher↔Class (homeroom), Child↔ChildGuardian↔User(Parent), Child↔Attendance↔Pickup, Child↔RegisteredPickupPerson, Child↔QuickHealthStatus/GrowthMeasurement/Incident, Incident↔IncidentPhoto, Class↔ActivityPhoto, Child↔Invoice↔Payment, TuitionFee↔Invoice, WeeklyMenu↔MenuEntry, User(Parent)↔Notification.
- **Constraints clearly supported:** role-required on every user; 1:1 User–Teacher; one attendance per child per day; one pickup per attendance; optional incident photos; invoice status limited to paid/unpaid; positive-amount checks on fee/invoice/payment amounts; at-least-one-of-height/weight on growth measurements.
- **Open Questions affecting the database:** 22 items (§3), classified A/B/C; none resolved by this design.
- **Decisions that must be made before implementation:** authentication/credential storage shape (#1), payment-provider-specific fields (#13), plus any Category-C portion of the others once the business decides them.
- **Files created:** `docs/database/database-design.md`, `docs/database/erd.puml`.
- **Files modified:** none.
- **Current Git status:** branch `develop`, up to date with `origin/develop`; only untracked `docs/` files pending (nothing staged, committed, or pushed by this task).
- **Confirmed:** no source code was modified; no EF Core entities were created; no migration was created; no Git commit or push was performed.
