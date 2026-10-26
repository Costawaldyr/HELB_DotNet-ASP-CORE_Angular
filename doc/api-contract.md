# GroupSwitch API Contract

## 1. Purpose

This document defines the intended HTTP contract between the Angular frontend and the ASP.NET Core backend.

The routes and DTOs below are a proposed contract derived from the GroupSwitch v1.1 requirements. They must be reconciled with the actual controllers or endpoint definitions before being treated as the final implemented API.

The API uses HTTP and JSON for ordinary operations. CSV and PDF exports return downloadable files. SignalR is used for real-time notifications, not as a replacement for REST operations.

## 2. General Conventions

- Use plural resource names for collections.
- Use stable identifiers for students, groups, assignments, requests, and study blocks.
- Use ISO 8601 date/time representations.
- Validate request bodies on the backend.
- Apply authentication and authorization on the server.
- Return consistent error responses.
- Do not expose internal database entities directly as API contracts.
- Do not trust user IDs or role values supplied by the frontend.

A version prefix such as `/api/v1` may be used if adopted consistently.

## 3. Authentication

| Method | Endpoint               | Purpose                                                        | Access                             |
| ------ | ---------------------- | -------------------------------------------------------------- | ---------------------------------- |
| POST   | `/api/v1/auth/login`   | Authenticate a user and issue a JWT                            | Public                             |
| POST   | `/api/v1/auth/refresh` | Renew an access token, if refresh tokens are implemented       | Authenticated/refresh-token policy |
| POST   | `/api/v1/auth/logout`  | End the client session and revoke a refresh token if supported | Authenticated                      |

The refresh and logout routes depend on the chosen token lifecycle. They must not be exposed as implemented unless the corresponding behaviour exists.

Example login request:

```json
{
  "email": "student@example.test",
  "password": "example-password"
}
```

Example login response:

```json
{
  "accessToken": "<jwt>",
  "tokenType": "Bearer",
  "expiresAt": "2026-10-26T17:00:00Z"
}
```

Do not return password hashes or sensitive account fields.

## 4. Students

| Method | Endpoint                       | Purpose                                            | Access                          |
| ------ | ------------------------------ | -------------------------------------------------- | ------------------------------- |
| GET    | `/api/v1/students`             | List students and permitted assignment information | Coordinator                     |
| GET    | `/api/v1/students/{studentId}` | Get a student and permitted details                | Coordinator or authorized owner |
| POST   | `/api/v1/students`             | Create a student                                   | Coordinator                     |
| PUT    | `/api/v1/students/{studentId}` | Update a student                                   | Coordinator                     |
| DELETE | `/api/v1/students/{studentId}` | Delete a student where business rules allow it     | Coordinator                     |

The backend must validate references and preserve assignment integrity when updating or deleting a student.

## 5. Study Blocks, Groups and Virtual Categories

| Method | Endpoint                                          | Purpose                                 | Access                                      |
| ------ | ------------------------------------------------- | --------------------------------------- | ------------------------------------------- |
| GET    | `/api/v1/study-blocks`                            | List study blocks                       | Authenticated, subject to data-access rules |
| POST   | `/api/v1/study-blocks`                            | Create a study block                    | Coordinator                                 |
| PUT    | `/api/v1/study-blocks/{blockId}`                  | Update a study block                    | Coordinator                                 |
| GET    | `/api/v1/study-blocks/{blockId}/groups`           | List groups in a block                  | Authenticated, subject to data-access rules |
| POST   | `/api/v1/study-blocks/{blockId}/groups`           | Create a group                          | Coordinator                                 |
| PUT    | `/api/v1/groups/{groupId}`                        | Update a group                          | Coordinator                                 |
| DELETE | `/api/v1/groups/{groupId}`                        | Delete a group where allowed            | Coordinator                                 |
| GET    | `/api/v1/virtual-categories`                      | List virtual categories                 | Coordinator                                 |
| POST   | `/api/v1/virtual-categories`                      | Create a virtual category               | Coordinator                                 |
| PUT    | `/api/v1/virtual-categories/{categoryId}`         | Update a virtual category               | Coordinator                                 |
| PUT    | `/api/v1/students/{studentId}/virtual-categories` | Replace or update category associations | Coordinator                                 |

Virtual categories are administrative data and must not be exposed to students unless an explicit requirement and authorization rule allow it.

## 6. Assignments

| Method | Endpoint                                     | Purpose                            | Access                          |
| ------ | -------------------------------------------- | ---------------------------------- | ------------------------------- |
| GET    | `/api/v1/students/{studentId}/assignments`   | List assignments for a student     | Coordinator or authorized owner |
| GET    | `/api/v1/study-blocks/{blockId}/assignments` | List assignments in a block        | Coordinator                     |
| POST   | `/api/v1/assignments`                        | Create an assignment               | Coordinator                     |
| PUT    | `/api/v1/assignments/{assignmentId}`         | Change one assignment              | Coordinator                     |
| DELETE | `/api/v1/assignments/{assignmentId}`         | Remove an assignment where allowed | Coordinator                     |

