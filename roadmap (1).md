# Roadmap — Python Backend & Agentic AI Mastery (5 Months / 22 Weeks / ~600 hrs)

> This is the quick-navigation companion to **Python-Backend-Agentic-AI-5Month-Curriculum.md**, which holds full concept breakdowns, deep-dives, project specs, and resources. Use this file weekly to know exactly where you are; use the main curriculum file for the actual study content.

---

## At a Glance

| Week(s) | Phase | Focus | Hrs |
|---|---|---|---|
| 1–4 | 1. Python Mastery | OOP, decorators, generators, descriptors, metaclasses, concurrency (GIL/async/threading/multiprocessing), testing/packaging | 112 |
| 5 | 2. Networking Fundamentals | OSI/TCP-IP, HTTP/TLS, REST/GraphQL/gRPC, WebSockets/SSE, JWT/OAuth2, LB/CDN/Nginx | 28 |
| 6–8 | 3. Backend Development | FastAPI, Pydantic, DI, middleware, auth, background tasks, WebSockets, testing | 84 |
| 9–10 | 4. Databases | SQL/Postgres, indexes, transactions, isolation levels, MVCC, SQLAlchemy/Alembic, Redis, Mongo | 56 |
| 11 | 5. Distributed Systems | CAP, Kafka/RabbitMQ/Celery, distributed locks, event-driven, sagas, idempotency | 28 |
| 12 | 6. DevOps | Docker/Compose, Kubernetes basics, CI/CD, Prometheus/Grafana/OpenTelemetry, cloud basics | 28 |
| 13–14 | 7. System Design | Scalability, caching, LB algorithms, 10 classic system case studies, trade-off framework | 56 |
| 15–16 | 8. LLM Fundamentals | Transformers/attention, tokenization, KV cache, sampling, fine-tuning/LoRA, RAG, vLLM | 56 |
| 17–18 | 9. Agentic AI | Agent loop, tool calling, memory, MCP/A2A, multi-agent patterns, 6-framework comparison | 56 |
| 19 | 10. Production AI Systems | Cost/latency optimization, eval pipelines, guardrails, security, scaling agents | 28 |
| 20 | 11. Open Source Reading | FastAPI/Pydantic/SQLAlchemy/LiteLLM/LangGraph/MCP SDK/CPython/Uvicorn | 28 |
| 21–22 | 12. Capstone Project | Enterprise AI Support & Knowledge Platform — full-stack + agentic synthesis | 56 |

**Total: ~600 hours over 22 weeks.** DSA runs in parallel every week (see below) and is not counted separately in the phase hours — it lives inside the daily 4-hour block.

---

