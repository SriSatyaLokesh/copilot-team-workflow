---
applyTo: "**"
---
# Team Copilot Workflow — Developer Guide

> Full guide: [docs/team-copilot-guide.md](../../docs/team-copilot-guide.md)
> Docs structure: [docs/docs-architecture.md](../../docs/docs-architecture.md)
> Research decisions: [docs/RESEARCH-DECISIONS.md](../../docs/RESEARCH-DECISIONS.md)
> **API architecture**: [docs/external-apis/](../../docs/external-apis/)

## The 5-Phase Issue Workflow

Every work item (feature, fix, task, story) is an **Issue** going through 5 phases:

| Phase | Command | Agent | What it does |
|-------|---------|-------|-------------|
| 1. Discuss | `/discuss` | Discuss | Defines requirements, creates Issue doc |
| 2. Research | Auto after Discuss | Research | Finds codebase context, reads external API docs |
| 3. Plan | `/plan` | Planner | Creates implementation task list |
| 4. Execute | `/execute` | TDD | Writes tests first, then code |
| 5. Verify | `/verify` | Verify | Checks completeness before PR |

## Specialist Agents

| Work Type | Agent | Command | When |
|-----------|-------|---------|------|
| Requirements | Discuss | `/discuss` | Phase 1 |
| Codebase research | Research | auto after Discuss | Phase 2 |
| Implementation plan | Planner | `/plan` | Phase 3 |
| Code + Tests | TDD | `/execute` | Phase 4 |
| Code review | Reviewer | `/code-review` | Before PR |
| Completeness check | Verify | `/verify` | Phase 5 |
| **New external API call** | **ApiBuilder** | `/add-new-api` | **Adding API integrations** |

## Request Architecture (Controller→Service→Wrapper→Transformer)

This project uses a strict layered architecture for all API calls:

```
Controller/Resolver (auth here) → Service (business logic)
  → APIWrapper (HTTP + transform) → External API
```

**Full rules**: `.github/instructions/api-architecture.instructions.md`

**When adding any external API call** → Use `/add-new-api` prompt and read:
- `docs/external-apis/[api-name]/` — fields, auth, query patterns
- `docs/apis/wrappers/[name]-wrapper.md` — existing wrapper methods

## Where Docs Live

```
docs/
├── issues/ISSUE-XXX-name.md          ← One per work item (all 5 phases)
├── apis/[domain]/[endpoint].api.md   ← One per internal API endpoint
├── apis/wrappers/[name]-wrapper.md   ← One per external API wrapper class
├── external-apis/[api-name]/         ← Auth, entities, field mappings per external API
├── flows/[flow-name]-flow.md         ← One per user journey
├── team-notes/[your-name]/           ← Personal notes (no conflicts)
└── templates/                        ← Copy to start new docs
```

## Daily Rules

1. **One session per Issue** — start fresh when switching Issues
2. **Read the Issue doc first**: `"Continue ISSUE-042 — read #docs/issues/ISSUE-042-name.md"`
3. **Read the external API doc before touching a wrapper**: `#docs/external-apis/dynamics/`
4. **Code + docs in same commit** — never separate them
5. **After changing an API** → run `/update-api-doc`

## All Slash Commands

| Command | When |
|---------|------|
| `/discuss` | Starting a new Issue |
| `/research` | Researching codebase (also auto after /discuss) |
| `/plan` | Creating an implementation plan |
| `/execute` | Implementing from an approved plan |
| `/verify` | Before creating a PR |
| `/add-new-api` | Adding a new external API call |
| `/code-review` | Reviewing any file or PR |
| `/generate-api-doc` | Documenting a new API endpoint |
| `/update-api-doc` | After changing an existing endpoint |
| `/create-pr-description` | Before opening a PR |
