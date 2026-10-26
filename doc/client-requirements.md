# GroupSwitch — Client Requirements and Technical Specifications

**Project:** GroupSwitch
**Document type:** Requirements and Technical Specifications
**Version:** 1.2 — Draft for Validation
**Status:** Proposed — Pending Review
**Backend technology:** ASP.NET Core / .NET
**Frontend technology:** Angular
**Database:** Relational database using Entity Framework Core
**Document language:** English

---

# 1. Project Overview

## 1.1 Purpose

GroupSwitch is an application designed to manage students, groups, assignment changes, and group-switch requests.

The application aims to simplify the management of student group assignments, automate parts of the allocation process, detect timetable conflicts, and provide a controlled workflow for students and administrators.

The system should provide a reliable, secure, maintainable, and production-compatible solution.

## 1.2 Main Objectives

The main objectives are to:

- Manage students, groups, blocks, and group capacities.
- Import student information from CSV files.
- Distribute students among groups according to defined rules.
- Allow students to submit group-switch requests.
- Allow administrators to review and process requests.
- Support coordinated exchanges involving multiple students.
- Detect timetable conflicts.
- Provide real-time updates when appropriate.
- Integrate external timetable information where technically possible.
- Provide AI-assisted analysis and candidate suggestions.
- Export information in CSV and PDF formats.
- Implement a reliable .NET backend.
- Provide logging and error-handling mechanisms.
- Verify security, performance, and production compatibility.
- Provide complete technical documentation and project presentations.

## 1.3 Scope

The project includes the following main components:

1. Angular frontend.
2. ASP.NET Core REST API.
3. Relational database.
4. Entity Framework Core data-access layer.
5. Authentication and authorization.
6. Student and group management.
7. Group-switch request management.
8. Timetable conflict detection.
9. Real-time communication using SignalR, if confirmed.
10. External timetable integration, subject to availability.
11. AI-assisted features, subject to technical and functional validation.
12. Logging, monitoring, error handling, and maintenance documentation.
13. Testing and production-readiness verification.

The final scope must be validated against the original client brief.

---

# 2. User Roles

## 2.1 Student

A student should be able to:

- Access the application securely.
- View their own information and group assignment.
- View available groups and relevant capacity information.
- Submit a group-switch request.
- View the status of their requests.
- Receive relevant updates about their requests.
- Participate in an exchange when the applicable rules permit it.

Students must not be able to access or modify administrative information without authorization.

## 2.2 Administrator

An administrator should be able to:

- Manage student records.
- Manage groups, blocks, and capacities.
- Import student data.
- View and process group-switch requests.
- Review proposed exchanges.
- Access relevant timetable-conflict information.
- Export data when authorized.
- Review operational errors and maintenance information through authorized mechanisms.
- Perform administrative actions according to assigned permissions.

## 2.3 Technical and Maintenance Team

The technical team is responsible for activities assigned to it, including:

- Maintaining the .NET backend.
- Investigating application errors.
- Reviewing logs.
- Prioritizing maintenance operations.
- Monitoring application performance.
- Verifying security requirements.
- Maintaining technical documentation.
- Preparing deployment and troubleshooting procedures.

The exact permissions and responsibilities of this role must be agreed upon.

---

# 3. Functional Requirements

## FR-01 — Student Management

The system shall allow authorized administrators to manage student records.

The information managed may include:

- Student identifier.
- Name and other required personal information.
- Academic block or cohort.
- Current group assignment.
- Relevant timetable information.
- Account status, where applicable.

The application shall validate required information and prevent unauthorized modifications.

The exact student data fields must be confirmed.

## FR-02 — Group and Block Management

The system shall allow authorized administrators to manage academic blocks and groups.

Each group may have:

- A unique identifier.
- A name or code.
- An associated academic block.
- A maximum capacity.
- A list of assigned students.
- Relevant scheduling information.

The application shall enforce the capacity rules defined by the client.

