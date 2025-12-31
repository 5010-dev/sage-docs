# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Constitution for Claude Code working with Sage.ai.

## Identity

You are working on **Sage.ai** - an AI investment mentor platform.
- This is a **monorepo** containing documentation, backend, frontend, and infrastructure.
- Follow specs in `docs/technical/` when implementing features.

## Source of Truth (Priority Order)

When documents conflict, follow this hierarchy:

1. **This file (CLAUDE.md)** - Overrides everything
2. **docs/technical/** - Backend, frontend, infrastructure specs
3. **docs/product/** - Product requirements
4. **docs/business/** - Business context (informational, not prescriptive)

If you find contradictions, **ask the user** before proceeding.

## Repository Structure

```
sage/
├── docs/              # Specifications (AUTHORITATIVE)
│   ├── business/      # Context only - not prescriptive
│   ├── technical/     # Backend, frontend, infrastructure specs
│   ├── product/       # Product requirements
│   ├── operations/    # GTM, campaigns
│   └── ai-guides/     # Claude Code patterns
├── wiki/              # GitHub Wiki sync (do not edit directly)
├── apps/
│   ├── backend/       # Nest.js 10.x + Prisma 5.x
│   └── frontend/      # React 18.3 + Vite 5
├── packages/          # Shared code (types, utils)
└── infra/             # IaC (Terraform/CDK)
```

**Note**: Some documents have Korean versions (suffix `-kr`). The English version is authoritative.

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

## Commands

```bash
# Install dependencies
pnpm install

# Development
pnpm dev              # Run all apps
pnpm dev:backend      # Run backend only
pnpm dev:frontend     # Run frontend only

# Build
pnpm build            # Build all
pnpm build:backend    # Build backend only
pnpm build:frontend   # Build frontend only

# Test
pnpm test             # Run all tests
pnpm test:backend     # Run backend tests
pnpm test:frontend    # Run frontend tests

# Lint
pnpm lint             # Lint all
```

## Key Documents

| Purpose | Document |
|---------|----------|
| MVP Scope | `docs/business/mvp-definition.md` |
| Features | `docs/product/product-spec.md` |
| Backend | `docs/technical/backend-spec.md` |
| Frontend | `docs/technical/frontend-spec.md` |
| Infrastructure | `docs/technical/infrastructure-spec.md` |
| Roadmap | `docs/business/development-roadmap.md` |

## Quick Checks

Before making changes, verify:

- [ ] Does this align with MVP scope? (`docs/business/mvp-definition.md`)
- [ ] Is this feature defined? (`docs/product/product-spec.md`)
- [ ] Does the tech stack match? (see Tech Stack section above)
- [ ] Are performance numbers preserved?
- [ ] Did I follow existing patterns?

---

**Last Updated**: 2025-12-31
