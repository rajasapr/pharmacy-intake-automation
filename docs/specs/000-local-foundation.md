# Spec 000: Local development foundation

**Status:** Ready for Claude Code planning  
**Owner:** Raja  
**Scope:** Establish the local-only API, database-backed workflow/event model, and dashboard shell using synthetic example cases.

## Goal

Create the foundation for the workflow before connecting Gmail or Azure: a developer can start the API and dashboard locally, view synthetic intake cases, and see per-step statuses and elapsed durations.

## Stack

- Python 3.12 and `uv` for the backend.
- FastAPI and Pydantic for the API and request/response schemas.
- PostgreSQL for cases and workflow events; local development may run it through Docker Compose.
- SQLAlchemy and Alembic for database access and schema migrations.
- React + TypeScript + Vite for the dashboard.
- Local synthetic fixtures only. No Gmail, Azure Document Intelligence, Power Automate, or CareTend integration in this spec.

## Functional requirements

1. The repository has a clear backend/frontend layout and documented local startup commands.
2. The API exposes a health endpoint and read endpoints for intake cases and their ordered workflow events.
3. The database stores a case, its current status, document type, created/updated timestamps, and workflow events with step, status, start/end times, duration, and safe error category.
4. The dashboard displays synthetic cases as cards with document type, current status, received time, total elapsed time, and latest stage.
5. Selecting a case shows an ordered timeline with each stage's status and duration. Waiting time and processing time are represented separately when the fixture provides enough timestamps.
6. The UI clearly labels all seeded data as synthetic/demo data.
7. The API, dashboard, and local database port bind to localhost only. No login is included in this phase.
8. Local database credentials use development-only values, not real secrets. `.env`, local data, attachment files, and `.venv` remain excluded from source control.

## Acceptance criteria

- A developer can follow the README to start PostgreSQL, the API, and the dashboard locally.
- The API health endpoint responds successfully and the dashboard can read cases from the API.
- At least three synthetic cases demonstrate distinct statuses, including `review_pending`, `desktop_running`, and `failed`.
- A case detail view shows workflow events in chronological order and displays durations from persisted timestamps rather than hard-coded UI values.
- Restarting the API does not erase cases/events already stored in PostgreSQL.
- The frontend, API, and database port listen on localhost, and no route accepts or processes real Gmail, Azure, or CareTend data.
- There is no authentication in this phase; documentation states that network exposure and real patient data are prohibited until the auth/security phase is complete.

## Non-goals

- Gmail API setup or OAuth consent.
- Attachment ingestion or storage.
- Azure Document Intelligence or LLMs.
- Human editing/approval actions.
- Power Automate Desktop execution.
- Cloud deployment, remote access, application authentication, or production PHI handling.

## Implementation guidance

- Keep the state/event data model small but extensible. Do not create a generic workflow engine.
- Use database seed data for reproducible synthetic cases; keep it obviously fake.
- Use UTC timestamps and calculate durations from timestamps on the backend or in a shared API schema.
- Keep API response fields and database columns free of unnecessary patient-like data. Demo names must be unmistakably fictional.
- Before editing, Claude Code should propose a plan with intended files, commands, and assumptions. Implement only this spec after Raja reviews the plan.
