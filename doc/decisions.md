# GroupSwitch Architecture Decision Records

## 1. Purpose

This document records the main architecture and technology decisions for GroupSwitch v1.1.

The goal is to explain why the application is structured in a particular way, which alternatives were considered, and which consequences the team must account for.

The records below are initial decision proposals based on the client requirements and the current repository structure. A proposal becomes an accepted project decision only after the team confirms it. An implementation decision must not be described as verified without evidence.

## 2. Decision Status

Use the following status values:

- **Proposed:** a possible solution that still requires confirmation.
- **Accepted:** the team has agreed to the decision.
- **Implemented:** the decision is reflected in the code.
- **Verified:** implementation and relevant tests or other evidence confirm the expected behaviour.
- **Superseded:** a later decision replaces this decision.
- **Deferred:** the decision is postponed.

A technology required by the client may be mandatory even when some implementation details remain undecided.

---

## ADR-001: Separate Angular Frontend and ASP.NET Core API

**Status:** Accepted in principle — implementation to be verified.

### Context

The client requires two distinct applications: a web frontend and a REST API implemented with ASP.NET Core.

### Decision

GroupSwitch uses:

- Angular for the frontend.
- ASP.NET Core for the backend API.
- HTTP and JSON for ordinary communication between them.

The applications are developed and run separately.

### Alternatives Considered

- A server-rendered application without a separate frontend.
- A frontend directly accessing the database.
- A single application containing both frontend and backend responsibilities.

### Consequences

- Frontend and backend can be developed and tested independently.
- The API contract must be documented and kept consistent with Angular.
- Authentication and authorization must be enforced by the backend.
- API configuration must identify the correct frontend origin and backend address.

---

## ADR-002: Use the Existing Four-Project Backend Structure

**Status:** Proposed for confirmation against the current solution.

### Context

The repository currently contains the following backend projects:

- `GroupSwitch.Api`
- `GroupSwitch.Business`
- `GroupSwitch.Data`
- `GroupSwitch.Tests`

The client recommends a small layered architecture and does not require a separate Infrastructure project.

### Decision

Keep the existing four-project structure unless a concrete requirement justifies changing it.

### Responsibilities

- `GroupSwitch.Api`: HTTP endpoints, API configuration, authentication, authorization, middleware, and SignalR integration.
- `GroupSwitch.Business`: business rules, validation, exchange search, assignment management, timetable conflict logic, and AI criteria processing.
- `GroupSwitch.Data`: EF Core configuration, database access, migrations, and persistence implementations.
- `GroupSwitch.Tests`: automated tests for business rules, security, integration, and critical workflows.

### Alternatives Considered

- Adding separate Domain, Application, and Infrastructure projects.
- Placing business logic directly in API controllers.
- Combining all backend responsibilities into one project.

### Consequences

- The structure remains manageable for the size and duration of the project.
- Responsibilities must be kept separate even though the number of projects is limited.
- Project references must not create circular dependencies.
- Additional projects should only be introduced if they provide a clear benefit.

---

## ADR-003: Use Entity Framework Core and a Relational Database

**Status:** Required technology; database provider to be confirmed.

### Context

The application must persist students, study blocks, groups, assignments, virtual categories, change requests, and user accounts.

### Decision

Use Entity Framework Core with a relational database.

SQLite is a possible option for a prototype, provided that the chosen provider supports the required persistence and transaction behaviour.

### Alternatives Considered

- A non-relational database as the primary store.
- Direct SQL access without EF Core.
- In-memory persistence as the application's permanent data store.

### Consequences

- Relationships and data integrity must be represented explicitly.
- Database constraints should protect important invariants where possible.
- Migrations must be managed and reviewed.
- Transaction and concurrency behaviour must be tested with the selected provider.

---

## ADR-004: Model Assignments Separately from Students and Groups

**Status:** Required by the client; implementation to be verified.

### Context

A student may study activities from several blocks simultaneously. For example, a student mainly following B3 may still need a B2 activity.

A student therefore cannot be represented by a single group field.

### Decision

Represent the relationship using a separate Assignment entity that references a Student and a Pedagogical Group.

The model must permit multiple active assignments for one student across different blocks while preventing multiple active assignments for the same student within one block.

### Alternatives Considered

- A single `GroupId` property on the Student entity.
- A separate assignment table without validating block consistency.
- Treating virtual categories as actual group assignments.

### Consequences

