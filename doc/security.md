# GroupSwitch Security

## 1. Purpose and Scope

Security is a mandatory part of GroupSwitch v1.1.

The application manages student information, pedagogical assignments, change requests, administrative categories, timetables, and exported files. The backend must enforce access control regardless of how requests are sent by the frontend.

This document defines the required security behaviour and verification approach. It does not claim that a control has been implemented or verified unless supporting code and test results exist.

## 2. User Roles

At least two roles are required.

### Student

A student may:

- View information explicitly permitted for students.
- View their own assignments and change-request statuses.
- Create a change request for their own valid assignment.
- Access only the information needed to select a requested group.

A student must not:

- Manage other students.
- Modify their own role.
- Manage groups, assignments, or virtual categories administratively.
- Accept or reject requests as a coordinator.
- View another student's private requests.
- Access coordinator-only exports or administrative analyses.

### Coordinator / Administrator

An authorized coordinator may:

- Manage students, study blocks, groups, and assignments.
- Manage virtual categories.
- Import data and generate initial distributions.
- Review, accept, or reject change requests.
- Search, review, and validate exchange proposals.
- Consult AI analyses and candidate solutions.
- Trigger timetable retrieval and access permitted exports.

Virtual categories must remain administrative information and must not be exposed to students.

The exact distinction between the Coordinator and Administrator roles must be documented if the implementation separates them.

## 3. Authentication

The backend must provide a real authentication mechanism.

Required behaviour:

- A login interface submits credentials to the backend over HTTPS in deployed environments.
- The backend verifies credentials.
- Passwords are stored as secure password hashes, never as plaintext.
- Successful authentication issues a signed JWT.
- Protected API endpoints validate the JWT.
- Invalid, expired, or improperly signed tokens are rejected.
- Login errors do not reveal whether a particular account exists.
- The frontend provides a logout action and clears its client-side authentication state.

ASP.NET Core Identity is the recommended option for account and password management, but the implementation must document the library actually selected.

### JWT validation

The backend must validate, as applicable:

- Token signature.
- Issuer.
- Audience.
- Expiration and validity period.
- Required claims and authorization policies.

Signing keys must be sufficiently strong and supplied through secure configuration. They must not be committed to Git.

Access tokens must have a limited lifetime. Refresh tokens are recommended when longer sessions are required, but their storage, expiration, rotation, and revocation rules must be documented if implemented.

## 4. Authorization

Authorization is enforced by the backend.

The API should require authentication by default, with explicit exceptions for endpoints such as login.

Role checks alone may not be sufficient. Resource ownership must also be verified.

Examples:

- When a student creates a request, the backend derives the student's identity from the authenticated principal.
- When a student reads a request, the backend checks that the request belongs to that student.
- When a coordinator accepts a request, the backend verifies the required coordinator permission and revalidates the request state.
- When an export is requested, the backend verifies the caller's permission even if the frontend hides the export button.
- When virtual categories are accessed, the backend enforces administrative access.

The backend must never trust a role, student ID, or ownership claim merely because it appears in a request body, query string, or frontend state.

## 5. Passwords, Tokens and Secrets

The following values must never be committed to source control:

- User passwords or password hashes copied from real environments.
- JWT signing keys.
- AI provider API keys.
- Hyperplanning credentials or session tokens.
- Production database credentials.
- Other credentials or private configuration.

Use .NET User Secrets for local development or an equivalent secure mechanism. Production secrets must be provided by the deployment environment.

A configuration example may be committed if it contains placeholders only.

Example placeholder configuration:

```json
{
  "Jwt": {
    "Issuer": "GroupSwitch",
    "Audience": "GroupSwitch.Client",
    "SigningKey": "<configure-outside-source-control>"
  }
}
```

This is an illustrative example, not a recommended production secret or a complete JWT configuration.

## 6. API Input Validation

All externally supplied data must be treated as untrusted.

Validate:

- Required fields and string lengths.
- Identifiers and referenced resources.
- Study-block and group compatibility.
- File size, format, and row contents during imports.
- Change-request status transitions.
- Group capacity and assignment constraints.
- AI output structure and values.
- Hyperplanning data before it enters the internal timetable model.

Use parameterized database access through EF Core or equivalent safe mechanisms. Do not build SQL statements by concatenating user input.

