# Service Blueprint

## Purpose

This blueprint defines the request, dependency, deployment, data and failure paths for the two-tier web service.

## Components

1. **Users/Clients** — Web, mobile or API clients.
2. **DNS/Internet** — Resolves the public service endpoint.
3. **Load Balancer** — Terminates/routes HTTPS traffic and removes unhealthy instances.
4. **Web/API Tier** — Stateless application layer containing the web/API service.
5. **Managed PostgreSQL** — Persistent application data.
6. **External API** — Optional third-party dependency such as payments, email or maps.
7. **Backup Storage** — Automated database backups/snapshots.
8. **Monitoring/Logging** — Metrics, logs, health checks and alerts.
9. **CI/CD + Registry** — Builds, tests and publishes application images before deployment.

## Flow Types

- **Request flow:** User → DNS → Load Balancer → Web/API Tier → response.
- **Data flow:** Web/API Tier ↔ PostgreSQL; Web/API Tier ↔ External API.
- **Backup flow:** PostgreSQL → Backup Storage.
- **Observability flow:** Application/infrastructure → Monitoring and Logging.
- **Deployment flow:** Source Repository → CI/CD → Container Registry → Orchestration → Application Tier.
- **Failure flow:** Detection → retry/fallback/replace/failover according to the failure type.

## Design Principle

The application tier is intended to remain as stateless as practical so instances can be replaced or scaled without losing persistent application data.
