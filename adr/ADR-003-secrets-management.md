# ADR-003 — Secrets Management

## Status
Accepted

## Decision
Store database credentials, API keys and other secrets in a managed secrets/configuration service.

## Context
Secrets must not be committed to source control or embedded in container images.

## Rationale
Centralized secret storage supports access control, auditing and rotation.

## Consequences
- Better security posture.
- Secret rotation can be performed without changing source code.
- The application gains a dependency on the secrets service at startup/runtime.