- Assignment changes can be scoped to one block.
- Other assignments remain unchanged when one assignment is modified.
- Group occupancy must be calculated from actual assignments.
- Tests must cover cross-block assignments and the one-assignment-per-block rule.

---

## ADR-005: Enforce Business Rules in the Backend

**Status:** Required by the client; implementation to be verified.

### Context

Frontend validation improves usability but can be bypassed by calling the API directly.

### Decision

The backend is authoritative for business validation.

It must validate group capacities, assignment consistency, request status transitions, exchange compatibility, timetable conflicts, and permissions.

### Alternatives Considered

- Enforcing rules only in Angular.
- Trusting data returned by the AI service.
- Trusting the result of an earlier frontend validation.

### Consequences

- API operations must revalidate critical conditions against current data.
- Business rules can be tested without the frontend.
- Validation failures must return understandable API responses.
- Database constraints and transactions must support critical invariants.

---

## ADR-006: Apply Exchange Cycles Atomically

**Status:** Required by the client; implementation to be verified.

### Context

A change may require a cycle involving three or more students. If one step fails, a partially applied exchange would leave the data inconsistent.

### Decision

The backend must validate and apply the exchange as a single atomic operation using a suitable database transaction.

The proposal must involve groups from the same study block and must not reuse a request or assignment within the same exchange.

### Alternatives Considered

- Updating each assignment independently without a transaction.
- Applying changes before validating all participants.
- Treating each participant's change as an unrelated operation.

### Consequences

- The backend must validate the full proposal before committing.
- Concurrent changes must be handled.
- A failure must roll back the complete exchange.
- Tests must inspect persisted state after both successful and failed operations.

---

## ADR-007: Use JWT Authentication and Backend Authorization

**Status:** Required by the client; implementation details to be confirmed.

### Context

GroupSwitch requires account management, login, JWT authentication, and role-based access.

At least Student and Coordinator/Administrator access levels are required.

### Decision

Use JWT authentication with backend authorization. ASP.NET Core Identity is the recommended option for account and password management, but the selected implementation must be documented.

The backend must verify the token and check both role permissions and resource ownership.

### Alternatives Considered

- Frontend-only authentication.
- A simulated login screen.
- Trusting role values submitted by the client.
- Allowing students to access requests based only on knowledge of their IDs.

### Consequences

- Signing keys and other secrets must remain outside Git.
- Passwords must be securely hashed.
- Protected endpoints must reject unauthenticated requests.
- Students must not access another student's private requests.
- Security tests must cover direct API calls.

---

## ADR-008: Keep Hyperplanning Retrieval Separate from Timetable Rules

**Status:** Required separation; retrieval implementation to be verified.

### Context

The application must retrieve real timetable data from the targeted Hyperplanning instance and detect conflicts across activities, including activities in different study blocks.

### Decision

Isolate web scraping or other approved retrieval logic in a dedicated component or service.

Map retrieved data to an internal timetable representation. Perform overlap calculations in deterministic C# business logic.

### Alternatives Considered

- Mixing HTML parsing into API controllers.
- Asking the AI model to determine timetable conflicts.
- Using fabricated timetable data as a replacement for the required real retrieval.

### Consequences

- Retrieval and parsing failures must be distinguishable from actual timetable conflicts.
- The internal timetable model must be documented.
- Conflict calculations can be tested using controlled fixtures.
- At least one real integration or demonstration must retrieve data from the targeted instance.

---

## ADR-009: Use AI as Decision Support, Not as the Decision Maker

**Status:** Required by the client; integration to be verified.

### Context

The AI must interpret a student's free-text reason and help the coordinator explore candidate solutions.

A chatbot interface is not required.

### Decision

Call the selected AI service from the backend. Request structured output containing a summary, preferences, explicitly declared constraints, uncertainties, and relevant activities.

Validate the response before using its criteria in the candidate-search workflow.

The backend remains responsible for capacity checks, assignment consistency, timetable conflict detection, and exchange validation. The coordinator makes the final decision.

### Alternatives Considered

- A chatbot that only summarizes requests.
- Letting the AI approve or reject requests.
- Allowing the AI to modify assignments directly.
- Treating AI-generated timetable claims as factual verification.

### Consequences

- The output schema must be documented.
- Invalid responses and service failures must be handled safely.
- Tests should use controlled AI responses rather than relying on a remote model for every run.
- Candidate solutions must be verified by deterministic rules and real timetable data.
- Unnecessary personal data must not be sent to the provider.

