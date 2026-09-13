# System Architecture — Kindergarten Management System

## 0. Document Information

- **Stage:** Phase 8 — System Architecture Design (design only — no source code, no EF Core entities, no migrations, no SQL, no frontend implementation).
- **Source of truth:** `CLAUDE.md`, `docs/requirements/functional-requirements.md`, `docs/requirements/non-functional-requirements.md`, `docs/business/user-roles.md`, `docs/business/use-cases.md`, `docs/business/business-rules.md`, `docs/requirements/acceptance-criteria.md`, `docs/database/database-design.md`, `docs/database/erd.puml`.
- **Status:** Draft — for review before Backend/Frontend project setup begins.
- **Approved technology stack (unchanged, not re-decided here):** ASP.NET Core Web API, Entity Framework Core, PostgreSQL/Supabase, React/Next.js + TypeScript, Tailwind CSS, React Hook Form, TanStack Query, Axios, Swagger/OpenAPI, Serilog, FluentValidation (`CLAUDE.md` §4).
- **Convention:** Where an architectural decision depends on an Open Question already recorded in the requirements/business/database documents, this document states the dependency explicitly and designs a boundary flexible enough to absorb either outcome, rather than guessing the outcome.

---

## 1. Architecture Overview

The system is a **layered monolith**, not a distributed system: one ASP.NET Core Web API backend, one Next.js frontend, one PostgreSQL database (hosted on Supabase). This matches `CLAUDE.md` §4's architecture rule (frontend never touches the database directly) and the project's timeline/team-size constraints.

**Major components:**

| Component | Responsibility |
|---|---|
| **Frontend (Next.js/React/TypeScript)** | Renders UI for the 5 roles, collects input, calls the backend REST API, manages client-side/server-state, never accesses the database. |
| **Backend API (ASP.NET Core Web API)** | Authenticates/authorizes requests, validates input, enforces business rules, is the only component that talks to the database, returns JSON. |
| **Database (PostgreSQL / Supabase)** | Persists the 20 tables defined in `docs/database/database-design.md`; accessed only through EF Core from the backend. |
| **External integrations (storage, payment)** | Isolated behind backend-side interfaces; concrete providers are pending Open Questions (media storage — `DECISIONS.md` D22; payment — D21) and are not chosen here. |

No microservices, no message broker, no CQRS/event sourcing, no EAV. A single backend solution with clear internal layers (§2) is sufficient for the scope in `docs/business/use-cases.md` (24 use cases) and keeps the system easy to explain and demo.

---

## 2. Backend Architecture

### 2.1 Layers

A single ASP.NET Core Web API solution, internally organized into four conceptual layers (as folders, or as a small number of projects — either is acceptable; folders are simplest for this team size):

```
Api            → Controllers, DTOs (request/response), middleware, Swagger config
Application    → Services (business logic), validators, interfaces for external concerns
Domain         → Entity/model classes, enums, domain constants
Infrastructure → EF Core DbContext, EF configurations, storage/payment implementations, Serilog setup
```

### 2.2 Responsibilities and dependency direction

- **Api (Controllers/DTOs):** Accepts HTTP requests, maps them to DTOs, calls Application services, maps results back to response DTOs, sets HTTP status codes. Contains no business logic and no direct EF Core/database calls.
- **Application (Services):** Contains business rules from `docs/business/business-rules.md` (e.g., BR-ATTENDANCE-02 "enrollment + class assignment required before check-in," BR-PICKUP-02 "non-parent pickup checked against registered list"). Services depend on the Domain layer and on small, purpose-specific interfaces (e.g., `IFileStorageService`, `IPaymentService` — see §10, §11) rather than on Infrastructure directly.
- **Domain:** Plain model classes mirroring the entities in `docs/database/database-design.md` (e.g., `Child`, `Class`, `Attendance`, `Invoice`). No framework dependencies here.
- **Infrastructure:** Implements the interfaces the Application layer depends on: the EF Core `DbContext` (data access), and — once their respective Open Questions are resolved — the concrete file storage and payment integrations. Also hosts Serilog configuration.

**Dependency direction:** `Api → Application → Domain`, and `Infrastructure → Application (interfaces) + Domain`. Nothing in Domain or Application depends on Infrastructure or Api. This is the standard "dependency inversion at the edges" shape — simple enough for a student project, but it keeps the two genuinely uncertain areas (storage, payment) swappable without touching business logic.

