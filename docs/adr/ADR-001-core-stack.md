# ADR-001: Core stack — Google ADK + Django + Cloud Run

- **Date:** 2026-07-09
- **Status:** Accepted

## Decision

BillAgent is built on:
- **Google ADK** for multi-agent orchestration (Extractor → Validator → Poster)
- **Django + Django REST Framework** for the API layer, persistence, and the human review queue (via Django admin)
- **Cloud Run** for deployment, with Cloud Build + Artifact Registry as the pipeline

## Context

BillAgent is a capstone/portfolio project targeting roles at Google Cloud Indonesia, GoTo, and Traveloka. The system needs: (1) orchestration of multiple LLM-backed agents with confidence-based escalation, (2) a human review queue with authentication and an admin UI, (3) a public, low-cost, low-maintenance deployment for a live demo.

The author's primary professional currently stacks are Node.js/Express and Laravel (PHP). Python is comfortable but not the primary daily stack.

Career alignment is an explicit, acknowledged factor: the target companies operate on Google Cloud, and demonstrating fluency in Google's agent tooling (ADK, Vertex AI, Cloud Run) is part of the project's purpose. We treat this as a legitimate architectural input, not a hidden bias.

## Alternatives considered

**Orchestration**
- **LangChain / LangGraph** — largest ecosystem and community examples. Rejected: heavier abstraction layers than needed for a 3-agent pipeline; weaker career signal for Google-ecosystem targets.
- **CrewAI** — fast to prototype role-based agents. Rejected: less control over escalation logic; smaller production footprint.
- **No framework (direct Vertex AI calls + own state machine)** — maximum control and learning. Rejected as primary approach: reinventing orchestration, retries, and tool-calling plumbing costs weeks that M1 doesn't have. Revisit if ADK's abstractions fight us.

**API/backend framework**
- **FastAPI** — the default choice for modern Python APIs; async-native, lighter. Rejected mainly because of one feature: **Django admin gives the human review queue for free** (auth, permissions, CRUD UI). Building that in FastAPI means custom UI work that adds no portfolio value. The review queue is core to the product (escalation is a first-class outcome, per ADR-002).
- **Express/NestJS (Node)** — the author's strongest stack. Rejected: ADK and the Google agent ecosystem are Python-first; mixing a Node API with Python agents means two runtimes, two deploy targets, more surface for a solo project.
- **Laravel (PHP)** — same reasoning as Node, plus weaker AI-tooling ecosystem.

**Deployment**
- **GKE** — overkill for a solo project; cluster cost and ops burden.
- **App Engine** — viable but legacy-flavored; Cloud Run is the current idiomatic serverless container platform and scales to zero (important for a demo that sits idle most of the time).
- **VPS (the author's prior habit)** — cheap and familiar, but no autoscaling story and weaker credibility for cloud-native targeting.

## Trade-offs accepted

- **Learning curve:** Django is not the author's primary stack; early velocity will be slower than Express/Laravel. Accepted — closing this gap is part of the project's goal.
- **ADK maturity:** ADK is younger than LangChain; fewer community answers when stuck. Mitigation: keep agent logic behind our own interfaces so a framework swap stays possible (see CLAUDE.md conventions).
- **Sync-first Django** vs async agent calls: agent invocations may be slow; we accept simple synchronous handling in M1 (with Cloud Run request timeouts) and defer task queues (Cloud Tasks/Celery) until measured latency demands it.
- **Vendor coupling:** the stack leans into Google Cloud deliberately. Portability is sacrificed knowingly; this is a portfolio project for a Google-ecosystem audience, not a vendor-neutral product.

## Outcome

*To be filled after Milestone 1: did Django admin actually save the review-queue build time? Did ADK's abstractions hold up for confidence-based escalation? Record real numbers (setup time, deploy pipeline reliability).*
