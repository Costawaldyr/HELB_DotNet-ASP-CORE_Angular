# GroupSwitch Deployment Guide

## 1. Purpose

This document describes how to configure, build, run, and verify GroupSwitch v1.1.

GroupSwitch consists of two separate applications:

- An Angular web frontend.
- An ASP.NET Core REST API with a relational database and external integrations.

The commands below are templates. Confirm the actual .NET target framework, Node.js version, package manager, database provider, project paths, and configuration keys in the repository before using them as definitive deployment instructions.

## 2. Prerequisites

The development or deployment environment needs:

- Git.
- The .NET SDK compatible with the backend projects.
- Node.js and the package manager required by `frontend_angular`.
- Access to the configured relational database.
- The configuration needed for JWT authentication.
- The required Hyperplanning access settings.
- AI provider configuration or the required local model.
- Network access for any services that must be reached.
- A supported PDF-generation dependency, if used by the implementation.

Use the versions documented by the repository and dependency files rather than assuming the latest version is compatible.

## 3. Repository Layout

```text
GroupSwitch/
├── backend/
│   ├── GroupSwitch.Api/
│   ├── GroupSwitch.Business/
│   ├── GroupSwitch.Data/
│   └── GroupSwitch.Tests/
├── frontend_angular/
│   ├── src/
│   └── package.json
└── GroupSwitch.sln
```

The frontend and backend must be configurable and runnable independently.

## 4. Clone and Inspect the Repository

```bash
git clone <repository-url>
cd GroupSwitch
```

Replace `<repository-url>` with the actual repository URL.

Inspect the solution and frontend package files. Confirm the target framework, startup project, test project, database provider, and package manager before proceeding.

## 5. Backend Setup

From the repository root, restore and build the solution:

```bash
dotnet restore GroupSwitch.sln
dotnet build GroupSwitch.sln
```

Run the API using the actual project path:

```bash
dotnet run --project backend/GroupSwitch.Api
```

The exact startup command may need a project filename or a configuration option depending on the repository.

Confirm the actual local API address from the startup output or launch settings. Do not assume a port that has not been configured.

## 6. Database Configuration

Configure the relational database through the application's supported configuration mechanism.

SQLite may be used if selected by the project. Another relational database may be used if the team has adopted it.

Before starting the application:

1. Confirm the selected database provider.
2. Configure the connection string outside source control where appropriate.
3. Verify that the database location is writable.
4. Apply migrations using the documented procedure.
5. Confirm that required tables and constraints exist.
6. Load only controlled development or demonstration data.

If EF Core migrations are used, a typical command is:

```bash
dotnet ef database update \
  --project backend/GroupSwitch.Data \
  --startup-project backend/GroupSwitch.Api
```

This is a template. It requires the EF Core tooling, the appropriate project files, and a correctly configured design-time database context.

Use a reviewed migration procedure in production. Do not assume that applying development migrations automatically is a suitable production deployment strategy.

## 7. Authentication Configuration

Configure JWT authentication outside the repository.

The backend needs the settings required by its actual authentication implementation, which may include:

- Issuer.
- Audience.
- Signing key or signing credentials.
- Access-token lifetime.
- Identity or user-store configuration.

Do not commit real secrets to `appsettings.json`, `.env` files, source files, or test fixtures.

Create the first coordinator or administrator using a documented, controlled initialization procedure. Do not rely on a public registration endpoint that automatically grants administrative privileges.

## 8. AI Configuration

Configure the selected AI integration from the backend.

Document:

- The provider or local model.
- Required model identifier.
- Endpoint or local service address, where applicable.
- Secret configuration.
- Timeout and failure behaviour.
- Expected structured response format.

A mock or simulated response may be used for automated tests and interface development. It must be clearly identified as simulated.

The application must retain its normal request-management functionality when the AI service is unavailable.

## 9. Hyperplanning Configuration

Configure access to the targeted Hyperplanning instance according to the approved retrieval method.

Document:

- The target instance.
- Authentication requirements.
- The retrieval or scraping procedure.
- The data fields extracted.
- The internal mapping format.
- The synchronization process.
- Error and freshness reporting.

Do not store real account credentials or session tokens in Git.

A test fixture is useful for automated tests, but it does not replace the required demonstration of retrieval from the actual targeted instance.

## 10. Angular Frontend Setup

From the frontend directory:

```bash
cd frontend
npm install
```

