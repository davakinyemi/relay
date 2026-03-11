# Relay — Functional Requirements

> **Status:** Draft v1.0
> **Last Updated:** 11.03.2026
> **Author:** Dave Akinyemi

---

## MoSCoW Priority Key

| Priority | Meaning                                                |
|:---------|:-------------------------------------------------------|
| **M**    | Must Have — MVP system is not releasable without this  |
| **S**    | Should Have — High value, included if time permits     |
| **C**    | Could Have — Nice to have, deferred to v2 if necessary |
| **W**    | Won't Have — Explicitly out of scope for v1.0          |

---

## FR-01: Task Management

| ID       | Priority | Requirement                                                                                                                                              |
|:---------|:---------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| FR-01-01 | M        | Relay shall accept task submissions via its REST API and persist each task to the database before returning a response                                   |
| FR-01-02 | M        | Each task shall have a unique identifier generated at the time of submission                                                                             |
| FR-01-03 | M        | Each task shall record a type, payload, current status, and timestamps for each status transition                                                        |
| FR-01-04 | M        | A submitted task shall survive a full service restart without loss                                                                                       |
| FR-01-05 | M        | Relay shall support the following task statuses: PENDING, PROCESSING, COMPLETED, FAILED, DEAD                                                            |
| FR-01-06 | M        | Relay shall accept an optional idempotency key on task submission — submitting the same key twice shall return the original task, not create a duplicate |
| FR-01-07 | S        | Relay shall support task priority levels — higher priority tasks shall be picked up before lower priority tasks                                          |

---

## FR-02: Workflow Orchestration

| ID       | Priority | Requirement                                                                                                                            |
|:---------|:---------|:---------------------------------------------------------------------------------------------------------------------------------------|
| FR-02-01 | M        | Relay shall accept workflow submissions consisting of an ordered sequence of two or more tasks                                         |
| FR-02-02 | M        | Steps within a workflow shall execute in the defined order — a step shall not begin until the previous step has completed successfully |
| FR-02-03 | M        | Each workflow shall have a unique identifier and a status that reflects the state of its current step                                  |
| FR-02-04 | M        | Workflow state shall be shared across steps — the output of one step shall be available as input to the next                           |
| FR-02-05 | M        | If any step in a workflow fails and exhausts its retries, the entire workflow shall transition to FAILED status                        |
| FR-02-06 | S        | A failed workflow shall be manually re-triggerable from the point of failure, without re-executing previously completed steps          |

---

## FR-03: Worker Execution

| ID       | Priority | Requirement                                                                                                                                           |
|:---------|:---------|:------------------------------------------------------------------------------------------------------------------------------------------------------|
| FR-03-01 | M        | Relay shall maintain a configurable pool of worker threads that continuously poll for PENDING tasks                                                   |
| FR-03-02 | M        | A task shall be processed by exactly one worker at a time — concurrent processing of the same task by multiple workers is not permitted               |
| FR-03-03 | M        | A worker claiming a task shall transition it to PROCESSING status atomically, using a database-level lock                                             |
| FR-03-04 | M        | A worker shall emit a periodic heartbeat while processing a task, updating a last_heartbeat_at timestamp                                              |
| FR-03-05 | M        | A task that has been PROCESSING without a heartbeat update for longer than a configurable timeout shall be automatically reset to PENDING             |
| FR-03-06 | M        | When all workers are occupied, new tasks shall remain in PENDING status until a worker becomes available — unbounded thread creation is not permitted |

---

## FR-04: Failure Handling

| ID       | Priority | Requirement                                                                                                                                    |
|:---------|:---------|:-----------------------------------------------------------------------------------------------------------------------------------------------|
| FR-04-01 | M        | A task that fails during execution shall be automatically retried up to a configurable maximum number of attempts                              |
| FR-04-02 | M        | Retry attempts shall use exponential backoff — the delay between attempts shall increase with each failure                                     |
| FR-04-03 | M        | Exponential backoff shall include random jitter to prevent multiple failed tasks from retrying simultaneously                                  |
| FR-04-04 | M        | Each failure shall record the error message and stack trace against the task for later inspection                                              |
| FR-04-05 | M        | A task that exhausts all retry attempts shall be moved to DEAD status and placed in the dead-letter queue                                      |
| FR-04-06 | M        | Tasks in the dead-letter queue shall be inspectable — their payload, error history, and attempt count shall be queryable                       |
| FR-04-07 | S        | A task in the dead-letter queue shall be manually re-queueable via the REST API                                                                |
| FR-04-08 | S        | Relay shall distinguish between retryable errors and fatal errors — a fatal error shall move the task directly to DEAD status without retrying |

---

## FR-05: Observability

| ID       | Priority | Requirement                                                                                                    |
|:---------|:---------|:---------------------------------------------------------------------------------------------------------------|
| FR-05-01 | M        | Relay shall provide a dashboard displaying current queue depth, grouped by task status                         |
| FR-05-02 | M        | The dashboard shall display task throughput — tasks completed per minute over a rolling time window            |
| FR-05-03 | M        | The dashboard shall display current error rate — failed tasks as a percentage of total attempts                |
| FR-05-04 | M        | The dashboard shall provide an execution timeline per task — showing each status transition with its timestamp |
| FR-05-05 | M        | The dashboard shall provide a dead-letter queue view — listing all DEAD tasks with their error details         |
| FR-05-06 | M        | The application shall emit a structured log entry at every task status transition                              |
| FR-05-07 | M        | The application shall expose a health check endpoint at /actuator/health                                       |
| FR-05-08 | S        | The dashboard shall display worker utilisation — how many workers are active versus idle                       |
| FR-05-09 | C        | The application shall expose metrics in a format consumable by an external monitoring tool                     |

---

## FR-06: REST API

| ID       | Priority | Requirement                                                                                               |
|:---------|:---------|:----------------------------------------------------------------------------------------------------------|
| FR-06-01 | M        | Relay shall expose a POST /api/v1/tasks endpoint for submitting a single task                             |
| FR-06-02 | M        | Relay shall expose a POST /api/v1/workflows endpoint for submitting a multi-step workflow                 |
| FR-06-03 | M        | Relay shall expose a GET /api/v1/tasks/{id} endpoint for retrieving the current state of a task           |
| FR-06-04 | M        | Relay shall expose a GET /api/v1/workflows/{id} endpoint for retrieving the current state of a workflow   |
| FR-06-05 | M        | Relay shall expose a GET /api/v1/tasks endpoint for listing tasks, filterable by status                   |
| FR-06-06 | M        | Relay shall expose a GET /api/v1/dashboard/summary endpoint returning aggregate metrics for the dashboard |
| FR-06-07 | M        | All API endpoints shall return appropriate HTTP status codes and structured error responses               |
| FR-06-08 | M        | All API endpoints shall be documented via OpenAPI 3.0, accessible at /swagger-ui.html                     |
| FR-06-09 | S        | Relay shall expose a POST /api/v1/tasks/{id}/requeue endpoint for manually re-queuing a dead-letter task  |