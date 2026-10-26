# GroupSwitch Test Plan

## 1. Purpose

This document defines the testing strategy for GroupSwitch v1.1.

The objective is to demonstrate that the application behaves correctly, protects restricted operations, preserves assignment consistency, and remains usable when external services fail.

Test status must reflect actual execution. A planned test is not a passed test.

## 2. Test Levels

### Unit Tests

Test isolated business rules without requiring a running frontend or external service.

Examples:

- Group-capacity validation.
- One active assignment per student per study block.
- Change-request validation.
- Exchange-cycle discovery.
- Timetable overlap calculation.
- AI response validation.
- CSV escaping and export-data selection.

### Integration Tests

Verify interactions between backend components and the relational database.

Examples:

- EF Core persistence and constraints.
- Transactional exchange processing.
- Request status transitions.
- Import behaviour for invalid rows.
- Export generation from recorded assignments.

### API and Authorization Tests

Verify HTTP responses, request validation, authentication, authorization, and resource ownership.

### Frontend End-to-End Tests

Where test tooling and time permit, verify essential workflows through Angular and the API.

### Manual Demonstration Tests

Use the agreed demonstration dataset to show the required workflows, including real Hyperplanning retrieval and SignalR updates.

Manual demonstration does not replace automated tests for deterministic business rules.

## 3. Test Environment and Data

Use a dedicated test database and fictitious student data.

The test environment should provide:

- The ASP.NET Core API.
- A relational database compatible with the application's configuration.
- Controlled test accounts for student and coordinator roles.
- A test Angular frontend where end-to-end tests are required.
- Mocked or stubbed AI responses for repeatable automated tests.
- Controlled timetable fixtures for conflict-detection tests.
- A separate integration setup for real Hyperplanning retrieval.

Do not use real student data in automated tests unless explicitly authorized and necessary.

## 4. Functional Test Cases

| ID    | Scenario                                                   | Expected result                                                                         |
| ----- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| FT-01 | Create and update a student                                | Valid changes are persisted                                                             |
| FT-02 | Assign one student to groups in two different study blocks | Both assignments can coexist                                                            |
| FT-03 | Assign a student to two active groups in the same block    | The second incompatible assignment is rejected                                          |
| FT-04 | Fill a group to its maximum capacity                       | Additional assignment is rejected                                                       |
| FT-05 | Change one assignment                                      | Other assignments belonging to the student remain unchanged                             |
| FT-06 | Create a change request without a reason                   | Validation rejects the request                                                          |
| FT-07 | Request a group belonging to a different study block       | The request is rejected                                                                 |
| FT-08 | Accept a valid request with available capacity             | Only the intended assignment changes                                                    |
| FT-09 | Search for a cycle involving at least three students       | A valid candidate cycle can be proposed                                                 |
| FT-10 | Force an error during exchange application                 | No partial exchange remains committed                                                   |
| FT-11 | Import a CSV containing valid and invalid rows             | Valid rows are handled according to the documented policy and invalid rows are reported |
| FT-12 | Generate an initial distribution                           | Eligibility, capacity and category-balancing rules are respected                        |
| FT-13 | Export all groups in a study block                         | Only assignments from that block are included                                           |
| FT-14 | Export an empty group                                      | The group appears with zero students                                                    |
| FT-15 | Export names with accents and compound surnames            | Text is preserved correctly                                                             |
| FT-16 | Export CSV and PDF                                         | Both files contain the expected group data                                              |
| FT-17 | Synchronize with Hyperplanning                             | Retrieved data is mapped or a clear failure is reported                                 |
| FT-18 | Detect a timetable overlap                                 | The backend identifies the overlap deterministically                                    |
| FT-19 | Receive a request update through SignalR                   | An authorized frontend updates without manual reload                                    |
| FT-20 | AI service is unavailable                                  | Ordinary request management remains available                                           |

## 5. Assignment and Exchange Invariants

Tests must verify these invariants before and after every relevant operation:

1. Every assignment references an existing student and group.
2. The assignment's group belongs to the correct study block.
3. A student has at most one active assignment in the same block.
4. Group capacity is not exceeded.
5. A change to one assignment does not change assignments in other blocks.
6. Every exchange proposal concerns a single study block.
7. No request or assignment is used more than once within the same exchange proposal.
8. An exchange either applies all intended changes or none.
9. Stale requests or assignments cannot be silently processed.
10. Candidate solutions are revalidated before acceptance.

Where possible, verify invariants against persisted database state, not only in-memory objects.

## 6. Timetable and Hyperplanning Tests

### Deterministic conflict detection

Test at least:

- Two sessions that do not overlap.
- Two sessions that overlap partially.
- Sessions with identical start and end times.
- Sessions where one ends exactly when the other begins.
- Sessions on different dates.
- An assignment involving activities from multiple study blocks.
- Missing or invalid timetable data.

The expected boundary behaviour must be documented. A common convention is to treat time intervals as half-open intervals, so a session ending at 10:00 does not overlap one starting at 10:00.

### Hyperplanning integration

Verify:

