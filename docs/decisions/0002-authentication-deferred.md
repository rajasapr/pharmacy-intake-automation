# Decision 0002: Defer application authentication until a later phase

**Status:** Accepted for local synthetic prototype  
**Date:** 2026-11-21

## Decision

Application login and role-based authorization are deferred until after the local foundation. The initial dashboard/API are for one developer on localhost and use synthetic data only. The app must not be bound to a LAN interface, exposed through a tunnel, or deployed publicly before authentication and security controls are implemented.

Gmail OAuth is a separate integration requirement: when Gmail intake is added, it must use OAuth with read-only Gmail access even though the application itself does not yet have user login.

## Rationale

This keeps the first build focused on workflow state, timing, and data boundaries while making the limits of an unauthenticated prototype explicit.

## Revisit when

- A second user needs access.
- The application is exposed beyond localhost.
- Any real patient data is proposed.
- A cloud deployment is planned.

Before any of these conditions, define identity provider, session handling, roles, audit access, secrets, and privacy/security controls in a new spec. The prototype is not approved for PHI merely because authentication is later added.