An assignment update must affect only the specified assignment. The API must not interpret an assignment change as replacing every assignment belonging to the student.

The backend must enforce the one-active-assignment-per-student-per-block rule and group-capacity constraints.

## 7. CSV Import and Initial Distribution

| Method | Endpoint                      | Purpose                                                                 | Access      |
| ------ | ----------------------------- | ----------------------------------------------------------------------- | ----------- |
| POST   | `/api/v1/imports/students`    | Import students from CSV                                                | Coordinator |
| POST   | `/api/v1/imports/assignments` | Import assignments from CSV                                             | Coordinator |
| POST   | `/api/v1/imports/groups`      | Import groups or study-block data if supported by the documented format | Coordinator |
| POST   | `/api/v1/distributions`       | Generate an initial distribution                                        | Coordinator |

For file uploads, use `multipart/form-data`.

An import should report row-level errors, duplicates, and unknown references without abandoning all otherwise valid rows. The exact partial-import and transaction policy must be documented.

Example import result:

```json
{
  "totalRows": 100,
  "importedRows": 94,
  "rejectedRows": 6,
  "errors": [
    {
      "rowNumber": 12,
      "code": "UNKNOWN_GROUP",
      "message": "The referenced group does not exist."
    }
  ]
}
```

The initial distribution must respect eligibility, group capacities, virtual-category balancing, and known exemptions or completed activities.

## 8. Change Requests

| Method | Endpoint                                     | Purpose                             | Access                          |
| ------ | -------------------------------------------- | ----------------------------------- | ------------------------------- |
| GET    | `/api/v1/change-requests`                    | List requests                       | Coordinator                     |
| GET    | `/api/v1/change-requests/{requestId}`        | View a request                      | Coordinator or authorized owner |
| POST   | `/api/v1/change-requests`                    | Create a request for one assignment | Student                         |
| POST   | `/api/v1/change-requests/{requestId}/accept` | Accept a request                    | Coordinator                     |
| POST   | `/api/v1/change-requests/{requestId}/reject` | Reject a request                    | Coordinator                     |

Proposed request DTO:

```json
{
  "assignmentId": "assignment-id",
  "requestedGroupId": "group-id",
  "reason": "I need to attend another activity at the current time."
}
```

The student identity must be derived from the authenticated user, not trusted from a client-supplied `studentId`.

The backend must verify that:

- The assignment belongs to the authenticated student.
- The assignment still exists and is valid.
- The requested group belongs to the same study block.
- The reason is present and valid.
- The request is not a duplicate or an invalid transition.
- Acceptance does not violate capacity, assignment, or applicable timetable rules.

Suggested status values are `Pending`, `Accepted`, and `Rejected`. Use the actual enum or serialization values consistently across the API and frontend.

## 9. Exchange Proposals

| Method | Endpoint                                         | Purpose                                          | Access      |
| ------ | ------------------------------------------------ | ------------------------------------------------ | ----------- |
| POST   | `/api/v1/exchange-proposals/search`              | Search for candidate exchange cycles             | Coordinator |
| GET    | `/api/v1/exchange-proposals/{proposalId}`        | View a proposal and its before/after assignments | Coordinator |
| POST   | `/api/v1/exchange-proposals/{proposalId}/accept` | Validate and apply the entire exchange           | Coordinator |
| POST   | `/api/v1/exchange-proposals/{proposalId}/reject` | Reject a proposal                                | Coordinator |

A valid exchange may involve three or more students. All participating requests and groups must concern the same study block.

The backend must prevent reuse of the same request or assignment within a proposal. It must revalidate current data at acceptance time and apply the exchange transactionally.

## 10. AI Analysis and Candidate Solutions

| Method | Endpoint                                       | Purpose                                                 | Access      |
| ------ | ---------------------------------------------- | ------------------------------------------------------- | ----------- |
| GET    | `/api/v1/change-requests/{requestId}/analysis` | View the structured analysis                            | Coordinator |
| POST   | `/api/v1/change-requests/{requestId}/analysis` | Trigger or refresh analysis if supported                | Coordinator |
| POST   | `/api/v1/solutions/search`                     | Search for candidate solutions using validated criteria | Coordinator |

The exact endpoints may differ depending on the implementation. AI provider credentials must never be sent to the frontend.

The analysis contract should distinguish:

- Summary.
- Preferences.
- Explicitly declared hard constraints.
- Uncertainties.
- Relevant activities.
- Checks still required.

