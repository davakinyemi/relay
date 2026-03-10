# Relay — Distributed Task Queue & Workflow Engine

> A task queue and workflow orchestration platform
> built with Spring Boot and PostgreSQL.

## Status

**In active development** — Requirements phase

## Overview

Relay is an infrastructure platform for durable, observable, 
asynchronous task execution. It provides:

- **Durable task queuing** — tasks survive service restarts
- **Workflow orchestration** — multi-step pipelines with 
  dependency management
- **At-least-once delivery** — with idempotent handler support
- **Automatic retries** — exponential backoff with dead-letter queues
- **Real-time observability** — live dashboard for queue depth, 
  throughput, and execution history

## Architecture

> Detailed architecture documentation: [`/docs/design`](/docs/design)

| Layer | Technology |
|---|---|
| Backend | Spring Boot 3 / Java 21 |
| Database | PostgreSQL (Neon) |
| Event Streaming | Apache Kafka (Upstash) |
| Frontend | Angular 19 / Angular Material |

## Infrastructure

> Full infrastructure documentation: [`/docs/design`](/docs/design)

| Concern | Service |
|---|---|
| Source Control | GitHub |
| CI/CD | GitHub Actions |
| Frontend Hosting | Cloudflare Pages |
| Backend Hosting | Render |
| Database | Neon.tech |
| Event Streaming | Upstash Kafka |

## Documentation

| Document | Description |
|---|---|
| [Requirements](/docs/requirements/) | Functional & non-functional requirements |
| [Architecture](/docs/design/) | System design & component diagrams |
| [ADRs](/docs/adr/) | Architecture Decision Records |
| [Setup Guide](/docs/guides/setup.md) | Local development setup |

## Reference Implementation

**GAP (GitHub Activity Processor)** is the canonical reference 
client built on top of Relay, demonstrating real-world workflow 
orchestration against the GitHub API.

→ [github.com/davakinyemi/gap](https://github.com/davakinyemi/gap)

## License

MIT
```