## FR-03 — Multiple Group Assignments

The system shall support the group-assignment structure required by the academic organization.

The data model must represent the relationship between students, groups, and academic activities without introducing invalid assignments.

The precise rules for students assigned to multiple groups must be confirmed.

## FR-04 — Virtual Categories

The system shall support the virtual categories required by the client.

The purpose, rules, and relationships of these categories with students and groups must be documented before implementation.

## FR-05 — CSV Import

The system shall allow authorized administrators to import student or group data from CSV files.

The import process should:

- Validate the file format.
- Validate required columns.
- Validate individual records.
- Detect duplicate or conflicting data.
- Report invalid records.
- Provide a clear import summary.
- Avoid leaving the database in an inconsistent state.

The handling of partial imports versus complete rollback must be agreed upon.

## FR-06 — Initial Student Distribution

The system shall support the initial distribution of students into groups.

The distribution process must respect the validated business rules, including group capacity and any applicable academic constraints.

The allocation algorithm should be testable and produce results that can be explained or audited where required.

The exact allocation criteria must be confirmed.

## FR-07 — Group-Switch Requests

Students shall be able to submit a request to change groups, subject to the application's rules.

A request should contain:

- The requesting student.
- The current group.
- The requested group or change.
- The submission date.
- The request status.
- Any required justification.

The system shall validate the request before accepting it.

## FR-08 — Request Processing

The system shall allow authorized administrators to review and process group-switch requests.

Possible statuses may include:

- Pending.
- Approved.
- Rejected.
- Cancelled.

The final list of statuses must be validated.

The system should record relevant status changes and prevent invalid transitions.

## FR-09 — Multi-Student Exchange

The system shall support coordinated group exchanges involving three or more students if this requirement is confirmed by the client.

An exchange must be validated against all applicable business rules before being committed.

The system must ensure that the exchange is processed atomically: either all required assignment changes succeed, or none of them are applied.

The implementation must account for:

- Group capacities.
- Assignment validity.
- Timetable conflicts, where applicable.
- Concurrent requests.
- Database transaction consistency.
- Failure recovery.

## FR-10 — CSV Export

The system shall allow authorized users to export permitted data in CSV format.

The exported information must follow the requested format and access-control rules.

The export process should handle special characters correctly and avoid exposing unauthorized personal information.

## FR-11 — PDF Export

The system shall support PDF exports for the reports or information confirmed by the client.

The required reports, document layout, and included fields must be defined.

## FR-12 — Timetable Data Retrieval

The system should retrieve timetable information from Hyperplanning if a supported and authorized integration is available.

The implementation must document:

- The integration method.
- Authentication requirements.
- Data format.
- Availability limitations.
- Error handling.
- Timeout and retry policies.
- Any restrictions imposed by the external system.

The availability of an API or another permitted integration method must be verified before implementation.

## FR-13 — Timetable Conflict Detection

The system shall detect timetable conflicts according to the rules agreed upon by the client.

Conflict detection should be deterministic for the same input data and rules.

The system should provide sufficient information to explain why a conflict was detected.

The exact conflict rules must be documented and tested.

## FR-14 — Real-Time Updates

The system may use SignalR to communicate relevant changes to connected users.

Potential updates include:

- Request status changes.
- Group-assignment changes.
- Administrative decisions.
- Relevant notifications.

Real-time communication must respect authentication and authorization requirements.

The system must not disclose information to users who are not authorized to receive it.

## FR-15 — AI-Assisted Analysis

The system may provide AI-assisted analysis of group-switch situations.

AI output must be treated as assistance rather than an automatically authoritative decision unless explicitly approved by the client.

The implementation must document:

- The purpose of the AI feature.
- The data sent to the AI service.
- Data-protection measures.
- Technical limitations.
- Error handling.
- Human review requirements.

## FR-16 — AI-Assisted Candidate Search

