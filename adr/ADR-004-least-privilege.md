# ADR-004 — Least-Privilege Access

## Status
Accepted

## Decision
Use separate service identities and grant only the permissions required for each component.

## Context
A compromised application should not automatically gain administrator-level access.

## Rationale
Least privilege limits blast radius and supports clearer security auditing.

## Consequences
- Safer runtime permissions.
- Requires initial IAM design.
- Permissions should be reviewed as the service evolves.
