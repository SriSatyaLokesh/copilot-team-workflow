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

> [!IMPORTANT]
> **Prerequisite: Known Codebase**
> Before starting the 5-phase workflow on an existing project, you MUST run the **`/acquire-codebase-knowledge`** process. This ensures the `docs/codebase/` templates are populated with factual evidence (Architecture, Stack, Conventions). Without this mapping, Copilot might hallucinate patterns or suggest incompatible technologies.

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

**Goal**: Confirm the primary branch, create an isolated workspace, verify a clean test baseline — before any requirements work.

**What it does**:
1. **Checks for a stored primary branch** in `.github/copilot-instructions.md`. If missing, asks once: *"What is your team's primary branch?"* (e.g. `dev`, `develop`, `main`) and writes the answer permanently — so future sessions never ask again.
2. **Shows current branch**, then asks: **A) Stay here** (already on right feature branch) or **B) Create new branch from primary** (for fresh work)
3. If B: `git checkout <primary> && git pull origin <primary> && git checkout -b <type>/[issue-id]-[slug]`
4. Runs `npm test` — confirms baseline is green before any new work
5. Creates work folder at `work/ISSUE-[id]-[slug]/` with `plan.md` and `result.md` templates
6. (If GitHub repo) Offers to create GitHub issue with `gh issue create`
7. (If GitHub repo) Offers to create feature branch
8. Appends entry to `logs/copilot/agent-activity.log`
9. Hands off to the **Discuss** agent automatically

> **Why confirm the primary branch instead of hardcoding `main`?** Teams differ — some use `dev`, `develop`, or `staging` as their integration branch. Hardcoding `main` means developers accidentally base feature branches on stale or wrong code. The answer is stored once in `copilot-instructions.md` so the prompt adapts to every team's setup.

> **Why A/B choice?** If you're mid-way through a feature and just want to continue, you don't want `/start-issue` forcing a new branch. Option A lets you keep context; Option B is for truly new work.

**Output**: Primary branch confirmed, feature branch ready, work folder created at `work/ISSUE-XXX-name/`, GitHub issue created (optional), activity logged, Discuss begins

---

### Phase 1 — Discuss (`/discuss`)

**Goal**: Write down requirements everyone agrees on, before anyone touches code.

**What happens**:
1. Discuss agent asks clarifying questions (one at a time, max 5)
2. Proposes 2-3 approaches with trade-offs
3. Gets explicit approval on final design
4. Writes Phase 1 (Requirements) in `work/ISSUE-XXX-name/plan.md`
5. Marks Phase 1 as `[x] Complete`
6. Appends entry to `logs/copilot/agent-activity.log`
7. Shows full 5-phase roadmap
8. Hands off to Research agent **automatically**

**Output**: `plan.md` Phase 1 complete, Research starts automatically

---

### Phase 2 — Research (automatic after Discuss)

**Goal**: Find existing patterns BEFORE planning. Never plan in a vacuum.

