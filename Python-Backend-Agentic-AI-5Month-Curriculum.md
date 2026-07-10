# Python Backend & Agentic AI Mastery — 5-Month Curriculum
### For a Java-background engineer moving to deep, framework-independent Python Backend + Agentic AI engineering
**Format:** 4 hrs/day × 7 days/week × ~22 weeks ≈ 600 hours

---

## How to Use This Document
Every phase gives you: **why it exists → what you'll be able to do → concepts (leveled) → projects → DSA parallel track → resources → checklists/interview prep.**
For every concept, apply the same study loop yourself (this is explicitly modeled for anchor topics per phase, and you should repeat it for every sub-topic):
`Definition → Why it exists → Internal working → Common mistakes → Real-world usage → Interview Qs → Mini exercise.`

---

# Table of Contents
1. Phase 1 — Python Mastery (Weeks 1–4)
2. Phase 2 — Networking Fundamentals (Week 5)
3. Phase 3 — Backend Development (Weeks 6–8)
4. Phase 4 — Databases (Weeks 9–10)
5. Phase 5 — Distributed Systems (Week 11)
6. Phase 6 — DevOps (Week 12)
7. Phase 7 — System Design (Weeks 13–14)
8. Phase 8 — LLM Fundamentals (Weeks 15–16)
9. Phase 9 — Agentic AI (Weeks 17–18)
10. Phase 10 — Production AI Systems (Week 19)
11. Phase 11 — Open Source Reading (Week 20, ongoing)
12. Phase 12 — Capstone Project (Weeks 21–22)
13. DSA Roadmap (parallel, all 22 weeks)
14. Weekly Schedule Template
15. Resource Library
16. Progress Tracking System

---

# PHASE 1 — Python Mastery
**Duration:** 4 weeks (112 hrs) | **Weeks 1–4**

## Phase Objective
You write Python but you don't yet *think* in Python — you think in Java and translate. Every later phase (FastAPI, async I/O, agent frameworks) assumes you understand Python's object model, memory model, and concurrency model at a level where you can predict behavior without running the code. This phase exists to remove the "AI wrote it, I don't fully know why it works" gap at its root, before frameworks add layers on top of it.

## Learning Outcomes
By the end you can:
- Explain what happens in memory when `a = b` runs, including reference counting and the GC generations.
- Write a decorator, a context manager, and a descriptor from scratch without looking it up.
- Explain the GIL's actual mechanism and correctly decide thread vs. process vs. async for a given workload.
- Read any medium-complexity Python file (e.g., inside FastAPI/SQLAlchemy source) and understand control flow immediately.

## Concepts

### Level 1 — Core Language
| Topic | Focus |
|---|---|
| Variables & references | Names are labels, not boxes; `id()`, mutability vs identity |
| Data types | int (arbitrary precision), float (IEEE 754 pitfalls), str (immutability, interning), bytes vs str |
| Control flow | `for/else`, `while/else`, walrus operator |
| Functions | Default args gotcha (mutable defaults), `*args/**kwargs`, closures, first-class functions |
| Collections | list/tuple/dict/set internals (dict is a hash table w/ open addressing since 3.6 insertion-order guarantee) |

**Anchor deep-dive — Mutable Default Argument Trap**
- *Definition:* Default arguments are evaluated once, at function definition time.
- *Why it exists:* Python evaluates the `def` statement once; defaults are stored as function attributes.
- *Internal working:* `func.__defaults__` is a tuple created at def-time; every call reuses the same object if it's mutable.
- *Common mistake:* `def add(item, bucket=[]):` — bucket persists across calls.
- *Real-world usage:* seen constantly in FastAPI route handlers with list/dict defaults — must use `None` + `if x is None: x = []`.
- *Interview Q:* "What does this print twice in a row?" (classic trap)
- *Mini exercise:* Write `add_item(item, bucket=[])`, call it 3 times, explain output, then fix it.

### Level 2 — OOP & Data Modeling
| Topic | Focus |
|---|---|
| Classes & instances | `__init__`, `__new__`, class vs instance attributes |
| Inheritance | MRO (C3 linearization), `super()` mechanics |
| Composition | favor over inheritance; mixins |
| Protocols / Structural typing | `Protocol`, duck typing formalized, `__len__`, `__iter__` |
| Dunder methods | `__eq__`, `__hash__`, `__repr__`, `__call__` |
| Dataclasses | `@dataclass`, `field(default_factory=)`, frozen, slots |

**Anchor deep-dive — MRO / `super()`**
- *Definition:* Method Resolution Order determines which class's method runs in multiple inheritance.
- *Internal working:* C3 linearization algorithm builds a deterministic linear order; `super()` walks this order, not just "the parent."
- *Common mistake:* Assuming `super()` always calls the immediate parent — it calls the *next in MRO*, which depends on the full class graph.
- *Interview Q:* Diamond inheritance — trace the MRO manually.
- *Mini exercise:* Build a 4-class diamond hierarchy, print `ClassName.__mro__`, predict output before running.

### Level 3 — Advanced Language Mechanics
| Topic | Focus |
|---|---|
| Decorators | function/class decorators, `functools.wraps`, parametrized decorators, stacking order |
| Generators & iterators | `yield`, `yield from`, generator protocol, memory efficiency |
| Context managers | `__enter__/__exit__`, `contextlib.contextmanager`, exception suppression |
| Descriptors | `__get__/__set__/__delete__`; how `property`, methods, and ORMs use them |
| Metaclasses | `type()`, `__new__` vs `__init__` in metaclasses, when frameworks (Pydantic, SQLAlchemy) use them |
| Memory management | refcounting, generational GC, `gc` module, circular references, weakrefs |
| Typing | `TypeVar`, `Generic`, `Protocol`, `overload`, static analysis with mypy/pyright |

**Anchor deep-dive — Descriptors (this is how `property`, ORMs, and Pydantic fields work)**
- *Definition:* An object with `__get__`/`__set__` controls attribute access on another class.
- *Why it exists:* Lets a class intercept `obj.attr` access to add validation, laziness, or storage logic — the mechanism behind `@property`, `functools.cached_property`, and every SQLAlchemy `Column`.
- *Internal working:* Python attribute lookup checks the class's `__dict__` for a descriptor *before* the instance `__dict__` (for data descriptors).
- *Common mistake:* Confusing instance attributes with descriptor-backed class attributes; shared mutable state across instances if descriptor stores data on itself instead of the instance.
- *Interview Q:* "Implement a `TypedAttribute` descriptor that validates type on assignment."
- *Mini exercise:* Build a `Validated` descriptor enforcing `int` type; attach to a class; prove it fires on every instance independently.

### Level 4 — Concurrency
| Topic | Focus |
|---|---|
| GIL | what it actually locks (bytecode execution, not I/O), why it exists (refcounting safety) |
| Threading | good for I/O-bound, `threading.Lock`, GIL releases on I/O calls |
| Multiprocessing | true parallelism, `Pool`, IPC cost, pickling constraints |
| AsyncIO | event loop, coroutines, `await`, tasks, `asyncio.gather`, cooperative multitasking |
| When to use what | CPU-bound → multiprocessing; I/O-bound + many connections → asyncio; I/O-bound + simple → threading |

**Anchor deep-dive — AsyncIO Event Loop**
- *Definition:* A single-threaded loop that schedules coroutines, running one until it hits an `await` on an I/O operation, then switching.
- *Why it exists:* Handle thousands of concurrent I/O-bound connections without the memory/context-switch cost of OS threads.
- *Internal working:* Coroutines are state machines; `await` yields control back to the loop; the loop polls I/O readiness (via `select`/`epoll`) and resumes the right coroutine.
- *Common mistake:* Calling a blocking function (e.g., `time.sleep`, sync `requests.get`) inside an async function — freezes the entire loop.
- *Real-world usage:* FastAPI's async endpoints, DB drivers like `asyncpg`, `httpx.AsyncClient`.
- *Interview Q:* "Why does one blocking call inside an async function stall unrelated requests?"
- *Mini exercise:* Write two coroutines, one that sleeps with `time.sleep` and one with `asyncio.sleep`, run concurrently, observe the difference.

