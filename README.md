# Sage.ai Documentation

> **Single Source of Truth** for all Sage.ai project documentation

This repository contains all authoritative documentation, specifications, and architecture guidelines for the Sage.ai platform - an AI-powered investment mentor that helps users make smarter financial decisions.

---

## Repository Overview

Sage.ai is organized across **4 independent repositories**:

### 1. **sage-docs** (THIS REPOSITORY)
- **Purpose**: Documentation and specifications
- **Repository**: [https://github.com/5010-dev/sage-docs](https://github.com/5010-dev/sage-docs)
- **Role**: Single Source of Truth for all specs and architecture docs

### 2. **sage-frontend**
- **Purpose**: React web application
- **Repository**: [https://github.com/5010-dev/sage-frontend](https://github.com/5010-dev/sage-frontend) *(to be created)*
- **Tech Stack**: React 18.3, Vite 5, TypeScript, Zustand, TanStack Query
- **Deployment**: AWS Amplify (automatic on push to `main`)

### 3. **sage-backend**
- **Purpose**: Nest.js API server
- **Repository**: [https://github.com/5010-dev/sage-backend](https://github.com/5010-dev/sage-backend) *(to be created)*
- **Tech Stack**: Nest.js 10.x, Prisma 5.x, PostgreSQL 18, Valkey 8.x
- **Deployment**: AWS ECS Fargate via GitHub Actions

### 4. **sage-infra**
- **Purpose**: Infrastructure as Code (Pulumi)
- **Repository**: [https://github.com/5010-dev/sage-infra](https://github.com/5010-dev/sage-infra) *(to be created)*
- **Tech Stack**: Pulumi 3.x (TypeScript)
- **Resources**: ECS, RDS, ElastiCache, ALB, Amplify

---

## Documentation Structure

```
sage-docs/
├── specs/                 # Technical specifications (AUTHORITATIVE)
│   ├── backend-spec.md           # Backend API, Agent Pipeline, SSE streaming
│   ├── frontend-spec.md          # React components, SSE client, state management
│   └── infrastructure-spec.md    # Pulumi, ECS, RDS, ElastiCache, Amplify
│
├── architecture/          # System architecture
│   └── system-overview.md        # Multi-repo structure, data flow, tech stack
│
├── business/              # Business context (informational, not prescriptive)
│   ├── mvp-definition.md         # MVP scope, timeline, success criteria
│   ├── business-plan.md          # Market analysis, revenue model
│   ├── development-roadmap.md    # Phase 1-4 feature timeline
│   ├── executive-brief.md        # Executive summary
│   ├── ai-team-strategy.md       # AI team composition
│   ├── branding-guide.md         # Brand identity
│   └── sage-overview-kr.md       # Overview (Korean)
│
├── product/               # Product requirements
│   └── product-spec.md           # Features, user journeys, Agent Pipeline
│
├── operations/            # GTM and operations
│   ├── operations-roadmap.md     # Operations timeline
│   ├── genesis-operations.md     # Launch operations
│   ├── live-ops-playbook.md      # Live operations guide
│   ├── mystery-campaign.md       # Mystery campaign strategy
│   └── discord-management.md     # Discord community management
│
├── docs/                  # Legacy documentation (to be migrated)
│   ├── technical/                # Old specs (use specs/ instead)
│   └── ai-guides/                # Claude Code development guides
│
├── wiki/                  # GitHub Wiki sync (auto-generated, do not edit)
│
├── CLAUDE.md              # AI development constitution
├── README.md              # This file
└── README-kr.md           # Korean version
```

---

## Quick Start

### For Developers

1. **Read the specs first**:
   - Backend: [`specs/backend-spec.md`](specs/backend-spec.md)
   - Frontend: [`specs/frontend-spec.md`](specs/frontend-spec.md)
   - Infrastructure: [`specs/infrastructure-spec.md`](specs/infrastructure-spec.md)

2. **Understand the architecture**:
   - [`architecture/system-overview.md`](architecture/system-overview.md)

3. **Check MVP scope**:
   - [`business/mvp-definition.md`](business/mvp-definition.md)
   - [`product/product-spec.md`](product/product-spec.md)

### For AI Assistants (Claude Code)

- Read [`CLAUDE.md`](CLAUDE.md) first - it's the constitution for AI development
- Always reference `specs/` when implementing features
- Follow terminology guidelines (Agent Pipeline, not "multi-agent system")
- Preserve performance targets: <2s first token, <0.5s context load

---

## Tech Stack Summary

### Frontend
- **Framework**: React 18.3 + Vite 5
- **State**: Zustand 4.x (client), TanStack Query 5.x (server)
- **Styling**: Tailwind CSS 3.x + shadcn/ui
- **Real-time**: SSE (Server-Sent Events) for AI streaming

### Backend
- **Framework**: Nest.js 10.x
- **Database**: PostgreSQL 18 (via Prisma 5.x)
- **Cache**: Valkey 8.x (Redis-compatible)
- **AI**: Claude Sonnet 4 + Haiku 4 (@anthropic-ai/sdk)
- **Auth**: Auth.js with Google OAuth

### Infrastructure
- **IaC**: Pulumi 3.x (TypeScript)
- **Compute**: AWS ECS Fargate
- **Database**: AWS RDS PostgreSQL 18
- **Cache**: AWS ElastiCache Valkey 8.x
- **CDN**: AWS Amplify (frontend), CloudFront
- **LB**: AWS ALB

---

## Key Features (MVP)

### 1. AI Chat with Warren Buffett Persona
- **Agent Pipeline**: Manager → Analyst → Persona → Risk
- **Streaming**: SSE for token-by-token response (<2s first token)
- **Context**: 20-message window (no RAG in MVP)

### 2. Shadow Portfolio
- **Virtual Trading**: No real money, practice-only
- **AI Signals**: Extracted from chat conversations
- **Performance**: Historical performance tracking

### 3. Proactive Analysis
- **Market Alerts**: BTC -5%, ETH -7% triggers notification
- **Deep Links**: Push notification → Auto-open chat with context
- **Multi-channel**: PWA push + Discord webhook

---

## Documentation Synchronization

Code repositories contain **local copies** of relevant specs:

```bash
# Frontend repo
sage-frontend/docs/frontend-spec.md  # Copied from specs/frontend-spec.md

# Backend repo
sage-backend/docs/backend-spec.md    # Copied from specs/backend-spec.md

# Infrastructure repo
sage-infra/docs/infrastructure-spec.md  # Copied from specs/infrastructure-spec.md
```

**When updating specs**:
1. Edit in `sage-docs/specs/` (Source of Truth)
2. Copy to relevant code repository
3. Commit with message: `Sync [spec] from sage-docs@[commit-hash]`

**Example**:
```bash
# Update frontend spec in sage-docs
cd sage-docs
git commit specs/frontend-spec.md -m "Update SSE reconnection logic"
git push

# Sync to frontend repo
cd ../sage-frontend
cp ../sage-docs/specs/frontend-spec.md docs/
git commit docs/frontend-spec.md -m "Sync frontend-spec from sage-docs@abc123"
git push
```

---

## Performance Targets (MUST NOT CHANGE)

These are **hard requirements**, not suggestions:

| Metric | Target | Measurement |
|--------|--------|-------------|
| SSE First Token | < 2s | Time from POST to first SSE token |
| Context Load | < 0.5s | Fetch 20 messages from PostgreSQL |
| API Response | < 200ms (P95) | REST endpoint latency |
| Cache Hit Rate | > 80% | Valkey hit/miss ratio |
| Hallucination Rate | < 1% | Risk Agent rejection rate |

---

## Terminology Guide

Use these terms **correctly** in all documentation and code:

| Correct Term | Definition | Wrong Usage |
|--------------|------------|-------------|
| **Agent Pipeline** | Manager → Analyst → Persona → Risk sequence | ❌ "multi-agent system" |
| **Agent** | Functional unit within ONE persona (Manager, Analyst, etc.) | ❌ Agent = Persona |
| **Persona** | Character + LLM combination (Warren Buffett + Claude) | ❌ "agent" when referring to persona |
| **SSE Streaming** | Server-Sent Events for AI response | ❌ "WebSocket" (not used in MVP) |
| **Shadow Portfolio** | Virtual/practice portfolio | ❌ "paper trading", "simulated" |

---

## Development Workflow

### Phase 1: Planning & Documentation (Current)
- ✅ Tech stack finalized
- ✅ Specs written (backend, frontend, infrastructure)
- ✅ Repository split planned
- 🔄 Amplify migration planned
- 🔄 SSE vs WebSocket decision: **Keep SSE**

### Phase 2: Infrastructure Setup (Next)
- Create sage-infra repository
- Define Pulumi resources (ECS, RDS, ElastiCache)
- Set up AWS Amplify app via Pulumi

### Phase 3: Backend Implementation
- Create sage-backend repository
- Implement Agent Pipeline
- Set up SSE streaming endpoints
- Connect to Claude API

### Phase 4: Frontend Implementation
- Create sage-frontend repository
- Implement SSE client
- Build chat interface
- Deploy to Amplify

---

## Contributing

### For Documentation Updates

1. **Specs** (`specs/`): Create PR with clear description
2. **Architecture** (`architecture/`): Discuss with team first
3. **Business/Product** (`business/`, `product/`): Requires product owner approval
4. **Operations** (`operations/`): Requires operations lead approval

### For Code Changes

Go to the relevant code repository:
- Frontend changes → [sage-frontend](https://github.com/5010-dev/sage-frontend)
- Backend changes → [sage-backend](https://github.com/5010-dev/sage-backend)
- Infrastructure changes → [sage-infra](https://github.com/5010-dev/sage-infra)

---

## FAQs

### Why split into multiple repositories?

1. **Independent deployment cycles**: Frontend, backend, infra deploy at different rates
2. **Team autonomy**: Each team can work independently
3. **Clearer boundaries**: Docs vs code separation
4. **Simplified CI/CD**: Each repo has focused pipelines

### Why not use Git submodules?

- **Complexity**: Submodules add overhead (init, update, nested commits)
- **Independence**: Each repo should work standalone
- **Simplicity**: Copy + README link is simpler for our team size

### Why Amplify instead of S3+CloudFront?

- **Automatic deployment**: Git push → auto build (no GitHub Actions needed)
- **PR previews**: Auto preview URL for every PR
- **Simplicity**: Less manual configuration
- **MVP speed**: Faster time to market

### Will we use WebSocket later?

Possibly in **Phase 2+** if we add:
- Real-time collaborative features (group chat, shared portfolio)
- Live market data streaming to UI (ticker)
- User interrupt during AI response

For MVP, **SSE is sufficient** and simpler.

---

## Contact & Support

- **GitHub Issues**: [sage-docs/issues](https://github.com/5010-dev/sage-docs/issues)
- **Discord**: *(to be added)*
- **Email**: *(to be added)*

---

## License

*(To be determined)*

---

**Last Updated**: 2026-01-05
**Migration Status**: Converted from monorepo to multi-repository structure