## Phase-by-Phase Milestone Gates
*(Do not move to the next phase until these are true — full detail in the main curriculum file's Checklists.)*

- [ ] **End of Phase 1:** Can build a decorator, a descriptor, and a context manager from memory; can explain the GIL correctly unaided.
- [ ] **End of Phase 2:** Can draw the OSI model and justify WebSocket vs SSE vs polling for 3 different scenarios.
- [ ] **End of Phase 3:** Built and tested a FastAPI app with dependency overrides; implemented JWT auth from scratch.
- [ ] **End of Phase 4:** Reproduced all 3 SQL read anomalies manually; explained N+1 and fixed it.
- [ ] **End of Phase 5:** Explained CAP with 2 real DB examples; built one idempotent endpoint.
- [ ] **End of Phase 6:** Green CI pipeline on every push; one working Grafana dashboard.
- [ ] **End of Phase 7:** 3 full system design docs completed; can do back-of-envelope estimation from memory.
- [ ] **End of Phase 8:** Built RAG from raw FAISS (no framework); ran a LoRA fine-tune end-to-end.
- [ ] **End of Phase 9:** Built tool-calling from raw API calls before touching any framework; implemented all 4 orchestration patterns.
- [ ] **End of Phase 10:** Eval suite with 20+ golden cases; demonstrated a prompt-injection mitigation.
- [ ] **End of Phase 11:** Traced one full request end-to-end through FastAPI's source unaided.
- [ ] **End of Phase 12:** Can whiteboard the capstone architecture in under 10 minutes and defend every decision.

---

## DSA Parallel Track (runs Weeks 1–22 alongside every phase)

| Weeks | Topic | Problems | Weekly Goal |
|---|---|---|---|
| 1–4 | Arrays, Strings, Sliding Window | 60 | 5–6/week |
| 5 | Linked Lists | 15 | 3/day pace |
| 6 | Stacks/Queues | 15 | 3/day pace |
| 7 | Binary Search | 15 | 2–3/day |
| 8–9 | Trees, BST | 37 | 4–5/week |
| 10 | Heap | 15 | 2–3/day |
| 11 | Trie | 8 | light week |
| 12–14 | Graphs (DFS/BFS, Union-Find, topological sort) | 25 | 8–9/week |
| 15 | Greedy | 15 | 2–3/day |
| 16 | Backtracking | 15 | 2–3/day |
| 17–20 | Dynamic Programming (1D→2D→interval→tree) | 35 | 8–9/week |
| 21 | Bit Manipulation | 10 | light week |
| 22 | Mixed review + timed mock contests | 20 | final push |

**Total ≈ 270 problems.** Recommended order matches the table top to bottom — see the main curriculum file for pattern-recognition tips per topic.

---

## Weekly Rhythm (4 hrs/day template)

| Day | Block 1 (1 hr) | Block 2 (1 hr) | Block 3 (1 hr) | Block 4 (1 hr) |
|---|---|---|---|---|
| Mon–Fri | New concept study (current phase) | Hands-on coding (current phase) | DSA (2 problems) | 30 min revision + 30 min buffer |
| Sat | Project work (3 hrs straight) | | | DSA mock contest (1 hr) |
| Sun | Weekly review + mock interview Qs (1.5 hrs) | Project work continued (1.5 hrs) | | Light DSA revision + plan next week (1 hr) |

Every 4th week (phase boundary), swap one weekday's DSA slot for finishing that phase's Project + Checklist.

---

## Project Checkpoints (Mini → Intermediate → Advanced per phase)
*(Full specs — objectives, features, folder structure, tech stack, expected hours — are in the main curriculum file.)*

| Phase | Mini | Intermediate | Advanced |
|---|---|---|---|
| 1. Python | CLI expense tracker | Custom ORM-lite (descriptors/metaclasses) | Concurrent scraper: threading vs multiprocessing vs asyncio benchmark |
| 2. Networking | Raw TCP echo server | Hand-rolled HTTP/1.1 parser | Protocol comparison lab (polling vs SSE vs WebSockets) |
| 3. Backend | Todo REST API + JWT | Multi-tenant Blog API | Real-time chat backend (WS + Redis pub/sub) |
| 4. Databases | E-commerce schema (3NF) + window functions | Redis caching layer for Blog API | Sharded KV store prototype |
| 5. Distributed Systems | Distributed lock demo | Kafka vs RabbitMQ comparison lab | Order-processing saga (Celery) |
| 6. DevOps | Dockerize a single service | Full-stack Compose + CI | Observability stack (Prometheus/Grafana/OTel) |
| 7. System Design | Rate limiter service | URL shortener (fully built) | 3 full system design docs |
| 8. LLM Fundamentals | Tokenizer + attention visualizer | RAG pipeline from scratch | LoRA fine-tune + evaluation |
| 9. Agentic AI | Raw tool-calling agent (no framework) | Multi-agent research assistant (LangGraph) | Multi-agent customer support system (MCP + memory + tracing) |
| 10. Production AI | Agent evaluation suite | Cost & security hardening | Productionized multi-agent system (queue-scaled) |
| 11. Open Source Reading | Annotated FastAPI request trace | Cross-repo pattern comparison report | Small real contribution or extension to an OSS repo |
| 12. Capstone | — | — | Enterprise AI Support & Knowledge Platform (full system) |

---

## Quick Reference — Where to Go for What
- **Deep concept explanations, anchor deep-dives, interview banks, resources:** `Python-Backend-Agentic-AI-5Month-Curriculum.md`
- **"What week am I in, what's next, am I on track":** this file
- **Weekly self-check:** copy the Progress Tracking block from the main curriculum file into a personal log each Sunday.

---

*Print or pin this file. Update the milestone checkboxes as you clear each phase — it's your single-glance progress dashboard for all 5 months.*