### Level 5 — Engineering Practices
Testing (pytest, fixtures, mocking, parametrize) · Logging (structured logging, log levels, handlers) · Packaging (pyproject.toml, `build`, publishing) · Virtual environments (venv, poetry, uv) · Project structure (src layout, `__init__.py` semantics)

## Projects

### Mini — CLI Expense Tracker
- **Objectives:** Practice OOP modeling, file persistence, and clean CLI design.
- **Features:** Add/edit/delete expenses, category tagging, monthly summary, CSV export.
- **Folder Structure:** `expense_tracker/{models.py, storage.py, cli.py, tests/}`
- **Technologies:** Python stdlib, `dataclasses`, `argparse`, `json`/`csv`.
- **Learning Outcomes:** Dataclasses, file I/O, basic OOP composition.
- **Expected Time:** 6–8 hrs.

### Intermediate — Custom ORM-Lite
- **Objectives:** Understand how ORMs map Python objects to storage using descriptors and metaclasses — build a miniature version yourself.
- **Features:** `Field` descriptors with type validation, a `Model` metaclass that auto-registers fields, save/load to JSON, simple query filter (`Model.filter(name="x")`).
- **Folder Structure:** `orm_lite/{fields.py, model.py, backend_json.py, tests/}`
- **Technologies:** Descriptors, metaclasses, `typing`.
- **Learning Outcomes:** Deep understanding of descriptor protocol + metaclass mechanics that power SQLAlchemy/Django/Pydantic internally.
- **Expected Time:** 10–12 hrs.

### Advanced — Concurrent Web Scraper Benchmark
- **Objectives:** Build the same scraping task three ways to internalize concurrency-model tradeoffs empirically, not theoretically.
- **Features:** Fetch N URLs and parse titles using (a) threading, (b) multiprocessing, (c) asyncio + `httpx`; a benchmarking harness comparing wall-clock time and CPU usage.
- **Folder Structure:** `scraper_bench/{threaded.py, multiproc.py, async_version.py, benchmark.py, results/}`
- **Technologies:** `threading`, `multiprocessing`, `asyncio`, `httpx`, `time`/`tracemalloc`.
- **Learning Outcomes:** Empirical, not theoretical, understanding of when each concurrency model wins.
- **Expected Time:** 14–16 hrs.

## Checklist
- [ ] Can explain reference counting + GC generations unaided
- [ ] Built a working decorator with arguments
- [ ] Built a descriptor-based validator
- [ ] Explained GIL correctly in an interview-style answer
- [ ] 3 concurrency models benchmarked on the same task

## Interview Bank (sample)
1. Why is `is` different from `==`? 2. What's the difference between `@staticmethod`, `@classmethod`, instance methods? 3. How does Python's `dict` handle collisions? 4. What's the difference between `deepcopy` and `copy`? 5. Explain `__slots__` and its memory tradeoff.

---

# PHASE 2 — Networking Fundamentals
**Duration:** 1 week (28 hrs) | **Week 5**

## Phase Objective
Backend engineering *is* networking with a database attached. Before you can reason about API design, load balancers, or why a WebSocket behaves differently from SSE, you need the OSI/TCP-IP model as a mental map. This phase is short but load-bearing for Phases 3, 5, 6, and 7.

## Learning Outcomes
Explain what happens from typing a URL to receiving a rendered response, across every layer. Choose the right protocol (REST/gRPC/WebSocket/SSE) for a given use case and justify it.

## Concepts
**Level 1 — Foundations:** OSI Model (7 layers), TCP/IP model, ARP, Routing, NAT, DNS resolution chain.
**Level 2 — Transport:** TCP (3-way handshake, flow control, congestion control) vs UDP (no guarantees, low latency) — when each is used (HTTP vs video streaming/gaming).
**Level 3 — Application layer:** HTTP/1.1 vs HTTP/2 (multiplexing) vs HTTP/3 (QUIC over UDP), HTTPS/TLS/SSL handshake, REST principles, GraphQL (schema, resolvers, N+1 problem), gRPC (protobuf, streaming, HTTP/2-based).
**Level 4 — Real-time & Auth:** WebSockets (full-duplex) vs SSE (server→client only, simpler) — decision table; Cookies vs Sessions vs JWT (stateless tradeoffs); OAuth2 flows (auth code, client credentials, PKCE); CORS (preflight, why browsers enforce it, not servers).
**Level 5 — Infra:** Reverse proxy vs Load balancer vs API Gateway (distinct roles, not synonyms), CDN, Caching layers (browser/CDN/app/DB), Nginx config basics, Rate limiting (token bucket, leaky bucket), Circuit breakers (closed/open/half-open states).

**Anchor deep-dive — JWT vs Sessions**
- *Definition:* Session = server stores state, client holds an opaque ID cookie. JWT = client holds a signed, self-contained token; server verifies signature, stores nothing.
- *Why it exists:* JWT solves horizontal scaling — no shared session store needed across servers.
- *Common mistake:* Storing sensitive data unencrypted in a JWT payload (it's signed, not encrypted); not handling revocation (stateless tokens can't be "logged out" without extra infra like a blocklist).
- *Interview Q:* "How do you revoke a JWT before expiry?" (short-lived tokens + refresh tokens + blocklist)
- *Mini exercise:* Implement JWT issue/verify manually with `pyjwt`, then break it by tampering payload and observe signature failure.

## Projects

### Mini — Raw TCP Echo Server/Client
- **Objectives:** Feel the transport layer directly, no abstractions.
- **Features:** Client connects, sends a message, server echoes it back; handle multiple sequential clients.
- **Folder Structure:** `tcp_echo/{server.py, client.py}`
- **Technologies:** Python `socket` stdlib only.
- **Learning Outcomes:** Sockets, blocking I/O, byte-level send/recv.
- **Expected Time:** 3–4 hrs.

### Intermediate — Hand-Rolled HTTP/1.1 Parser
- **Objectives:** See exactly what a web framework does with raw bytes before you hand it off to Uvicorn.
- **Features:** Parse request line, headers, and body from a raw socket stream; return a valid minimal HTTP/1.1 response; support `GET` and `POST`.
- **Folder Structure:** `mini_http/{parser.py, server.py, tests/}`
- **Technologies:** `socket`, manual string/byte parsing.
- **Learning Outcomes:** HTTP wire format, header parsing edge cases, why frameworks exist.
- **Expected Time:** 6–8 hrs.

### Advanced — Protocol Comparison Lab
- **Objectives:** Justify protocol choice with real measurements, not just theory.
- **Features:** Implement the same "live price ticker" feature 3 ways — polling, SSE, and WebSockets; measure latency and bandwidth for each under simulated load.
- **Folder Structure:** `protocol_lab/{polling.py, sse_server.py, ws_server.py, load_test.py}`
- **Technologies:** FastAPI (minimal), `websockets`, `httpx`, a simple load generator.
- **Learning Outcomes:** Concrete, measured tradeoffs between real-time protocols.
- **Expected Time:** 8–10 hrs.

## Checklist
- [ ] Can draw OSI model and map real tools to each layer
- [ ] Explained TCP handshake and TLS handshake back-to-back
- [ ] Justified WebSocket vs SSE vs polling for 3 different scenarios

---

# PHASE 3 — Backend Development
**Duration:** 3 weeks (84 hrs) | **Weeks 6–8**

