---
layout: default
parent: "Copilot Team Workflow"
title: "Part 02 - The 5-Phase Issue Workflow"
description: "Discuss → Research → Plan → Execute → Verify. The structured workflow every developer follows."
nav_order: 3
---

# Learn: The 5-Phase Issue Workflow


---

## Why Phases?

Most teams "just start coding" when they get a new task. The developer opens VS Code, asks Copilot to "add authentication", and gets code that:
- Doesn't match the existing auth pattern
- Doesn't account for the edge cases that were discussed in Slack
- Works but nobody knows which requirements it was supposed to satisfy

The 5-phase workflow prevents this. Every piece of work — feature, bug fix, improvement, task — goes through the same structured path.

**The principle**: Copilot is only as good as the context you give it. The phases are how you build that context systematically.

---

## The 5 Phases

```
/start-issue  → Create branch + baseline first  ← ALWAYS START HERE
      ↓
① DISCUSS     → Define WHAT before HOW
      ↓ (auto-triggers)
② RESEARCH    → Find WHERE before PLANNING
      ↓
③ PLAN        → Design HOW before CODING
      ↓
④ EXECUTE     → Code with TESTS first
      ↓
⑤ VERIFY      → Check before PR
      ↓
/finish-branch → Merge / PR / Keep / Discard
```

---

### Before Phase 1 — Start Issue (`/start-issue`) — **Always begin here**

**Goal**: Create an isolated branch and verify a clean test baseline before any requirements work.

**What it does**:
1. Checks you are not on `main`/`master`
2. Pulls latest from `main`
3. Creates branch: `issue/[issue-id]-[slug]`
4. Runs `npm test` — confirms baseline is green before any new work
5. Creates the Issue doc at `docs/issues/[issue-id]-[slug].md`
6. Hands off to the **Discuss** agent

> **Why `/start-issue` instead of just selecting the Discuss agent?** `/start-issue` enforces the branch-first rule. Going straight to Discuss means you might discuss and plan on `main`, then forget to branch. `/start-issue` makes the branch the first non-negotiable step.

**Output**: Feature branch created, Issue doc initialized, Discuss begins

---

### Phase 1 — Discuss (`/discuss`)

**Goal**: Write down requirements everyone agrees on, before anyone touches code.

**Output**: Phase 1 section in the Issue doc

---

### Phase 2 — Research (automatic after Discuss)

**Goal**: Find existing patterns BEFORE planning. Never plan in a vacuum.

