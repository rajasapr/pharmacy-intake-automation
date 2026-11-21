# Pharmacy Intake Automation — Claude Code Starter

This is the learning workspace for building the pharmacy intake workflow in small, reviewable increments. The first increment is a **local project foundation**: API, persisted workflow events, and a dashboard showing synthetic cases and step timings. Gmail intake follows next.

## First development cycle

1. Extract this folder and open it in a terminal.
2. Initialize the local Python environment with `uv` as described below.
3. Start Claude Code from this folder: `claude`.
4. Ask Claude Code to read `CLAUDE.md`, `docs/architecture/overview.md`, and `docs/specs/000-local-foundation.md`, then propose a plan without editing files.
5. Review the plan, then ask Claude Code to implement only that spec.
6. Review the diff and explanation before moving on to Gmail or Azure.

Suggested first prompt:

> Read `CLAUDE.md`, `docs/architecture/overview.md`, and `docs/specs/000-local-foundation.md`. Explain the first implementation slice in your own words and propose a small plan using the selected stack. Do not edit files yet. Call out assumptions or decisions I should review.

After reviewing the plan, ask Claude Code to implement only the local foundation. Use synthetic cases only. Do not add application authentication in this phase; keep both services bound to localhost. Do not expose the app to a network or use real patient data.

## Initialize with uv

From this folder in PowerShell:

```powershell
uv init --bare
uv python install 3.12
uv python pin 3.12
uv sync
```

`uv sync` creates the project environment. Use `uv run ...` for Python commands so `uv` keeps the environment aligned with `pyproject.toml` and `uv.lock`.

## Intended system, built incrementally

1. Local foundation: API, workflow/event data model, and dashboard with synthetic cases and timings.
2. Gmail test-label intake and attachment storage.
3. Azure AI Document Intelligence classification and extraction by document type.
4. Human review dashboard with field correction and explicit approval.
5. Windows desktop runner and Power Automate Desktop workflow dispatch.
6. Application authentication, authorization, operational controls, and approved deployment before any network exposure or real patient data.

This starter contains specifications and project guidance, not the application implementation.

## Safety for development

- Use synthetic records only. Gmail integration will use a dedicated test mailbox and Gmail OAuth when that feature is implemented.
- Do not put patient data, real attachments, OAuth client secrets, or access tokens in this repository, prompts, or source control.
- Do not connect a personal Gmail mailbox containing patient information to the prototype.
- The first local phase has no application login. Bind development servers to `127.0.0.1` only; do not expose them to a LAN, tunnel, or public internet.
- The desktop automation must run only after a human has reviewed and approved the dashboard record. Prescription creation or changes must not be performed from unreviewed extraction output.
