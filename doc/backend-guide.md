# GroupSwitch Backend Guide

## 1. Purpose

This document describes the backend architecture and code organization of GroupSwitch v1.1.

GroupSwitch is a web application for managing students, study blocks, pedagogical groups, student assignments, change requests, exchange proposals, timetable conflicts, and exports.

The backend exposes a REST API consumed by the Angular frontend. It is responsible for data persistence, authentication, authorization, business rules, timetable validation, and coordination with external services.

This document describes the intended architecture. A feature must only be considered implemented when its code and tests demonstrate the required behaviour.

## 2. Solution Structure

The current solution uses the following structure:

```text
GroupSwitch/
├── backend/
│   ├── GroupSwitch.Api/
│   ├── GroupSwitch.Business/
│   ├── GroupSwitch.Data/
│   └── GroupSwitch.Tests/
├── frontend_angular/
│   ├── src/
│   │   └── app/
│   │       ├── core/
│   │       ├── layout/
│   │       ├── features/
│   │       ├── shared/
│   │       ├── app.routes.ts
│   │       └── app.config.ts
│   └── package.json
└── GroupSwitch.sln
```

### 2.1. GroupSwitch.Api

**Responsibility:** expose the HTTP API and configure the application.

This project should contain:

- API controllers or endpoint definitions.
- Request and response DTOs.
- Authentication and authorization configuration.
- Dependency injection configuration.
- Middleware for error handling and request processing.
- SignalR hubs and endpoint mappings.
- Application startup and configuration.
- API documentation configuration, if enabled.

API endpoints must not contain complex business logic. They should validate request structure, invoke the appropriate business service, and translate the result into an HTTP response.

### 2.2. GroupSwitch.Business

**Responsibility:** implement the application's business rules.

This project should contain services and domain logic for:

- Student and study-block management.
- Pedagogical group management and capacity rules.
- Student assignments.
- Change-request processing.
- Exchange-cycle search and validation.
- Initial student distribution.
- Timetable conflict detection.
- AI result validation and candidate-solution search.
- CSV and PDF export preparation, where appropriate.

The business layer must not rely on frontend behaviour to enforce rules. It should remain testable without making real HTTP requests to Angular, Hyperplanning, or an AI provider.

### 2.3. GroupSwitch.Data

**Responsibility:** persist and retrieve application data.

This project should contain:

- Entity Framework Core `DbContext`.
- EF Core entity configurations.
- Database migrations.
- Database queries and repositories, if repositories are used.
- Persistence-related implementations.
- Database initialization or controlled seed data, where required.

The database must be relational. SQLite is a possible choice, provided that the selected database supports the required constraints and transaction behaviour.

The persistence model must distinguish students, groups, assignments, virtual categories, change requests, user accounts, and any timetable or AI-analysis data that the implementation needs to retain.

### 2.4. GroupSwitch.Tests

**Responsibility:** verify application behaviour automatically.

Tests should cover, at minimum:

- Business rules and capacity constraints.
- Multiple assignments across different study blocks.
- Change-request validation.
- Exchange cycles involving at least three students.
- Transaction atomicity.
- Timetable conflict detection.
- AI output validation and failure handling.
- Authentication and authorization.
- CSV and PDF exports.
- API and persistence integration.

Tests must use controlled data. They must not depend on a paid AI service or a live Hyperplanning instance for every test.

## 3. Main Business Concepts

### Student

Represents a person enrolled in the programme.

### Study Block

Represents a study block, such as B1, B2, or B3.

### Pedagogical Group

Represents a group associated with a study block and a maximum capacity.

### Assignment

Connects a student to a pedagogical group.

A student may have active assignments in several study blocks. However, a student must not have more than one active assignment in the same study block.

A change to one assignment must not modify the student's assignments in other blocks.

### Virtual Category

Represents an administrative or pedagogical characteristic, such as B1, B2, B3, an external exemption, or a reduced study programme.

Virtual categories guide administrative management and initial distribution. They are not replacements for actual pedagogical assignments and are not visible to students.

### Change Request

Represents a student's request to move one specific assignment from its current group to a requested group within the same study block.

A request contains a mandatory reason and a status: Pending, Accepted, or Rejected.

### Exchange Proposal

Represents a proposed cycle of assignment changes. A proposal may involve three or more students and must concern groups from the same study block.

## 4. Dependency Direction

The projects should have clear responsibilities and limited coupling.

A recommended dependency arrangement is:

- `GroupSwitch.Api` references `GroupSwitch.Business` and the required persistence abstractions or registration entry points.
- `GroupSwitch.Business` contains business rules and contracts required by those rules.
- `GroupSwitch.Data` implements persistence and other data-access integrations.
- `GroupSwitch.Tests` references the projects needed to test their behaviour.