**What happens**:
1. Research agent reads Phase 1 requirements
2. Searches the codebase for:
   - Files that will likely need to change
   - Existing patterns to follow (reuse, don't reinvent)
   - Related services, middleware, schemas
   - Tests that may need updating
3. Fills Phase 2 of the Issue doc with findings
4. Presents a handoff button: "Create Plan →"

**Real example of what Research finds**:
```
Files to modify:
  - src/middleware/auth.ts
  - src/api/auth/route.ts

Existing patterns to follow:
  - OTP rate limiting: src/middleware/otp-rate-limit.ts
    (use same Redis key pattern: `rate:${type}:${identifier}`)

Risk:
  - Rate limiter must exempt admin users — check user.role before applying
```

**Why this matters**: The Plan agent uses this output. Without Research, the Planner proposes new patterns that already exist in the codebase, leading to duplicate code.

**Agent used**: `research.agent.md`
**Output**: Phase 2 section in the Issue doc

---

### Phase 3 — Plan ("Create Plan →" button)

**Goal**: Create an ordered implementation checklist before writing any code.

**What happens**:
1. Developer clicks **"Create Plan →"** from the Research handoff (or selects the `Planner` agent directly)
2. Planner agent reads Phase 1 (requirements) + Phase 2 (research findings)
3. Creates a structured plan with:
   - Architecture decisions (and why)
   - Ordered task checklist
   - Test plan
   - Open questions needing human input
4. Developer reviews and approves
5. Planner writes Phase 3 to Issue doc
6. Handoff button: "Start Implementation (TDD) →"

**Example plan output**:
```markdown
## Architecture Decision
Use the existing Redis `rate:${type}:${identifier}` key pattern.
Window: 15 min (not 5 — confirmed with product team in Discuss phase).

## Tasks
- [ ] Test: rate limit middleware (unit)
- [ ] Implement: rate-limit.ts middleware
- [ ] Test: auth controller with rate limit (integration)
- [ ] Apply: middleware to POST /auth/login
- [ ] Test: admin bypass behavior
- [ ] Implement: admin role check before rate limit
```

**Agent used**: `plan.agent.md`
**Output**: Phase 3 section in the Issue doc

---

### Phase 4 — Execute (`/execute`)

**Goal**: Write code following TDD — tests prove the code works before moving on.

**What happens**:
1. Developer runs `/execute` (or uses the "Start Implementation" handoff button)
2. TDD agent reads Phase 3 task list
3. For each task:
   - Writes the test file first
   - Writes the implementation to make the test pass
   - Commits: `issue-042 test: rate limit middleware` then `issue-042 feat: rate limit middleware`
   - Marks task `[x]` in Issue doc Phase 4 progress tracker
4. Logs each commit to `logs/copilot/agent-activity.log`

**Commit pattern**:
```
issue-042 test: rate limit middleware
issue-042 feat: rate limit middleware (red→green)
issue-042 test: admin bypass for rate limit
issue-042 feat: admin bypass (red→green)
```

**Why one commit per test cycle?** If a test regresses later, you can pin exact commits to where it broke.

**Copilot updates**:
- `docs/issues/issue-042-login-rate-limiting.md` → Execution notes, progress tracker updated
- `docs/apis/auth/login.api.md` → "Rate limiting" section added to the API doc

**Agent used**: `tdd.agent.md`
**Output**: Working code, Phase 4 progress tracker updated

> **3 or more independent tasks?** Use `parallel-builder.agent.md` instead. It checks which tasks are truly independent, dispatches them to TDD Implementer subagents simultaneously, then integrates the results and runs the full test suite. Only use this when tasks touch completely different files — shared files = hidden dependencies = merge conflicts.

---

### Phase 5 — Verify (`/verify`)

**Goal**: Independent check that everything is done before creating a PR.

**What happens**:
1. Developer runs `/verify`
2. Verify agent runs the full checklist:
   - Requirements: each acceptance criterion from Phase 1 → ✅/❌/⚠️
   - Tests: `npm test` — all passing?
   - Plan: all tasks from Phase 3 complete?
   - Docs: API docs updated if endpoints changed?
   - Code quality: TypeScript errors? Lint errors? Console.logs?
   - Security: No hardcoded credentials?
3. Produces a verification report
4. Writes Phase 5 to Issue doc
5. If all clear: marks Issue `status: done`

**Example report**:
```markdown
## Verification Report — issue-042

Requirements: 3/3 ✅
  ✅ Rate limiting after 5 failures
  ✅ 15-minute window
  ✅ Admin bypass

Tests: 12/12 ✅
  ✅ Unit: 8/8
  ✅ Integration: 4/4

Verdict: ✅ READY FOR PR
```

**Agent used**: `verify.agent.md`
**Output**: Phase 5 in Issue doc, green light for PR

---

### After Phase 5 — Finish Branch (`/finish-branch`)

**Goal**: Final gate before merging — runs tests + quality checks, then offers 4 structured options.

**What it does**:
1. Re-runs the full test suite (`npm test`)
2. Runs TypeScript check (`npx tsc --noEmit`) and lint
3. Reads the Issue doc to verify all Phase 1 requirements are met
4. Presents 4 options:
   - **Merge to main locally** — merge the branch into main now
   - **Push and create a PR** — push branch and open a GitHub PR
   - **Keep as-is** — preserve the branch, handle merging later
   - **Discard** — delete the branch (asks for typed confirmation first)
5. Updates Issue doc `status: done` after merge or PR

> **Why use `/finish-branch` instead of just running `git merge`?** It enforces the final test + requirements gate — prevents merging green-code-but-broken-requirements or broken lint.

---

## The Issue Document

Every Issue has ONE file tracking all 5 phases:

```
docs/issues/issue-042-login-rate-limiting.md
```

```markdown
---
issue-id: "issue-042"
title: "Login Rate Limiting"
status: "execute"          ← updates through: discuss → research → plan → execute → verify → done
branch: "issue/issue-042-login-rate-limiting"
---

# issue-042: Login Rate Limiting

## Phase 1: Discuss [x]
Requirements agreed: ...
Acceptance criteria: ...

## Phase 2: Research [x]
Files to modify: ...
Patterns found: ...

## Phase 3: Plan [x]
Architecture decisions: ...
Task checklist: ...

## Phase 4: Execute [/]     ← in progress
[ ] Task 1
[x] Task 2

## Phase 5: Verify [ ]
```

---

### The `related-flow` and `related-apis` fields

The Issue doc frontmatter has two fields every developer should fill in:

```yaml
related-flow: "auth-flow"         ← name of the flow doc in docs/flows/
related-apis: ["auth/login"]      ← paths to API docs in docs/apis/
```

These fields help Copilot **find the right context automatically** during Research and Execute phases. When the Research agent reads your Issue doc, it uses these links to pull in `docs/flows/auth-flow.md` and `docs/apis/auth/login.api.md` — giving it the full picture of what your code is supposed to do.

Fill them in before starting Research. Leave them blank and Copilot may miss important business rules and endpoint contracts.

---

## Starting a Session on an Existing Issue

This is the most important habit: **always re-read the Issue doc** when resuming work.

```
"Continue issue-042. Read #docs/issues/issue-042-login-rate-limiting.md first."
```

Copilot sees the full context: what was decided, what was researched, what the plan says, where execution left off. No repeated explanations.

---

## Summary

| Phase | How to start | Time | Who decides to stop |
|:---|:---|:---|:---|
| Start Issue | `/start-issue` | 2-5 min | Branch + baseline green |
| Discuss | Select `Discuss` agent | 10-15 min | Developer approves requirements |
| Research | automatic (handoff from Discuss) | 5-10 min | Research agent presents findings |
| Plan | Click `Create Plan →` button | 15-20 min | Developer approves plan |
| Execute | `/execute` | Varies | All tasks complete and committed |
| Verify | `/verify` | 10 min | Verify agent gives green light |
| Finish | `/finish-branch` | 2-5 min | Developer chooses merge/PR/keep/discard |

**Next: API Architecture →** [03-api-architecture.md](./03-api-architecture.md)

---

## Further Reading

- [VS Code Copilot — Agents](https://code.visualstudio.com/docs/copilot/chat/chat-agents) — how agent handoffs and the `send: true` auto-trigger work
- [Test-Driven Development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development) — the red→green→refactor cycle that Phase 4 is built on
- [Conventional Commits](https://www.conventionalcommits.org/) — the `feat:`, `test:`, `docs:` commit format used in this workflow
- Full example issue doc: [issue-001-login-rate-limiting.example.md](../docs/issues/issue-001-login-rate-limiting.example.md)
---

[← Part 01: .github Folder Explained](./01-github-folder-explained.md) · 📚 [Learn Series](../index.md) · [Part 03: API Architecture →](./03-api-architecture.md)
