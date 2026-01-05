# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Constitution for Claude Code working with Sage.ai.

## Identity

You are working on **Sage.ai** - an AI investment mentor platform.
- This is the **sage-docs** repository - Single Source of Truth for all documentation.
- Code repositories (frontend, backend, infra) reference these specs.
- Follow specs in `specs/` when implementing features.

## Multi-Repository Structure

Sage.ai is organized across 4 independent repositories:

### 1. sage-docs (THIS REPOSITORY)
- **Purpose**: Documentation and specifications
- **Repository**: `https://github.com/5010-dev/sage-docs`
- **Role**: Single Source of Truth

### 2. sage-frontend
- **Purpose**: React web application
- **Repository**: `https://github.com/5010-dev/sage-frontend`
- **Deployment**: AWS Amplify

### 3. sage-backend
- **Purpose**: Nest.js API server
- **Repository**: `https://github.com/5010-dev/sage-backend`
- **Deployment**: AWS ECS Fargate

### 4. sage-infra
- **Purpose**: Infrastructure as Code (Pulumi)
- **Repository**: `https://github.com/5010-dev/sage-infra`
- **Deployment**: Pulumi CLI

## Source of Truth (Priority Order)

When documents conflict, follow this hierarchy:

1. **This file (CLAUDE.md)** - Overrides everything
2. **specs/** - Technical specifications (backend, frontend, infrastructure)
3. **product/** - Product requirements
4. **business/** - Business context (informational, not prescriptive)

If you find contradictions, **ask the user** before proceeding.

## Repository Structure (sage-docs)

```
sage-docs/
├── specs/             # Technical specifications (AUTHORITATIVE)
│   ├── backend-spec.md
│   ├── frontend-spec.md
│   └── infrastructure-spec.md
├── architecture/      # System architecture documentation
│   └── system-overview.md
├── business/          # Business context (not prescriptive)
│   ├── mvp-definition.md
│   ├── business-plan.md
│   └── development-roadmap.md
├── product/           # Product requirements
│   └── product-spec.md
├── operations/        # GTM, campaigns
│   └── operations-roadmap.md
├── docs/              # Legacy docs (to be migrated)
│   ├── technical/
│   ├── ai-guides/
│   └── ...
├── wiki/              # GitHub Wiki sync (auto-generated)
├── CLAUDE.md          # This file
└── README.md          # Links to all repositories
```

**Note**: Some documents have Korean versions (suffix `-kr`). The English version is authoritative.

## Documentation Synchronization

Code repositories contain **local copies** of relevant specs for quick reference:
- `sage-frontend/docs/frontend-spec.md` (copied from `specs/frontend-spec.md`)
- `sage-backend/docs/backend-spec.md` (copied from `specs/backend-spec.md`)
- `sage-infra/docs/infrastructure-spec.md` (copied from `specs/infrastructure-spec.md`)

**When updating specs**:
1. Update in `sage-docs/specs/` (Source of Truth)
2. Copy to relevant code repository
3. Commit with message: "Sync [spec] from sage-docs@[commit]"

## Tech Stack (MUST Use Exactly)

| Layer | Technology |
|-------|------------|
| Backend | Nest.js 10.x, Prisma 5.x, TypeScript |
| Frontend | React 18.3, Vite 5, TypeScript, Zustand 4.x, TanStack Query 5.x |
| Database | PostgreSQL 18, Valkey 8.x |
| AI | Claude Sonnet 4, Claude Haiku 4 (@anthropic-ai/sdk) |
| Auth | Auth.js (@auth/core) with Google OAuth |
| Infra | ECS Fargate, S3/CloudFront, RDS, ElastiCache |
| Package Manager | pnpm |

## Terminology (MUST Use Correctly)

| Term | Definition | Wrong Usage |
|------|------------|-------------|
| **Persona** | Character + LLM combination | ❌ "multi-agent" for personas |
| **Agent** | Functional unit within ONE persona | ❌ Agent = Persona |
| **Agent Pipeline** | Manager → Analyst → Persona → Risk | ❌ "multi-agent system" |
| **Multi-Persona** | Multiple LLMs (Claude + GPT + Gemini) | ❌ Using in MVP context |

## MUST Do

- **Reference specs**: Always check `docs/technical/` before implementing
- **Preserve consistency**: When editing one doc, check related docs for conflicts
- **Use exact versions**: Nest.js 10.x, Prisma 5.x, PostgreSQL 18, Valkey 8.x
- **Keep performance numbers**: 2s first token, 0.5s context load, <1% hallucination
- **Follow existing patterns**: Check existing code before creating new patterns

## MUST NOT Do

- **Don't invent features**: If not in `docs/product/product-spec.md`, it doesn't exist
- **Don't add RAG/vector DB**: MVP uses 20-message window only
- **Don't suggest EKS**: MVP uses ECS Fargate only
- **Don't mix personas in MVP**: Only Wallet Buffett (Claude) exists in MVP
- **Don't change performance targets**: They are hard requirements, not suggestions

## Key Documents (sage-docs)

| Purpose | Document |
|---------|----------|
| System Overview | `architecture/system-overview.md` |
| MVP Scope | `business/mvp-definition.md` |
| Features | `product/product-spec.md` |
| Backend Spec | `specs/backend-spec.md` |
| Frontend Spec | `specs/frontend-spec.md` |
| Infrastructure Spec | `specs/infrastructure-spec.md` |
| Roadmap | `business/development-roadmap.md` |

## Commands (in Code Repositories)

### Frontend (sage-frontend)
```bash
pnpm install          # Install dependencies
pnpm dev              # Run dev server
pnpm build            # Build for production
pnpm preview          # Preview production build
pnpm lint             # Lint code
pnpm test             # Run tests
```

### Backend (sage-backend)
```bash
pnpm install          # Install dependencies
pnpm dev              # Run dev server
pnpm build            # Build for production
pnpm start:prod       # Start production server
pnpm lint             # Lint code
pnpm test             # Run tests
pnpm prisma:migrate   # Run database migrations
pnpm prisma:generate  # Generate Prisma Client
```

### Infrastructure (sage-infra)
```bash
pnpm install          # Install dependencies
pulumi preview        # Preview infrastructure changes
pulumi up             # Apply infrastructure changes
pulumi destroy        # Destroy infrastructure
```

## Quick Checks

Before making changes, verify:

- [ ] Does this align with MVP scope? (`business/mvp-definition.md`)
- [ ] Is this feature defined? (`product/product-spec.md`)
- [ ] Does the tech stack match? (see Tech Stack section above)
- [ ] Are performance numbers preserved?
- [ ] Did I follow existing patterns?
- [ ] Am I in the correct repository? (docs vs code)

---

**Last Updated**: 2026-01-05
**Migration**: Converted from monorepo to multi-repository structure