The system may suggest potential candidates for a group exchange based on validated criteria.

Suggestions must be checked against the actual business rules before being accepted.

The AI must not bypass authorization, group-capacity constraints, or timetable validation.

If the AI service is unavailable, essential application functions should remain usable wherever technically possible.

## FR-17 — Authentication and Authorization

The system shall protect access to restricted functionality.

The implementation may use JWT-based authentication, subject to the project's security requirements.

The backend shall verify permissions for protected operations, rather than relying exclusively on frontend restrictions.

Administrative operations must be available only to authorized users.

The authentication lifecycle, token configuration, and role definitions must be documented.

## FR-18 — Reliability and Data Protection

The system shall handle invalid input, application errors, and integration failures in a controlled manner.

The application should prevent accidental exposure of sensitive information.

The implementation should protect the consistency of group assignments and requests.

Personal data must be processed and logged according to the applicable data-protection requirements.

## FR-19 — Testing and Demonstration

The project shall include tests appropriate to the criticality of its functions.

Testing should cover:

- Business rules.
- Data validation.
- API behavior.
- Authentication and authorization.
- Group-capacity constraints.
- Timetable-conflict detection.
- Multi-student exchange transactions.
- Error handling.
- Logging behavior where relevant.
- External integration failures.
- Performance of important operations.

A demonstration should present the validated functionality and clearly identify any limitations.

## FR-20 — Project Documentation

The project shall include technical and functional documentation sufficient to understand, install, configure, test, maintain, and demonstrate the application.

The documentation shall distinguish implemented features from proposed features and known limitations.

---

# 4. Additional Technical and Operational Requirements

The following requirements extend the functional scope with the technical responsibilities requested for the project. They must be reviewed and approved by the relevant stakeholders.

## OPS-01 — Logging and Error Handling

### Objective

Implement logging and error-handling mechanisms to improve system reliability, traceability, and maintainability.

### Requirements

The .NET backend shall use structured logging.

The logging mechanism should record relevant information such as:

- Timestamp.
- Log level.
- Application component.
- Operation being performed.
- Correlation or trace identifier.
- Relevant technical context.
- Exception details when appropriate.

The following log levels should be used consistently:

| Level       | Purpose                                                              |
| ----------- | -------------------------------------------------------------------- |
| Trace       | Detailed diagnostic information when explicitly enabled              |
| Debug       | Information useful during development and troubleshooting            |
| Information | Significant normal application events                                |
| Warning     | Unexpected situations that do not necessarily interrupt an operation |
| Error       | Failed operations requiring investigation                            |
| Critical    | Severe failures affecting the application or an essential function   |

### Error handling

The backend shall handle errors consistently.

The implementation should:

- Validate incoming requests.
- Return appropriate HTTP status codes.
- Use a consistent error-response structure.
- Distinguish validation errors from unexpected server failures.
- Avoid returning stack traces or internal implementation details to clients.
- Log unexpected exceptions.
- Include a correlation identifier when appropriate.
- Preserve database consistency when an operation fails.
- Provide understandable error messages to users.

For ASP.NET Core, the implementation may use centralized exception-handling middleware and standardized Problem Details responses.

### Security considerations

Logs must not contain:

- Passwords.
- Authentication tokens.
- API keys.
- Secrets.
- Unnecessary personal information.
- Sensitive request payloads.

Log access, retention, and deletion policies must be defined according to the project's requirements.

### Acceptance criteria

- Errors are returned using a consistent format.
- Validation failures produce appropriate responses.
- Unexpected exceptions are logged.
- Sensitive values are excluded from logs.
- Critical operations preserve data consistency when they fail.
- Automated tests cover important error scenarios.

## OPS-02 — Maintenance Operations and Prioritization

### Objective

Identify maintenance operations and rank them according to their impact, urgency, and technical risk, in collaboration with the relevant teams.

### Maintenance workflow

Each maintenance operation should document:

