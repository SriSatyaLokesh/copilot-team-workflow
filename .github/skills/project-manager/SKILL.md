# Project Management Skill — Reference Guide

> This skill was adapted from the Zoey AI Agents project management pattern.
> It serves as the **reference architecture** for our 5-phase Issue workflow.

---

## What This Skill Defines

The project manager skill shows HOW to run the full development lifecycle
using specialized agent delegation. The core pattern:

```
User Request
     ↓
Project Manager (@project-manager agent)
     ↓
Discuss (defines requirements)
     ↓
Research (finds codebase context)
     ↓
Plan (creates implementation plan)
     ↓
Execute / Specialist Skills (builds the thing)
     ├── Backend work → TDD agent, backend instructions
     ├── Frontend work → TDD agent, frontend instructions
     └── Docs work → update-api-doc, documentation
     ↓
Verify (checks completeness + correctness)
```

---

## Specialist Skill Mapping (from Zoey → Our Workflow)

| Zoey Skill | Our Equivalent | When Used |
|-----------|---------------|-----------|
| `@python-developer` | TDD agent + backend instructions | API endpoints, services, DB |
| `@frontend-developer` | TDD agent + frontend instructions | React components, pages |
| `@documentation-writer` | `/update-api-doc`, `/generate-api-doc` | After code changes |
| `@project-management` | `project-manager.agent.md` | Orchestrating full Issues |

---

## Project Docs Structure (Adapted from Zoey)

Zoey uses `.copilot_docs/[MM-YYYY]/[feature-name]/` — we use `docs/issues/` 
but the same principle applies: **one folder of docs per work item**.

```
docs/
├── issues/
│   └── ISSUE-042-login-rate-limiting.md   ← All 5 phases in one file
├── apis/
│   └── auth/login.api.md                  ← Updated during Execute phase
└── flows/
    └── auth-flow.md                        ← Updated when flow changes
```

The **Issue doc** is our equivalent of Zoey's `requirements.md` + `work-summary.md` 
combined into a single 5-phase document.

---

## Delegation Pattern (How to Brief a Specialist)

When the Execute agent works on a task, provide this level of context:

### For Backend Work
```
Execute the backend tasks in ISSUE-042 plan.

Context:
- Follow backend standards: .github/instructions/backend.instructions.md
- Existing pattern to follow: src/middleware/otp-rate-limit.ts
- Files to create/modify: [list from Research phase]
- Run tests after each task: npm test
- Commit after each passing cycle
```

### For Frontend Work
```
Execute the frontend tasks in ISSUE-042 plan.

Context:
- Follow frontend standards: .github/instructions/frontend.instructions.md
- Component library: @/components/ui/*
- Pages folder: src/app/
- Run tests after each task: npm test
```

### For API Documentation
```
/update-api-doc

Updated: src/api/auth/route.ts (added rate limiting)
Update: docs/apis/auth/login.api.md
- Add rate limit error responses (429)
- Update business rules section
```

---

## Requirements Doc Standards (from Zoey)

Every good requirements doc (our Issue doc Phase 1-3) must include:

- [ ] Clear description and business goal
- [ ] Specific, **measurable** acceptance criteria (not "make it faster" → "respond in <200ms")
- [ ] Technical requirements and API specs
- [ ] Architecture and design decisions
- [ ] Testing strategy
- [ ] Security considerations

---

## Work Summary Standards (from Zoey)

Every completion doc (our Issue doc Phase 5) must include:

- [ ] Summary of what was accomplished
- [ ] Acceptance criteria status with % completion
- [ ] All files changed (created / modified) with line counts
- [ ] Implementation decisions and reasoning
- [ ] Test results and coverage
- [ ] Known issues and limitations
- [ ] Lessons learned

---

## Monthly Organization (Zoey Pattern, Optional for Us)

If the team wants monthly snapshots of work done, mirror Zoey's pattern:

```
docs/issues/
├── 03-2026/                          ← ongoing Issues this month
│   ├── ISSUE-042-login-rate-limiting.md
│   └── ISSUE-043-quotation-export.md
└── 02-2026/                          ← completed Issues, archived
    └── ISSUE-001-user-auth.md
```

Use month folders when the team grows beyond ~10 active Issues.

---

## Parallel vs Sequential Work (from Zoey)

### Sequential (dependencies)
```
1. Backend API first (Python Developer)
2. API contract shared → Frontend builds against it
3. Both done → Documentation added
```

### Parallel (independent)
```
ISSUE-042 backend ←→ ISSUE-043 frontend   (different systems, no conflict)
Both run in separate background agent sessions simultaneously
```

---

## Source Reference

This skill was adapted from:
- Zoey AI Agents: Project Management SKILL.md (shared Feb 2026)
- Zoey AI Agents: Developer Guide (shared Feb 2026)

The 5-phase workflow (Discuss → Research → Plan → Execute → Verify) is our
adaptation of Zoey's (Requirements → Delegation → Tracking → Completion).
