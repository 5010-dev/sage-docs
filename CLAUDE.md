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
- `docs/technical/` - System architecture for backend (Next.js), frontend (React), infrastructure (AWS)
- `docs/product/` - Product specifications, user journey, persona design, UX guidelines
- `docs/operations/` - GTM strategy, viral campaign, community management, live operations
- `docs/ai-guides/` - AI-native development patterns for Claude Code/Cursor

## Key Architecture Decisions

The project follows an **AI-Native development approach** with these core principles:

### Backend Architecture

- **Framework**: Next.js 15 API Routes (not microservices - monolithic for MVP)
- **ORM**: Drizzle ORM for type-safe PostgreSQL access
- **AI System**: Multi-agent orchestration with Claude 3.5 Sonnet/Haiku
  - **Manager Agent**: Sonnet 3.5 - Routes user intent
  - **Analyst Agent**: Haiku 3.5 - Fetches facts (price, news, sentiment)
  - **Persona Agent**: Sonnet 3.5 - Warren Buffett philosophy interpretation
  - **Risk Agent**: Sonnet 3.5 - Cross-validation, hallucination prevention
- **Context Management**: Simple 20-message window (no RAG for MVP)
- **Auth**: Auth.js v5 with Google OAuth
- **Performance targets**: 2 seconds to first token (streaming), 0.5s context load

### Frontend Architecture

- **Main App**: Next.js 15 App Router + React 19 + Tailwind CSS 4
- **Sites**: Vite + React + TypeScript (WhyBitcoinFallen.com, Sage.ai landing)
- **State Management**: Zustand (lightweight, simple)
- **Streaming**: Vercel AI SDK for SSE chat streaming
- **PWA**: Service Worker for push notifications + deep linking
- **Performance targets**: <2s page load, 60 FPS UI

### Infrastructure Architecture

- **Environment strategy**: dev + prod (no staging - same as ORE)
- **Branch mapping**: dev branch → dev environment, main → prod
- **Compute**: ECS Fargate (not Kubernetes - simpler for MVP)
- **Database**: PostgreSQL RDS (db.t3.micro for MVP)
- **Cache**: Redis ElastiCache (cache.t3.micro for MVP)
- **Storage**: S3 + CloudFront for static sites
- **Serverless**: Lambda for market analysis (15-minute cron)
- **Progressive scaling**: ECS → 10K users, then consider EKS if needed

### Hallucination Prevention System

**Critical difference from typical LLM apps**: Zero tolerance for factual errors

1. **Tool Enforcement**: All numerical data MUST come from tools, not LLM generation
2. **Multi-Agent Cross-Validation**: Analyst (facts) → Persona (interpretation) → Risk (verify)
3. **Cached Responses**: Redis caching for identical questions (5-minute TTL for market data)
4. **Target**: <1% hallucination rate (measured by user reports)

## Common Development Commands

This is a documentation repository with no build system. Common operations:

### Documentation Management

```bash
# Check all markdown links
markdown-link-check docs/**/*.md

# Validate markdown syntax
markdownlint docs/**/*.md

# Find TODO items
grep -r "TODO\|FIXME" docs/ --include="*.md"
```

### Content Validation

```bash
# Search for broken internal links
grep -r "](docs/" docs/ --include="*.md"

# Check for outdated version references
grep -r "Version:" docs/ --include="*.md"

# Verify all code examples have language tags
grep -r '```$' docs/ --include="*.md"
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
- **AWS Infrastructure**: ECS Fargate, RDS PostgreSQL, ElastiCache Redis, Lambda, S3/CloudFront
- **Vercel AI SDK**: Streaming abstraction (not Vercel hosting - AWS only)

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
- **Google OAuth**: Auth.js integration (no email/password for MVP)
- **Discord Webhooks**: #market-alerts channel for community notifications
- **Web Push API**: PWA push notifications (VAPID protocol)

## Important Notes

- This is purely documentation - no executable code in this repository
- All architectural decisions prioritize **simplicity** over **scalability** for MVP
  - Monolithic Next.js (not microservices)
  - 20-message context window (no RAG)
  - ECS Fargate (not EKS)
- Performance targets are **hard requirements**, not aspirational:
  - 2 seconds to first token
  - <1% hallucination rate
  - 0.5 seconds context load
- The **8-week MVP timeline** is aggressive but achievable with Claude Code:
  - Week 1-2: 1 developer can set up full infrastructure
  - Week 3-4: Claude integration takes ~3 days with AI assistance
  - Week 5-6: Portfolio tracking is straightforward CRUD
  - Week 7-8: PWA push is well-documented, Lambda cron is simple
- **Dual Hook GTM** is critical:
  - WhyBitcoinFallen.com must launch Week 2
  - Sage.ai landing must convert at 10%+

## Comparison with Project ORE

For developers familiar with ORE documentation structure:

| Aspect | Project ORE | Sage.ai |
|--------|-------------|---------|
| **Domain** | AR location-based game | AI investment mentor |
| **Backend** | Rust + Go microservices | Next.js API Routes |
| **Frontend** | Unity AR | Next.js + Vite |
| **AI** | None (game logic) | Claude 3.5 multi-agent |
| **Timeline** | 16 weeks | 8 weeks |
| **Scale Target** | 100K users | 10K users |
| **Special Features** | Genesis 1000 | None (equal access) |
| **Complexity** | High (8 services) | Low (monolith) |

**Philosophy**: Sage.ai prioritizes **speed to market** and **AI quality** over **architectural elegance**.

---

**Last Updated**: 2025년 12월 17일
**Version**: 1.0
**Maintainer**: Sam (dev@5010.tech)