- Operation identifier.
- Description.
- Reason for the operation.
- Affected component.
- Impact on users or business operations.
- Severity and priority.
- Responsible person or team.
- Dependencies.
- Proposed corrective action.
- Planned date.
- Current status.
- Verification evidence.
- Closure date.

### Proposed priority classification

| Priority      | Meaning                                                                                | Example                                                           |
| ------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| P1 — Critical | Essential functionality unavailable, severe security issue, or risk of data corruption | Group assignments are corrupted or the application is unavailable |
| P2 — High     | Major functionality degraded with significant operational impact                       | Administrators cannot process requests                            |
| P3 — Normal   | Limited impact with an acceptable workaround                                           | An export format needs correction                                 |
| P4 — Low      | Minor improvement or non-urgent technical enhancement                                  | Improve a diagnostic message                                      |

This classification is a proposal and must be validated by the project stakeholders.

### Collaboration

Maintenance priorities should be reviewed with the relevant stakeholders, such as:

- Development team.
- Technical lead.
- Project owner.
- Application users or administrators.
- Infrastructure or operations team, if applicable.

### Acceptance criteria

- Maintenance operations are documented.
- Each operation has a priority and an owner.
- Critical issues are identified and escalated.
- Progress and resolution are traceable.
- Completed operations include verification evidence.

## OPS-03 — Technical Implementation in the .NET Backend

### Objective

Implement the application's technical operations in a maintainable ASP.NET Core backend.

### Architecture principles

The backend should separate responsibilities between:

- API controllers.
- Application or service layer.
- Business logic.
- Data-access layer.
- External integrations.
- Cross-cutting concerns such as logging and exception handling.

The final architecture must remain consistent with the existing repository and the project requirements.

### Implementation requirements

The backend should:

- Expose REST endpoints with documented contracts.
- Validate requests at the API boundary.
- Apply business rules in appropriate application or domain components.
- Use dependency injection.
- Use asynchronous operations where appropriate.
- Support cancellation for operations where it is useful.
- Centralize error handling.
- Use transactions for operations requiring atomicity.
- Avoid placing complex business logic directly in controllers.
- Isolate external integrations behind clear interfaces where appropriate.
- Keep configuration separate from application code.
- Include tests for important business rules and endpoints.

### Database operations

The implementation should ensure that:

- Entity Framework Core mappings are consistent.
- Database changes are managed through migrations when applicable.
- Transactions protect operations that must succeed or fail as a whole.
- Concurrent requests do not violate business rules.
- Database errors are handled without exposing internal details.

### Acceptance criteria

- The backend builds successfully.
- Important business rules are covered by automated tests.
- API responses follow documented contracts.
- Database consistency is protected for critical operations.
- The implementation follows the agreed architecture.
- No feature is described as complete without supporting implementation and test evidence.

## OPS-04 — Security and Performance Verification

### Objective

Verify that the application satisfies the security and performance requirements agreed upon for the project.

### Security verification

The verification should cover:

- Authentication.
- Authorization and role restrictions.
- Access to resources belonging to other users.
- Input validation.
- Protection against unauthorized data modification.
- Safe error responses.
- Secret and configuration management.
- Dependency vulnerabilities.
- Sensitive information in logs.
- Protection of API endpoints.
- Data protection for exports and external integrations.

Where applicable, the team should review risks related to common web-application vulnerabilities.

### Performance verification

The verification should cover important operations, including:

- Retrieving student and group data.
- Searching for potential exchange candidates.
- Importing CSV files.
- Processing group-switch requests.
- Executing multi-student exchanges.
- Exporting CSV or PDF files.
- Loading timetable information.

The team should define measurable performance targets before claiming that the application meets them.

Possible measurements include:

- Response time.
- Throughput.
- Error rate.
- Database query duration.
- Memory consumption.
- Behavior under representative concurrent load.

### Acceptance criteria