Avoid circular project references. If the current implementation requires a different reference arrangement, document the reason rather than introducing an unnecessary project.

## 5. Request Processing

A typical request follows this sequence:

1. Angular sends an HTTP request to the API.
2. ASP.NET Core authenticates the request when authentication is required.
3. Authorization policies check the user's permissions.
4. The API validates the request structure.
5. The business layer validates the operation against current business rules.
6. The data layer reads or changes persistent data.
7. The operation commits successfully or is rolled back.
8. The API returns a documented HTTP response.
9. Where required, the backend publishes a SignalR event after the business operation succeeds.

SignalR notifications must not announce a successful change before the corresponding database transaction has committed.

## 6. Important Business Rules

The backend must enforce these rules:

- A student may have assignments in several study blocks.
- A student may have at most one active assignment per study block.
- An assignment must reference an existing student and a group belonging to the relevant study block.
- Group capacity must never be exceeded.
- A change request concerns one specific assignment.
- The requested group must belong to the same study block as the current group.
- Accepting a change must preserve all unrelated assignments.
- An exchange cycle must be applied entirely or not at all.
- An assignment or request must not be reused more than once in the same exchange proposal.
- Timetable conflicts must be calculated deterministically in C# from retrieved timetable data.
- AI output must be validated before it influences candidate searches.
- AI unavailability must not prevent ordinary request management.
- Authorization must be enforced by the backend.

## 7. Transactions and Concurrent Operations

Exchange processing must use a transaction appropriate to the selected database and EF Core configuration.

Before committing an exchange, the backend must validate the current assignments, request statuses, group compatibility, and relevant capacity constraints.

A failed exchange must not leave some students moved while others remain unchanged.

Concurrency must also be considered. Two coordinators or concurrent operations must not be able to exceed group capacity or process the same request inconsistently. Use suitable database constraints, concurrency checks, transactions, or a combination of these mechanisms.

## 8. External Integrations

### Hyperplanning

Hyperplanning retrieval must be isolated behind a dedicated component or service.

Its responsibilities include retrieving timetable data from the targeted instance, mapping the result to a stable internal representation, and reporting retrieval or parsing failures.

The business layer must calculate overlaps using the retrieved dates and times. It must not ask the AI model to decide whether two sessions overlap.

The actual target instance, access method, permitted retrieval mechanism, and data format must be documented once confirmed.

### AI Service

The AI integration must run from the backend.

Its responsibilities include interpreting a free-text reason and returning structured criteria such as:

- Summary.
- Preferences.
- Explicitly declared hard constraints.
- Uncertainties or ambiguities.
- Relevant activities.
- Points that require verification.

The backend must validate and normalize the result before using it to search for candidate solutions. The AI must not modify assignments, authorize users, or make the final decision.

### SignalR

SignalR informs connected and authorized clients about relevant changes. REST remains the mechanism for reading and modifying application resources.

Events should identify the relevant request or study block without exposing unnecessary personal data.

## 9. API Contracts and Error Handling

API request and response formats should be documented in `api-contract.md`.

The API should return consistent validation and error responses. Unexpected exceptions must not expose stack traces, credentials, internal configuration, or database details to clients.

Use structured server-side logging with correlation information where practical. Do not log passwords, access tokens, or unnecessary personal data.

## 10. Configuration and Secrets

Environment-specific settings must be kept outside source code.

Examples include:

- Database connection settings.
- JWT signing configuration.
- AI provider configuration and API keys.
- Hyperplanning access settings.
- Allowed frontend origins.
- Logging levels.

Use .NET User Secrets during local development or an equivalent environment-specific secret mechanism. Production secrets must be supplied securely by the deployment environment.

## 11. Development and Quality Checks

Before considering a change complete:

1. Build the solution.
2. Run the relevant automated tests.
3. Add tests for new business rules and failure scenarios.
4. Check authorization on sensitive endpoints.
5. Verify transaction and concurrency behaviour where relevant.
6. Update the API contract and related documentation.
7. Confirm that no secrets or real student data were committed.
8. Verify that the feature works through the Angular frontend where applicable.

## 12. Open Implementation Decisions

The following details must be confirmed against the repository:

- The selected relational database and its version.
- The exact entity and service names.
- Whether repositories are used.
- The authentication and token-storage design.
- The PDF-generation library.
- The Hyperplanning retrieval mechanism and data mapping.
- The AI provider or local model.
- The exact REST routes and DTO definitions.
- The database concurrency strategy.

Do not document an unimplemented feature as operational merely because its intended design is described here.
