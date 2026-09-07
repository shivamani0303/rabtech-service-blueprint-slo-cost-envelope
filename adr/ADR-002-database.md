# ADR-002 — Managed PostgreSQL

## Status
Accepted

## Decision
Use a managed PostgreSQL-compatible database for persistent application data.

## Context
The service needs durable relational storage while minimizing operational overhead.

## Rationale
A managed database reduces manual patching, backup administration and routine maintenance.

## Consequences
- Lower operational burden.
- Easier backup and recovery integration.
- Higher direct service cost than self-hosting.
- Application must handle connection failures gracefully.
