# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sage.ai** is an AI-powered investment mentor platform for cryptocurrency investors. The platform provides personalized AI mentoring through conversational interface with Warren Buffett philosophy, powered by Claude Sonnet 4.

This repository (`sage-docs`) contains all technical specifications, business documentation, and AI development guides for the platform.

**Key Differentiators**:
- **Wallet Buffett AI Mentor**: Claude Sonnet 4-based multi-agent system
- **Shadow Portfolio**: Transparent performance tracking (Target: Benchmark +3%p, Hit Rate 72.3%)
- **Proactive Analysis**: 15-min automatic market analysis with PWA Push + Discord notifications
- **Conversational Onboarding**: No surveys - profile inferred from natural conversation

## Repository Structure

This is a **documentation-only repository** with markdown files organized by purpose:

- `docs/business/` - Business plan, MVP definition, AI team strategy, market analysis
- `docs/technical/` - System architecture for backend (Nest.js), frontend (React), infrastructure (AWS)
- `docs/operations/` - GTM strategy, viral campaign, community management, live operations
- `docs/ai-guides/` - AI-native development patterns for Claude Code

## Key Architecture Decisions

The project follows an **AI-Native development approach** with **Clean Architecture Lite**:

### Backend Architecture (Nest.js + Prisma)

- **Framework**: Nest.js 10.x (TypeScript native, modular structure)
- **ORM**: Prisma 5.x (type-safe, intuitive migrations)
- **Architecture Pattern**: Layered + Domain (Clean Lite)
- **AI System**: Multi-agent orchestration with Claude Sonnet 4/Haiku 4
  - **Manager Agent**: Haiku 4 - Routes user intent
  - **Analyst Agent**: Haiku 4 - Fetches facts (price, news, sentiment)
  - **Persona Agent**: Sonnet 4 - Warren Buffett philosophy interpretation
  - **Risk Agent**: Haiku 4 - Cross-validation, hallucination prevention
- **Context Management**: Simple 20-message window (no RAG for MVP)
- **Auth**: Auth.js (@auth/core) with Google OAuth
- **Streaming**: Nest.js built-in SSE (Server-Sent Events)
- **Async Jobs**: BullMQ 5.x with Valkey (Memory extraction, alert sending)
- **Scheduling**: @nestjs/schedule (15-minute market polling)
- **Performance targets**: 2 seconds to first token (streaming), 0.5s context load

### Frontend Architecture (Complete Separation)

- **Build Tool**: Vite 5 (all 3 frontend apps)
- **Framework**: React 18.3 + TypeScript
- **Apps**: Main App + Landing + WhyBitcoinFallen (all SPAs)
- **State Management**:
  - Server State: TanStack Query 5.x (caching, real-time sync)
  - UI State: Zustand 4.x (lightweight, simple)
- **Streaming**: EventSource (SSE) for Claude chat
- **PWA**: Service Worker for push notifications + deep linking
- **Performance targets**: <2s page load, 60 FPS UI

### Infrastructure Architecture

- **Environment strategy**: dev + prod (no staging)
- **Branch mapping**: dev branch → dev environment, main → prod
- **Backend**: ECS Fargate + Node.js (Nest.js)
- **Frontend**: S3 + CloudFront (static hosting for 3 SPAs)
- **Database**: PostgreSQL 18 RDS (db.t3.micro for MVP)
- **Cache**: Valkey 8.x ElastiCache (cache.t3.micro for MVP)
- **Serverless**: Lambda for market analysis (15-minute cron)
- **Progressive scaling**: ECS → 10K users, then consider EKS if needed
- **CI/CD**: GitHub Actions (separate pipelines for backend/frontend)
- **IaC**: Pulumi (TypeScript-based infrastructure as code)
- **Monitoring**: Sentry (error tracking) + CloudWatch (logs/metrics) + Discord alerts

### Hallucination Prevention System

**Critical difference from typical LLM apps**: Zero tolerance for factual errors

1. **Tool Enforcement**: All numerical data MUST come from tools, not LLM generation
2. **Multi-Agent Cross-Validation**: Analyst (facts) → Persona (interpretation) → Risk (verify)
3. **Cached Responses**: Valkey caching for identical questions (5-minute TTL for market data)
4. **Target**: <1% hallucination rate (measured by user reports)

## Common Development Commands

### Backend (Nest.js)

```bash
# Development server
pnpm run start:dev              # Start with watch mode
pnpm run start:debug            # Start in debug mode

# Build
pnpm run build                  # Production build
pnpm run start:prod             # Run production build

# Database (Prisma)
npx prisma generate             # Generate Prisma Client
npx prisma migrate dev          # Create and apply migration
npx prisma migrate deploy       # Apply migrations in production
npx prisma studio               # Open Prisma Studio (DB GUI)
npx prisma db seed              # Run seed script

# Testing
pnpm run test                   # Run unit tests
pnpm run test:watch             # Run tests in watch mode
pnpm run test:cov               # Generate coverage report
pnpm run test:e2e               # Run e2e tests

# Linting
pnpm run lint                   # Run ESLint
pnpm run format                 # Format with Prettier
```

### Frontend (Vite + React)

