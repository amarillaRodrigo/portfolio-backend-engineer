# HieloPedido — Offline-First B2B Order Management

**Role.** Backend engineer (sole author of the backend services).
**Stack.** Java 25 · Spring Boot · PostgreSQL 15 · Docker · JWT (HMAC-SHA256).
**Status.** Production-grade reference implementation, code on disk.

---

## Problem

Wholesale ice distributors operate from cold-storage warehouses and delivery trucks — environments where mobile connectivity is unreliable or absent. Sales reps need to register orders in the field and have them reach the central system without manual reconciliation once the signal returns.

A naive REST backend would reject duplicate submissions on retry, lose orders when the network drops mid-sync, and force the client to implement brittle offline queues. The challenge was to design a backend that accepts the same submission N times safely, preserves order across reconnects, and uses tokens that survive a long offline window without becoming a security liability.

## Approach

Split the system into two Spring Boot microservices behind a clear contract:

- **sync-service** — the entry point. Validates JWTs, enforces idempotency, rotates refresh tokens, and forwards accepted payloads to the order service.
- **order-service** — owns the transactional database. Manages orders, stock validation, and the product catalog.

The contract between client and server is built around **idempotency keys** carried on every write. The client generates the key once per logical order and retries safely until the server acknowledges. The **Outbox pattern** propagates committed orders to downstream consumers without losing events on a crash.

## Architecture

```
Flutter mobile client
        │  (HTTPS, JWT bearer)
        ▼
┌──────────────────────────────┐
│ sync-service   (port 8080/81)│
│  ─ JWT validation & refresh  │
│  ─ Idempotency middleware    │
│  ─ Outbox publisher          │
└──────────────────────────────┘
        │  (internal)
        ▼
┌──────────────────────────────┐
│ order-service   (port 8082)  │
│  ─ Orders + stock + catalog  │
│  ─ PostgreSQL 15 (Docker)    │
└──────────────────────────────┘
```

## Key Decisions

- **Refresh-token rotation on every use.** A leaked refresh token is detectable on the next request because the previous one is invalidated. Offline windows don't widen the attack surface beyond the configured refresh TTL.
- **Idempotency keys live at the API boundary, not in the database.** The sync service owns deduplication; the order service trusts that what it receives is logically unique. This keeps the transactional DB simple.
- **Outbox table inside each service.** No external message broker. Events are written in the same transaction as the business state change, then a publisher drains them. Atomicity over throughput.
- **Two databases, not two schemas.** `sync_db` and `order_db` isolated by Docker. Each service owns its schema end-to-end. Strictly enforced in the OpenAPI contract.
- **HMAC-SHA256 JWT with a 32-byte minimum secret.** Both services refuse to start if `JWT_SECRET` is missing or short — a fail-fast guard against a known deployment footgun.

## Results

- Connectivity loss of any duration is recoverable without manual intervention.
- Duplicate orders are impossible at the API boundary.
- JWT refresh rotation is verified end-to-end with integration tests (see `docs/testing-integration.md`).
- Extensible to additional client apps (admin web, third-party integrations) without changing the sync contract.

## Links

- Repository: `D:\programacion\ice`
- Spec: `PRD.md`, `docs/ARCHITECTURE.md`, `docs/API_REFERENCE.md`
- Public OpenAPI: `docs/openapi.json`