AI-generated criteria are untrusted input. The backend must validate them and use deterministic business logic to assess candidates. The AI must not make the final decision or directly modify assignments.

## 11. Hyperplanning and Timetables

| Method | Endpoint                                  | Purpose                                                          | Access                          |
| ------ | ----------------------------------------- | ---------------------------------------------------------------- | ------------------------------- |
| POST   | `/api/v1/timetables/synchronization`      | Trigger retrieval from the targeted Hyperplanning instance       | Coordinator                     |
| GET    | `/api/v1/timetables/students/{studentId}` | View permitted timetable information                             | Coordinator or authorized owner |
| POST   | `/api/v1/timetables/conflicts/check`      | Check conflicts for a specified set of activities or assignments | Coordinator or permitted caller |

The synchronization operation must report success or failure clearly. Timetable conflicts must be calculated by C# using the retrieved timetable data.

The API must not claim that timetable data is current when retrieval failed or when its freshness is unknown.

## 12. CSV and PDF Exports

| Method | Endpoint                                            | Purpose                             | Access      |
| ------ | --------------------------------------------------- | ----------------------------------- | ----------- |
| GET    | `/api/v1/exports/study-blocks/{blockId}/groups.csv` | Export all groups in a block as CSV | Coordinator |
| GET    | `/api/v1/exports/groups/{groupId}.csv`              | Export one group as CSV             | Coordinator |
| GET    | `/api/v1/exports/study-blocks/{blockId}/groups.pdf` | Export all groups in a block as PDF | Coordinator |
| GET    | `/api/v1/exports/groups/{groupId}.pdf`              | Export one group as PDF             | Coordinator |

The backend must verify authorization on every export request.

Exports must use the actual assignments recorded at generation time, include empty groups, and exclude assignments from other blocks.

CSV output must document its delimiter, encoding, headers, escaping rules, and spreadsheet-formula mitigation. PDF output must handle accents, long lists, repeated table headers, and page breaks.

Successful responses should use an appropriate content type and a meaningful download filename.

## 13. SignalR

SignalR uses a Hub endpoint, for example:

`/hubs/group-switch`

The actual endpoint must match the backend and Angular configuration.

Suggested event names include:

- `ChangeRequestCreated`
- `ChangeRequestUpdated`
- `ExchangeApplied`
- `TimetableSynchronizationCompleted`

These are proposed names, not a statement that the events already exist.

Events must be published only after the related business operation succeeds. Private events must be sent only to authorized recipients. Clients must handle disconnections and reconnect without treating an event as proof that an operation succeeded.

## 14. HTTP Status Codes

| Status                     | Meaning                                                                          |
| -------------------------- | -------------------------------------------------------------------------------- |
| 200 OK                     | Request succeeded and returns a representation                                   |
| 201 Created                | A resource was created                                                           |
| 202 Accepted               | An operation was accepted for asynchronous processing                            |
| 204 No Content             | Request succeeded without a response body                                        |
| 400 Bad Request            | Invalid request or validation failure                                            |
| 401 Unauthorized           | Authentication is missing or invalid                                             |
| 403 Forbidden              | Authenticated user lacks permission                                              |
| 404 Not Found              | Resource does not exist or is not accessible under the chosen disclosure policy  |
| 409 Conflict               | Current state prevents the operation, such as a capacity or concurrency conflict |
| 413 Content Too Large      | Uploaded file exceeds the permitted size                                         |
| 415 Unsupported Media Type | Unsupported request content type                                                 |
| 429 Too Many Requests      | Rate limit exceeded, if configured                                               |
| 500 Internal Server Error  | Unexpected server-side failure                                                   |
| 503 Service Unavailable    | A required service is temporarily unavailable                                    |

Use status codes consistently. Do not return `200 OK` for every error.

## 15. Error Response Format

Use a consistent error format, preferably ASP.NET Core Problem Details.

Example:

```json
{
  "type": "https://example.invalid/problems/validation-error",
  "title": "Request validation failed",
  "status": 400,
  "detail": "One or more fields are invalid.",
  "errors": {
    "reason": ["A reason is required."]
  }
}
```

The example URI is illustrative and must be replaced with a real documentation URI or an agreed stable identifier.

Error responses must not expose stack traces, SQL statements, tokens, secrets, or internal implementation details.

## 16. Contract Verification

Before finalizing this document:

1. Compare every route with the actual API.
2. Confirm DTO property names and data types.
3. Confirm enum values and date/time formats.
4. Verify the authentication requirement for each endpoint.
5. Test status codes for success, validation errors, forbidden access, missing resources, and conflicts.
6. Verify that exports use the correct content types and filenames.
7. Add integration tests for critical endpoints.
8. Update this document when the contract changes.
