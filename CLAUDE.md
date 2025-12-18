# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sage.ai** is an AI-powered investment mentor platform for cryptocurrency investors. Unlike Project ORE (AR location-based game), Sage.ai focuses on providing personalized AI mentoring through conversational interface with Warren Buffett philosophy.

This repository (`sage-docs`) contains all technical specifications, business documentation, and AI development guides for the platform.

**Key Differentiators**:
- **Wallet Buffett AI Mentor**: Claude 3.5-based multi-agent system
- **Shadow Portfolio**: Transparent performance tracking (Target: ROI +18.5%, Hit Rate 72.3%)
- **Proactive Alerts**: Market monitoring with PWA Push + Discord notifications
- **Conversational Onboarding**: No surveys - profile inferred from natural conversation

## Repository Structure

This is a **documentation-only repository** with markdown files organized by purpose:

- `docs/business/` - Strategy documents, MVP definition, branding guide, market analysis
- `docs/technical/` - System architecture for backend (Django), frontend (React), infrastructure (AWS)
- `docs/product/` - Product specifications, user journey, persona design, UX guidelines
- `docs/operations/` - GTM strategy, viral campaign, community management, live operations
- `docs/ai-guides/` - AI-native development patterns for Claude Code/Cursor

## Key Architecture Decisions

The project follows an **AI-Native development approach** with **Clean Architecture Lite**:

### Backend Architecture (Django + DRF)

- **Framework**: Django 5.1 + Django REST Framework (완전 분리 아키텍처)
- **ORM**: Django ORM with PostgreSQL
- **Architecture Pattern**: Clean Architecture Lite (Service Layer + Domain + Infrastructure)
- **AI System**: Multi-agent orchestration with Claude 3.5 Sonnet/Haiku
  - **Manager Agent**: Sonnet 3.5 - Routes user intent
  - **Analyst Agent**: Haiku 3.5 - Fetches facts (price, news, sentiment)
  - **Persona Agent**: Sonnet 3.5 - Warren Buffett philosophy interpretation
  - **Risk Agent**: Sonnet 3.5 - Cross-validation, hallucination prevention
- **Context Management**: Simple 20-message window (no RAG for MVP)
- **Auth**: Django-Allauth with Google OAuth
- **Streaming**: Django Channels for SSE (Server-Sent Events)
- **Admin**: Django Admin for rapid data management (MVP accelerator)
- **Performance targets**: 2 seconds to first token (streaming), 0.5s context load

### Frontend Architecture (Complete Separation)

- **Build Tool**: Vite 5 (all 3 frontend apps)
- **Framework**: React 18.3 + TypeScript
- **Apps**: Main App + Landing + WhyBitcoinFallen (모두 SPA)
- **State Management**:
  - Server State: TanStack Query (caching, real-time sync)
  - UI State: Zustand (lightweight, simple)
- **API Client**: Auto-generated from Django OpenAPI (drf-spectacular)
- **Streaming**: EventSource (SSE) for Claude chat
- **PWA**: Service Worker for push notifications + deep linking
- **Performance targets**: <2s page load, 60 FPS UI

### Infrastructure Architecture

- **Environment strategy**: dev + prod (no staging)
- **Branch mapping**: dev branch → dev environment, main → prod
- **Backend**: ECS Fargate + Daphne (ASGI server for Django Channels)
- **Frontend**: S3 + CloudFront (static hosting for 3 SPAs)
- **Database**: PostgreSQL RDS (db.t3.micro for MVP)
- **Cache**: Redis ElastiCache (cache.t3.micro for MVP)
- **Serverless**: Lambda for market analysis (15-minute cron)
- **Progressive scaling**: ECS → 10K users, then consider EKS if needed
- **CI/CD**: GitHub Actions (separate pipelines for backend/frontend)

### Hallucination Prevention System

**Critical difference from typical LLM apps**: Zero tolerance for factual errors

1. **Tool Enforcement**: All numerical data MUST come from tools, not LLM generation
2. **Multi-Agent Cross-Validation**: Analyst (facts) → Persona (interpretation) → Risk (verify)
3. **Cached Responses**: Redis caching for identical questions (5-minute TTL for market data)
4. **Target**: <1% hallucination rate (measured by user reports)

## Common Development Commands

### Backend (Django)

```bash
# Development server
python manage.py runserver

# Database migrations
python manage.py makemigrations     # Generate migration files
python manage.py migrate            # Apply migrations
python manage.py showmigrations     # Show migration status

# Admin & User management
python manage.py createsuperuser    # Create admin account
python manage.py changepassword <username>

# Django shell
python manage.py shell              # Python shell with Django context
python manage.py shell_plus         # Enhanced shell (if django-extensions installed)

# Static files
python manage.py collectstatic      # Collect static files for production

# Testing
python manage.py test               # Run tests
pytest                              # If using pytest-django

# Django Channels (SSE/WebSocket)
daphne config.asgi:application      # Run ASGI server
```

### Frontend (Vite + React)

```bash
# Development
cd apps/frontend-app
pnpm dev                            # Start dev server

# Build
pnpm build                          # Production build
pnpm preview                        # Preview production build

# Type generation from Django OpenAPI
cd apps/backend
python manage.py spectacular --file schema.yml
cd ../frontend-app
npx openapi-typescript-codegen --input ../backend/schema.yml --output ./src/api
```

### Documentation Management

```bash
# Check all markdown links
markdown-link-check docs/**/*.md

# Validate markdown syntax
markdownlint docs/**/*.md

# Find TODO items
grep -r "TODO\|FIXME" docs/ --include="*.md"
```

