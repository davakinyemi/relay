# Relay — Non-Functional Requirements

> **Status:** Draft v1.0
> **Last Updated:** 11.03.2026
> **Author:** Dave Akinyemi

---

## NFR-01: Reliability & Durability

| ID        | Priority | Requirement                                                                                                                                                                                |
|-----------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-01-01 | M        | No submitted task shall be permanently lost due to a worker crash, JVM restart, or application redeployment                                                                                |
| NFR-01-02 | M        | PostgreSQL shall be the single source of truth for all task and workflow state — no in-memory state shall be the sole record of a task's existence                                         |
| NFR-01-03 | M        | A task stuck in `PROCESSING` status without a heartbeat update for longer than a configurable timeout shall be automatically recovered — no manual database intervention shall be required |
| NFR-01-04 | M        | The system shall guarantee at-least-once execution of every task — handlers must therefore be designed to be idempotent                                                                    |
| NFR-01-05 | S        | The system shall be recoverable from a full application restart with no loss of `PENDING` or `PROCESSING` tasks                                                                            |

--- 

## NFR-02: Performance

| ID        | Priority | Requirement                                                                                                                                                                                    |
|-----------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-02-01 | M        | The REST API shall return a response to the caller immediately upon task acceptance — actual task execution shall occur asynchronously in the background and shall not block the HTTP response |
| NFR-02-02 | M        | The REST API shall respond to task submission requests within 200ms at the 95th percentile — the response shall confirm acceptance, not completion                                             |
| NFR-02-03 | M        | A `PENDING` task shall be picked up by a worker within 5 seconds of becoming eligible under normal load                                                                                        |
| NFR-02-04 | M        | Worker thread count shall be bounded — the system shall not create unbounded threads under high task volume                                                                                    |
| NFR-02-05 | S        | End-to-end engine overhead — the time between a task being enqueued and its handler being invoked, excluding handler execution time — shall not exceed 10 seconds under normal load            |
| NFR-02-06 | S        | Memory and CPU usage shall scale predictably with task throughput — no resource spikes disproportionate to load                                                                                |
| NFR-02-07 | S        | The system shall sustain a throughput of at least 100 task completions per minute on the target deployment hardware                                                                            |

---

## NFR-03: Scalability

| ID        | Priority | Requirement                                                                                                                                                                                                    |
|-----------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-03-01 | M        | Multiple instances of Relay shall be deployable simultaneously without risk of the same task being processed by more than one worker — this shall be enforced at the database level, not the application level |
| NFR-03-02 | S        | Worker pool size shall be configurable via environment variable, requiring no code change                                                                                                                      |
| NFR-03-03 | C        | The system architecture shall not preclude horizontal scaling — design decisions that would fundamentally block multi-instance deployment are not permitted                                                    |

---

## NFR-04: Maintainability

| ID        | Priority | Requirement                                                                                                                                                                                |
|-----------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-04-01 | M        | The codebase shall follow a layered architecture with clear separation between the API layer, domain/engine layer, and persistence layer — cross-layer dependencies shall only flow inward |
| NFR-04-02 | M        | Retry logic, heartbeat logic, and dead-letter logic shall each be independently modifiable without requiring changes to the database schema or the task execution path                     |
| NFR-04-03 | M        | All configurable values (timeouts, retry limits, backoff parameters, pool sizes) shall be externalised to `application.yml` — no magic numbers in source code                              |
| NFR-04-04 | M        | The codebase shall follow consistent naming conventions and package structure throughout                                                                                                   |
| NFR-04-05 | S        | No component shall carry TODO comments or commented-out code in the submitted version — technical debt shall be tracked in GitHub Issues, not inline                                       |

---

## NFR-05: Testability

| ID        | Priority | Requirement                                                                                                                                              |
|-----------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-05-01 | M        | Core engine logic (retry calculation, backoff, state transitions) shall be unit-testable without a running database, Kafka broker, or worker thread pool |
| NFR-05-02 | M        | Integration tests shall use an embedded or containerised database — no dependency on a live Neon connection to run the test suite                        |
| NFR-05-03 | M        | The full application shall be runnable on a developer's local machine using Docker Compose with a single command                                         |
| NFR-05-04 | S        | Test coverage for the engine core (state machine, retry logic, heartbeat recovery) shall be measurable and shall not fall below 80% line coverage        |

---

## NFR-06: Observability

| ID        | Priority | Requirement                                                                                                                                                                          |
|-----------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-06-01 | M        | Every task status transition shall emit a structured log entry in JSON format, containing the task ID, previous status, new status, and timestamp                                    |
| NFR-06-02 | M        | Every task failure shall log the full error message and stack trace against the task record — not only to stdout                                                                     |
| NFR-06-03 | M        | Log levels shall be configurable per environment via `application.yml` without code change                                                                                           |
| NFR-06-04 | M        | An operator shall be able to determine the current state of any task from the dashboard alone, without querying the database directly                                                |
| NFR-06-05 | S        | API error responses shall include a machine-readable error code, a human-readable message, and the offending field or context — a bare `500 Internal Server Error` is not acceptable |
| NFR-06-06 | S        | The application shall expose a `/actuator/health` endpoint that reports database and Kafka connectivity status                                                                       |

---

## NFR-07: Security

| ID        | Priority | Requirement                                                                                                                                                                                                                              |
|-----------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-07-01 | M        | All REST API endpoints shall require a valid API key passed as a request header — unauthenticated requests shall receive a `401 Unauthorized` response before any processing occurs                                                      |
| NFR-07-02 | M        | API keys shall be stored as environment variables — they shall never appear in source code or version control                                                                                                                            |
| NFR-07-03 | M        | Task payloads shall be validated on ingestion — malformed, unexpected, or oversized payloads shall be rejected with a `400 Bad Request` before entering the queue                                                                        |
| NFR-07-04 | M        | The engine shall not execute arbitrary code submitted in a task payload — executable content (scripts, shell commands, class references) shall not be a valid task payload type                                                          |
| NFR-07-05 | M        | All API communication shall occur over HTTPS — plaintext HTTP shall not be accepted in any deployed environment                                                                                                                          |
| NFR-07-06 | M        | Workflow execution history and task payloads shall not be accessible to unauthenticated callers — all read endpoints on task state, workflow state, and the dead-letter queue require a valid API key                                    |
| NFR-07-07 | S        | API key authentication shall be enforced before the engine acts on any downstream system (database writes, Kafka publishes, external HTTP calls) — the engine shall not be usable as an authenticated proxy by an unauthenticated caller |
| NFR-07-08 | S        | Database credentials, Kafka credentials, and API keys shall each be distinct secrets — no shared credentials across services                                                                                                             |
| NFR-07-09 | S        | Authenticated API calls shall be rate-limited per key — a configurable request-per-minute ceiling shall be enforced to prevent runaway workflow submission                                                                               |

---

## NFR-08: Developer Experience

| ID        | Priority | Requirement                                                                                                                                                                |
|-----------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NFR-08-01 | M        | The public API shall not expose internal implementation details — task status values, error codes, and field names shall be stable, documented terms                       |
| NFR-08-02 | M        | All API endpoints shall be documented in OpenAPI 3.0 with request/response examples for both success and error cases                                                       |
| NFR-08-03 | S        | When a workflow step fails, the error response shall identify the specific step that failed, the input it received, and the error that occurred — not only the workflow ID |
| NFR-08-04 | S        | A local Docker Compose setup shall be provided that allows a client developer to run Relay against a local database with no external dependencies                          |
