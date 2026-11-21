# Pharmacy intake automation — architecture overview

**Status:** Prototype direction  
**Date:** 2026-11-21

## Goal

Capture pharmacy intake emails and attachments, classify and extract document fields, let a person correct and approve the result, then execute the matching CareTend desktop workflow and show every stage and its timing in a dashboard.

## Component boundaries

| Component | Prototype choice | Later production direction |
| --- | --- | --- |
| Email connector | Gmail API, read-only OAuth, test-label polling | Dedicated organization mailbox; event/push ingestion after polling is proven |
| API and workflow | Python 3.12, FastAPI, Pydantic | Same API, deployed behind authenticated access |
| Database | PostgreSQL | Azure Database for PostgreSQL or approved managed equivalent |
| Attachment store | Ignored local development folder | Private Azure Blob Storage container |
| Document processing | Azure Document Intelligence SDK | Classifier plus document-specific extraction models; optionally a reviewed LLM step if evidence shows it is needed |
| Dashboard | React + TypeScript + Vite | Same dashboard with application authentication and role-based authorization |
| Job queue | PostgreSQL-backed job/event records | Azure Service Bus when backend and desktop runner are on separate machines |
| Desktop execution | Windows runner on the same development machine starts PAD | Registered Power Automate machine/cloud flow or an approved outbound runner design |

## Workflow ownership

The backend owns the case state and is the source of truth. Document processing proposes a document type and fields. The server validates the type against an explicit allowlist and maps it to a PAD flow. A model must never invent a flow name or directly launch a desktop action.

Each email may have zero or more attachments. Each supported attachment becomes its own document case linked to the source email. An email without a supported attachment is recorded as `no_attachment` or `unsupported`, then shown in the dashboard's exception view rather than silently discarded.

The case state is persisted, not held only in a running process. A suggested state sequence is:

`received → attachment_saved → classified → extracted → review_pending → approved → desktop_queued → desktop_running → completed`

Other terminal or recovery states include `no_attachment`, `unsupported`, `rejected`, `failed`, and `retry_pending`. A failed desktop run must not be retried blindly if CareTend may already have accepted the entry; the case should be reconciled before retry.

## Audit events and timing

Record a workflow event whenever a stage starts, completes, fails, is retried, or is approved/rejected. Each event should include case ID, step, status, UTC timestamp, duration when complete, attempt number, correlation/run ID, and a non-sensitive error category. Avoid storing PHI in logs.

The dashboard should distinguish active processing time from waiting time. In particular, show time waiting for human review separately from time spent reviewing, and time queued for the desktop runner separately from PAD execution time. Overall elapsed time is useful but is not a substitute for per-step timings.

## Human approval and desktop boundary

Extracted fields remain a draft. The review page shows source document, document type, extracted values, confidence/source context where available, validation issues, and editable fields. Approval creates an immutable approved-data snapshot and records reviewer and timestamp. Any edits after approval require a new review/approval version.

Only the approved snapshot may be sent to the desktop runner. The runner claims a job once, passes a job identifier or protected file reference to PAD, and reports `started`, `completed`, or `failed` plus a run ID. PAD flow selection uses a static mapping such as `new_patient → RegisterPatient`, `prior_authorization → CreateAuthorization`, and `prescription_renewal → RenewPrescription`. `unknown` must stop for review.

For local development, the runner and API can run on the same Windows machine. Use PAD's documented local run URL/console mechanism only after the data contract is settled. For a remote backend, use a registered machine and supported Power Automate trigger or an outbound-authenticated runner; do not expose a Windows desktop by opening an inbound port.

## Stack and sequencing decisions

- Use `uv` to manage Python and the backend environment; keep `uv.lock` in source control and `.venv` ignored.
- Use PostgreSQL even in local development so the persisted workflow and event model match the intended architecture.
- Use React with TypeScript and Vite for the dashboard. Keep the API separate from the frontend.
- Store document bytes outside PostgreSQL. The database stores metadata, field values, workflow states, approvals, and event history.
- Start with Gmail polling, not Gmail push notifications. Add push delivery only after the end-to-end polling flow is stable.
- Use Azure Document Intelligence for classification/layout/extraction. Begin with synthetic forms and confidence-aware review. Do not assume one generic model will identify every pharmacy document type; document-specific labeled examples may be required.
- Do not add LangChain or LangGraph to the foundation. Use explicit API services and a persisted state machine first. Consider LangGraph only if a later spec needs multi-step model reasoning or durable model-driven branching.
- Defer application login/authentication to a later phase. Until then, run only on localhost with synthetic data. Gmail OAuth is still mandatory when Gmail is added.

## Security and production gate

This prototype is not approved for real patient data. Before production or any network exposure, define and implement authentication, authorization, secret management, encrypted storage and transport, retention/deletion, audit access, monitoring, backup/restore, and an organizational HIPAA/privacy review. A cloud provider's BAA or eligible service does not by itself make the application compliant.