**No generic repository/unit-of-work layer is introduced.** EF Core's `DbContext` already provides that role; adding a repository abstraction on top of it for straightforward CRUD would be the kind of unnecessary abstraction this project explicitly avoids. Services depend on `DbContext` (or a thin, feature-specific query interface only where genuinely useful) directly.

### 2.3 Where things belong

- **Validation:** Input-shape/field validation (required fields, formats) happens in the Api layer via FluentValidation validators bound to request DTOs, before a request reaches a Service — consistent with NFR-SEC-02 ("backend validates regardless of frontend"). Cross-entity **business-rule** validation (e.g., "invoice must be unpaid before payment," BR-TUITION-05) happens in the Application layer's Services, because it requires reading related data, not just the shape of one request.
- **Authorization:** Role checks happen at the Api layer, on the Controller/endpoint (via ASP.NET Core's `[Authorize(Roles = ...)]` or policy-based authorization), mapped to the 5 roles in `docs/business/user-roles.md`. This is independent of whichever authentication mechanism is eventually chosen (§5).
- **Database access:** Only in Infrastructure, only through EF Core's `DbContext`. No other layer opens a database connection.

### 2.4 EF Core / PostgreSQL naming note

`docs/database/database-design.md` uses `snake_case` table/column names (idiomatic PostgreSQL). EF Core entity classes will use normal C# `PascalCase` properties; a naming convention (e.g., the common `EFCore.NamingConventions` package or explicit Fluent API mapping) should be configured in Infrastructure so the two stay aligned without renaming either side. This is a technical mapping detail, not a schema change.

---

## 3. Frontend Architecture

### 3.1 Structure

```
app/ (or pages/)      → routes, grouped by role area (admin, teacher, accounting, medical, parent, auth)
components/           → reusable UI building blocks (forms, tables, cards, layout)
features/ or modules/ → feature-oriented groupings (attendance, pickup, health, tuition, ...)
lib/api/              → Axios instance + one client function per backend resource
hooks/                → TanStack Query hooks (useAttendanceHistory, useInvoices, ...)
types/                → TypeScript types mirroring backend DTOs
```

Route grouping by role keeps navigation aligned with `docs/business/user-roles.md`'s Actor → Use Case matrix, and makes it easy to gate a whole route group behind a role check.

### 3.2 API communication

- All server communication goes through a single configured **Axios** instance (base URL, JSON headers, and — once decided — the authentication credential attachment point for whichever mechanism is chosen in §5).
- **TanStack Query** wraps every Axios call: `useQuery` for reads (attendance history, invoices, dashboard stats, ...), `useMutation` for writes (check-in, record incident, register pickup person, pay invoice, ...), with cache invalidation tied to the related query keys after a successful mutation.
- **React Hook Form** manages form state and client-side validation for UX responsiveness only (e.g., required-field hints before submit); it never replaces backend validation (NFR-SEC-02) — the backend re-validates and is authoritative.

### 3.3 Client/server responsibility

- The frontend renders UI and orchestrates calls to the backend API; it holds no direct database credentials and no Supabase client-side database access, even though Supabase offers a JS SDK for that — this project deliberately does not use that capability, to respect `CLAUDE.md` §4.
- Next.js may render initial page shells server-side for performance, but all *data* still flows through the same REST API — there is no server-side code path that talks to PostgreSQL directly from the Next.js server either.

### 3.4 Loading / error / empty states

Every TanStack Query consumer follows the same three-state pattern (`isLoading`, `isError`, empty-data check) before rendering real content, consistent with NFR-USE-02. Error states show a generic, safe message (never raw backend exception text — see §8).

---

## 4. Request Flow

```
React component
  → TanStack Query hook (useMutation/useQuery)
    → Axios call
      → ASP.NET Core Web API (Controller)
        → [Authentication check] → [Authorization/role check]
        → Model binding + FluentValidation (request DTO)
        → Application Service (business rules from business-rules.md)
          → DbContext (EF Core)
            → PostgreSQL / Supabase
          ← query/command result
        ← Service returns a result or throws a domain/validation error
      ← Controller maps result to response DTO + HTTP status
    ← Axios receives JSON
  ← TanStack Query updates cache
← React re-renders with new data / error / loading state
```

This is the same flow for every functional area (attendance, pickup, health, incidents, media, tuition, menu, notifications, dashboard) — only the specific Controller/Service/table differ (see §16 traceability).

---

## 5. Authentication and Authorization Boundary

**Open Question #1 (authentication strategy) is unresolved and is not decided here.** The architecture instead fixes *where* authentication and authorization happen, so any of the options under evaluation (`DECISIONS.md` D20 — ASP.NET Identity with cookies, JWT bearer tokens, or an external provider integrated via NextAuth) can be plugged in without changing Controllers, Services, or the frontend's data layer beyond the credential-attachment point in the Axios instance.

- **Authentication** happens at the edge of the backend request pipeline (ASP.NET Core authentication middleware), before a request reaches any Controller action. Its job is only to establish *who* the caller is and *which role* they hold (`FR-AUTH-01`).
- **Authorization** happens immediately after, at the Controller/endpoint level, using role-based checks against the 5 roles (`FR-AUTH-02`, `BR-AUTH-02`). Because it is role-based rather than tied to a specific authentication technology, it does not need to change when the authentication mechanism is finalized.
- If permission granularity is later confirmed to be finer than role-only (Open Question #2 in `business-rules.md`), ASP.NET Core's policy-based authorization can express that without restructuring this boundary — it is additive, not a redesign.
- The frontend's only authentication-related responsibility is attaching whatever credential the chosen mechanism requires to outgoing Axios requests, and redirecting unauthenticated/unauthorized users; it never independently decides what a user is allowed to do — that is always re-checked by the backend.

---

## 6. Database Access

- **The backend is the only component that connects to PostgreSQL/Supabase.** The frontend never receives a database connection string or Supabase service/anon key that would allow direct table access.
- **EF Core `DbContext`** (in Infrastructure) is the single point of database access for the whole backend. It exposes `DbSet<T>` per entity, matching the 20 tables in `docs/database/database-design.md`, and encapsulates all queries and commands used by Application-layer Services.
- The existing database design is respected as-is: no additional tables, columns, or constraints are introduced by this architecture document. (See §15 for the one design note this phase surfaced, not a contradiction.)

---

## 7. API Boundary

- **REST endpoints** are the only way the frontend reaches backend functionality — one Controller per functional area (Auth, Attendance, Pickup, Health, Incidents, Media, Tuition, Menu, Notifications, Users/Classes/Teachers, Dashboard), matching the modules already established in `docs/requirements/functional-requirements.md`.
- **AuthController** (`POST /api/auth/login`, `GET /api/auth/me`) is implemented — see `DECISIONS.md` D20 for the full authentication strategy (backend-issued JWT, no NextAuth, no BFF).
- **DTOs** (request and response) are distinct from Domain entities and from the database schema. Their job is to expose only what a given role/action needs (e.g., a parent's invoice view never returns another child's data) and to insulate the API contract from internal schema changes.
- **The full endpoint list, exact routes, and DTO field lists are out of scope for this document** — they belong to the upcoming API Design phase. This document only fixes the boundary and responsibility, not the catalog.

---

## 8. Error Handling

- A single **global exception-handling middleware** in the Api layer catches unhandled exceptions, logs the full detail via Serilog (§9), and returns a generic, safe JSON error body with HTTP `500` — never a stack trace or internal exception message to the client (NFR-SEC-03).
- **Validation errors** (FluentValidation failures on a request DTO) return HTTP `400` with field-level messages generated from the validators — no invented error-code taxonomy, since the exact response/error envelope format is itself a pending decision (`DECISIONS.md` §23 — "API response format" / "Error response format"; NFR-API-03 Open Question). This document fixes *that* validation errors are structured and consistent, not their exact JSON shape.
- **Business-rule violations** (e.g., attempting check-in without an enrollment/class assignment — BR-ATTENDANCE-02) are raised by a Service as a distinguishable error type and translated by the Api layer into an appropriate client status (`400` or `409`, exact choice deferred to API Design) with a business-readable message — never a raw exception message.
- **Unexpected errors** always fall through to the global handler above; no Controller or Service should return a raw exception to the client under any circumstance.

---

## 9. Logging

- **Serilog** is configured once, at the Api host startup, and used from Api middleware and Application Services — not scattered ad hoc across the codebase.
- Logged: request/response summaries (route, status code, duration), authentication/authorization failures, business-rule rejections, and unhandled exceptions (full detail, server-side only).
- **Never logged:** passwords, tokens, secrets, API keys (`CLAUDE.md` §12, NFR-LOG-02) — and, consistent with that same principle, full request/response **bodies for health, incident, and personal-data endpoints** should not be logged wholesale; log that an action occurred (who, when, which child ID) rather than the health/incident content itself, to avoid incidentally accumulating sensitive personal data in log storage.
- Log retention/monitoring tooling is an Open Question (`non-functional-requirements.md` §13 item 7) and is not decided here.

---

## 10. File/Media Handling

Incident photos (`FR-INCIDENT-02`) and activity photos (`FR-MEDIA-01`) both need a place to live, but **the storage provider is an unresolved Open Question** (`DECISIONS.md` D22). The architecture isolates this behind a single Application-layer interface, e.g. `IFileStorageService` (upload → returns a reference string; retrieve/serve by reference), implemented once in Infrastructure.

- Controllers/Services only ever work with the storage-agnostic reference string already modeled in the database (`incident_photos.file_reference`, `activity_photos.file_reference` — `docs/database/database-design.md` §5). They never construct provider-specific URLs or SDK calls directly.
- Switching providers later (e.g., Supabase Storage, Cloudinary, S3) means writing one new Infrastructure implementation of `IFileStorageService` — no change to Controllers, Services, DTOs, or the database schema.
- Upload-time concerns (file type/size checks) belong in the Api layer, before a file reaches the storage interface (see §13).
- Whether teachers can upload video (not just photos) is a separate Open Question (`functional-requirements.md` Open Question #10) and is not assumed here; the interface is designed for "a file," not specifically "a photo," so it would not need to change if video is confirmed later — only the allowed-type validation would.

---

## 11. Payment Integration Boundary

The payment provider is an unresolved Open Question (`DECISIONS.md` D21). Payment functionality is isolated behind an Application-layer `IPaymentService` interface (e.g., "initiate payment for this invoice," "handle a payment confirmation"), implemented once in Infrastructure against whichever provider is chosen (VNPay, MoMo, PayOS, Stripe, or another option — none selected here).

- Core tuition/invoice logic (`BR-TUITION-01` through `BR-TUITION-08`) — fee catalog, invoice creation, the `unpaid`/`paid` status transition — lives entirely in the Application layer and the `invoices`/`payments` tables, and does not know which provider is used.
- Only the mechanics of *talking to* the provider (redirect/callback handling, provider-specific fields) live in the Infrastructure implementation, consistent with `payments.external_reference` being a generic, nullable field in the database design.
- How a failed/declined payment should be handled is an open item (`use-cases.md` Open Question #3) and is intentionally not designed here; the boundary above just ensures that whatever behavior is decided can be implemented inside `IPaymentService` and the Application layer without leaking provider specifics elsewhere.

---

## 12. Configuration and Environment

- All environment-specific values — PostgreSQL/Supabase connection string, authentication settings (once chosen), file-storage credentials (once chosen), payment-provider credentials (once chosen) — are read from ASP.NET Core's configuration system (`appsettings.{Environment}.json` for non-secret defaults, environment variables / user-secrets / a secret manager for anything sensitive). **Nothing sensitive is hard-coded or committed to source control**, consistent with NFR-DEPLOY-02 and `CLAUDE.md` §12.
- The frontend reads its own environment-specific values (e.g., API base URL) via Next.js's environment-variable conventions, keeping a clear line between what is safe to expose to the browser and what is not.

---

## 13. Security Considerations

- **Authentication:** mechanism deferred (§5), but always required before any role-specific action (FR-AUTH-01).
- **Authorization:** role-based checks on every endpoint (FR-AUTH-02); no endpoint is reachable without an explicit role requirement.
- **Input validation:** always re-validated server-side (NFR-SEC-02), regardless of frontend checks.
- **Sensitive data handling:** child health/incident/personal data is only ever returned to roles permitted by `docs/business/user-roles.md`; logging excludes sensitive content (§9).
- **API security:** consistent, safe error responses (§8); no internal detail ever reaches the client.
- **File upload considerations:** validate file type and size before handing to `IFileStorageService`; never trust or reuse a client-supplied filename directly for storage; never allow an uploaded file to be executed.
- **Secret management:** environment-based only (§12); no secret ever appears in a Git-tracked file.

---

## 14. Deployment Overview

A simple, realistic shape appropriate for a university project — not a fixed infrastructure decision, since the hosting/deployment platform is itself an open NFR item (`non-functional-requirements.md` §12 Open Question):

```
Frontend (Next.js)  → any Node-compatible hosting (e.g., a PaaS or static/SSR host)
Backend (ASP.NET Core Web API) → any container- or PaaS-style .NET host
Database → Supabase-hosted PostgreSQL (already decided, D16)
```

No load balancers, no multi-region setup, no container orchestration platform — a single instance of the frontend and a single instance of the backend, both pointing at the one Supabase database, is sufficient to build and demonstrate this project within the timeline.

---

## 15. Architecture Decisions and Trade-offs

**Why this shape fits the project:**

- A layered monolith is far easier for a small student team to build, understand, debug, and present than any distributed alternative — and nothing in the requirements (24 use cases, single-kindergarten scale, `non-functional-requirements.md` NFR-SCALE-01) calls for more.
- Skipping a generic repository layer over EF Core removes boilerplate without losing testability — Services can still be tested against an in-memory/test `DbContext`.
- Isolating only the two genuinely uncertain integrations (storage, payment) behind interfaces — rather than wrapping everything in interfaces "for flexibility" — keeps the codebase simple while still protecting the two areas that really do depend on unresolved decisions.

**Decisions made:**
- Single ASP.NET Core Web API + Next.js frontend, no microservices.
- Four-layer backend organization (Api / Application / Domain / Infrastructure), no repository/unit-of-work abstraction over EF Core.
- Role-based authorization at the Controller level, independent of the authentication mechanism.
- Storage and payment isolated behind purpose-specific interfaces; nothing else is abstracted "just in case."
- Global exception-handling middleware as the single place unhandled errors are turned into safe responses.

**Decisions explicitly left open (dependent on unresolved Open Questions):**
- The authentication mechanism itself (Open Question #1 / `DECISIONS.md` D20).
- Whether authorization needs to go beyond role-only (Open Question #2).
- The storage provider for photos (Open Question #10 / D22) and whether teacher video upload is in scope.
- The payment provider and failed-payment handling (Open Question #13 / D21, and `use-cases.md` Open Question #3).
- The exact API response/error envelope shape (NFR-API-03 Open Question).
- The hosting/deployment platform and CI/CD process (NFR-DEPLOY Open Question).

---

## 16. Traceability to Functional Areas

| Functional area | Architecture path |
|---|---|
| Authentication | `AuthController` → `AuthService` + `ITokenService` + `IPasswordHasher` → `IAuthRepository` → `AppDbContext` → `users`, `roles` (D20) |
| Child management | `ChildrenController` → `ChildService` (BR-STUDENT-01/02) → `DbContext` → `children` |
| Class management | `ClassesController` → `ClassService` (BR-CLASS-01/02) → `DbContext` → `classes` |
| Attendance | `AttendanceController` → `AttendanceService` (BR-ATTENDANCE-01/02/03) → `DbContext` → `attendances` |
| Pickup | `PickupController` → `PickupService` (BR-PICKUP-01/02/03) → `DbContext` → `pickups`, `registered_pickup_persons` |
| Health | `HealthController` → `HealthService` (BR-HEALTH-01–04) → `DbContext` → `quick_health_statuses`, `growth_measurements` |
| Incidents | `IncidentsController` → `IncidentService` (BR-INCIDENT-01–04) → `DbContext` + `IFileStorageService` → `incidents`, `incident_photos` |
| Activity media | `MediaController` → `MediaService` (BR-MEDIA-01/02) → `DbContext` + `IFileStorageService` → `activity_photos` |
| Tuition | `TuitionController` → `TuitionService` (BR-TUITION-01–03/07/08) → `DbContext` → `tuition_fees`, `invoices` |
| Payments | `PaymentsController` → `PaymentService` + `IPaymentService` (BR-TUITION-05/06) → `DbContext` → `payments` |
| Weekly menu | `MenuController` → `MenuService` (BR-MENU-01) → `DbContext` → `weekly_menus`, `menu_entries` |
| Notifications | `NotificationsController` → `NotificationService` (BR-NOTIFICATION-01) → `DbContext` → `notifications` |
| Dashboard | `DashboardController` → `DashboardService` (BR-DASHBOARD-01/02) → `DbContext` (computed over `children`/`attendances`/`invoices`/`payments`) |

---

## 17. Architecture Diagram

See `docs/architecture/architecture.puml` (PlantUML, consistent with `DECISIONS.md` D32's choice of PlantUML for project diagrams).

---

## 18. Consistency Check Against Database Design

No contradiction with `docs/database/database-design.md` or `docs/database/erd.puml` was found. One note surfaced during this phase (not a contradiction, no schema change made): the generic `file_reference` (photos) and `external_reference` (payments) text fields already chosen in the database design turn out to be exactly what a provider-agnostic architecture needs — the database design was already built compatibly with keeping storage/payment swappable, so no change was necessary.
