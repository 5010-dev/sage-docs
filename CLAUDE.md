# CLAUDE.md

Constitution for Claude Code working with Sage.ai documentation.

## Identity

You are working on **Sage.ai** - an AI investment mentor platform.
- This is a **documentation-only** repository. There is no executable code.
- All specs are for a product **not yet built**.

## Source of Truth (Priority Order)

When documents conflict, follow this hierarchy:

1. **This file (CLAUDE.md)** - Overrides everything
2. **docs/technical/** - Backend, frontend, infrastructure specs
3. **docs/product/** - Product requirements
4. **docs/business/** - Business context (informational, not prescriptive)

If you find contradictions, **ask the user** before proceeding.

## Terminology (MUST Use Correctly)

| Term | Definition | Wrong Usage |
|------|------------|-------------|
| **Persona** | Character + LLM combination | ❌ "multi-agent" for personas |
| **Agent** | Functional unit within ONE persona | ❌ Agent = Persona |
| **Agent Pipeline** | Manager → Analyst → Persona → Risk | ❌ "multi-agent system" |
| **Multi-Persona** | Multiple LLMs (Claude + GPT + Gemini) | ❌ Using in MVP context |

## MUST Do

- **Reference specs**: Always check `docs/technical/` before answering architecture questions
- **Preserve consistency**: When editing one doc, check related docs for conflicts
- **Use exact versions**: Nest.js 10.x, Prisma 5.x, PostgreSQL 18, Valkey 8.x
- **Keep performance numbers**: 2s first token, 0.5s context load, <1% hallucination
- **Link, don't duplicate**: Point to spec docs instead of copying content

## MUST NOT Do

- **Don't invent features**: If not in `docs/product/product-spec.md`, it doesn't exist
- **Don't add RAG/vector DB**: MVP uses 20-message window only
- **Don't suggest EKS**: MVP uses ECS Fargate only
- **Don't mix personas in MVP**: Only Wallet Buffett (Claude) exists in MVP
- **Don't create code files**: This repo has documentation only
- **Don't change performance targets**: They are hard requirements, not suggestions

## Out of Scope (REFUSE These)

Refuse and explain why if asked to:

- Write implementation code (→ "This is a docs-only repo")
- Add features not in product spec (→ "Check docs/product/product-spec.md first")
- Use technologies not in tech stack (→ "See docs/technical/backend-spec.md Section 1")
- Discuss Phase 2+ features as if they're MVP (→ "That's Phase 2+, MVP scope is in docs/business/mvp-definition.md")

## When Uncertain

1. **Check the spec first**: Read the relevant doc before answering
2. **Ask if conflicting**: "I found X in doc A but Y in doc B. Which is correct?"
3. **Don't guess numbers**: If a metric isn't documented, say so

## Repository Structure

```
docs/
├── business/      # Context only - not prescriptive
├── technical/     # AUTHORITATIVE for implementation
├── product/       # AUTHORITATIVE for features
├── operations/    # GTM, campaigns
└── ai-guides/     # Claude Code patterns
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
- [ ] Does the tech stack match? (`docs/technical/backend-spec.md` Section 1)
- [ ] Are performance numbers preserved?
- [ ] Did I update related documents?

---

**Last Updated**: 2025-12-30
