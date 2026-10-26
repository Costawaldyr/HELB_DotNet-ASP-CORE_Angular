# GroupSwitch Maintenance Guide

## 1. Purpose

This document defines how to diagnose problems, record defects, prioritize fixes, and maintain GroupSwitch v1.1.

It covers the Angular frontend, ASP.NET Core API, business layer, EF Core persistence, authentication, SignalR, Hyperplanning retrieval, AI integration, and CSV/PDF exports.

## 2. Maintenance Principles

- Reproduce a problem before changing code.
- Identify whether the cause is in the frontend, API, business rules, database, or an external integration.
- Preserve assignment and transaction invariants.
- Add or update automated tests when fixing a defect.
- Avoid committing secrets, real student data, or sensitive logs.
- Update documentation when behaviour or configuration changes.
- Do not claim a fix is complete until the relevant checks have passed.

## 3. Issue Priority

Use the following project convention.

| Priority      | Definition                                                                         | Example                                          |
| ------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------ |
| P1 — Critical | Security breach, corrupted assignments, or a critical workflow that cannot operate | An exchange leaves students partially reassigned |
| P2 — High     | A major required feature fails without an acceptable workaround                    | Coordinator cannot process change requests       |
| P3 — Medium   | A limited feature fails or a workaround is available                               | PDF export fails for a particular long list      |
| P4 — Low      | Minor presentation, wording, or non-critical documentation issue                   | A label is unclear                               |

Priorities describe the proposed maintenance process and are not official client grading categories.

## 4. Maintenance Register

Maintain an issue register in the repository's chosen issue tracker or another agreed location.

Each issue should contain:

- Issue identifier.
- Date reported.
- Short description.
- Affected component.
- Reproduction steps.
- Expected behaviour.
- Actual behaviour.
- Priority and status.
- Assigned owner, if applicable.
- Related tests.
- Fix or workaround.
- Verification evidence.
- Closure date.

Recommended statuses are `Open`, `Investigating`, `In Progress`, `Blocked`, `Ready for Verification`, and `Closed`.

## 5. Incident Handling

1. Record the observed problem and its impact.
2. Check whether the issue affects security, data integrity, or assignment consistency.
3. Reproduce the problem using test data where possible.
4. Inspect the API response, server logs, and relevant database state.
5. Identify the root cause.
6. Implement the smallest safe correction.
7. Add a regression test.
8. Run relevant automated tests.
9. Verify the result in the affected workflow.
10. Update the issue and documentation.

If there is a risk of unauthorized access or corrupted assignments, stop the affected operation until the risk is understood.

## 6. Troubleshooting

### 6.1. API returns HTTP 500

Check:

- Server logs and the correlation identifier, if available.
- The full server-side exception.
- Database connectivity and migration state.
- Missing configuration values.
- External-service failures.
- Whether a recent change introduced the error.

Do not expose stack traces or internal exceptions to the frontend in deployed environments.

### 6.2. API returns HTTP 401

Check:

- Whether the user has authenticated.
- Whether the token is present and correctly transmitted.
- Whether the token has expired.
- Whether issuer, audience, signature, and signing-key configuration match.
- Whether the client is using the correct API environment.

Do not solve a 401 error by disabling authentication.

### 6.3. API returns HTTP 403

Check:

- The user's authenticated role and claims.
- The endpoint's authorization policy.
- Resource ownership checks.
- Whether the operation is restricted to the coordinator.

Do not grant broader permissions simply to make a request succeed.

### 6.4. Assignment or capacity conflict

Check:

- The student's current assignments.
- The study block associated with the current and requested groups.
- The group's current effective occupancy and capacity.
- Whether another operation changed the data concurrently.
- Whether the request has already been processed.

Revalidate the operation against persisted state. Never bypass the capacity or one-assignment-per-block rules to resolve the error.

### 6.5. Exchange leaves unexpected data

Treat this as a high-priority data-integrity issue.

Check:

- The transaction boundary.
- The order in which assignments are validated and updated.
- Database concurrency behaviour.
- Whether all exchange participants belong to the same block.
- Whether an assignment or request is duplicated in the proposal.
- Whether an exception was caught without rolling back the operation.

