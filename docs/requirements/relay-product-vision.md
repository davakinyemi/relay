# Product Vision - Relay

> **Status:** Draft v1.0
> **Last Updated:** 10.03.2026
> **Author:** Dave Akinyemi

---

## What is Relay?

Relay is a portfolio project — a working distributed task queue and workflow engine built with Spring Boot and PostgreSQL. It exists to demonstrate a concrete understanding of the systems concepts covered in Designing Data-Intensive Applications, specifically around durable task execution, failure recovery, and operational observability.

---

## The Core Idea

A task queue decouples the act of requesting work from the act of doing work. Without this separation, failures are silent, work is lost on restart, and systems become fragile under load. Relay makes work durable, observable, and recoverable — every task has a full lifecycle history that can be inspected and queried.

Relay adds a workflow layer on top: multistep sequences where each step depends on the previous, with automatic retry on failure and a dead-letter queue for tasks that cannot be recovered.

---

## Stakeholders

- **Developer:** building and maintaining Relay
- **Client:** application submitting tasks to Relay via its REST API (e.g. GAP)
- **Operator:** monitoring the live dashboard

---

## What Success Looks Like

- A submitted task survives a service restart without loss
- A failed task is automatically retried and eventually reaches the dead-letter queue if unrecoverable
- The dashboard shows queue depth, throughput, error rate, and execution timelines in real time
- GAP runs continuously against Relay, keeping the dashboard populated with live data
- The codebase demonstrates clearly: at-least-once delivery, idempotency, the outbox pattern, backpressure, and derived data

---

## Out of Scope — v1.0

- API authentication and authorization
- Multi-tenancy
- Distributed workers across multiple JVMs
- Recurring / scheduled tasks
- Visual workflow builder

---

## Relationship to GAP

GAP is a separate application that submits GitHub webhook events to Relay as multistep workflows. It exists to prove Relay works under a real, continuous workload and to keep the dashboard populated during demos. Relay knows nothing about GAP — the only connection between them is Relay's REST API.

---