## AI Development Context

When working on this documentation:

1. **Technical Accuracy**: All specifications are production-ready, not theoretical
2. **Implementation Guidance**: Code examples should be copy-pasteable
3. **Performance Focus**: Specific numbers (2s, 0.5s) are hard requirements
4. **No Genesis Features**: Unlike ORE, Sage.ai has no special founding community features
5. **Conversational Onboarding**: No upfront surveys - profile inferred from chat
6. **Context Simplicity**: 20-message window only (no RAG, no summarization for MVP)

## File Relationships

Key document dependencies to understand when editing:

- `docs/business/mvp-definition.md` → Drives all technical specs (8-week timeline)
- `docs/technical/backend-spec.md` → Core architecture reference
- `docs/product/product-spec.md` → Defines 3 core features (Chat, Portfolio, Alerts)
- `docs/ai-guides/backend-ai-guide.md` → Claude Code prompts for implementation
- `docs/business/development-roadmap.md` → 8-week sprint breakdown + 2026 quarterly goals

## Documentation Standards

When updating documentation:

1. **Version control**: Update version numbers and dates at document end
2. **Cross-references**: Use relative links: `[Backend Spec](../technical/backend-spec.md)`
3. **Code examples**: All TypeScript/React code must be production-ready
4. **Performance metrics**: Always include numbers (not "fast" but "2 seconds")
5. **No Genesis references**: Sage.ai is open to all, no founding member special treatment

## Project Status

- **Current Phase**: Phase 1 - Core 3 documents (WIKI_HOME, CLAUDE, backend-spec) ✅
- **MVP Timeline**: 8 weeks (Week 1-2: Infrastructure, Week 3-4: Chat, Week 5-6: Portfolio, Week 7-8: Alerts)
- **Target Metrics**: 500 MAU (MVP), 5K MAU (Q1 2026), 10K MAU (Q4 2026)
- **No Code**: This repository contains only documentation - implementation in `sage-platform` repository (to be created)

## External Dependencies

Documentation references these external systems:

### AI & Infrastructure
- **Claude API**: Anthropic Claude 3.5 Sonnet/Haiku (no other LLMs for MVP)
- **Anthropic Python SDK**: Direct integration for multi-agent orchestration
- **Django Channels**: SSE streaming for real-time Claude responses
- **AWS Infrastructure**: ECS Fargate, RDS PostgreSQL, ElastiCache Redis, Lambda, S3/CloudFront

### Market Data
- **CoinGecko API**: Price data for 6 cryptocurrencies (BTC, ETH, SOL, BNB, DOGE, XRP)
  - Rate limit: 50 calls/minute
  - Caching: Redis 5-minute TTL
- **CryptoPanic API**: News aggregation
  - Rate limit: 100 calls/day
  - Caching: Redis 10-minute TTL
- **Alternative.me**: Fear & Greed Index
  - Rate limit: Unlimited
  - Caching: Redis 30-minute TTL

### Auth & Notifications
- **Google OAuth**: Django-Allauth integration (no email/password for MVP)
- **Discord Webhooks**: #market-alerts channel for community notifications
- **Web Push API**: PWA push notifications (VAPID protocol)

## Important Notes

- This is purely documentation - no executable code in this repository
- All architectural decisions prioritize **rapid MVP development** with **clean architecture**:
  - Complete backend/frontend separation (Django + React SPA)
  - Clean Architecture Lite (Service Layer + Domain) for maintainability
  - Django Admin for 2x faster data management
  - 20-message context window (no RAG for MVP)
  - ECS Fargate (not EKS)
- Performance targets are **hard requirements**, not aspirational:
  - 2 seconds to first token
  - <1% hallucination rate
  - 0.5 seconds context load
- The **8-week MVP timeline** is achievable with Django's batteries-included approach:
  - Week 1-2: Django Admin accelerates backend setup
  - Week 3-4: Python AI ecosystem (Anthropic SDK) integration is straightforward
  - Week 5-6: Django ORM makes CRUD operations simple
  - Week 7-8: Django Channels for SSE, Lambda for market alerts
- **Trade-offs accepted for speed**:
  - Type safety: Python dynamic typing vs TypeScript (mitigated with mypy + Pydantic)
  - Deployment: Separate backend/frontend pipelines (ECS Fargate + S3/CloudFront)
  - Admin dependency: Django Admin lock-in (but 2x faster initial development)
- **Dual Hook GTM** is critical:
  - WhyBitcoinFallen.com must launch Week 2
  - Sage.ai landing must convert at 10%+

## Comparison with Project ORE

For developers familiar with ORE documentation structure:

| Aspect | Project ORE | Sage.ai |
|--------|-------------|---------|
| **Domain** | AR location-based game | AI investment mentor |
| **Backend** | Rust + Go microservices | Django + DRF (Python) |
| **Frontend** | Unity AR | React + Vite (SPA) |
| **Architecture** | Microservices (8 services) | Clean Lite monolith |
| **AI** | None (game logic) | Claude 3.5 multi-agent |
| **Timeline** | 16 weeks | 8 weeks |
| **Scale Target** | 100K users | 10K users |
| **Special Features** | Genesis 1000 | None (equal access) |
| **Complexity** | High (distributed) | Medium (monolith + hexagonal) |

**Philosophy**: Sage.ai prioritizes **rapid MVP** with **clean architecture foundation** for future scaling.

---

**Last Updated**: 2025년 12월 17일
**Version**: 2.1 - Django + Clean Lite
**Architecture**: Django + DRF + Clean Architecture Lite + Complete Separation
**Maintainer**: Sam (dev@5010.tech)
