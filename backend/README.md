# Backend

This folder hosts the backend code for CapeHike Secure. It is organized to match the architecture and responsibilities described in the root README (two Spring Boot deployables: API and Worker, plus a shared module and infra assets).

Current status: this directory is intentionally empty. The structure below prescribes what will be created as the implementation proceeds.

## Prescribed structure

backend/
- capehike-api/
  - Purpose: Public REST API (Spring Boot). Implements authentication/authorization, request validation, domain services, persistence, and acts as a Temporal client to start/signal workflows.
  - Typical layout:
    - src/main/java/... — controllers, services, repositories
    - src/main/resources/application.yml — config per profile (local, dev, prod)
    - src/test/java/... — tests
    - build.gradle or pom.xml — build configuration
- capehike-worker/
  - Purpose: Background worker (Spring Boot + Temporal Worker). Runs workflows and activities for scheduled check-ins, SOS flows, and notification retries. Internal service (no public ingress).
  - Typical layout:
    - src/main/java/... — workflows, activities, worker bootstrap
    - src/main/resources/application.yml — Temporal + DB settings
    - src/test/java/... — workflow/activity tests
    - build.gradle or pom.xml — build configuration
- capehike-common/
  - Purpose: Shared library for DTOs, domain models, mappers, validation, and optionally Temporal workflow/activity interfaces used by both API and Worker.
  - Typical layout:
    - src/main/java/... — shared types and helpers
    - build.gradle or pom.xml — build configuration (Java library)
- infra/
  - Purpose: Local and deployment assets.
  - Contents (to be added as needed):
    - docker-compose.yml — local dev stack (PostgreSQL, Redis, Temporal, API, Worker)
    - k8s/ — manifests or Helm charts for API, Worker, PostgreSQL, Redis, Temporal
    - migration/ — database migration files (Flyway/Liquibase), if not kept inside API/Worker modules
    - ops/ — runbooks, dashboards, alerts (optional)

Notes:
- Database: PostgreSQL is the primary store. Redis is optional (rate limiting, short-lived cache).
- Temporal: The Worker polls named task queues (for example: `checkin-queue`, `sos-queue`). The API uses the Temporal SDK as a client to start or signal workflows.
- Secrets and configuration: Use environment variables or a secrets manager. Do not commit secrets. Prefer profile-specific configuration files.

## Responsibilities
- capehike-api (public):
  - Exposes REST endpoints for mobile/web clients.
  - Performs authentication (JWT), authorization, validation.
  - Reads/writes domain state in PostgreSQL via Spring Data.
  - Interacts with Temporal via the SDK to start/signals workflows.
  - Optional: uses Redis for rate limiting/caching.
- capehike-worker (internal):
  - Hosts Temporal workers that execute workflows/activities.
  - Polls Temporal task queues and coordinates long-running, retryable processes (scheduled check-ins, SOS escalation, notifications).
  - Reads/writes audit and supplementary state in PostgreSQL.
  - Calls external providers (Twilio/FCM/Email) for notifications.
- capehike-common (shared):
  - Common DTOs, domain models, exceptions, constants, and interfaces.
  - Encourages type safety and eliminates duplication across API/Worker.
- infra:
  - Container orchestration and local dev setup.
  - Observability wiring (metrics, tracing, logs) and deployment configs.

## Alignment with architecture (root README)
- C4 L2: Two Spring Boot deployables — API (public ingress) and Worker (internal only).
- Both API and Worker connect to PostgreSQL (JDBC) and Temporal (SDK client vs polling task queues).
- Worker is the only service with outbound egress to notification providers.
- Shared types live in a common module to keep API and Worker consistent.

## Naming and packages
- Group ID / base package suggestion: `io.capehike` (or your organization domain).
- Module names reflect responsibilities: `capehike-api`, `capehike-worker`, `capehike-common`.

## Next steps
1) Create the three modules (`capehike-api`, `capehike-worker`, `capehike-common`) using your chosen build tool (Gradle or Maven) with standard Java 17+ layout.
2) Add `infra/` with initial local runtime artifacts (e.g., docker-compose for PostgreSQL, Temporal, Redis; API and Worker images).
3) In `capehike-common`, define shared DTOs/domain models and, if desired, Temporal interfaces.
4) In `capehike-api`, scaffold minimal endpoints for hikes and check-ins; wire JWT auth and database access.
5) In `capehike-worker`, scaffold the Scheduled Check-in and SOS workflows with empty activities; verify the worker connects to Temporal.
6) Add database migrations (Flyway/Liquibase) either in API, Worker, or a dedicated migration location under `infra/`.
7) Wire observability (metrics, tracing) and CI pipelines.

This document will evolve with the codebase. As modules are created, include per-module READMEs describing build/run instructions, configuration keys, and operational tips.