- Security checks are documented.
- Important authorization rules are tested.
- Representative performance measurements are recorded.
- Identified problems are prioritized.
- Results and limitations are documented.

**Compliance must not be claimed until the relevant checks have been performed and their results recorded.**

## OPS-05 — Production Environment Compatibility

### Objective

Ensure that the application can be configured, deployed, and operated in the intended production environment.

### Requirements

The project should document and verify:

- Supported .NET and Node.js versions, as applicable.
- Database engine and version.
- Required environment variables.
- Secret management.
- Database connection configuration.
- Database migrations.
- HTTPS configuration.
- CORS configuration.
- Frontend and backend deployment settings.
- External service dependencies.
- Required network access.
- Logging configuration.
- Health checks.
- Error behavior when external services are unavailable.

The exact deployment environment must be confirmed with the team.

### Deployment and recovery

Where applicable, deployment documentation should include:

- Build and deployment steps.
- Environment-specific configuration.
- Database migration procedure.
- Pre-deployment checks.
- Rollback procedure.
- Backup and recovery procedure.
- Post-deployment verification.

### Acceptance criteria

- Installation and configuration steps are reproducible.
- The application starts in the target environment.
- Required dependencies are available and correctly configured.
- Critical application functions are verified after deployment.
- Known environment limitations are documented.

Running the application successfully on a developer's computer is not sufficient evidence of production compatibility.

---

# 5. Documentation and Communication Requirements

## DOC-01 — Technical Documentation

### Objective

Produce clear and complete technical documentation describing the application's operation and maintenance.

### Required documentation

The documentation should include:

1. Project overview and objectives.
2. System architecture.
3. Component responsibilities.
4. Database model and relationships.
5. API documentation.
6. Authentication and authorization.
7. Installation and configuration.
8. Database migrations.
9. Logging and error handling.
10. Maintenance procedures.
11. Testing procedures.
12. Deployment instructions.
13. Troubleshooting guidance.
14. External integration requirements and limitations.
15. Known issues and future improvements.

### API documentation

The API documentation should describe:

- Endpoint path and HTTP method.
- Purpose.
- Required permissions.
- Request parameters and body.
- Response format.
- Possible HTTP status codes.
- Validation errors.
- Authentication requirements.
- Relevant examples.

OpenAPI/Swagger may be used if compatible with the project's implementation.

### Installation documentation

The installation guide should explain:

- Prerequisites.
- Required software.
- Repository setup.
- Configuration variables.
- Database setup.
- Migration execution.
- Backend startup.
- Frontend startup.
- Test execution.
- Common installation errors.

### Maintenance documentation

The maintenance guide should explain:

- How to locate and interpret logs.
- How to investigate common failures.
- How to identify the affected component.
- How to prioritize issues.
- How to validate a correction.
- How to perform relevant maintenance tasks.
- How to deploy a fix.
- How to restore service when a documented recovery procedure exists.

## DOC-02 — Technical Communication

The team shall prepare a technical presentation for the relevant technical stakeholders.

The presentation should include:

- Project objectives.
- Architecture and technology choices.
- Main implemented features.
- API and database organization.
- Logging and error-handling strategy.
- Testing approach.
- Security and performance verification.
- Production-readiness considerations.
- Known limitations.
- Outstanding technical risks.
- Proposed next steps.

The presentation must distinguish implemented functionality from planned or unverified functionality.

## DOC-03 — Management Presentation

The team shall prepare a concise presentation suitable for a non-technical management audience.

It should focus on:

- The problem addressed by GroupSwitch.
- Project objectives.
- Main features.
- Expected operational benefits.
- Demonstrated results.
- Project status.
- Major risks and limitations.
- Remaining work.
- Proposed next steps.

Technical details should be explained in accessible language and included only when necessary for decision-making.

---

# 6. Security Requirements

The following principles should guide implementation:

