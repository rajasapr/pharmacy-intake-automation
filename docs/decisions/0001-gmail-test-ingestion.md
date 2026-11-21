# Decision 0001: Gmail test ingestion approach

**Status:** Accepted for prototype; revisit before production use  
**Date:** 2026-11-21

## Context

The workflow begins with pharmacy-related emails and attachments. We need a low-cost way to exercise the intake boundary before adding document extraction, the approval dashboard, and desktop UI automation.

## Decision

- Start with the Gmail API against a dedicated test mailbox containing synthetic data.
- Use a configured Gmail search query scoped to a test label and poll on demand during development.
- Request read-only Gmail access. Do not change email state.
- Keep intake deterministic: the connector identifies matching messages, retrieves metadata and attachment bytes, and assigns stable IDs. It does not use an LLM to decide whether it may access or modify mailbox data.
- Separate the mailbox connector from storage and later processing so the same downstream workflow can eventually accept another mail provider.
- Defer Gmail push notifications and Pub/Sub until the basic flow is proven and there is a real need for near-real-time intake.

## Consequences

- The first iteration is straightforward to run and debug, but it is not an always-on production service.
- Polling may see the same message more than once, so the intake store must be idempotent.
- Production use with patient information requires a separately reviewed organizational, security, privacy, and compliance design. The prototype uses synthetic data only.