## Phase Objective
This is where Python + networking fuse into shippable services. FastAPI is the vehicle because its internals (Pydantic validation, dependency injection, ASGI) are transparent enough to learn *how* a framework works, not just its API — directly serving your goal of not depending on "vibe coding."

## Learning Outcomes
Build a multi-router FastAPI app with auth, background tasks, file streaming, and tests, and explain what ASGI/Uvicorn does underneath every request.

## Concepts
**Level 1:** FastAPI basics (path/query/body params), Pydantic models (validation, `BaseModel`, validators), project structure (routers, services, repositories layering).
**Level 2:** Dependency Injection (`Depends`, scoping, overriding for tests), Middleware (request/response hooks, ordering), Configuration (`pydantic-settings`, env-based config), Logging setup.
**Level 3:** Authentication (password hashing w/ bcrypt/argon2, JWT issuing) vs Authorization (RBAC, scopes), Background tasks vs Celery (when in-process isn't enough), File uploads (streaming vs buffering), Validation edge cases (nested models, custom validators).
**Level 4:** Async endpoints done right (avoiding blocking calls), WebSockets in FastAPI, Server-Sent Events for streaming LLM responses, Testing (pytest + `TestClient`/`httpx.AsyncClient`, dependency overrides, fixtures for DB).

**Anchor deep-dive — Dependency Injection in FastAPI**
- *Definition:* `Depends()` lets FastAPI resolve and inject a callable's return value into your route function before it runs.
- *Why it exists:* Decouples business logic from request parsing; enables swapping real DB sessions for test mocks without touching route code.
- *Internal working:* FastAPI builds a dependency graph per request, resolves it (including nested `Depends`), caches results per-request by default.
- *Common mistake:* Creating a new DB connection per dependency call instead of reusing a request-scoped session; forgetting `yield`-based dependencies need try/finally for cleanup.
- *Interview Q:* "How would you inject a different DB for integration tests without changing route code?"
- *Mini exercise:* Build a `get_db` yield-dependency, override it in tests with an in-memory SQLite session.

## Projects

### Mini — Todo REST API with Auth
- **Objectives:** Solidify FastAPI + Pydantic + JWT auth fundamentals end-to-end.
- **Features:** Signup/login, JWT-protected CRUD on todos, per-user data isolation, full pytest suite with dependency overrides.
- **Folder Structure:** `todo_api/{main.py, routers/, models.py, auth.py, tests/}`
- **Technologies:** FastAPI, Pydantic, SQLite/SQLAlchemy, `pyjwt`, pytest, `httpx.AsyncClient`.
- **Learning Outcomes:** Auth flow, dependency injection, testable route design.
- **Expected Time:** 10–12 hrs.

### Intermediate — Multi-Tenant Blog API
- **Objectives:** Handle real production concerns — background work, file handling, abuse prevention.
- **Features:** Tenant-scoped posts/comments, background email notification on new comment, image upload for post covers, per-IP rate limiting middleware.
- **Folder Structure:** `blog_api/{main.py, routers/, services/, middleware/, tests/}`
- **Technologies:** FastAPI, PostgreSQL, SQLAlchemy, Celery (or `BackgroundTasks` first), Redis (rate limiting).
- **Learning Outcomes:** Middleware design, background task tradeoffs, multi-tenancy patterns.
- **Expected Time:** 16–18 hrs.

### Advanced — Real-Time Chat Backend
- **Objectives:** Build a horizontally-scalable real-time system, not just a single-process demo.
- **Features:** WebSocket chat rooms, Redis pub/sub so messages fan out across multiple server instances, JWT-authenticated socket connections, message persistence.
- **Folder Structure:** `chat_backend/{main.py, ws_manager.py, pubsub.py, auth.py, tests/}`
- **Technologies:** FastAPI WebSockets, Redis Pub/Sub, PostgreSQL, JWT.
- **Learning Outcomes:** Real-time architecture that survives running >1 server instance — the crux of scaling WebSockets.
- **Expected Time:** 18–20 hrs.

## Checklist
- [ ] Built and tested a FastAPI app with dependency overrides
- [ ] Implemented JWT auth from scratch (not copy-pasted)
- [ ] Explained ASGI vs WSGI difference confidently

---

# PHASE 4 — Databases
**Duration:** 2 weeks (56 hrs) | **Weeks 9–10**

## Phase Objective
Every backend and every RAG system is bottlenecked by data storage decisions. You need SQL fluency plus an understanding of what's happening under the hood (indexes, MVCC, isolation) so query performance and correctness aren't a guessing game.

## Learning Outcomes
Design a normalized schema, write window functions and CTEs confidently, explain isolation levels with concrete anomaly examples, and use SQLAlchemy/Alembic as tools you understand rather than magic.

## Concepts
**Level 1:** SQL fundamentals, PostgreSQL setup, basic joins, normalization (1NF–3NF).
**Level 2:** Indexes (B-tree internals, when indexes hurt writes), Transactions (ACID), Isolation levels (read uncommitted → serializable, with concrete anomaly demos: dirty read, non-repeatable read, phantom read), MVCC (how Postgres avoids read locks), Locks (row vs table, deadlocks).
**Level 3:** Advanced SQL — Window functions, CTEs (recursive too), Views vs Materialized Views, Stored Procedures.
**Level 4:** Scaling — Partitioning (range/list/hash), Replication (sync/async, leader-follower), Sharding (strategies + tradeoffs). ORMs — SQLAlchemy Core vs ORM, session lifecycle, lazy vs eager loading (N+1 problem), Alembic migrations.
**Level 5:** Redis (data structures, use cases: cache, pub/sub, rate limiting, sessions), MongoDB (document model, when NoSQL beats SQL — flexible schema, high write throughput).

**Anchor deep-dive — MVCC (Multi-Version Concurrency Control)**
- *Definition:* Postgres keeps multiple versions of a row so readers never block writers and vice versa.
- *Why it exists:* Traditional locking read/write causes contention; MVCC lets a transaction see a consistent snapshot without locking readers.
- *Internal working:* Each row has hidden `xmin`/`xmax` transaction IDs; a query sees only rows visible to its snapshot; `VACUUM` cleans up dead row versions.
- *Common mistake:* Assuming "SELECT never blocks" means no consistency issues — long-running transactions can bloat tables (dead tuples pile up) without regular vacuuming.
- *Interview Q:* "Why does Postgres need VACUUM and what happens if you never run it?"
- *Mini exercise:* Open two `psql` sessions, demonstrate a non-repeatable read at `READ COMMITTED`, then show it disappears at `REPEATABLE READ`.

## Projects

### Mini — E-Commerce Schema Design
- **Objectives:** Practice normalization and advanced SQL on a realistic domain.
- **Features:** Fully normalized (3NF) schema for products/orders/customers/inventory; 15 queries including 3 window-function queries (running totals, ranking, moving average).
- **Folder Structure:** `ecommerce_schema/{schema.sql, queries.sql, erd.md}`
- **Technologies:** PostgreSQL, `psql`.
- **Learning Outcomes:** Normalization judgment, window function fluency.
- **Expected Time:** 8–10 hrs.

### Intermediate — Caching Layer for Blog API
- **Objectives:** Add a real caching layer with correct invalidation to an existing service (Phase 3's Blog API).
- **Features:** Cache-aside pattern for post reads, explicit invalidation on write, TTL fallback, cache-hit/miss metrics.
- **Folder Structure:** extends `blog_api/{cache.py, ...}`
- **Technologies:** Redis, FastAPI, SQLAlchemy.
- **Learning Outcomes:** Cache invalidation is the hard part — this project makes that concrete.
- **Expected Time:** 8–10 hrs.

### Advanced — Sharded Key-Value Store Prototype
- **Objectives:** Understand sharding by building a (small, educational) version yourself.
- **Features:** Consistent-hashing-based routing across N storage nodes (in-process dicts or SQLite files), add/remove a node and observe rebalancing, basic replication for fault tolerance.
- **Folder Structure:** `sharded_kv/{router.py, node.py, hashring.py, tests/}`
- **Technologies:** Python, consistent hashing (from Phase 7), sockets or in-process simulation.
- **Learning Outcomes:** Concrete mental model for how real sharded systems (Cassandra, DynamoDB) route and rebalance.
- **Expected Time:** 12–14 hrs.

## Checklist
- [ ] Reproduced all 3 read anomalies manually
- [ ] Wrote a recursive CTE unaided
- [ ] Explained N+1 problem and fixed it with eager loading

---

# PHASE 5 — Distributed Systems
**Duration:** 1 week (28 hrs) | **Week 11**

## Phase Objective
Agentic AI systems and modern backends are inherently distributed (multiple services, queues, workers). This phase gives you the vocabulary and mental models (CAP, idempotency, sagas) to design systems that don't silently corrupt data under failure.

## Learning Outcomes
Explain CAP theorem tradeoffs with real system examples, design an idempotent API, and choose between Kafka/RabbitMQ/Celery correctly.

## Concepts
CAP Theorem (with concrete DB examples: Postgres=CP-leaning, Cassandra=AP-leaning) · Replication (revisited in a distributed context: sync vs async, multi-leader conflicts) · Consistency models (strong/eventual) · Leader election (Raft basics) · Message Queues (general concept: decoupling producers/consumers, at-least-once vs exactly-once delivery) — realized concretely via: Kafka (log-based, partitions, consumer groups) vs RabbitMQ (broker-based, routing) vs Celery (task queue built on top of a broker) · Distributed locks (Redis Redlock, caveats) · Event-driven architecture · Microservices (bounded contexts) · Service discovery · Observability (traces/metrics/logs = the 3 pillars) · Idempotency (idempotency keys in payment APIs) · Saga pattern (choreography vs orchestration for distributed transactions).

**Anchor deep-dive — Idempotency**
- *Definition:* An operation that produces the same result no matter how many times it's executed.
- *Why it exists:* Networks fail; clients retry; without idempotency a retried "charge card" call double-charges.
- *Internal working:* Client sends an idempotency key; server stores (key → result) and returns cached result on retry instead of re-executing.
- *Common mistake:* Making the key based on request content only (two legitimately different requests can collide) instead of a client-generated unique key.
- *Interview Q:* "Design an idempotent payment endpoint."
- *Mini exercise:* Build a FastAPI endpoint with an idempotency-key header backed by Redis.

## Projects

### Mini — Distributed Lock Demo
- **Objectives:** Make a race condition visible, then fix it with a distributed lock.
- **Features:** Two worker processes incrementing a shared counter in Redis; show the race without a lock, then fix it with Redis-based locking (Redlock-style).
- **Folder Structure:** `dist_lock/{worker.py, lock.py}`
- **Technologies:** Redis, Python `multiprocessing`.
- **Learning Outcomes:** Why naive shared state fails under concurrency across processes/machines.
- **Expected Time:** 4–5 hrs.

### Intermediate — Message Queue Comparison Lab
- **Objectives:** Feel the practical difference between a broker (RabbitMQ) and a log (Kafka) on the same task.
- **Features:** Implement the same "order event → 2 consumers" pipeline on both RabbitMQ and Kafka; compare delivery guarantees, replay behavior, and consumer group semantics.
- **Folder Structure:** `mq_lab/{rabbitmq_version/, kafka_version/, notes.md}`
- **Technologies:** RabbitMQ, Kafka, Python clients (`pika`, `confluent-kafka`).
- **Learning Outcomes:** Concrete criteria for choosing Kafka vs RabbitMQ vs Celery.
- **Expected Time:** 8–10 hrs.

### Advanced — Order Processing Saga
- **Objectives:** Handle distributed transactions correctly under partial failure.
- **Features:** Order → Payment → Inventory across 3 mock services using Celery + Redis; compensating-transaction rollback path when any step fails; idempotency keys on every step.
- **Folder Structure:** `saga_demo/{order_service/, payment_service/, inventory_service/, orchestrator.py}`
- **Technologies:** Celery, Redis, FastAPI (per mock service).
- **Learning Outcomes:** Saga orchestration pattern, idempotency in a multi-service failure scenario.
- **Expected Time:** 12–14 hrs.

## Checklist
- [ ] Explained CAP with 2 real DB examples
- [ ] Built one idempotent endpoint
- [ ] Chose Kafka vs RabbitMQ for 3 given scenarios and justified it

---

# PHASE 6 — DevOps
**Duration:** 1 week (28 hrs) | **Week 12**

## Phase Objective
You can't call something "production-ready" if you can't containerize, deploy, and monitor it. This phase operationalizes everything built so far.

## Learning Outcomes
Containerize a multi-service app, wire a CI/CD pipeline, and stand up basic monitoring.

## Concepts
Linux essentials (processes, permissions, systemd, shell scripting) · Docker (images vs containers, layers, multi-stage builds) · Docker Compose (multi-service local orchestration) · Kubernetes basics (Pods, Deployments, Services, ConfigMaps — conceptual, not deep ops) · GitHub Actions / CI-CD (build-test-deploy pipeline) · Monitoring (the practice: what to measure — latency/errors/traffic/saturation — the "four golden signals") using Prometheus (metrics scraping) + Grafana (dashboards) + OpenTelemetry (traces) · Deployment (strategies: rolling, blue-green, canary; rollback) · Nginx as reverse proxy/deploy target · Cloud basics (AWS/GCP/Azure — compute, storage, managed DB overview, no need to master one deeply yet).

## Projects

### Mini — Dockerize a Single Service
- **Objectives:** Learn Docker fundamentals hands-on with a minimal, correct image.
- **Features:** Multi-stage Dockerfile for the Phase 1/3 app, `.dockerignore`, environment-based config, health-check endpoint.
- **Folder Structure:** `dockerized_app/{Dockerfile, .dockerignore, app/}`
- **Technologies:** Docker.
- **Learning Outcomes:** Image layering, multi-stage build size optimization.
- **Expected Time:** 4–5 hrs.

### Intermediate — Full Stack Compose + CI
- **Objectives:** Orchestrate a real multi-service app and automate its pipeline.
- **Features:** Dockerize the Phase 3 Blog API + Postgres + Redis with Docker Compose; GitHub Actions pipeline (lint → test → build image → push to registry).
- **Folder Structure:** `blog_api/{docker-compose.yml, .github/workflows/ci.yml, Dockerfile}`
- **Technologies:** Docker Compose, GitHub Actions.
- **Learning Outcomes:** Multi-service local orchestration, automated CI gating.
- **Expected Time:** 8–10 hrs.

### Advanced — Observability Stack
- **Objectives:** Make the running system's health and performance visible.
- **Features:** Expose Prometheus metrics from the Blog API, deploy Prometheus + Grafana via Compose, build one dashboard (request latency, error rate, throughput), add OpenTelemetry tracing across 2 endpoints, choose and justify one deployment strategy (rolling/blue-green/canary) for this service.
- **Folder Structure:** `observability_stack/{prometheus.yml, grafana/, otel_config.yml}`
- **Technologies:** Prometheus, Grafana, OpenTelemetry.
- **Learning Outcomes:** The three observability pillars applied to a real service, plus deployment-strategy tradeoffs.
- **Expected Time:** 10–12 hrs.

## Checklist
- [ ] Multi-stage Dockerfile under 200MB image size
- [ ] Green CI pipeline on every push
- [ ] One working Grafana dashboard showing request latency

---

# PHASE 7 — System Design
**Duration:** 2 weeks (56 hrs) | **Weeks 13–14**

## Phase Objective
This phase converts everything learned into interview-ready and real-world architecture judgment — it's synthesis, not new tools.

## Learning Outcomes
Design any of the classic systems below on a whiteboard in 35–40 minutes, stating tradeoffs explicitly.

## Concepts & Practice Systems
Scalability fundamentals (vertical vs horizontal, stateless services) · Caching strategies (cache-aside, write-through, write-back) · Load balancing algorithms (round robin, least connections, consistent hashing) · Database scaling (read replicas, sharding revisited) · Rate limiter design · **Design Trade-offs** (the explicit evaluation framework applied to every case study below: consistency vs availability, latency vs throughput, cost vs performance, simplicity vs flexibility — always state which side you're choosing and why) · **Case studies (design each, then compare to real published architectures):** Chat System, Notification System, URL Shortener, Instagram (feed + media), WhatsApp (E2E + delivery guarantees), YouTube (upload + transcoding + CDN), Uber (geospatial + matching), Netflix (recommendation + CDN), Payment Gateway (idempotency + saga), Booking System (inventory locking, overbooking prevention).

**Anchor deep-dive — Consistent Hashing**
- *Definition:* A hashing scheme where adding/removing a node only remaps ~1/N of keys, not all of them.
- *Why it exists:* Naive `hash(key) % N` remaps almost everything when N changes — catastrophic for caches/shards.
- *Interview Q:* "Why not just use modulo hashing for a distributed cache?"
- *Mini exercise:* Implement consistent hashing with virtual nodes in Python; simulate adding a node and measure % of keys remapped.

## Projects

### Mini — Rate Limiter Service
- **Objectives:** Implement, not just describe, a system-design classic.
- **Features:** Token-bucket rate limiter as a FastAPI middleware backed by Redis; configurable per-user limits; returns proper `429` + `Retry-After`.
- **Folder Structure:** `rate_limiter/{middleware.py, redis_bucket.py, tests/}`
- **Technologies:** FastAPI, Redis.
- **Learning Outcomes:** Token bucket vs leaky bucket implemented, not just whiteboarded.
- **Expected Time:** 5–6 hrs.

### Intermediate — URL Shortener (fully built)
- **Objectives:** Take one classic system past the whiteboard into a working service.
- **Features:** Base62 short-code generation, redirect endpoint, click-analytics counter, cache-aside layer for hot links.
- **Folder Structure:** `url_shortener/{main.py, encoder.py, cache.py, tests/}`
- **Technologies:** FastAPI, PostgreSQL, Redis.
- **Learning Outcomes:** Turning a design doc into a real, scalable service.
- **Expected Time:** 10–12 hrs.

### Advanced — 3 Full System Design Docs
- **Objectives:** Build interview-grade design documents for 3 systems from the case-study list (pick different tradeoff profiles, e.g., Chat System, Notification System, and one of Instagram/Netflix/Uber).
- **Features (per doc):** Requirements (functional/non-functional), back-of-envelope estimation (QPS, storage, bandwidth), high-level architecture, deep dive on one bottleneck, explicit trade-off statements, and a comparison to the real system's known public architecture where available.
- **Folder Structure:** `system_design_docs/{chat_system.md, notification_system.md, third_system.md}`
- **Technologies:** N/A (design artifact) — diagrams described in text/Mermaid.
- **Learning Outcomes:** Interview-ready structured design communication.
- **Expected Time:** 12–15 hrs.

## Checklist
- [ ] 3 full system design docs completed
- [ ] Can do back-of-envelope capacity estimation from memory
- [ ] Mock-interviewed (self or peer) at least twice

---

# PHASE 8 — LLM Fundamentals
**Duration:** 2 weeks (56 hrs) | **Weeks 15–16**

## Phase Objective
You've built agentic projects via "vibe coding" — this phase makes the LLM itself stop being a black box, which is essential before Phase 9's agent design and Phase 10's production RAG.

## Learning Outcomes
Explain attention math conceptually, tokenization's effect on cost/behavior, and build a RAG pipeline without LangChain to prove you understand the primitives frameworks wrap.

## Concepts
**Level 1:** Transformers (encoder/decoder/decoder-only), Attention (Q/K/V intuition, why it replaced RNNs), Embeddings (vector semantics), Tokenization (BPE, why "strawberry" trips up token-level counting).
**Level 2:** Context windows (cost/latency scaling), KV Cache (why it makes autoregressive decoding fast, memory tradeoff), Prompt engineering (system/user/few-shot, structured prompting).
**Level 3:** Sampling strategies (temperature, top-k, top-p, beam search) and their effect on determinism/creativity, Inference basics, Fine-tuning (full vs parameter-efficient), LoRA/QLoRA (low-rank adapters, why they're cheap).
**Level 4:** Vector databases (FAISS internals, HNSW graph-based ANN search), RAG (chunking strategies, retrieval + reranking, why naive RAG fails on multi-hop questions), Evaluation (faithfulness, relevance metrics), Guardrails, Hallucination causes and mitigations, Model serving (vLLM's continuous batching + paged attention).

**Anchor deep-dive — KV Cache**
- *Definition:* Cached Key/Value projections from previous tokens so each new token generation doesn't recompute attention over the whole sequence from scratch.
- *Why it exists:* Autoregressive generation is O(n²) without caching; KV cache makes each new token O(n) by reusing prior computation.
- *Common mistake:* Ignoring memory cost — cache size grows linearly with context length × layers × heads, which is why long-context serving is memory-bound, not compute-bound.
- *Interview Q:* "Why does GPU memory, not compute, usually limit max concurrent requests for LLM serving?"
- *Mini exercise:* Implement a minimal from-scratch attention function in NumPy, then add manual KV caching and benchmark token generation speed with vs without it.

## Projects

### Mini — Tokenizer + Attention Visualizer
- **Objectives:** De-mystify tokenization and attention with a tiny, inspectable example.
- **Features:** BPE tokenizer walkthrough on sample text, NumPy attention computation on a short sequence, heatmap visualization of attention weights.
- **Folder Structure:** `attn_viz/{tokenizer.py, attention.py, visualize.py}`
- **Technologies:** NumPy, `matplotlib`.
- **Learning Outcomes:** Attention math is no longer a black box.
- **Expected Time:** 6–8 hrs.

### Intermediate — RAG Pipeline From Scratch
- **Objectives:** Understand every stage of RAG by building it without a framework, before ever using LangChain/LlamaIndex.
- **Features:** Document chunking (with overlap), embedding generation, FAISS index build, top-k retrieval, prompt assembly, generation call; basic evaluation (does retrieved chunk actually contain the answer?).
- **Folder Structure:** `rag_from_scratch/{chunker.py, embed.py, index.py, retrieve.py, generate.py, eval.py}`
- **Technologies:** FAISS, an embeddings API or local model, an LLM API.
- **Learning Outcomes:** Full RAG pipeline internals — the exact thing frameworks wrap.
- **Expected Time:** 14–16 hrs.

### Advanced — LoRA Fine-Tune + Evaluation
- **Objectives:** Go from "using models" to "adapting models."
- **Features:** Fine-tune a small open model (e.g., a 1–3B parameter model) with LoRA on a custom instruction dataset; before/after evaluation on held-out prompts.
- **Folder Structure:** `lora_finetune/{prepare_data.py, train.py, evaluate.py, configs/}`
- **Technologies:** Hugging Face `transformers` + `peft`, a small open-weight model.
- **Learning Outcomes:** Parameter-efficient fine-tuning end-to-end, including why it's cheap.
- **Expected Time:** 16–20 hrs.

## Checklist
- [ ] Explained attention mechanism without notes
- [ ] Built RAG with raw FAISS + OpenAI/local embeddings, no framework
- [ ] Ran a LoRA fine-tune end-to-end

---

# PHASE 9 — Agentic AI
**Duration:** 2 weeks (56 hrs) | **Weeks 17–18**

## Phase Objective
This is your stated end goal. Everything before this (async Python, backend APIs, distributed systems, LLM internals) is prerequisite machinery an agent runtime is built from. This phase builds agents from primitives first, frameworks second — so frameworks become "convenient wrappers I understand," not magic.

## Learning Outcomes
Build a working multi-agent system with tool calling, memory, and human-in-the-loop from raw API calls before touching any framework; then evaluate 6 frameworks with informed technical opinions.

## Concepts
**Core architecture:** Agent loop (perceive → plan → act → observe), Planning (single-shot vs iterative/ReAct), Reflection (self-critique loops), Memory (short-term/context vs long-term/vector store vs episodic), Tool calling / Function calling (schema generation, execution, result injection), Structured outputs (JSON mode, Pydantic-validated responses), MCP (Model Context Protocol — standardized tool/resource exposure), A2A protocol (agent-to-agent communication standard), Multi-agent patterns: Supervisor (central router delegates), Router (classify-then-dispatch), Planner-Executor (split thinking from doing), Human-in-the-loop (approval gates), Retries & self-correction (structured error feedback to the model), Streaming (token + tool-call streaming to UI), Observability (tracing agent steps, tool latency, token cost per run).

**Anchor deep-dive — Tool/Function Calling Internals**
- *Definition:* The model outputs a structured request (function name + JSON args) instead of free text; your code executes it and feeds the result back in.
- *Why it exists:* LLMs can't access live data or take actions natively — tool calling is the bridge to the outside world.
- *Internal working:* You send a tool schema (name, description, JSON schema for params) alongside the prompt; the model is trained to emit a matching structured call when appropriate; your runtime parses it, executes, appends a "tool result" message, and calls the model again.
- *Common mistake:* Trusting model-generated arguments without validation (e.g., letting it construct a raw SQL string); not handling the case where the model hallucinates a non-existent tool.
- *Interview Q:* "Walk through what happens, end-to-end, when an agent decides to call a tool."
- *Mini exercise:* Implement tool calling from raw API calls (no framework): define 2 tools, parse the model's structured request yourself, execute, and loop the result back in.

## Framework Comparison
| Framework | Why it exists | Architecture | Strengths | Weaknesses | Use when | Avoid when |
|---|---|---|---|---|---|---|
| **LangGraph** | Graph-based control flow for agents needing explicit state machines | Nodes/edges over a shared state object | Fine-grained control, great for complex branching workflows, good observability via LangSmith | Steeper learning curve, verbose for simple agents | Multi-step workflows with conditional branching/loops | Simple single-tool chatbots |
| **CrewAI** | Role-based multi-agent collaboration, quick to prototype | Agents with roles/goals + Tasks + Crew orchestrator | Fast to stand up multi-agent "teams," intuitive mental model | Less control over exact execution order, can be opaque under the hood | Rapid prototyping of collaborative agent teams | Production systems needing precise control |
| **PydanticAI** | Type-safe agent framework from the Pydantic team | Agent wraps a model + tools with Pydantic-validated I/O | Strong typing/validation, lightweight, plays well with FastAPI | Newer, smaller ecosystem | You want strict schema validation and FastAPI-native feel | Need a huge pre-built tool/integration ecosystem |
| **AutoGen** | Multi-agent conversation framework from Microsoft | Agents converse with each other in a chat loop | Good for research-style multi-agent conversation patterns | Conversation-centric abstraction can feel indirect for pure task pipelines | Research/simulation of agent dialogue | Simple deterministic pipelines |
| **LlamaIndex** | Data framework for connecting LLMs to your data (RAG-first) | Indices/retrievers/query engines, agents built on top | Best-in-class data connectors and indexing strategies | Agent story is secondary to its RAG/indexing focus | RAG-heavy applications | Pure agent orchestration without much data retrieval |
| **Agno** (formerly Phidata) | Lightweight, fast agent framework with built-in memory/knowledge | Simple Agent object with tools/memory/knowledge built-in | Fast to build, batteries-included (memory, storage, UI) | Smaller community, less battle-tested at scale | Quick, memory-equipped agents without heavy setup | Highly custom control-flow needs |

## Projects

### Mini — Raw Tool-Calling Agent (No Framework)
- **Objectives:** Prove you understand the tool-calling loop before any framework hides it.
- **Features:** Single agent with 3 tools (e.g., calculator, web search, file reader) built directly against the model API; manual parse-execute-reinject loop; basic retry on malformed tool calls.
- **Folder Structure:** `raw_agent/{agent.py, tools.py, loop.py}`
- **Technologies:** Anthropic/OpenAI API directly, no agent framework.
- **Learning Outcomes:** The tool-calling loop internalized, not abstracted away.
- **Expected Time:** 8–10 hrs.

### Intermediate — Multi-Agent Research Assistant (LangGraph)
- **Objectives:** Apply the planner-executor pattern with real branching control flow.
- **Features:** Planner agent breaks down a research question → researcher agent uses a web/RAG tool → writer agent drafts the answer → human-approval gate before finalizing.
- **Folder Structure:** `research_assistant/{graph.py, nodes/, tools/, state.py}`
- **Technologies:** LangGraph, a search or RAG tool, an LLM API.
- **Learning Outcomes:** Graph-based agent orchestration, human-in-the-loop gating.
- **Expected Time:** 14–16 hrs.

### Advanced — Multi-Agent Customer Support System
- **Objectives:** Combine every agentic pattern into one realistic production-shaped system.
- **Features:** Router agent classifies incoming tickets → specialist agents (billing/technical/general) each with scoped tools exposed via MCP → long-term memory via a vector store → full tracing of every agent step and tool call.
- **Folder Structure:** `support_agents/{router.py, agents/, mcp_tools/, memory/, tracing.py}`
- **Technologies:** Chosen framework (LangGraph/PydanticAI), MCP, a vector DB, an observability tool.
- **Learning Outcomes:** End-to-end multi-agent architecture with memory, tool governance, and observability — this feeds directly into the Phase 12 capstone.
- **Expected Time:** 20–24 hrs.

## Checklist
- [ ] Built tool calling from raw API calls before using any framework
- [ ] Implemented all 4 orchestration patterns (supervisor/router/planner-executor/human-in-loop) at least once
- [ ] Can articulate framework tradeoffs unprompted in an interview setting

---

# PHASE 10 — Production AI Systems
**Duration:** 1 week (28 hrs) | **Week 19**

## Phase Objective
Bridges "I can build an agent" to "I can ship one that survives real traffic, cost constraints, and failure modes."

## Learning Outcomes
Add evaluation, cost control, and security hardening to an existing agent system, and explain each production tradeoff you made.

## Concepts
Cost/latency optimization (caching, batching, model routing — cheap model for simple tasks, strong model for hard ones) · Evaluation pipelines for agents (golden datasets, LLM-as-judge, regression testing on prompts) · Guardrails in production (input/output filtering, PII redaction) · Observability (full trace of agent runs, token cost dashboards) · Security (prompt injection defenses, tool permission scoping, sandboxing code execution tools) · Scaling agent workloads (async fan-out, queue-based agent job processing) · Versioning prompts and agent configs like code.

## Projects

### Mini — Agent Evaluation Suite
- **Objectives:** Learn to measure agent quality instead of eyeballing it.
- **Features:** 20-case golden dataset for the Phase 9 support agent, automated scoring via LLM-as-judge, regression check that fails CI if quality drops.
- **Folder Structure:** `agent_eval/{golden_set.json, judge.py, run_eval.py}`
- **Technologies:** An LLM API for judging, pytest for the regression gate.
- **Learning Outcomes:** Concrete, repeatable agent quality measurement.
- **Expected Time:** 6–8 hrs.

### Intermediate — Cost & Security Hardening
- **Objectives:** Make the agent system safe and affordable to run.
- **Features:** Model routing (cheap model for simple classification, strong model for complex reasoning), per-run cost tracking dashboard, a prompt-injection test suite with at least 10 attack patterns and documented mitigations.
- **Folder Structure:** `agent_hardening/{router.py, cost_tracker.py, injection_tests/}`
- **Technologies:** Same stack as Phase 9 advanced project + a cost-logging store.
- **Learning Outcomes:** Practical security and cost governance for LLM systems.
- **Expected Time:** 8–10 hrs.

### Advanced — Productionized Multi-Agent System
- **Objectives:** Take the Phase 9 advanced project all the way to "would survive real traffic."
- **Features:** Everything above combined, plus a queue-based scaling layer (Celery/Redis) so concurrent agent runs don't block each other, and prompt/config versioning so changes are auditable like code.
- **Folder Structure:** extends `support_agents/{..., queue.py, versioning/}`
- **Technologies:** Celery, Redis, existing agent stack.
- **Learning Outcomes:** The full "agent → production system" transition, directly feeding the Phase 12 capstone.
- **Expected Time:** 12–14 hrs.

## Checklist
- [ ] Built an eval suite with at least 20 golden test cases
- [ ] Demonstrated a prompt-injection attempt and its mitigation
- [ ] Added per-run cost tracking

---

# PHASE 11 — Open Source Reading
**Duration:** 1 week dedicated + ongoing throughout | **Week 20**

## Phase Objective
Reading production-grade code is how you graduate from "can build toy projects" to "can operate at senior engineer level." This phase is a skill (how to read a codebase), not just a reading list.

## Learning Outcomes
Read an unfamiliar production codebase and, within a few hours, explain its entry point, one full request path, and the architectural pattern behind its core abstraction.

## How to Read a Repo (method, apply to every repo below)
1. Read `README` + architecture docs first, not code.
2. Find the entry point (CLI, `main.py`, or top-level `__init__.py`).
3. Trace one real request/call end-to-end before reading anything else.
4. Ignore test folders, migration folders, and vendored/generated code on first pass.
5. Ask: "What problem does this abstraction solve that a simpler version couldn't?"

## Recommended Repositories
| Repo | Read first | Ignore initially | Key lesson |
|---|---|---|---|
| **FastAPI** | `routing.py`, `dependencies/` | `openapi/` generation internals | How dependency injection + ASGI integrate cleanly |
| **Pydantic** | `main.py` (`BaseModel`), `fields.py` | Rust core (`pydantic-core`) internals | How validation + descriptors + metaclasses combine |
| **SQLAlchemy** | `orm/session.py`, `engine/base.py` | Dialect-specific SQL compilers | Unit of Work pattern, identity map |
| **LiteLLM** | `main.py`, `router.py` | Provider-specific adapters | Unified interface pattern over heterogeneous APIs |
| **LangGraph** | `graph/state.py`, `graph/graph.py` | Prebuilt agent templates | State machines as a first-class agent abstraction |
| **MCP SDK** | `server/` and `client/` core classes | Transport-layer edge cases | Protocol design for tool/resource exposure |
| **CPython** | `Objects/dictobject.c` (conceptually, or the Python-level explanation) | Full compiler/parser internals | How the interpreter actually executes what you write |
| **Uvicorn** | `protocols/http/`, `server.py` | Windows-specific event loop code | What an ASGI server actually does per request |

## Projects

### Mini — Annotated Trace of FastAPI
- **Objectives:** Practice the repo-reading method on a codebase you already use daily.
- **Features:** A written, annotated trace of one full request through FastAPI's source (route match → dependency resolution → response), with your own comments at each step.
- **Folder Structure:** `oss_reading/fastapi_trace.md`
- **Technologies:** N/A — reading + writing exercise.
- **Learning Outcomes:** Confidence reading framework internals instead of treating them as magic.
- **Expected Time:** 4–5 hrs.

### Intermediate — Pattern Comparison Report
- **Objectives:** Recognize that the same design patterns recur across "different" tools.
- **Features:** A short report identifying one shared pattern (e.g., Unit of Work, Registry, Builder) across at least 3 of the recommended repos, with code excerpts explained in your own words (no verbatim reproduction).
- **Folder Structure:** `oss_reading/pattern_report.md`
- **Technologies:** N/A.
- **Learning Outcomes:** Pattern recognition that transfers to codebases you've never seen.
- **Expected Time:** 6–8 hrs.

### Advanced — Small Contribution or Extension
- **Objectives:** Prove comprehension by producing something, not just reading.
- **Features:** Either (a) submit a small real PR (docs fix, small bug fix, or test addition) to one of the recommended repos, or (b) build a small plugin/extension against one of them (e.g., a custom FastAPI middleware, a custom SQLAlchemy type, or a LangGraph custom node) that exercises its extension points.
- **Folder Structure:** `oss_reading/contribution/` or a fork link.
- **Technologies:** Matches the chosen repo's stack.
- **Learning Outcomes:** The final proof that you can operate inside a production codebase, not just read about it.
- **Expected Time:** 8–12 hrs.

## Checklist
- [ ] Traced one full request end-to-end in FastAPI's source
- [ ] Explained SQLAlchemy's Unit of Work pattern in your own words
- [ ] Identified one design pattern reused across ≥3 of these repos

---

# PHASE 12 — Capstone Project
**Duration:** 2 weeks (56 hrs) | **Weeks 21–22**

## Phase Objective
Synthesize all 11 phases into one enterprise-scale system you can fully explain, deploy, and defend in an interview.

## Learning Outcomes
Design, build, deploy, and defend one enterprise-scale system that integrates backend engineering, data layer, distributed systems, DevOps, and agentic AI — with the ability to justify every decision unaided in an interview setting.

## Capstone: "Enterprise AI Support & Knowledge Platform"
A multi-tenant SaaS platform combining a production backend with a multi-agent RAG-powered support system.

**Architecture (described):**
- **Client** → **API Gateway/Nginx** → **FastAPI Backend Service** (auth, tenants, tickets) and **Agent Orchestration Service** (separate microservice)
- **Auth:** JWT + refresh tokens, RBAC per tenant
- **Databases:** PostgreSQL (relational: users/tenants/tickets), Redis (cache + session + rate limiting), Vector DB/FAISS or pgvector (knowledge base embeddings)
- **Queues:** Celery + Redis/RabbitMQ for async ticket processing and agent job dispatch
- **Agentic layer:** Router agent classifies incoming ticket → specialist agents (billing/technical/general) each with scoped tools via MCP → RAG tool over tenant-specific knowledge base → human-in-the-loop escalation for low-confidence responses
- **Observability:** OpenTelemetry traces across API + agent calls, Prometheus metrics, Grafana dashboards, per-agent-run cost tracking
- **Security:** Prompt-injection filtering on all agent inputs, tool permission scoping per tenant, PII redaction in logs
- **Deployment:** Dockerized services, Docker Compose for local dev, Kubernetes manifests for staging/prod, GitHub Actions CI/CD (lint → test → build → deploy)
- **Testing:** Unit tests (pytest), integration tests (API + DB), agent evaluation suite (golden dataset + LLM-as-judge)

**Folder structure (top-level):**
```
capstone/
  backend/        # FastAPI service: auth, tenants, tickets
  agent-service/  # agent orchestration microservice
  shared/         # shared schemas, auth utils
  infra/          # Dockerfiles, k8s manifests, nginx conf
  .github/workflows/
  tests/
```

**Expected implementation timeline:** Week 21 = backend + DB + auth + queues; Week 22 = agent layer + RAG + observability + deployment + eval suite.

## Final Checklist
- [ ] End-to-end request traceable from client to agent tool call and back
- [ ] Can defend every architectural decision (why Postgres+Redis+vector DB, why this agent pattern, why this queue) unaided
- [ ] Deployed locally via Docker Compose with green CI

---

# DSA ROADMAP (Parallel Track — all 22 weeks, ~45 min/day within the 4-hr budget)
| Topic | Problems | Difficulty progression | Pattern tip | Weekly goal |
|---|---|---|---|---|
| Arrays | 25 | Easy→Medium→Hard | Two pointers, prefix sums | 5–6/week (Wks 1–4) |
| Strings | 20 | Easy→Medium | Sliding window, hashing | alongside arrays |
| Sliding Window | 15 | Medium | Fixed vs variable window | Wk 3–4 |
| Linked Lists | 15 | Easy→Medium | Fast/slow pointers | Wk 5 |
| Stacks/Queues | 15 | Easy→Medium | Monotonic stack | Wk 6 |
| Binary Search | 15 | Medium | Search space reduction | Wk 7 |
| Trees | 25 | Easy→Hard | DFS/BFS traversal templates | Wk 8–9 |
| BST | 12 | Medium | In-order = sorted | Wk 9 |
| Heap | 15 | Medium | Top-K pattern | Wk 10 |
| Trie | 8 | Medium | Prefix matching | Wk 11 |
| Graphs (DFS/BFS) | 25 | Medium→Hard | Union-Find, topological sort | Wk 12–14 |
| Greedy | 15 | Medium | Prove exchange argument | Wk 15 |
| Backtracking | 15 | Medium→Hard | State-space tree pruning | Wk 16 |
| Dynamic Programming | 35 | Medium→Hard | 1D→2D→interval→tree DP progression | Wk 17–20 |
| Bit Manipulation | 10 | Easy→Medium | XOR tricks | Wk 21 |
| Mixed review/mocks | 20 | Mixed | Timed contest simulation | Wk 22 |

**Total: ~270 problems over 22 weeks (~2/day average).** Recommended order: Arrays/Strings → Sliding Window → Linked List → Stack/Queue → Binary Search → Trees → Heap → Trie → Graphs → Greedy → Backtracking → DP → Bit Manipulation → Mixed mocks.

---

# WEEKLY SCHEDULE TEMPLATE (4 hrs/day, working-professional realistic)

| Day | Hour 1 | Hour 2 | Hour 3 | Hour 4 (split into 45+45+30 or similar) |
|---|---|---|---|---|
| Mon | Current Phase — new concept study | Current Phase — hands-on coding | DSA (2 problems) | 30 min revision of yesterday |
| Tue | Current Phase — new concept study | Current Phase — hands-on coding | DSA (2 problems) | 30 min revision |
| Wed | Current Phase — new concept study | Current Phase — hands-on coding | DSA (2 problems) | 30 min revision |
| Thu | Current Phase — new concept study | Current Phase — hands-on coding | DSA (2 problems) | 30 min revision |
| Fri | Current Phase — new concept study | Current Phase — hands-on coding | DSA (2 problems) | 30 min revision |
| Sat | Project work (mini/intermediate/advanced, whichever is active) — full 3 hrs | | | DSA mock contest (1 hr) |
| Sun | Weekly review: redo checklist, mock interview Qs from the week (1.5 hrs) | Project work continued (1.5 hrs) | | Light DSA revision, plan next week (1 hr) |

**Notes:**
- Every 4th week (end of a phase), replace one weekday's DSA slot with the Phase's Project + Checklist completion.
- Keep a running "concepts I couldn't explain out loud" log — revisit weekly, not just at phase end.

---

# RESOURCE LIBRARY

### Phase 1 — Python
- **Docs:** docs.python.org (official, always primary)
- **Books:** *Fluent Python* (Ramalho) — the single best deep-dive; *Effective Python* (Slatkin)
- **YouTube:** ArjanCodes (design patterns/pythonic code), mCoding (internals)
- **Repo:** CPython source for curious deep-dives

### Phase 2 — Networking
- **Book:** *Computer Networking: A Top-Down Approach* (Kurose & Ross)
- **Docs:** MDN Web Docs (HTTP, CORS, WebSockets sections)
- **Course:** freeCodeCamp networking crash course (YouTube)

### Phase 3 — Backend
- **Docs:** fastapi.tiangolo.com (exceptionally good official docs), Pydantic docs
- **YouTube:** ArjanCodes FastAPI series
- **Repo:** tiangolo/full-stack-fastapi-template

### Phase 4 — Databases
- **Book:** *Designing Data-Intensive Applications* (Kleppmann) — essential, re-read across phases 4–5
- **Docs:** postgresql.org/docs, SQLAlchemy official docs
- **Site:** use-the-index-luke.com (indexing deep dive)

### Phase 5 — Distributed Systems
- **Book:** *Designing Data-Intensive Applications* (continued)
- **Papers:** Google's "MapReduce," Kafka's original LinkedIn paper, Amazon's "Dynamo" paper
- **Docs:** kafka.apache.org/documentation, celeryproject.org

### Phase 6 — DevOps
- **Docs:** docs.docker.com, kubernetes.io/docs (concepts section only for now)
- **Course:** KodeKloud Docker/K8s basics

### Phase 7 — System Design
- **Book:** *System Design Interview Vol 1 & 2* (Alex Xu)
- **Repo:** donnemartin/system-design-primer
- **YouTube:** Gaurav Sen, ByteByteGo

### Phase 8 — LLM Fundamentals
- **Paper:** "Attention Is All You Need" (original transformer paper)
- **Blog:** Jay Alammar's "The Illustrated Transformer"
- **Docs:** platform.openai.com/docs, huggingface.co/docs
- **Repo:** karpathy/nanoGPT (build a GPT from scratch — do this)

### Phase 9 — Agentic AI
- **Docs:** langchain-ai.github.io/langgraph, docs.crewai.com, ai.pydantic.dev, modelcontextprotocol.io
- **Papers:** "ReAct: Synergizing Reasoning and Acting in Language Models"
- **Repo:** each framework's official GitHub examples folder

### Phase 10–12
- **Docs:** OpenTelemetry docs, OWASP LLM Top 10 (prompt injection reference)
- **Blog:** Anthropic and OpenAI engineering blogs for production LLM system patterns

### DSA
- **Site:** NeetCode (patterns-first, matches your goal of understanding over memorization), LeetCode
- **Book:** *Elements of Programming Interviews in Python*

---

# PROGRESS TRACKING SYSTEM

Use this per phase (copy the block):

```
## Phase N Review
- [ ] All concept checklists complete
- [ ] Mini project done
- [ ] Intermediate project done
- [ ] Advanced project done
- [ ] Self-assessment: explained 3 anchor deep-dives out loud, unaided, in <2 min each
- [ ] Interview bank: answered 80%+ without notes
- [ ] DSA weekly goal met
- [ ] Logged remaining gaps → carried to next week's revision slot
```

**Milestone gates (don't advance until true):**
- End of Phase 1 → can build a decorator + descriptor + context manager from memory
- End of Phase 4 → can explain isolation levels with a live demo
- End of Phase 9 → can build tool-calling from raw API calls before using any framework
- End of Phase 12 → can whiteboard the capstone architecture in under 10 minutes

---

*This is a living document — update checklists weekly, and treat every "anchor deep-dive" pattern as the template to replicate for every remaining concept listed in table form. Consistency over 600 hours, not intensity in any single day, is what will close the gap between "AI wrote this and it worked" and "I designed this and know why it works."*