- Retrieval from the actual targeted instance.
- Parsing of the required activity, date, time, and group information.
- Mapping to the internal timetable model.
- Handling of missing or changed fields.
- Reporting of authentication, connectivity, and parsing failures.
- No false claim of successful synchronization when retrieval fails.

Automated unit tests should use saved or controlled fixtures. At least one integration or demonstration scenario must exercise the real targeted instance, as required by the client.

## 7. AI Integration Tests

AI tests must not depend on a paid or remote model for every run.

| ID    | Scenario                                                  | Expected result                                                    |
| ----- | --------------------------------------------------------- | ------------------------------------------------------------------ |
| AI-01 | Valid structured AI response                              | The backend validates and normalizes it                            |
| AI-02 | Malformed JSON or missing required fields                 | The response is rejected or handled safely                         |
| AI-03 | Unknown or unsupported criteria                           | Invalid criteria are not used blindly                              |
| AI-04 | Ambiguous free-text preference                            | The uncertainty is retained for coordinator review                 |
| AI-05 | AI service timeout or outage                              | The normal request workflow remains available                      |
| AI-06 | Valid criteria produce candidate searches                 | The backend uses the criteria to search for candidates             |
| AI-07 | Candidate conflicts with actual timetable data            | The backend flags or rejects the candidate as incompatible         |
| AI-08 | Candidate exceeds group capacity                          | The candidate is not presented as valid                            |
| AI-09 | Student has activities in multiple blocks                 | Relevant activities across those blocks are considered             |
| AI-10 | Prompt contains instructions attempting to override rules | The result cannot bypass deterministic validation or authorization |

The tests must demonstrate that the AI interprets the request but does not decide timetable conflicts, modify assignments, or make the final approval decision.

## 8. Security Tests

| ID    | Scenario                                                       | Expected result                                                          |
| ----- | -------------------------------------------------------------- | ------------------------------------------------------------------------ |
| ST-01 | No JWT on a protected endpoint                                 | 401 or the documented equivalent                                         |
| ST-02 | Invalid or expired JWT                                         | Request is rejected                                                      |
| ST-03 | Student calls a coordinator endpoint                           | Access is denied                                                         |
| ST-04 | Student requests another student's private request             | Access is denied                                                         |
| ST-05 | Client changes its submitted role or student ID                | Backend ignores untrusted identity claims and denies unauthorized access |
| ST-06 | Unauthorized user calls an export endpoint directly            | Access is denied                                                         |
| ST-07 | Unauthorized client attempts to receive private SignalR events | Event is not delivered                                                   |
| ST-08 | AI output contains invalid criteria                            | Criteria cannot bypass business validation                               |

Verify that error responses do not disclose passwords, signing keys, API keys, SQL statements, or stack traces.

## 9. CSV and PDF Export Tests

The following cases are mandatory:

1. CSV export for a group containing several students.
2. PDF export for a group containing several students.
3. Export of multiple groups from one study block.
4. Inclusion of empty groups.
5. Preservation of accents and compound names.
6. Exclusion of assignments from other study blocks.
7. Rejection of export requests from unauthorized users.

Also verify CSV headers, documented delimiter, UTF-8 encoding, quoting and escaping, spreadsheet-formula mitigation, PDF page breaks, repeated table headers, and readable long lists.

## 10. SignalR Tests

Verify that:

- A successful request creation publishes the relevant event.
- A successful acceptance or rejection publishes an update.
- A failed business operation does not publish a false success event.
- The event identifies the relevant request or activity.
- Authorized clients receive the event.
- Unauthorized clients do not receive private information.
- The frontend handles temporary disconnection and reconnection clearly.

## 11. Performance and Reliability

The client brief does not specify numeric performance targets. Establish measurable targets before claiming performance compliance.

Evaluate, as relevant:

- API response times for common list and detail operations.
- Initial distribution with the agreed test dataset.
- Exchange-cycle search with a representative number of pending requests.
- Timetable conflict checks with realistic timetable volumes.
- CSV and PDF generation for a large study block.
- Behaviour when the database, AI provider, or Hyperplanning is unavailable.

Record the dataset size, environment, test procedure, observed results, and limitations.

## 12. Defect Severity

Use the following proposed classification:

- **P1 — Critical:** data corruption, security bypass, or a core operation that makes the application unusable.
- **P2 — High:** major required workflow fails with no acceptable workaround.
- **P3 — Medium:** a non-critical feature fails or a workaround exists.
- **P4 — Low:** minor visual, wording, or documentation issue.

This is a project tracking convention, not a client-specified grading scale.

## 13. Entry and Exit Criteria

### Entry Criteria

- The relevant code builds.
- Test configuration is available.
- Required test data is prepared.
- Dependencies and external-service assumptions are documented.

### Exit Criteria

- Mandatory tests have been executed.
- Critical and high-severity failures are resolved or explicitly reported.
- Security and business-rule failures are not hidden.
- Test results and unresolved limitations are documented.
- The demonstration can reproduce the essential workflows.

## 14. Test Evidence

For each test, record:

- Test ID.
- Preconditions and test data.
- Execution steps.
- Expected result.
- Actual result.
- Pass, Fail, Blocked, or Not Run.
- Date and evidence reference.

Do not mark a test as Passed based only on code review or a successful build.
