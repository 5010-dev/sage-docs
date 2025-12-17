# Architecture Overview

## High-Level Architecture

Sage.ai follows a **Hexagonal Architecture (Ports & Adapters)** pattern with complete backend/frontend separation.

```
┌─────────────────────────────────────────────────────────────┐
│                     React SPA Frontend                       │
│         (Vite 6 + React 19 + TypeScript)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Main App   │  │   Landing    │  │ Bitcoin      │      │
│  │              │  │    Page      │  │  Fallen      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │ REST API + SSE
                            │
┌─────────────────────────────────────────────────────────────┐
│                  Django Backend (Hexagonal)                  │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │              Presentation Layer                     │    │
│  │  (DRF ViewSets, Serializers, Django Channels)      │    │
│  └────────────────────────────────────────────────────┘    │
│                            ▲                                 │
│                            │                                 │
│  ┌────────────────────────────────────────────────────┐    │
│  │              Application Layer                      │    │
│  │  (Use Cases, Services, Business Logic)             │    │
│  └────────────────────────────────────────────────────┘    │
│                            ▲                                 │
│                            │                                 │
│  ┌────────────────────────────────────────────────────┐    │
│  │              Domain Layer                           │    │
│  │  (Entities, Value Objects, Ports)                  │    │
│  └────────────────────────────────────────────────────┘    │
│                            ▲                                 │
│                            │                                 │
│  ┌────────────────────────────────────────────────────┐    │
│  │              Infrastructure Layer                   │    │
│  │  (Adapters: DB, AI, External APIs, Cache)          │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌─────────────────────────────────────────────────────────────┐
│                    External Services                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Claude   │  │CoinGecko │  │ Discord  │  │  Redis   │   │
│  │   API    │  │   API    │  │ Webhook  │  │  Cache   │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Key Architectural Decisions

### 1. Complete Backend/Frontend Separation

**Why**:
- Independent scaling
- Technology flexibility
- Clear API boundaries
- Better development workflow

**Trade-offs**:
- Separate deployment pipelines
- Type synchronization via OpenAPI
- CORS configuration needed

### 2. Hexagonal Architecture

**Why**:
- Domain logic independence
- Easy testing (mock adapters)
- Technology replacement flexibility
- Clear separation of concerns

**Structure**:
```
domain/           # Pure Python, no dependencies
  entities/       # Business objects
  ports/          # Interfaces (ABC)
application/      # Use cases, orchestration
infrastructure/   # Adapters (Claude, DB, APIs)
presentation/     # DRF ViewSets, Channels
```

### 3. Django + DRF (not Next.js or FastAPI)

**Why Django**:
- Django Admin: 2x faster MVP development
- Django ORM: Mature, auto-migrations
- Django-Allauth: Google OAuth in 5 minutes
- Django Channels: SSE for Claude streaming
- Battery-included: 8-week timeline critical

**Trade-offs**:
- Type safety: Python vs TypeScript (mitigated with mypy + Pydantic)
- Async: Not pure async (but sufficient for MVP scale)

### 4. Celery Beat (not Lambda)

**Why**:
- Direct Django ORM access (no VPC setup)
- Unified codebase (no separate Lambda deploy)
- Cost-effective (no Lambda invocation fees)
- Better for 15-minute recurring tasks

**Usage**:
- 15-minute market analysis
- Automatic context generation
- Alert distribution

## Multi-Agent AI System

```
[User Question]
      ↓
[Manager Agent] ─── Intent classification
      ↓
[Analyst Agent] ─── Fetch facts (price, news)
      ↓
[Persona Agent] ─── Warren Buffett interpretation
      ↓
[Risk Agent] ────── Validate & fact-check
      ↓
[Final Response]
```

**Goal**: <1% hallucination rate through multi-agent cross-validation

## Data Flow

### Chat Flow
```
User Input → DRF API → Chat Service → Claude Multi-Agent → SSE Stream → React UI
                ↓
         Message Repository → PostgreSQL
                ↓
         User Profile Update (inferred from conversation)
```

### Shadow Portfolio Flow
```
Claude Response → Signal Extraction → Portfolio Service → Shadow Trade Repository
                                              ↓
                                        PostgreSQL + Redis Cache
                                              ↓
                                    ROI Calculation (daily cron)
```

### Proactive Analysis Flow
```
Celery Beat (15분) → Market Analysis Task → CoinGecko + CryptoPanic + Fear & Greed
                              ↓
                   Context Generation + Threshold Check
                              ↓
                   Discord Webhook + PWA Push (if alert needed)
```

## Security & Compliance

### Authentication
- Django-Allauth: Google OAuth 2.0
- Session-based auth: HttpOnly cookies
- CSRF protection: Django built-in

### Audit Log
- Separate `audit_log` table (append-only)
- Tracks: user actions, data changes, API calls
- Fields: table_name, record_id, action, old_values, new_values, user_id, timestamp

### API Security
- DRF permissions: IsAuthenticated
- Rate limiting: Django-ratelimit
- CORS: django-cors-headers (whitelist only)

## Performance Targets

| Metric | Target |
|--------|--------|
| Chat response (first token) | <2 seconds |
| Context load (20 messages) | <0.5 seconds |
| Proactive analysis interval | 15 minutes |
| Concurrent users (MVP) | 1,000 |
| Hallucination rate | <1% |

## Infrastructure

### Development Environment
- Django development server
- PostgreSQL local
- Redis local
- Hot reload for frontend (Vite)

### Production (AWS)
- **Backend**: ECS Fargate (Docker)
  - Django app container
  - Celery worker container
  - Celery Beat scheduler container
- **Database**: RDS PostgreSQL 16+ (db.t3.micro)
- **Cache**: ElastiCache Redis 7.x (cache.t3.micro)
- **Frontend**: S3 + CloudFront (3 SPAs)
- **CI/CD**: GitHub Actions

### Deployment Strategy
- **dev branch** → dev environment
- **main branch** → production environment
- No staging (rapid iteration for MVP)

---

**Last Updated**: 2025년 12월 17일
**Architecture Version**: 3.0 - Django + Hexagonal + Celery
