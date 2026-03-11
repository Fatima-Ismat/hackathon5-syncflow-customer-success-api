# SyncFlow Customer Success API

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=flat-square&logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-009688?style=flat-square&logo=fastapi)
![OpenAPI](https://img.shields.io/badge/OpenAPI-3.1-6BA539?style=flat-square&logo=openapi-initiative)
![Hackathon](https://img.shields.io/badge/Hackathon%205-Stage%202-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)

---

## Owner

**Ismat Fatima**

---

## Hackathon Stage

**Hackathon 5 — Stage 2 (Specialization Phase)**
NovaSync Technologies | SyncFlow Customer Success Digital FTE

---

## Project Overview

SyncFlow Customer Success API is a **production-ready FastAPI backend** for an AI-assisted customer support system. It functions as a **Digital Full-Time Employee (Digital FTE)** — an AI agent that autonomously receives, processes, and resolves inbound customer support requests across multiple channels.

The system handles the complete support lifecycle: customer identification, AI-generated responses via an integrated support agent, automatic ticket creation with SLA tracking, escalation workflows routing to specialist queues, and detailed analytics. It is designed to operate at scale alongside human agents, dramatically reducing ticket volume that requires human intervention.

Built on top of the Stage 1 prototype (fully preserved), Stage 2 wraps every Stage 1 capability in a robust HTTP API layer with CRM-style customer tracking, structured ticketing, multi-channel message normalization, and agent performance metrics.

---

## Business Goal

The **Customer Success Digital FTE** is designed to solve the core tension facing SaaS support teams: **ticket volume grows faster than hiring**. As a Digital FTE, this system:

- Handles the full lifecycle of inbound support interactions autonomously
- Maintains consistent brand voice across email, WhatsApp, and web form channels
- Enforces SLA commitments automatically based on customer plan tier
- Routes sensitive issues (legal, billing, security, angry customers) to the right human specialist queue
- Frees human agents to focus on complex, high-value interactions
- Provides real-time analytics so support managers can monitor agent performance

The goal is zero-touch resolution for the majority of inbound tickets, with reliable, auditable escalation for every edge case that genuinely requires a human.

---

## Current Status

| Component | Status |
|-----------|--------|
| FastAPI backend server | Running |
| Swagger / OpenAPI docs | Available at `/docs` |
| `POST /support/submit` endpoint | Tested and working |
| Ticket lifecycle system | Implemented |
| CRM customer lookup | Implemented |
| Channel adapters (email, WhatsApp, web form) | Implemented |
| AI agent pipeline | Integrated via `agent_bridge.py` |
| Metrics and analytics endpoints | Implemented |
| Stage 2 backend architecture | Complete |
| Frontend (Next.js) | Available in `frontend/` |

---

## Core Features

- **AI Support Agent** — integrated via `agent_bridge.py`, processes natural language support messages, generates channel-appropriate responses, scores sentiment and confidence
- **Multi-Channel Support Ingestion** — normalized message pipeline for Email, WhatsApp (Meta Cloud API), and Web Form; each channel has its own adapter with `normalize()` and `send_reply()` methods
- **CRM Customer Lookup and Identity Tracking** — 3-strategy identification (direct ref → email lookup → phone/WhatsApp → guest profile), with plan, VIP status, and account health
- **Automatic Ticket Creation** — every support submission generates a structured ticket with reference ID, SLA deadline, priority, and tags
- **Ticket Status Management** — 5-state lifecycle (OPEN → IN_PROGRESS → WAITING_CUSTOMER | ESCALATED → RESOLVED) with validated state transitions
- **Ticket Reply Workflow** — human agents can add replies via `POST /tickets/{ticket_ref}/reply`, triggering status transitions
- **Ticket Escalation Workflow** — 17 escalation reason codes routing to 8 specialist queues (billing, legal, security, sales, enterprise CSM, senior support, technical support, CSM retention)
- **Sentiment Analysis** — anger, frustration, and urgency scoring integrated into priority assignment and escalation decisions
- **Confidence Scoring** — agent KB search returns a 0–1 confidence score; low confidence triggers escalation
- **SLA Tracking** — per-plan SLA matrix (Starter / Growth / Business / Enterprise × Critical / High / Medium / Low priority) with breach detection
- **Metrics and Analytics Endpoints** — ticket volume, response counts, escalation breakdown, channel distribution, auto-resolution rate, average confidence, average resolution time
- **Swagger / OpenAPI Documentation** — full interactive documentation at `/docs` with pre-filled examples for every endpoint
- **Production-Style Backend Architecture** — modular service layer (CRM, channels, workers), exception handlers, CORS middleware, Pydantic v2 request/response models

---

## Stage 2 Scope

Stage 2 represents the **Specialization Phase** — the transition from prototype thinking to production-style backend architecture.

**Scope delivered in Stage 2:**

- HTTP API layer (FastAPI) exposing all agent and CRM operations as REST endpoints
- CRM logic: customer records, identifier index, conversation threading
- Ticket system: full lifecycle, state machine, SLA matrix, escalation routing
- Agent workflow: Stage 1 agent integrated via bridge, priority computation, sentiment scoring
- Message processing: 9-stage pipeline orchestrator in `workers/message_worker.py`
- Support endpoints: 10 HTTP endpoints covering all support operations
- Metrics: event log pattern with time-windowed analytics queries

**Not in Stage 2 scope (extensible in a future stage):**
- Live database (PostgreSQL/Redis) — current implementation uses in-memory stores ready to swap for SQLAlchemy sessions
- Deployment / infrastructure scaling — Dockerfile, cloud deployment, load balancing
- Real external API integrations (Gmail API, Meta Graph API live credentials)

All Stage 1 artifacts (prototype agent, MCP server, context datasets, escalation rules, brand voice guide, prompt history, test cases) are **fully preserved and untouched**.

---

## Architecture Diagram

```
  Support Channels (Web Form / Email / WhatsApp)
                    |
                    v
           FastAPI Backend API
            (backend/main.py)
                    |
                    v
       Message Processing Pipeline
        (workers/message_worker.py)
                    |
         -----------+-----------
         |          |           |
         v          v           v
    AI Agent    CRM Layer   Metrics &
    (bridge)   (tickets,   Analytics
               customers)  (events)
         |          |
         v          v
    Escalation   Ticket
    Queue        Lifecycle
    Routing      + SLA
         |
         v
  Human Agent Queues
  (legal / billing / security /
   sales / enterprise / senior-support /
   technical / csm-retention)
```

---

## Project Structure

```
Hackathon5-Customer-Success-FTE-Stage2/
│
├── backend/                    FastAPI application layer
│   ├── main.py                 All HTTP endpoints, request/response models, CORS
│   └── agent_bridge.py         Stage 1 ↔ Stage 2 integration layer
│
├── crm/                        CRM service modules
│   ├── ticket_service.py       Ticket lifecycle, SLA matrix, state machine, escalation routing
│   ├── customer_service.py     Customer identification (3-strategy lookup), profile service
│   ├── knowledge_service.py    KB search, suggest_solution, rank_answers, confidence scoring
│   └── metrics_service.py      Event log, metrics summary, channel breakdown, sentiment distribution
│
├── channels/                   Channel adapter implementations
│   ├── email_channel.py        Email (Gmail-style) — normalize + send_reply
│   ├── whatsapp_channel.py     WhatsApp (Meta Cloud API) — normalize + send_reply
│   └── web_form_channel.py     Web form — structured field parsing + send_reply
│
├── workers/                    Background pipeline orchestration
│   └── message_worker.py       9-stage message processing pipeline
│
├── database/                   Data layer definitions
│   ├── models.py               SQLAlchemy ORM (7 tables: customers, tickets, conversations, etc.)
│   └── schema.sql              PostgreSQL DDL with seed data and indexes
│
├── frontend/                   Next.js frontend interface
│   ├── app/page.tsx            Web support form landing page
│   ├── app/ticket-status/      Ticket status lookup page
│   ├── app/admin/              Admin metrics dashboard
│   ├── components/             SupportForm, TicketCard, MetricsCard components
│   └── lib/api.ts              Typed API client (TypeScript)
│
├── src/                        Stage 1 prototype (preserved, untouched)
│   └── agent/
│       ├── customer_success_agent.py   Core Stage 1 agent with CLI demo
│       └── mcp_server.py               5 MCP tools with tool registry
│
├── context/                    Stage 1 domain knowledge (preserved)
│   ├── company-profile.md
│   ├── product-docs.md
│   ├── sample-tickets.json     55 customer tickets across 3 channels
│   ├── escalation-rules.md
│   └── brand-voice.md
│
├── specs/                      Stage 1 design specifications (preserved)
│   ├── agent-skills.md
│   ├── customer-success-fte-spec.md
│   ├── discovery-log.md
│   └── prompt-history.md       10-iteration prompt journal
│
├── tests/
│   ├── test_cases.md           Stage 1: 25 edge case scenarios
│   └── api_tests.md            Stage 2: 34 integration test scenarios
│
├── requirements.txt
└── README.md
```

---

## API Endpoints Summary

### Base URL
```
http://127.0.0.1:8000
```

### Interactive Docs
```
http://127.0.0.1:8000/docs
```

---

### System

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Health check — returns service status, version, timestamp |

---

### Support

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/support/submit` | **Primary endpoint.** Submit inbound support message, run full 9-stage AI pipeline, return ticket ref + AI response |

**Request body:**
```json
{
  "channel": "web_form",
  "customer_ref": "C-1042",
  "message": "I cannot reset my password — the reset link is not arriving."
}
```

**Response includes:** `ticket_ref`, `response`, `should_escalate`, `priority`, `sentiment`, `agent_confidence`, `kb_section`, `sla_deadline`

---

### Tickets

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/tickets` | List all tickets — filter by `status`, `priority`, `customer` |
| `GET` | `/tickets/{ticket_ref}` | Get full ticket record by reference ID |
| `POST` | `/tickets/{ticket_ref}/reply` | Add a human agent reply, transitions ticket to IN_PROGRESS |
| `POST` | `/tickets/{ticket_ref}/escalate` | Manually escalate ticket to specialist queue |

---

### Customers

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/customers/{customer_ref}` | Get customer profile, plan, health score, CSAT, recent ticket history |

---

### Metrics

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/metrics/summary` | Agent performance summary — volume, quality, escalation breakdown |
| `GET` | `/metrics/channels` | Per-channel ticket volume, response rate, confidence scores |
| `GET` | `/metrics/sentiment` | Sentiment distribution (positive / neutral / frustrated / angry) |

All metrics endpoints accept an `hours` query parameter (1–168) for time-window filtering.

---

## CRM / Ticketing Design

The CRM backend manages the full support lifecycle:

**Customer Records** — each customer has a `customer_ref`, plan tier (Starter / Growth / Business / Enterprise), VIP flag, account health score, CSAT average, and MRR. New customers are created as guest profiles on first contact.

**Customer Identification** — 3-strategy lookup: direct `customer_ref` match → email address extraction → phone/WhatsApp number normalization. Unknown contacts are provisioned as guest profiles automatically.

**Ticket References** — format `TKT-YYYYMMDD-XXXX`, unique per ticket, used for all subsequent operations (reply, escalate, status lookup).

**Ticket State Machine:**
```
OPEN → IN_PROGRESS → WAITING_CUSTOMER → RESOLVED
                   → ESCALATED ────────→ RESOLVED
```

**SLA Matrix (hours until breach):**

| Plan | Critical | High | Medium | Low |
|------|----------|------|--------|-----|
| Starter | 4h | 12h | 24h | 48h |
| Growth | 2h | 6h | 12h | 24h |
| Business | 1h | 2h | 4h | 8h |
| Enterprise | 30m | 1h | 2h | 4h |

**Escalation Queue Routing:**

| Reason Code | Queue |
|-------------|-------|
| `legal_threat` | legal-team |
| `security_incident` | security-team |
| `refund_request` | billing-team |
| `pricing_negotiation` | sales-team |
| `enterprise_renewal` | enterprise-csm |
| `cancellation_churn_risk` | csm-retention |
| `high_anger_score`, `profanity` | senior-support |
| `low_kb_confidence` | technical-support |

---

## Metrics and Analytics

The metrics layer records every pipeline event to an in-memory event log and exposes time-windowed queries:

- **Ticket Statistics** — tickets created, AI responses generated, escalations triggered, resolutions completed, SLA breaches
- **Channel Usage Distribution** — breakdown of ticket volume and response metrics by channel (email / whatsapp / web_form)
- **Sentiment Analysis** — distribution of positive / neutral / frustrated / angry sentiment across all inbound messages in the time window; useful for detecting CSAT trends before scores are collected
- **Confidence Scores** — average agent KB search confidence, KB usage rate (percentage of tickets where KB answer was found)
- **SLA / Processing Metrics** — average resolution time, auto-resolution rate, escalation rate

---

## How to Demo This Project

**Step-by-step demo for judges:**

1. **Start the backend** — run `uvicorn backend.main:app --reload --port 8000`
2. **Open Swagger** — navigate to `http://127.0.0.1:8000/docs`
3. **Submit a support message** — `POST /support/submit` with `{"channel": "web_form", "customer_ref": "C-1042", "message": "I cannot reset my password"}`; note the `ticket_ref` in the response
4. **View all tickets** — `GET /tickets` to see the ticket list
5. **View ticket details** — `GET /tickets/{ticket_ref}` using the reference from step 3
6. **View customer info** — `GET /customers/C-1042` to see the customer profile and ticket history
7. **View metrics** — `GET /metrics/summary` for agent performance stats, `GET /metrics/channels` for channel breakdown

**Optional:** Submit an escalation-triggering message:
```json
{"channel": "email", "customer_ref": "C-1042", "message": "I need a full refund immediately or I will take legal action."}
```
Then check the ticket — it will be escalated to `legal-team` with `critical` priority.

---

## Run Instructions

**Requirements:** Python 3.9+, Node.js 18+ (frontend only)

### Backend

```bash
# 1. Install Python dependencies
pip install -r requirements.txt

# 2. Start FastAPI server
uvicorn backend.main:app --reload --port 8000

# 3. Open Swagger interactive docs
# http://127.0.0.1:8000/docs
```

### Frontend (optional)

```bash
cd frontend
npm install
npm run dev
# http://localhost:3000
```

### Stage 1 Agent (still works standalone)

```bash
python src/agent/customer_success_agent.py
python src/agent/mcp_server.py
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend Framework | Python 3.9+ / FastAPI 0.110+ |
| Data Validation | Pydantic v2 |
| API Documentation | Swagger / OpenAPI 3.1 (auto-generated) |
| Database Schema | SQLAlchemy 2.0 ORM + PostgreSQL DDL |
| Channel Adapters | Custom Python adapters (Email, WhatsApp, Web Form) |
| AI Agent | Claude Agent SDK (Stage 1) via `agent_bridge.py` |
| Pipeline Orchestration | Custom 9-stage message worker |
| Frontend | Next.js 14 (App Router) + TypeScript + Tailwind CSS |
| CORS | FastAPI `CORSMiddleware` |

---

## Stage 2 Completion Checklist

| Component | Status | File |
|-----------|--------|------|
| FastAPI backend | Complete | `backend/main.py` |
| Agent bridge (Stage 1 integration) | Complete | `backend/agent_bridge.py` |
| CRM database schema (SQLAlchemy) | Complete | `database/models.py` |
| PostgreSQL schema + seed data | Complete | `database/schema.sql` |
| Ticket lifecycle service | Complete | `crm/ticket_service.py` |
| Customer identification service | Complete | `crm/customer_service.py` |
| Knowledge base search service | Complete | `crm/knowledge_service.py` |
| Metrics & analytics service | Complete | `crm/metrics_service.py` |
| Email channel adapter | Complete | `channels/email_channel.py` |
| WhatsApp channel adapter | Complete | `channels/whatsapp_channel.py` |
| Web form channel adapter | Complete | `channels/web_form_channel.py` |
| Message processing pipeline | Complete | `workers/message_worker.py` |
| Integration tests (34 scenarios) | Complete | `tests/api_tests.md` |
| Sentiment scoring | Complete | `backend/agent_bridge.py` |
| Automatic priority assignment | Complete | `crm/ticket_service.py` |
| SLA tracking + breach detection | Complete | `crm/ticket_service.py` |
| Ticket escalation routing | Complete | `crm/ticket_service.py` |
| Ticket tagging system | Complete | `workers/message_worker.py` |
| Agent confidence scoring | Complete | `backend/agent_bridge.py` |
| Frontend support form | Complete | `frontend/app/page.tsx` |
| Frontend ticket status page | Complete | `frontend/app/ticket-status/` |
| Frontend admin dashboard | Complete | `frontend/app/admin/` |

---

## Submission Note

This repository represents **Hackathon 5 — Stage 2 (Specialization Phase)**. It demonstrates the transition from the Stage 1 incubation/prototype phase into a **production-style backend architecture** — a complete, working HTTP API that any frontend, integration, or automation tool can call.

Every Stage 1 artifact (prototype agent, MCP server, 55 sample tickets, escalation rules, brand voice guide, 10-iteration prompt history, 25 edge case tests) is preserved unchanged. Stage 2 builds on top of that foundation rather than replacing it, demonstrating the full evolution from prototype to production.

**Owner:** Ismat Fatima
**Hackathon:** Hackathon 5 | Customer Success Digital FTE Track
**Stage:** Stage 2 — Specialization Phase | March 2026
**Built with:** Python 3.9+ | FastAPI | Pydantic v2 | SQLAlchemy | Next.js 14 | Claude Agent SDK