Add a regression test that verifies the complete persisted state after a forced failure.

### 6.6. CSV import reports invalid rows

Check:

- The documented delimiter and encoding.
- Required column names.
- Duplicate identifiers.
- Unknown student, group, or study-block references.
- Incorrectly escaped separators or quotation marks.
- Whether the import policy allows valid rows to be processed alongside invalid rows.

The import result should identify the affected row and a useful error code or message.

### 6.7. CSV export opens incorrectly in a spreadsheet

Check:

- UTF-8 encoding and the documented BOM choice.
- The selected delimiter.
- Column headers.
- Escaping of separators, quotes, and line breaks.
- Spreadsheet formula-injection mitigation.
- The content type and downloaded filename.

Use test values containing accents, compound names, commas, quotes, and values beginning with spreadsheet formula characters.

### 6.8. PDF export has missing or clipped information

Check:

- Font support for accented characters.
- Page-break behaviour.
- Repeated table headers.
- Long names and long group lists.
- Whether group headings remain associated with the correct students.
- Whether empty groups are included.

### 6.9. Hyperplanning synchronization fails

Check:

- Reachability of the targeted instance.
- Authentication and session requirements.
- Whether the source page or data structure has changed.
- Parsing errors and missing fields.
- Date/time interpretation and time-zone assumptions.
- Whether the application is displaying stale data as if it were current.

Report retrieval failures clearly. Do not fabricate timetable records to make a synchronization appear successful.

### 6.10. Timetable conflicts appear incorrect

Check:

- The source timetable records.
- Session dates and start/end times.
- Time-zone handling.
- Interval-boundary rules.
- Whether activities from multiple study blocks are included.
- Whether the conflict result was computed from real mapped data.

The conflict calculation must remain deterministic and implemented in C#. Do not use an AI response as proof that a timetable is compatible.

### 6.11. AI analysis is unavailable or invalid

Check:

- Backend configuration and secret availability.
- Provider or local-model availability.
- Timeout and error handling.
- JSON structure and required fields.
- Validation of criteria before candidate search.
- Whether the normal request workflow remains usable.

AI failure must not disable ordinary request consultation or processing.

### 6.12. SignalR updates do not arrive

Check:

- Hub URL and frontend configuration.
- Authentication on the Hub connection.
- CORS and network configuration where applicable.
- Client connection state and reconnection behaviour.
- Event names and payload structure.
- Whether the backend published the event after the database transaction succeeded.
- Whether the recipient is authorized for that event.

Do not publish a success event before the underlying operation commits.

## 7. Routine Maintenance

### Before a significant release

- Build the backend and frontend.
- Run the automated test suite.
- Review authentication and authorization changes.
- Check database migrations.
- Verify assignment and exchange invariants.
- Test exports and their authorization.
- Check AI and Hyperplanning failure handling.
- Confirm that no secrets are tracked by Git.
- Update the README and affected technical documents.

### During development

- Keep dependencies maintained.
- Review dependency updates for compatibility.
- Use database migrations for schema changes.
- Keep test fixtures reproducible.
- Remove obsolete code and unused configuration.
- Review logs for accidental exposure of personal data.

## 8. Database and Recovery

Database backup and recovery procedures depend on the database selected and the deployment environment.

Document:

- Database engine and version.
- Database file or connection configuration.
- Backup frequency and location.
- Retention period.
- Restore procedure.
- Migration procedure.
- Validation checks after restoration.

For SQLite, a backup procedure must account for database consistency and any active journal or write-ahead-log configuration.

A backup must not be considered reliable until a restore has been tested.

## 9. Change Management

For a significant change:

1. Describe the requirement or defect.
2. Identify affected components and business invariants.
3. Record important architecture decisions.
4. Implement the change in a separate branch where practical.
5. Add tests.
6. Review the diff for security and data-integrity risks.
7. Run the relevant checks.
8. Update documentation.
9. Merge only after the agreed review and validation process.

## 10. Closing an Issue

Close an issue only when:

- The cause is understood or the remaining uncertainty is documented.
- The correction or accepted workaround is recorded.
- Relevant tests pass.
- No known regression violates a mandatory business rule.
- Required documentation is updated.
- Verification evidence is available.