Return understandable validation errors without revealing internal implementation details.

## 7. Protection of Business Operations

Authentication does not replace business validation.

Even an authorized coordinator must not be able to:

- Exceed group capacity.
- Assign a student to an incompatible group.
- Accept a stale or already processed request.
- Apply an exchange involving incompatible study blocks.
- Cause a partially committed exchange.
- Reuse the same assignment more than once in a single proposal.

Revalidate critical conditions at the time of the operation. A previous frontend validation or candidate search is not sufficient because the underlying data may have changed.

## 8. SignalR Security

SignalR connections must use the application's authentication mechanism.

The backend must ensure that:

- Unauthenticated clients cannot receive protected events.
- Students cannot receive coordinator-only notifications.
- Clients cannot freely join groups belonging to another user or role.
- Event payloads contain only the information recipients are allowed to see.
- Authentication expiration and reconnection are handled safely.

The client must not treat a SignalR event as authorization to perform an operation. Subsequent API requests must still pass normal authorization checks.

## 9. Personal Data and AI

Only the minimum information required for AI analysis should be transmitted to the selected AI service.

Where possible, send the free-text reason without the student's name, student ID, email address, or other unnecessary identifiers.

The backend must treat both the original text and the AI output as untrusted data. An AI response must never:

- Grant permissions.
- Modify assignments directly.
- Approve or reject a request.
- Override group-capacity rules.
- Determine timetable conflicts without timetable-based verification.

AI unavailability or invalid output must not prevent ordinary request management.

The project should document the chosen provider or local model, the data transmitted, the output format, configuration, and known limitations.

## 10. Logging and Error Disclosure

Logs should support troubleshooting without unnecessarily exposing personal information.

Do not log:

- Plaintext passwords.
- JWTs or refresh tokens.
- API keys or signing secrets.
- Unnecessary personal data.
- Full sensitive request content unless there is a documented and justified need.

Unexpected errors must be logged server-side with sufficient diagnostic context. Client responses must not reveal stack traces, SQL queries, local file paths, or secret configuration.

Use correlation identifiers where practical to connect a frontend report with server-side logs.

## 11. CSV and PDF Export Security

Exports must be protected by backend authorization.

The export service must:

- Include only the requested study block or group.
- Use current recorded assignments.
- Avoid unnecessary personal data.
- Escape CSV values correctly.
- Mitigate spreadsheet formula interpretation for untrusted values.
- Return clear errors when generation fails.

A user must not gain access to exports by calling the URL directly without the required role.

## 12. HTTPS, Browser and Configuration Security

Deployed environments must use HTTPS.

Configure cross-origin access for the actual frontend origin rather than allowing every origin by default.

If JWTs are stored in browser local storage, document the risk of token theft through cross-site scripting. If cookies are used, configure appropriate cookie protections and CSRF defences.

Avoid exposing development-only endpoints, detailed exception pages, or test credentials in production.

## 13. Security Test Requirements

Automated tests must verify at least:

- An unauthenticated request cannot access protected endpoints.
- A student cannot perform coordinator-only operations.
- A student cannot access another student's private requests.
- Invalid and expired JWTs are rejected.
- A client cannot elevate its role by changing request data.
- Direct API calls are subject to the same authorization rules as frontend actions.
- SignalR events are not delivered to unauthorized recipients.
- Unauthorized exports are refused.
- Invalid AI output cannot bypass business rules.
- Failed exchanges do not leave partial changes.

Tests should include both successful authorized operations and denied operations.

## 14. Security Verification Record

Record the following for each security test:

| Field           | Description                               |
| --------------- | ----------------------------------------- |
| Test ID         | Unique identifier                         |
| Scenario        | Behaviour being tested                    |
| Expected result | Required security outcome                 |
| Actual result   | Observed result                           |
| Evidence        | Test output, log reference, or screenshot |
| Status          | Not Run, Passed, Failed, or Blocked       |
| Date            | Execution date                            |

Do not mark a security requirement as Verified until the relevant test has passed and evidence is available.

## 15. Scope

GroupSwitch does not require enterprise-scale identity infrastructure or mandatory multifactor authentication under the supplied client brief.

However, real authentication, JWT validation, role management, resource-level authorization, secret protection, and automated security tests are required. A simulated login screen or frontend-only role check is not sufficient.