```bash
# Development
cd apps/frontend-app
pnpm dev                        # Start dev server

# Build
pnpm build                      # Production build
pnpm preview                    # Preview production build

# Type checking
pnpm type-check                 # Run TypeScript compiler check
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
4. **No Genesis Features**: Sage.ai has equal access for all users
5. **Conversational Onboarding**: No upfront surveys - profile inferred from chat
6. **Context Simplicity**: 20-message window only (no RAG, no summarization for MVP)

## File Relationships

Key document dependencies to understand when editing:

- `MVP_technical_paper.md` → Core technology stack reference
- `docs/business/mvp-definition.md` → Defines MVP scope (3-month timeline)
- `docs/business/business-plan.md` → Business model and GTM strategy
- `docs/business/branding-guide.md` → Brand identity and messaging
- `docs/technical/backend-spec.md` → Nest.js architecture reference
- `docs/product/product-spec.md` → Defines 3 core features (Chat, Portfolio, Alerts)
- `docs/ai-guides/backend-ai-guide.md` → Claude Code prompts for implementation

## Documentation Standards

When updating documentation:

1. **Version control**: Update version numbers and dates at document end
2. **Cross-references**: Use relative links: `[Backend Spec](../technical/backend-spec.md)`
3. **Code examples**: All TypeScript/React code must be production-ready
4. **Performance metrics**: Always include numbers (not "fast" but "2 seconds")
5. **Technology accuracy**: Use correct versions (Nest.js 10.x, Prisma 5.x, etc.)

## Project Status

- **Current Phase**: Documentation complete, ready for implementation
- **MVP Timeline**: 3 months (Month 1: Infrastructure, Month 2: Core features, Month 3: Polish & Launch)
- **Target Metrics**: 500 MAU (MVP), 5K MAU (Q1 2026), 10K MAU (Q4 2026)
- **No Code**: This repository contains only documentation - implementation in `sage-platform` repository (to be created)

## External Dependencies

Documentation references these external systems:

### AI & Infrastructure
- **Claude API**: Anthropic Claude Sonnet 4 / Haiku 4 (no other LLMs for MVP)
- **@anthropic-ai/sdk**: TypeScript SDK for direct integration
- **Nest.js SSE**: Built-in Server-Sent Events for real-time Claude responses
- **AWS Infrastructure**: ECS Fargate, RDS PostgreSQL 18, ElastiCache Valkey 8.x, Lambda, S3/CloudFront

### Market Data
- **CoinGecko API**: Price data for 6 cryptocurrencies (BTC, ETH, SOL, BNB, DOGE, XRP)
  - Rate limit: 50 calls/minute
  - Caching: Valkey 5-minute TTL
- **Alternative.me**: Fear & Greed Index
  - Rate limit: Unlimited
  - Caching: Valkey 30-minute TTL

### Auth & Notifications
- **Auth.js (@auth/core)**: Google OAuth integration
- **Discord Webhooks**: 3 channels for alerts (#errors, #performance, #business-metrics)
- **Web Push API**: PWA push notifications (VAPID protocol)

## Important Notes

- This is purely documentation - no executable code in this repository
- All architectural decisions prioritize **rapid MVP development** with **clean architecture**:
  - Complete backend/frontend separation (Nest.js + React SPA)
  - Layered + Domain architecture for maintainability
  - Prisma ORM for type-safe database access
  - 20-message context window (no RAG for MVP)
  - ECS Fargate (not EKS)
- Performance targets are **hard requirements**, not aspirational:
  - 2 seconds to first token
  - <1% hallucination rate
  - 0.5 seconds context load
- The **3-month MVP timeline** is achievable with TypeScript fullstack approach:
  - Month 1: Nest.js + Prisma setup is rapid with TypeScript
  - Month 2: @anthropic-ai/sdk integration is straightforward
  - Month 3: BullMQ for async jobs, @nestjs/schedule for cron
- **Trade-offs accepted for speed**:
  - Monorepo complexity: Considered Turborepo but keeping simple for MVP
  - Type safety: Full TypeScript stack (backend + frontend)
  - Deployment: Separate backend/frontend pipelines (ECS Fargate + S3/CloudFront)
- **Dual Hook GTM** is critical:
  - WhyBitcoinFallen.com must launch Month 1
  - Sage.ai landing must convert at 10%+

## Technology Stack Summary

### Core Stack

| Component | Technology | Version | Why |
|-----------|-----------|---------|-----|
| **Backend** | Nest.js | 10.x | TypeScript native, modular |
| **ORM** | Prisma | 5.x | Type-safe, intuitive |
| **Frontend** | React + Vite | 18.3 + 5.x | Fast HMR, modern |
| **State** | Zustand + TanStack Query | 4.x + 5.x | Simple + powerful |
| **AI** | Claude Sonnet 4 + Haiku 4 | - | Best reasoning + cost |
| **DB** | PostgreSQL | 18 | 5-year LTS, JSON 30% faster |
| **Cache** | Valkey | 8.x | Redis-compatible, OSS stable |
| **Queue** | BullMQ | 5.x | Reliable background jobs |

### Infrastructure

- **Container**: AWS ECS Fargate
- **Static**: S3 + CloudFront
- **IaC**: Pulumi (TypeScript)
- **Monitoring**: Sentry + CloudWatch + Discord

---

**Last Updated**: 2025년 12월 26일
**Version**: 5.0 - PostgreSQL 18 + Valkey + Pulumi Stack
**Architecture**: Layered + Domain (Clean Lite), TypeScript Fullstack
**Maintainer**: Sam (dev@5010.tech)