**What happens**:
1. Research agent reads `plan.md` Phase 1 (requirements)
2. Searches the codebase for:
   - Files that will likely need to change
   - Existing patterns to follow (reuse, don't reinvent)
   - Related services, middleware, schemas
   - Tests that may need updating
3. Writes Phase 2 (Research) in `work/ISSUE-XXX-name/plan.md`
4. Marks Phase 2 as `[x] Complete`
5. Appends entry to `logs/copilot/agent-activity.log`
6. Completes with message: **"Run `/plan` to create implementation plan"**
7. **Does NOT auto-hand off** — user must explicitly run `/plan`

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

### Phase 3 — Plan (`/plan`)

**Goal**: Create an ordered implementation checklist before writing any code.

**What happens**:
1. Developer runs `/plan` (after Research completes)
2. Planner agent reads `plan.md` Phase 1 (requirements) + Phase 2 (research findings)
3. Verifies Phases 1 and 2 are marked complete (gate check)
4. Creates a structured plan with:
   - Architecture decisions (and why)
   - Ordered task checklist
   - Test plan
   - Open questions needing human input
5. Developer reviews and approves
6. Planner writes Phase 3 to `work/ISSUE-XXX-name/plan.md`
7. Marks Phase 3 as `[x] Complete`
8. Appends entry to `logs/copilot/agent-activity.log`
9. Completes with message: **"Run `/execute work/ISSUE-XXX-name` to begin implementation"**
10. **Does NOT auto-hand off** — user must explicitly run `/execute`

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
**Output**: `plan.md` Phase 3 complete

---

### Phase 4 — Execute (`/execute`)

**Goal**: Write code following TDD — tests prove the code works before moving on.

**What happens**:
1. Developer runs `/execute work/ISSUE-XXX-name`
2. Execute prompt reads `plan.md` Phases 1-3
3. **Offers choice of execution mode**:

   **Mode A: Agent Mode** (Recommended for complex features)
   - Execute prompt generates a context bundle with:
     - Requirements (Phase 1)
     - Research findings (Phase 2)
     - Implementation plan (Phase 3)
     - File locations and quality standards
   - Developer copies bundle to **new Copilot chat**
   - Agent mode implements with full workspace awareness
   - When done, developer returns and runs `/verify`
   - ✅ Best for: refactoring, large features, exploration

   **Mode B: TDD Agent** (Strict discipline)
   - Invokes `@tdd` agent in same chat
   - For each task:
     - Writes failing test first
     - Implements minimal code to pass
     - Refactors while keeping tests green
     - Commits: `git commit -m "feat: <task> Fixes #XX"`
     - Updates `result.md` Phase 4 progress
   - ✅ Best for: routine features, bug fixes, learning TDD

4. Whichever mode completes:
   - Updates `work/ISSUE-XXX-name/result.md` Phase 4
   - Marks Phase 4 as `[x] Complete`
   - (If GitHub repo) Offers to create PR with `gh pr create --title "<type>: <title>" --body "Fixes #XX\n\n<summary>"`
   - Appends entry to `logs/copilot/agent-activity.log`
   - Completes with message: **"Run `/verify work/ISSUE-XXX-name` for quality gate"**

**Commit pattern** (Mode B - TDD Agent):
```
feat: rate limit middleware Fixes #42
feat: admin bypass for rate limit Fixes #42
```

**Why GitHub issue reference?** Links commits to the GitHub issue for traceability.

**Copilot updates**:
- `work/ISSUE-042-login-rate-limiting/result.md` → Phase 4 execution notes
- `docs/apis/auth/login.api.md` → "Rate limiting" section added to the API doc (if endpoint changed)

**Agents/Prompts used**: `execute.prompt.md` → `tdd.agent.md` (Mode B) or agent mode (Mode A)  
**Output**: Working code, `result.md` Phase 4 complete, GitHub PR created (optional)

> **3 or more independent tasks?** Use `parallel-builder.agent.md` instead. It checks which tasks are truly independent, dispatches them to TDD Implementer subagents simultaneously, then integrates the results and runs the full test suite. Only use this when tasks touch completely different files — shared files = hidden dependencies = merge conflicts.

---

### Phase 5 — Verify (`/verify`)

**Goal**: Independent check that everything is done before creating a PR.

**What happens**:
1. Developer runs `/verify work/ISSUE-XXX-name`
2. Verify agent runs the full checklist:
   - Requirements: each acceptance criterion from Phase 1 → ✅/❌/⚠️
   - Tests: `npm test` — all passing?
   - Plan: all tasks from Phase 3 complete?
   - Docs: API docs updated if endpoints changed?
   - Code quality: TypeScript errors? Lint errors? Console.logs?
   - Security: No hardcoded credentials?
3. Produces a verification report
4. Writes Phase 5 to `work/ISSUE-XXX-name/result.md`
5. Marks Phase 5 as `[x] Complete`
6. Appends entry to `logs/copilot/agent-activity.log`
7. If verdict is **✅ READY**:
   - (If GitHub repo + PR exists) Offers to merge PR with `gh pr merge --squash --delete-branch`
   - (If GitHub repo + no PR) Offers to create and merge PR
8. If verdict is **⛔ NOT READY**:
   - Lists blocking issues
   - Developer fixes issues, then re-runs `/verify`

**Example report**:
```markdown
## Verification Report — ISSUE-042

Requirements: 3/3 ✅
  ✅ Rate limiting after 5 failures
  ✅ 15-minute window
  ✅ Admin bypass

Tests: 12/12 ✅
  ✅ Unit: 8/8
  ✅ Integration: 4/4

Verdict: ✅ READY
```

**Agent used**: `verify.agent.md`  
**Output**: `result.md` Phase 5 complete, verdict (READY or NOT READY), GitHub PR merged (optional)

---

### After Phase 5 — Finish Branch (`/finish-branch`)

**Goal**: Final decision on what to do with the completed work.

**What it does**:
1. Reads `result.md` Phase 5 verification report
2. (If GitHub repo) Checks if PR exists with `gh pr view`
3. Presents 4 options:
   - **Merge to primary locally** — merge the branch into your primary branch (e.g. `dev`)
   - **Push and create a PR** — push branch and open a GitHub PR (if GitHub repo)
   - **Keep as-is** — preserve the branch and work folder, handle merging later
   - **Discard** — delete the branch and work folder (asks for typed confirmation first)
4. (If GitHub repo) Uses `gh pr merge` or `gh pr create` automatically
5. Archives work folder to `work/.archived/ISSUE-XXX-name/` after merge (optional)

> **Why use `/finish-branch` instead of just running `git merge`?** GitHub integration is handled automatically, and verification report is reviewed one final time before merge.

---

## The Work Folder Structure

Every issue gets ONE folder with TWO files tracking intent vs reality:

```
work/ISSUE-042-login-rate-limiting/
  ├── plan.md      ← Intent: Phases 1-3 (Discuss, Research, Plan)
  └── result.md    ← Reality: Phases 4-5 (Execute, Verify)
```

**Why split into two files?**
- Prevents merge conflicts when multiple developers work on different issues
- Separates "what we want to build" from "what we built"
- Easier for agents to read specific sections

**plan.md structure**:
```markdown
# ISSUE-042: Login Rate Limiting

**Status**: execute  
**Branch**: feat/042-login-rate-limiting  
**GitHub Issue**: #42 (if created)  
**Started**: 2026-03-12  

---

## Phase 1: Discuss — Requirements
**Status**: [x] Complete

### What We're Building
Add rate limiting to the login endpoint to prevent brute-force attacks.

### Acceptance Criteria
- [ ] Max 5 login attempts per email per 15 minutes
- [ ] Admin users bypass rate limit
- [ ] 429 response with Retry-After header when limited

---

## Starting a Session on an Existing Issue

This is the most important habit: **always re-read the work folder** when resuming work.

```
"Continue ISSUE-042. Read work/ISSUE-042-login-rate-limiting/plan.md and result
- [ ] Implement: rate-limit.ts middleware
- [ ] Test: auth controller with rate limit (integration)
- [ ] Apply: middleware to POST /auth/login
```

**result.md structure**:
```markdown
# ISSUE-042: Login Rate Limiting — Execution Results

---

## Phase 4: Execute — Implementation
**Status**: [x] Complete

### What Was Implemented
- Created src/middleware/rate-limit.ts with Redis-based rate limiting
- Applied middleware to POST /auth/login route
- Added admin role bypass before rate limit check

### Commits
- `feat: rate limit middleware Fixes #42` (commit abc123)
- `feat: admin bypass for rate limit Fixes #42` (commit def456)

### Deviations from Plan
- Used 10-minute window instead of 15 (product approved in Slack)

---

## Phase 5: Verify — Quality Gate
**Status**: [x] Complete

### Verification Report
**Requirements**: 3/3 ✅  
**Tests**: 12/12 ✅  
**Code Quality**: ✅ No errors  

**Verdict**: ✅ READY

### GitHub PR
- PR #42 created: https://github.com/.../pull/42
- Merged to `dev` on 2026-03-12
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
- Full example issue doc: [issue-001-login-rate-limiting.example.md](https://github.com/SriSatyaLokesh/copilot-team-workflow/blob/main/docs/issues/issue-001-login-rate-limiting.example.md)
---

[← Part 01: .github Folder Explained](./01-github-folder-explained.md) · 📚 [Learn Series](/) · [Part 03: API Architecture →](./03-api-architecture.md)
