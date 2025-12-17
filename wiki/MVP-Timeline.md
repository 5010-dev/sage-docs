# MVP Timeline (8 Weeks)

## Overview

**Goal**: Launch beta with 3 core features in 8 weeks
- Chat with Wallet Buffett AI
- Shadow Portfolio tracking
- Proactive market analysis (15-minute intervals)

---

## Week 1-2: Foundation

### Infrastructure
- [ ] AWS account setup + Terraform modules
- [ ] ECS Fargate cluster (dev + prod)
- [ ] RDS PostgreSQL 16+ (db.t3.micro)
- [ ] ElastiCache Redis 7.x (cache.t3.micro)
- [ ] S3 buckets + CloudFront distributions

### Backend Foundation
- [ ] Django 5.1+ project setup
- [ ] Hexagonal architecture folder structure
- [ ] Django ORM models (User, Chat, Message, Profile)
- [ ] Django-Allauth Google OAuth
- [ ] Django Admin customization
- [ ] Alembic-style migrations

### Frontend Foundation
- [ ] Turborepo monorepo setup
- [ ] Vite 6 + React 19 projects (3 apps)
- [ ] Tailwind CSS 4 + shadcn/ui
- [ ] TanStack Query setup
- [ ] Zustand store setup

### Sites
- [ ] WhyBitcoinFallen.com (Vite + React)
- [ ] Sage.ai landing page

---

## Week 3-4: Core Chat

### AI Integration
- [ ] Anthropic Python SDK setup
- [ ] Multi-agent orchestration
  - [ ] Manager Agent (intent routing)
  - [ ] Analyst Agent (fact gathering)
  - [ ] Persona Agent (Buffett philosophy)
  - [ ] Risk Agent (validation)
- [ ] System prompts (Wallet Buffett persona)
- [ ] Tool definitions (market data, calculations)

### Backend Chat
- [ ] Chat service (business logic)
- [ ] Message repository (Django ORM)
- [ ] Django Channels setup (SSE streaming)
- [ ] Context management (20-message window)
- [ ] User profile inference (from conversation)

### External APIs
- [ ] CoinGecko adapter (price data)
- [ ] CryptoPanic adapter (news)
- [ ] Alternative.me adapter (Fear & Greed)
- [ ] Redis caching (5/10/30 min TTL)

### Frontend Chat
- [ ] Chat UI (message bubbles, input)
- [ ] SSE client (EventSource)
- [ ] Streaming message display
- [ ] Markdown rendering
- [ ] Code syntax highlighting

### Testing
- [ ] Unit tests (domain layer)
- [ ] Integration tests (API endpoints)
- [ ] Hallucination rate measurement

---

## Week 5-6: Shadow Portfolio

### Backend Portfolio
- [ ] Shadow trade repository
- [ ] Portfolio service (ROI calculation)
- [ ] Signal extraction from Claude responses
- [ ] Daily ROI cron (Celery Beat)
- [ ] Benchmark comparison (BTC, ETH)

### Frontend Portfolio
- [ ] Portfolio page
- [ ] Performance chart (Recharts)
- [ ] [담아보기] button in chat
- [ ] Trade history table
- [ ] ROI vs benchmark display

### Market Data Integration
- [ ] Real-time price updates (WebSocket or polling)
- [ ] News feed integration
- [ ] Fear & Greed index display

---

## Week 7-8: Proactive Analysis + Deployment

### Celery Setup
- [ ] Celery + Redis configuration
- [ ] Celery Beat scheduler setup
- [ ] Celery worker Dockerfile

### Market Analysis Task
- [ ] 15-minute market analysis task
- [ ] Context generation (volatility, sentiment, news)
- [ ] Alert threshold logic
- [ ] Discord webhook integration
- [ ] PWA push notification integration

### Deep Linking
- [ ] `/chat/new?context=market_alert` routing
- [ ] Pre-filled context in chat input
- [ ] Analytics tracking

### Deployment
- [ ] Docker multi-stage builds
- [ ] ECS task definitions (Django + Celery worker + Beat)
- [ ] GitHub Actions CI/CD
- [ ] Environment variables management (AWS Secrets Manager)
- [ ] Health checks
- [ ] Monitoring (CloudWatch)

### Testing & QA
- [ ] Integration tests (full flow)
- [ ] Performance tests (2-second response target)
- [ ] Security audit (OWASP top 10)
- [ ] Load testing (1,000 concurrent users)

### Beta Launch
- [ ] 10-20 beta testers recruited
- [ ] Feedback collection system
- [ ] Bug tracking (GitHub Issues)
- [ ] Documentation finalization

---

## Success Criteria (Week 8)

| Metric | Target |
|--------|--------|
| Beta testers | 10-20 users |
| WhyBitcoinFallen visitors | 1,000+/day |
| Sage.ai MAU | 500+ |
| Hallucination rate | <1% |
| Chat response time | <2 seconds |
| Shadow trades tracked | 10+ trades |
| System uptime | 99%+ |

---

## Post-MVP (Phase 2)

### Q1 2026
- Multi-chat support
- Multilingual (Korean, English, Japanese, Chinese)
- Advanced portfolio features (custom benchmarks)
- Community features (Discord integration)

### Q2 2026
- Paid subscription tiers (Pro $19.99, Premium $49.99)
- Advanced analytics
- Mobile app (React Native)
- API for developers

---

**Last Updated**: 2025년 12월 17일
**Timeline**: 8 weeks → Beta launch