- Apply authentication to protected endpoints.
- Enforce authorization on the backend.
- Validate user input.
- Protect sensitive configuration values.
- Avoid exposing internal exception details.
- Avoid logging secrets and unnecessary personal data.
- Apply appropriate access controls to student information and exports.
- Review external dependencies.
- Use HTTPS in production.
- Define suitable policies for personal-data retention and access.

The final security requirements must be reviewed against the deployment environment and applicable organizational policies.

---

# 7. Business Rules and Data Integrity

The system must preserve the integrity of student assignments and group capacities.

The following rules require explicit validation:

1. A student cannot be assigned to an invalid group.
2. Group capacities must not be exceeded.
3. Unauthorized users cannot modify protected assignments.
4. A rejected or cancelled request must not be applied as an approved change.
5. Multi-student exchanges must be atomic.
6. Concurrent requests must not create invalid assignments.
7. Timetable conflicts must be evaluated according to the agreed rules.
8. Failed operations must not leave partial changes.
9. Administrative decisions should be traceable where required.
10. Imported data must be validated before it is used.

Additional business rules may be added after review of the original client brief.

---

# 8. Proposed API Organization

The following endpoint structure is a proposal, not a statement that these endpoints already exist.

| Area           | Example endpoint                         | Purpose                             |
| -------------- | ---------------------------------------- | ----------------------------------- |
| Authentication | `POST /api/auth/login`                   | Authenticate a user                 |
| Students       | `GET /api/students`                      | Retrieve authorized student records |
| Students       | `GET /api/students/{id}`                 | Retrieve a student                  |
| Groups         | `GET /api/groups`                        | Retrieve groups                     |
| Groups         | `GET /api/groups/{id}`                   | Retrieve group details              |
| Assignments    | `GET /api/assignments`                   | Retrieve authorized assignments     |
| Requests       | `POST /api/switch-requests`              | Submit a switch request             |
| Requests       | `GET /api/switch-requests`               | Retrieve permitted requests         |
| Requests       | `PATCH /api/switch-requests/{id}/status` | Process a request                   |
| Imports        | `POST /api/imports/students`             | Import student data                 |
| Exports        | `GET /api/exports/students.csv`          | Export permitted student data       |
| Health         | `GET /health`                            | Check application health            |

The final endpoint paths, DTOs, permissions, and status codes must be aligned with the implemented application.

---

# 9. Testing Strategy

## 9.1 Unit Tests

Unit tests should cover:

- Business rules.
- Group-capacity validation.
- Request status transitions.
- Timetable-conflict detection.
- Candidate-selection logic.
- Input validation.
- Error-mapping logic where appropriate.

## 9.2 Integration Tests

Integration tests should cover:

- API endpoints.
- Database persistence.
- Entity Framework Core mappings.
- Authentication and authorization.
- Database transactions.
- Import and export operations.
- Error-handling middleware.
- External-service failure scenarios.

## 9.3 Security Tests

Security tests should verify:

- Unauthorized requests are rejected.
- Users cannot access prohibited data.
- Students cannot perform administrative operations.
- Sensitive information is not exposed in error responses.
- Invalid input is handled safely.

## 9.4 Performance Tests

Performance testing should focus on representative scenarios and agreed targets.

Results should record the test conditions, dataset size, observed response times, errors, and limitations.

## 9.5 Acceptance Evidence

Each completed requirement should be supported by appropriate evidence, such as:

- Automated test results.
- Manual test records.
- API responses.
- Build results.
- Deployment checks.
- Performance measurements.
- Security review findings.
- Documentation review.

---

# 10. Maintenance Priority Matrix

| Priority      | Description                                                     | Response approach                      |
| ------------- | --------------------------------------------------------------- | -------------------------------------- |
| P1 — Critical | Data corruption, major outage, or severe security incident      | Immediate escalation and investigation |
| P2 — High     | Major feature unavailable or substantial operational disruption | Prioritized corrective action          |
| P3 — Normal   | Limited defect with an acceptable workaround                    | Planned correction                     |
| P4 — Low      | Minor improvement or non-urgent technical debt                  | Schedule when resources permit         |