For a reproducible installation, prefer the repository's committed lockfile and use the appropriate package-manager command, such as `npm ci`, when supported by the project.

Start the development server using the script defined in `package.json`, for example:

```bash
npm start
```

If the project uses a different script, use the actual script listed in `package.json`.

Configure the frontend API base URL and SignalR Hub URL for the selected environment. Do not hardcode production addresses into application components.

## 11. Running the Tests

From the repository root:

```bash
dotnet test GroupSwitch.sln
```

This command assumes the test projects are included in the solution and configured for `dotnet test`.

Run frontend tests using the scripts available in `frontend_angular/package.json`.

Before a release, verify the required cases for:

- Assignment constraints.
- Change requests and exchange cycles.
- Transaction atomicity.
- Timetable conflicts.
- AI response validation and service failures.
- Authentication and authorization.
- SignalR notifications.
- CSV and PDF exports.

Record test results rather than assuming that a successful build means all requirements are satisfied.

## 12. Environment Separation

Use separate configuration for local development, testing, demonstration, and production where applicable.

| Setting       | Local development                | Test                                     | Production                             |
| ------------- | -------------------------------- | ---------------------------------------- | -------------------------------------- |
| Database      | Local development database       | Isolated test database                   | Configured production database         |
| JWT secrets   | Local secret configuration       | Test-only secrets                        | Secure deployment configuration        |
| AI service    | Real or mock, clearly identified | Prefer controlled mock responses         | Configured real service or local model |
| Hyperplanning | Test fixture or approved target  | Controlled fixture and integration setup | Approved target instance               |
| Logging       | Developer-friendly diagnostics   | Test diagnostics                         | Controlled operational logging         |
| HTTPS         | Development configuration        | Test environment configuration           | Required                               |

Do not copy production credentials or real student data into local or test environments.

## 13. Production Readiness Checklist

Before deployment, confirm:

- [ ] Backend and frontend build successfully.
- [ ] Automated tests have been executed and results recorded.
- [ ] JWT validation and role-based authorization work.
- [ ] Resource-level access checks work.
- [ ] Database migrations are reviewed.
- [ ] Group capacity and assignment invariants are protected.
- [ ] Exchange operations are atomic.
- [ ] Secrets are supplied outside source control.
- [ ] HTTPS is enabled.
- [ ] CORS is restricted to the required origins.
- [ ] Error responses do not expose internal details.
- [ ] Logs do not contain passwords, tokens, or unnecessary personal data.
- [ ] Hyperplanning retrieval failures are reported clearly.
- [ ] AI failures do not block ordinary request management.
- [ ] SignalR events are authorized and published after successful operations.
- [ ] CSV and PDF exports are protected and validated.
- [ ] Backup and recovery procedures are documented and tested.
- [ ] A controlled initial administrator-creation procedure exists.

## 14. Health Checks and Verification

If health endpoints are implemented, document their exact routes and meanings.

A health check should distinguish, where appropriate, between:

- The API process being responsive.
- The database being available.
- Required dependencies being available.

Avoid exposing detailed internal infrastructure information through public health endpoints.

After deployment, verify:

1. The API starts successfully.
2. Database access works.
3. Authentication succeeds with a controlled account.
4. A protected endpoint rejects unauthenticated access.
5. The Angular frontend can call the API.
6. SignalR can establish an authorized connection.
7. A basic business operation succeeds.
8. Export endpoints return valid files to an authorized user.
9. Error handling works when a dependency is unavailable.

## 15. Rollback and Recovery

The rollback process depends on the deployment platform and database.

Before a release:

- Record the deployed application version or commit.
- Keep the previous deployable build where practical.
- Back up the database according to the documented procedure.
- Review schema changes for backward compatibility.
- Identify whether a migration can be reversed safely.

If a release causes a serious problem:

1. Stop or restrict the affected operation if data integrity is at risk.
2. Collect relevant logs and identify the release.
3. Roll back the application when safe.
4. Restore data only when necessary and according to the recovery plan.
5. Verify assignment invariants after recovery.
6. Record the incident and corrective action.

Do not automatically reverse database changes without understanding their impact on stored data.

## 16. Deployment Record

For each deployment, record:

- Version or commit.
- Deployment date.
- Environment.
- Database migration applied.
- Configuration changes, excluding secret values.
- Test results.
- Known issues.
- Recovery point or backup reference.
- Person responsible, where applicable.