---

## ADR-010: Use SignalR for Real-Time Notifications

**Status:** Required by the client; implementation to be verified.

### Context

The frontend must visibly update when a relevant backend operation succeeds, without requiring manual page reload.

### Decision

Use a SignalR Hub for real-time notifications while retaining REST for reading and modifying application resources.

At minimum, demonstrate notifications when a change request is created, accepted, or rejected.

### Alternatives Considered

- Requiring manual refresh.
- Periodic polling as the only notification mechanism.
- Using SignalR as the main mechanism for all CRUD operations.

### Consequences

- The frontend must manage Hub connection state.
- Events must be authenticated and authorized.
- The backend must publish success events only after the relevant operation commits.
- Clients must handle temporary connection loss.
- Tests must ensure that unauthorized clients do not receive private events.

---

## ADR-011: Provide CSV and PDF Exports from the Backend

**Status:** Required by the client; implementation to be verified.

### Context

The coordinator must export one group or all groups in a selected study block.

### Decision

Generate CSV and PDF files on the backend using the actual recorded assignments.

CSV output must document its delimiter, encoding, headers, escaping, and spreadsheet-formula mitigation.

PDF output must remain readable when lists span several pages and must preserve accents and compound names.

### Alternatives Considered

- Generating exports only in the browser.
- Exporting all student assignments regardless of the selected block.
- Omitting empty groups.

### Consequences

- Every export request requires backend authorization.
- Export queries must be scoped to the selected group or block.
- Empty groups must be represented.
- Tests must cover both file formats, block isolation, accents, and unauthorized access.

---

## ADR-012: Use a Consistent API Error Contract

**Status:** Proposed; implementation to be verified.

### Context

The frontend needs predictable responses for validation failures, unauthorized operations, conflicts, and unexpected errors.

### Decision

Use consistent HTTP status codes and a documented error structure, preferably ASP.NET Core Problem Details.

### Alternatives Considered

- Returning HTTP 200 for all outcomes.
- Returning unstructured exception text.
- Exposing stack traces to clients.

### Consequences

- Angular can display errors consistently.
- Tests can verify expected status codes and response fields.
- Internal exception details remain in controlled server-side logs.
- The API contract must document important failure scenarios.

---

## ADR-013: Use Automated Tests for Business Rules and Security

**Status:** Required by the client; coverage to be verified.

### Context

The application has important integrity constraints and protected operations.

### Decision

Use automated unit and integration tests for the core business rules, API security, persistence, AI response validation, and export behaviour.

Use controlled fixtures for timetable and AI tests. Reserve real external integration checks for appropriate integration or demonstration scenarios.

### Alternatives Considered

- Relying exclusively on manual demonstration.
- Treating a successful build as proof of correctness.
- Depending on a live AI service for every test.

### Consequences

- Tests must be reproducible.
- Critical failure cases must be tested, not just successful paths.
- Test results must be recorded.
- Requirements must not be marked verified without evidence.

---

## 3. Decisions Still to Confirm

The following choices should be explicitly recorded when the team has made them:

- Exact .NET target framework and dependency versions.
- Relational database provider.
- Entity and DTO naming conventions.
- Whether a repository abstraction is needed.
- JWT access-token lifetime and refresh-token strategy.
- Token storage and frontend security considerations.
- AI provider or local model and structured output schema.
- Hyperplanning target instance, access method, and parsing approach.
- PDF-generation library.
- CSV delimiter and encoding.
- SignalR event names and recipient rules.
- Initial-distribution algorithm and reproducibility strategy.
- Database concurrency approach.
- Deployment environment and backup procedure.

## 4. Decision Review

Review an architecture decision when:

- A mandatory requirement cannot be met with the current approach.
- A significant security or data-integrity issue is identified.
- A dependency or external service changes.
- A simpler solution would meet the same requirements more reliably.
- The current implementation diverges from the documented decision.

When replacing a decision, preserve the reason for the change and mark the old record as Superseded.

## 5. Review Record

| Field                       | Value            |
| --------------------------- | ---------------- |
| Project                     | GroupSwitch v1.1 |
| Review date                 | To be completed  |
| Reviewed by                 | To be completed  |
| Decisions accepted          | To be completed  |
| Decisions requiring changes | To be completed  |
| Open risks                  | To be completed  |
| Follow-up actions           | To be completed  |
