# Non-Functional Requirements — Kindergarten Management System

## 0. Document Information

- **Source of truth:** The approved project specification (`CLAUDE.md` §1–§21) and `docs/requirements/functional-requirements.md`. No requirement below goes beyond what is stated or directly implied there.
- **Status:** Draft — for review and approval before Business Rules, Use Cases, Database Design, Architecture, and API Design proceed.
- **Out of scope for this document:** database schema, API endpoints, UI screens, system architecture, and any choice of authentication, payment, storage, notification, or deployment/hosting technology. These remain pending decisions (see `DECISIONS.md`).
- **Convention used in this document:**
  - **Requirement** states only what is explicitly stated or directly implied by the specification, kept qualitative (no invented numeric target) unless the specification itself gives a number.
  - **Rationale** cites the specification section or functional requirement(s) the item is grounded in.
  - Where the specification does not define a measurable target or a technical decision, the gap is recorded as an **Open Question** rather than assumed.
  - This document does not resolve any Open Question already listed in `functional-requirements.md`.

---

## 1. Performance

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-PERF-01 | Performance | The system should respond to routine daily actions (login, attendance/pickup check-in, viewing a child's record) quickly enough not to disrupt classroom and front-desk operations. | Attendance and pickup (FR-ATT-01, FR-PICKUP-01/02) happen at time-sensitive moments (drop-off/pickup); the specification gives no numeric response-time target. | Medium |
| NFR-PERF-02 | Performance | Reporting and export operations (revenue report, Excel/PDF export) may take longer than interactive actions but must not block other users' concurrent use of the system. | FR-TUITION-03, FR-TUITION-04. | Low |

**Open Question:** No numeric performance targets (e.g., maximum response time, report generation time) are specified.

---

## 2. Security

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-SEC-01 | Security | Access to functionality and data must be restricted according to the user's role, matching the 5 defined roles. | CLAUDE.md §2, §5; FR-AUTH-02, CC-01. | High |
| NFR-SEC-02 | Security | All input must be validated on the backend regardless of any frontend validation. | CLAUDE.md §11. | High |
| NFR-SEC-03 | Security | Internal exceptions and stack traces must never be exposed directly to end users; errors must be handled consistently. | CLAUDE.md §11. | High |
| NFR-SEC-04 | Security | Logs must never contain passwords, tokens, secrets, API keys, or other unnecessary sensitive data. | CLAUDE.md §12. | High |
| NFR-SEC-05 | Security | Data handled by sensitive functional areas (authentication, health records, online payment, personal/child information) must be protected against unauthorized access. | CLAUDE.md §5, §9; FR-HEALTH-*, FR-TUITION-06, FR-PICKUP-*. | High |
| NFR-SEC-06 | Security | Once an authentication approach is implemented, it must undergo a security review before being considered complete. | CLAUDE.md §5 ("Sau khi triển khai phải review security"). | High |

**Open Question:** The specific mechanisms for securing data in transit and at rest (e.g., transport security, encryption at rest) are not specified and depend on the authentication strategy and hosting/deployment decisions, which are not yet finalized (`DECISIONS.md` D20).

---

## 3. Availability & Reliability

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-AVAIL-01 | Availability | The system should be usable whenever staff, teachers, and parents need it for daily operations (attendance, pickup, health, tuition, communication). | Implied by the daily-operational nature of FR-ATT, FR-PICKUP, FR-HEALTH modules; no formal uptime target is stated. | Medium |
| NFR-AVAIL-02 | Reliability | Attendance and pickup-verification behavior must be consistent and predictable, since it relates to confirming who takes a child from the school. | FR-ATT-01, FR-PICKUP-01/02 are safety-relevant, time-sensitive flows. | High |

**Open Question:** No uptime/availability SLA (e.g., percentage uptime, maximum downtime) is specified in the approved scope.

---

## 4. Usability

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-USE-01 | Usability | The UI must be responsive across common device sizes, since teachers and parents are expected to use the system on different devices. | CLAUDE.md §10 ("responsive UI"). | High |
| NFR-USE-02 | Usability | The system must present loading, error, and empty states consistently to the user. | CLAUDE.md §10. | Medium |
| NFR-USE-03 | Usability | Accessibility must be addressed "at an appropriate level" for the system's users. | CLAUDE.md §10 ("accessibility ở mức phù hợp"). | Medium |
| NFR-USE-04 | Usability | Time-sensitive workflows used by teachers (e.g., quick health status entry, attendance/pickup) must be simple enough for fast daily use. | The specification itself describes some entries as "quick" (FR-HEALTH-01: "Ghi nhận nhanh"). | High |

**Open Question:** No specific accessibility standard or level (e.g., a named conformance level) is specified.

---

## 5. Maintainability

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-MAINT-01 | Maintainability | Code must remain simple, readable, and appropriate for the project's scale; unnecessary abstraction and over-engineering must be avoided. | CLAUDE.md §6. | High |
| NFR-MAINT-02 | Maintainability | Authorization/permission logic must be defined in one consistent place rather than duplicated/hard-coded across the codebase. | CLAUDE.md §5; FR-USER-02. | High |
| NFR-MAINT-03 | Maintainability | Each significant feature must be verified through build and test before being considered complete. | CLAUDE.md §13. | Medium |

---

## 6. Scalability

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-SCALE-01 | Scalability | The system must support the expected scale of a single kindergarten (5 role groups: admin/leadership, teachers, accounting/office, medical staff, parents) without requiring a highly scalable, distributed architecture. | Project overview describes a single-kindergarten management system (CLAUDE.md §1). | Medium |

**Open Question:** No specific expected number of children, classes, staff, or concurrent users is defined in the specification.

---

## 7. Compatibility

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-COMPAT-01 | Compatibility | The frontend must work on common web browsers and be usable on both desktop and mobile screen sizes. | CLAUDE.md §10 ("responsive UI"). | High |

**Open Question:** No explicit browser support matrix (specific browsers/minimum versions) or device support list is specified.

---

## 8. Logging & Monitoring

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-LOG-01 | Logging | The backend must log relevant application events using Serilog. | CLAUDE.md §12; D8. | High |
| NFR-LOG-02 | Logging | Logs must exclude passwords, access tokens, refresh tokens, secrets, API keys, and other unnecessary sensitive information. | CLAUDE.md §12. | High |

**Open Question:** No monitoring/alerting tooling, log retention period, or log review process is specified.

---

## 9. Data Integrity

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-DATA-01 | Data Integrity | The database design must define appropriate primary keys, foreign keys, constraints, and required/nullable fields before implementation, reviewed against business rules. | CLAUDE.md §8. | High |
| NFR-DATA-02 | Data Integrity | Errors must be handled consistently so invalid or inconsistent data is not silently accepted by the system. | CLAUDE.md §11. | High |

---

## 10. Backup & Recovery

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-BACKUP-02 | Backup & Recovery | The system's data should be recoverable in the event of accidental loss or corruption. | General implication of §8/§18's caution around destructive operations; no explicit backup policy is stated. | Medium |

**Open Question:** No backup schedule, retention period, or recovery procedure is specified; this depends on the database hosting/operations approach, which is not yet finalized.

---

## 11. API Quality

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-API-01 | API Quality | The backend must expose functionality only through a RESTful API; the frontend must never access the database directly. | CLAUDE.md §4, §9; D17, D18. | High |
| NFR-API-02 | API Quality | API endpoints must use clear, consistent naming and correct HTTP methods. | CLAUDE.md §9. | High |
| NFR-API-03 | API Quality | API responses and error formats must be consistent across endpoints, using proper HTTP status codes. | CLAUDE.md §9. | High |
| NFR-API-04 | API Quality | The API must be documented via Swagger/OpenAPI. | CLAUDE.md §9; D9. | Medium |

**Open Question:** The exact API response format and error response format are listed as pending decisions and are not finalized (`DECISIONS.md` §23, "API response format" / "Error response format").

---

## 12. Deployment / Environment

| ID | Category | Requirement | Rationale | Priority |
|---|---|---|---|---|
| NFR-DEPLOY-02 | Deployment | Sensitive configuration (connection strings, credentials, secrets) must not be committed to source control. | Consistent with CLAUDE.md §12's prohibition on exposing secrets and the project's existing `.gitignore` setup for environment files. | High |
| NFR-DEPLOY-03 | Deployment | Database schema changes must go through a reviewed migration process; migrations must not be applied without review. | CLAUDE.md §8; D19. | High |

**Open Question:** The hosting/deployment platform, CI/CD process, and environment strategy (e.g., staging vs. production) are not specified.

---

## 13. Open Questions (NFR-specific)

These are unresolved non-functional details, separate from the Open Questions already recorded in `functional-requirements.md`. They are not resolved by this document.

1. **(Performance)** No numeric response-time or throughput targets are defined for any operation, interactive or batch.
2. **(Security)** Specific transport security and encryption-at-rest requirements are not defined; they depend on the still-pending authentication strategy and hosting decisions.
3. **(Availability)** No uptime/availability SLA is defined.
4. **(Usability)** No specific accessibility conformance level is named.
5. **(Scalability)** No expected number of children, classes, staff, or concurrent users is defined.
6. **(Compatibility)** No explicit browser/device support matrix is defined.
7. **(Logging & Monitoring)** No monitoring/alerting tool, log retention period, or log review process is defined.
8. **(Backup & Recovery)** No backup schedule, retention period, or recovery procedure is defined.
9. **(API Quality)** Standard API response and error response formats are not finalized (tracked as a pending decision in `DECISIONS.md`).
10. **(Deployment)** Hosting platform, CI/CD pipeline, and environment strategy are not decided.