The team must agree on response targets and escalation procedures. No specific resolution time is assumed in this document.

## Maintenance Register Template

| Field        | Description                                     |
| ------------ | ----------------------------------------------- |
| ID           | Unique maintenance reference                    |
| Title        | Short description                               |
| Component    | Affected application component                  |
| Impact       | User or business impact                         |
| Priority     | P1, P2, P3, or P4                               |
| Owner        | Responsible person or team                      |
| Action       | Proposed corrective action                      |
| Status       | Open, In Progress, Blocked, Resolved, or Closed |
| Verification | Test or evidence confirming the result          |
| Dates        | Reported, planned, and completed dates          |

---

# 11. Definition of Done

A requirement should be considered complete only when the applicable conditions are satisfied:

- The implementation is available in the repository.
- The code builds successfully.
- Relevant automated tests pass.
- Important errors are handled.
- Authorization and data-integrity rules are respected.
- The implementation has been reviewed where required.
- Relevant documentation is updated.
- Known limitations are recorded.
- Verification evidence is available.

For security, performance, and production compatibility, completion requires specific verification evidence rather than an assumption based on successful compilation.

---

# 12. Requirements Status and Traceability

Each requirement should have a status.

| Status      | Meaning                                                       |
| ----------- | ------------------------------------------------------------- |
| Proposed    | Suggested but not yet approved                                |
| Confirmed   | Accepted by the relevant stakeholders                         |
| In Progress | Implementation or verification has started                    |
| Implemented | Code or documentation has been produced                       |
| Verified    | Evidence demonstrates that the requirement has been satisfied |
| Deferred    | Intentionally postponed with a recorded reason                |

A requirement must not be marked **Verified** solely because its code has been written.

The team should maintain traceability between:

- Requirement ID.
- Implementation component.
- Related API endpoint.
- Relevant tests.
- Documentation section.
- Verification evidence.
- Current status.

---

# 13. Open Questions

The following questions should be resolved before the requirements are finalized:

1. Which .NET version is required?
2. Which database engine and version will be used?
3. What are the exact student and group data fields?
4. What are the official rules for initial student allocation?
5. What are the exact rules for multi-student exchanges?
6. Which timetable integration methods are authorized and available?
7. Which AI provider or local model is required?
8. Which user roles and permissions are mandatory?
9. Is SignalR required for the first version?
10. Which CSV and PDF reports are required?
11. Which production environment will host the application?
12. Which security standards or organizational policies must be followed?
13. What response-time targets and test loads should be used?
14. What are the logging retention and access requirements?
15. Who approves maintenance priorities?
16. What are the expected deployment, rollback, backup, and recovery procedures?
17. Which requirements are mandatory for the first release?
18. What evidence is required for final acceptance?

---

# 14. Final Deliverables

The expected project deliverables should include, subject to stakeholder approval:

- Angular frontend source code.
- ASP.NET Core backend source code.
- Database model and migrations.
- API documentation.
- Technical architecture documentation.
- Installation and configuration guide.
- Logging and error-handling implementation.
- Maintenance register and prioritization procedure.
- Automated tests and test results.
- Security and performance verification report.
- Production compatibility and deployment guide.
- Troubleshooting and maintenance guide.
- Technical presentation.
- Management presentation.
- List of known limitations and future improvements.

---

# 15. Approval

This document is a draft for review. Functional requirements, additional operational requirements, technical choices, acceptance criteria, and deployment expectations must be validated by the relevant stakeholders before being treated as binding project requirements.

| Role                   | Name            | Approval date   |
| ---------------------- | --------------- | --------------- |
| Client / Project Owner | To be confirmed | To be confirmed |
| Technical Lead         | To be confirmed | To be confirmed |
| Development Team       | To be confirmed | To be confirmed |
