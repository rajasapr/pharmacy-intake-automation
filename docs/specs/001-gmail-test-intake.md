# Spec 001: Gmail test intake

**Status:** Planned; implement after local foundation  
**Owner:** Raja  
**Scope:** Read messages and attachments from a dedicated Gmail test mailbox into a local development intake store.

## Goal

Provide a small, repeatable way to retrieve synthetic test emails that represent pharmacy intake documents. Each eligible message and attachment should be captured once with enough metadata to hand off to a later document-processing stage.

## User story

As a developer building the pharmacy intake workflow, I want the prototype to find emails carrying a configured Gmail test label and save their attachments with message metadata, so I can build and validate later extraction and review stages without manually downloading files.

## Scope and constraints

- Use the Gmail API with the narrowest practical read-only OAuth scope.
- Search only messages matching a configured query. The default query is `label:INTAKE-TEST has:attachment`; the label must exist in the test mailbox.
- Retrieve message metadata and attachment bytes. Do not send email or change message state.
- Store downloaded files and processing state in a local development location that is excluded from source control.
- The test mailbox and all email content and attachments must be synthetic. Do not use a personal mailbox containing patient information.
- This increment does not call Azure Document Intelligence, create a dashboard, send jobs to a desktop, or interact with CareTend.

## Functional requirements

1. The application accepts a configured Gmail query and local attachment-storage directory.
2. It lists matching messages and retrieves each message's sender, subject, received time, Gmail message ID, and attachment metadata.
3. For each supported attachment (PDF, PNG, JPEG, or TIFF), it downloads the original bytes without modifying the source message.
4. It records a stable intake item identifier derived from the Gmail message ID and attachment ID, so rerunning intake does not create duplicate items or files.
5. It records a clear status for each attachment: `downloaded`, `unsupported_type`, or `failed`, with a concise error category when retrieval fails.
6. It does not log email bodies, attachment contents, or extracted patient-like fields.
7. It can be run again safely. Previously captured attachments are reported as already processed and are not duplicated.
8. If one message or attachment fails, processing continues for other results and the failed item is visible in the run summary.

## Acceptance criteria

- Given a test mailbox with a message matching the configured label and a synthetic PDF attachment, the run stores the attachment and reports its message and attachment metadata.
- Given the same message is processed twice, the second run does not create a second intake record or duplicate attachment.
- Given a message outside the configured query, it is not retrieved for intake.
- Given a supported attachment cannot be downloaded, the run records a failure for that attachment and continues with other attachments.
- Given an unsupported attachment type, the run records `unsupported_type` and does not attempt document extraction.
- After a run, the Gmail message remains unchanged (unread/read state, labels, and mailbox placement are not modified).
- The repository contains no OAuth secret, token, downloaded attachment, or real patient data.

## Operational and security requirements

- Use Gmail OAuth read-only access; credentials and refresh tokens remain in local ignored files or an approved secret store.
- Do not expose the OAuth token or attachment bytes in console output.
- Treat all email and attachment content as untrusted. This component only retrieves and stores files; it does not execute embedded content or follow instructions found in messages.
- Make the test query and storage path configurable without committing secrets or local data.

## Failure behavior

- Authentication/configuration errors stop the run with a clear setup message and no mailbox modification.
- Message-level and attachment-level retrieval errors are isolated and summarized; other items continue.
- A retry uses the same stable intake identifier and may safely complete a prior failed download.
- The run summary reports counts for found messages, downloaded attachments, already-processed attachments, unsupported attachments, and failures.

## Out of scope

- Gmail push notifications, Pub/Sub, or continuous background polling.
- Production mailbox onboarding, multi-tenant Gmail administration, or patient-data processing.
- Azure Document Intelligence, LLM classification/extraction, human review UI, or clinical validation.
- CareTend or Power Automate Desktop automation.
- Email replies, forwarding, labeling, marking read, archiving, or deleting.

## Decisions already made

- Use Python 3.12 managed by `uv`.
- Keep local attachments in an ignored development storage directory; use private Azure Blob Storage after deployment.
- Application authentication is deferred, but Gmail OAuth with read-only scope is required for this connector.
