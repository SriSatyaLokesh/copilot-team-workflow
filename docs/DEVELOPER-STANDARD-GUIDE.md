# Developer Standard Guide — GitHub Copilot Team Setup

> **Version**: 1.0 | **For**: All developers on the team
> **Purpose**: The single reference every developer reads before using Copilot in this project.

---

## Table of Contents
1. [How Copilot Works in This Project](#1-how-copilot-works-in-this-project)
2. [The 5-Phase Issue Workflow](#2-the-5-phase-issue-workflow)
3. [API Architecture — The Non-Negotiable Pattern](#3-api-architecture--the-non-negotiable-pattern)
4. [Docs Structure — Where Everything Lives](#4-docs-structure--where-everything-lives)
5. [Available Agents — Who to Ask for What](#5-available-agents--who-to-ask-for-what)
6. [Slash Commands Reference](#6-slash-commands-reference)
7. [Day-to-Day Developer Workflow](#7-day-to-day-developer-workflow)
8. [Adding a New External API Call](#8-adding-a-new-external-api-call)
9. [Writing Documentation](#9-writing-documentation)
10. [Hooks — What Runs Automatically](#10-hooks--what-runs-automatically)
11. [Team Lead Setup Guide](#11-team-lead-setup-guide)
12. [Why We Chose This Approach](#12-why-we-chose-this-approach)

---

## 1. How Copilot Works in This Project

Copilot is not just an autocomplete tool here. It's configured to behave as a structured team assistant through:

| Component | What It Is | Where |
|-----------|-----------|-------|
| **Instructions** | Rules auto-loaded per file type | `.github/instructions/*.instructions.md` |
| **Agents** | Specialist assistants with specific roles | `.github/agents/*.agent.md` |
| **Prompts** | Slash commands for repeatable tasks | `.github/prompts/*.prompt.md` |
| **Skills** | Bundled knowledge packs the agent reads | `.github/skills/*/SKILL.md` |
| **Hooks** | Scripts that run automatically on events | `.github/hooks/*/` |

### How Context Works

Copilot loads context **automatically** based on what file you have open:

| File you're editing | Instructions loaded |
|---------------------|---------------------|
| `src/api/**` | `api-architecture.instructions.md` |
| `src/wrappers/**` | `api-architecture.instructions.md` |
| `src/services/**` | `api-architecture.instructions.md` |
| `**/*.tsx`, `**/*.jsx` | `frontend.instructions.md` |
| `**/*.test.*` | `testing.instructions.md` |
| Any file | `developer-guide.instructions.md` (always on) |

> **Rule**: You don't have to tell Copilot "we use zod for validation" every session —
> it's in the instructions and loads automatically.

---

## 2. The 5-Phase Issue Workflow

Every unit of work — feature, bug fix, task, improvement — is an **Issue** and follows 5 phases.

### Overview

```
New Request
    ↓
① Discuss → Define scope, requirements, acceptance criteria
    ↓ (auto hands off)
② Research → Find existing patterns, affected files, related docs
    ↓
③ Plan → Create implementation task list, architecture decisions
    ↓
④ Execute → TDD: write tests first, then code, commit per task
    ↓
⑤ Verify → Check requirements met, tests passing, docs updated
    ↓
Pull Request
```

### Commands

| Phase | How to Start | Agent Activated |
|-------|-------------|----------------|
| ① Discuss | `/discuss` | Discuss → auto-triggers Research |
| ② Research | Automatic after Discuss | Research |
| ③ Plan | `/plan` | Planner |
| ④ Execute | `/execute` | TDD |
| ⑤ Verify | `/verify` | Verify |

### The Issue Document

Every Issue has **one file** that tracks all 5 phases:

```
docs/issues/ISSUE-042-login-rate-limiting.md
```

It starts empty (just Phase 1 section) and fills up as the Issue progresses.
This one file is your source of truth — not Jira, not Slack, this file.

**Start every Copilot session with:**
```
"Continue ISSUE-042. Read #docs/issues/ISSUE-042-login-rate-limiting.md first."
```

---

## 3. API Architecture — The Non-Negotiable Pattern

This project has a strict layered architecture. Every developer must know this.

### The Lifecycle

```
HTTP Request
      ↓
① Controller / Resolver     ← Parse, validate input. Auth & Authorization ONLY.
      ↓
② Service                   ← Business logic. Decides what to call, applies rules.
      ↓
③ API Wrapper               ← HTTP communication only. One wrapper per external API.
      ↓
④ External API              ← 3rd party (Dynamics, Stripe, etc.)
      ↓
⑤ API Wrapper               ← Calls Transformer on the response.
      ↓
⑥ Transformer               ← Maps external field names → internal field names.
      ↓
⑦ Service                   ← Receives clean typed internal objects.
      ↓
⑧ Controller / Resolver     ← Returns response to client.
```

### What Goes in Each Layer

| Layer | Allowed | Forbidden |
|-------|---------|----------|
| Controller | Parse, validate, auth check, call service | Business logic, HTTP calls, DB queries |
| Service | Business rules, orchestration, calling wrapper | HTTP calls, direct DB, response formatting |
| Wrapper | HTTP calls, auth header, retry, call transformer | Business logic, decisions about WHAT to call |
| Transformer | Field name mapping, type conversion | Business logic, decisions |

> **Full rules**: `.github/instructions/api-architecture.instructions.md` (auto-loaded when you open any service/wrapper file)

---

## 4. Docs Structure — Where Everything Lives

```
docs/
├── DEVELOPER-STANDARD-GUIDE.md   ← You are here
├── team-copilot-guide.md          ← Detailed guide (full version)
├── docs-architecture.md           ← This structure explained
├── RESEARCH-DECISIONS.md          ← Why we chose this approach
│
├── external-apis/                 ← 🔑 KEY: Read BEFORE writing wrapper code
│   ├── README.md                  ← Index of all external APIs
│   └── [api-name]/
│       ├── README.md              ← Auth, base URL, rate limits, gotchas
│       └── [entity].api.md        ← Field names, query examples per entity
│
├── apis/                          ← Our own internal API endpoints
│   ├── [domain]/
│   │   └── [endpoint].api.md      ← One per endpoint
│   └── wrappers/
│       └── [name]-wrapper.md      ← All wrapper methods documented
│
├── flows/                         ← User journey documentation
│   └── [flow-name]-flow.md        ← One per business flow
│
├── issues/                        ← Active work items
│   └── ISSUE-XXX-name.md          ← One per Issue, all 5 phases inside
│
├── decisions/                     ← Architecture Decision Records
│   └── ADR-XXX-decision.md
│
├── team-notes/                    ← Personal notes per developer
│   └── [your-name]/               ← Only YOU write here — no merge conflicts
│
└── templates/                     ← Copy these to start new docs
    ├── issue-template.md          ← Start a new Issue
    ├── api-doc-template.md        ← Document an internal API endpoint
    ├── api-wrapper-doc-template.md ← Document a wrapper class
    ├── external-api-doc-template.md ← Document an external API
    └── flow-doc-template.md       ← Document a user journey
```

### The 10-Second Rule

Before starting any coding session, spend 10 seconds opening the right doc:

| What you're working on | Read this first |
|------------------------|----------------|
| Adding an endpoint | `docs/apis/[domain]/[endpoint].api.md` |
| Working on a wrapper | `docs/external-apis/[api-name]/[entity].api.md` |
| Continuing an Issue | `docs/issues/ISSUE-XXX-name.md` |
| Understanding a flow | `docs/flows/[flow-name]-flow.md` |
| Making an arch decision | `docs/decisions/` |

---

## 5. Available Agents — Who to Ask for What

| Agent | Best For | How to Use |
|-------|---------|-----------|
| **Project Manager** | Starting any new piece of work | Select "Project Manager" agent |
| **Discuss** | Clarifying requirements for a new Issue | `/discuss` |
| **Research** | Exploring codebase before planning | Auto, or select Research agent |
| **Planner** | Creating implementation plan from research | `/plan` |
| **TDD** | Writing code (tests first, always) | `/execute` |
| **Reviewer** | Code review before PR | `/code-review` |
| **Verify** | Final check — all requirements met? | `/verify` |
| **API Builder** | Adding new external API integrations | `/add-new-api` |

> **Tip**: Start every piece of work with `Project Manager` — it routes you to the right next agent automatically.

---

## 6. Slash Commands Reference

| Command | Phase | What happens |
|---------|-------|-------------|
| `/discuss` | Phase 1 | Start new Issue, define requirements → auto-triggers research |
| `/plan` | Phase 3 | Create implementation plan per Issue |
| `/execute` | Phase 4 | TDD execution of plan tasks |
| `/verify` | Phase 5 | Full verification report before PR |
| `/add-new-api` | Execute | Full API integration workflow |
| `/code-review` | Any | Multi-dimension code review |
| `/generate-api-doc` | After code | Create new internal API endpoint doc |
| `/update-api-doc` | After code change | Update existing API doc from code changes |
| `/create-pr-description` | Before PR | Generate PR description |

---

## 7. Day-to-Day Developer Workflow

### Starting a New Issue (Morning)

```
1. Create branch: git checkout -b issue/ISSUE-042-login-rate-limiting
2. Open Copilot Chat → select "Project Manager" agent
3. Describe what you need to build
4. /discuss → requirements defined → auto-research kicks in
5. Review research findings
6. /plan → approve the plan
7. /execute → TDD implementation begins
```

### Continuing an Issue (After a Break)

```
1. Open Copilot Chat
2. Say: "I'm continuing ISSUE-042. Read #docs/issues/ISSUE-042-login-rate-limiting.md"
3. Check Phase 4 progress tracker — pick up from where you left off
```

### Before Committing

```bash
# Run tests
npm test

# Check for issues
npm run lint
npx tsc --noEmit

# If you changed an API endpoint
# In Copilot Chat: /update-api-doc
```

### Before Opening a PR

```
1. /verify → get the verification report
2. Fix any ❌ items the Verify agent found
3. /create-pr-description → paste into GitHub PR
4. Request review from a teammate → they run /code-review
```

### When Working on a Wrapper or External API Call

```
1. STOP — open docs/external-apis/[api-name]/ FIRST
2. Check the field name mapping table
3. Check auth method and headers
4. Then open /add-new-api prompt
```

---

## 8. Adding a New External API Call

This is the most common complex task in this project. Do it in this exact order:

### Step 1 (if not done): Document the External API

If `docs/external-apis/[api-name]/[entity].api.md` doesn't exist:

```
1. Copy docs/templates/external-api-doc-template.md
2. Fill in: auth method, base URL, field name mapping table, query examples, gotchas
3. Save to docs/external-apis/[api-name]/[entity].api.md
4. Commit this doc BEFORE writing any code
```

> ⚠️ Never write wrapper code without first documenting the external field names.
> Copilot will use the wrong field names if this doc doesn't exist.

### Step 2: Run /add-new-api

```
# In Copilot Chat:
/add-new-api
→ "Which external API?" → dynamics
→ "Which entity?" → accounts  
→ "Which operation?" → get by ID
```

### Step 3: API Builder agent creates (in order):
1. **Transformer** — external field → internal field mapping
2. **Wrapper method** — HTTP call + error handling + calls transformer
3. **Service method** — business logic
4. **Controller/Resolver endpoint** — validation + auth + calls service
5. **Tests** — one per layer, TDD

### Step 4: Update docs
```
/update-api-doc  ← updates docs/apis/wrappers/[name]-wrapper.md
/update-api-doc  ← updates docs/apis/[domain]/[endpoint].api.md
```

---

## 9. Writing Documentation

### The Code + Docs Rule

**Never commit a code change without updating the related doc in the same commit.**

```bash
# Bad commit
git commit -m "feat: add rate limiting to login"

# Good commit
git commit -m "feat: ISSUE-042 add rate limiting to login

Updated:
- docs/apis/auth/login.api.md (added 429 error, rate limit rules)
- docs/issues/ISSUE-042-login-rate-limiting.md (Phase 4 progress)"
```

### Quick Template Guide

| Situation | Template to copy | Save to |
|-----------|-----------------|---------|
| New Issue | `docs/templates/issue-template.md` | `docs/issues/ISSUE-XXX-name.md` |
| New internal endpoint | `docs/templates/api-doc-template.md` | `docs/apis/[domain]/[endpoint].api.md` |
| New external API entity | `docs/templates/external-api-doc-template.md` | `docs/external-apis/[name]/[entity].api.md` |
| New wrapper class | `docs/templates/api-wrapper-doc-template.md` | `docs/apis/wrappers/[name]-wrapper.md` |
| New user journey | `docs/templates/flow-doc-template.md` | `docs/flows/[flow-name]-flow.md` |

### Using Copilot for Docs

```
# Generate new API doc from code
/generate-api-doc
(Copilot reads the active file and creates the doc)

# Update existing doc after changing code
/update-api-doc
(Copilot reads git diff and updates only what changed)
```

---

## 10. Hooks — What Runs Automatically

Hooks run scripts at specific moments in the Copilot agent lifecycle.

| Hook | When it runs | What it does |
|------|-------------|-------------|
| `session-auto-commit` | When agent session ends | Auto-commits and pushes any unsaved changes |
| `governance-audit` | On every prompt | Scans for dangerous patterns (rm -rf, DROP TABLE, credential exposure) |

### How to Add a Hook to Your Project

```bash
# Copy the hook folder to your project
cp -r .github/hooks/governance-audit /path/to/your-project/.github/hooks/

# Make scripts executable (on Mac/Linux)
chmod +x .github/hooks/governance-audit/*.sh

# Commit to main branch (hooks activate immediately after commit)
git add .github/hooks/
git commit -m "chore: add governance audit hook"
git push
```

---

## 11. Team Lead Setup Guide

### First-Time Project Setup (Do Once)

```
1. Copy .github/ folder from this repo to your project
2. Update .github/copilot-instructions.md:
   - Fill in "What This Project Is"
   - Fill in your tech stack
   - Add your specific conventions

3. Create docs/external-apis/ for each external API your project calls:
   - Copy docs/templates/external-api-doc-template.md
   - Fill in auth, base URL, field mapping tables

4. Create docs/flows/ for each business flow:
   - auth-flow.md, order-flow.md, etc.

5. Brief the team on this guide (30 min)
```

### What Developers DO NOT need to do

- ❌ Install plugins
- ❌ Configure API keys for Copilot
- ❌ Modify agent files
- ❌ Learn about frontmatter syntax

All they need:
- ✅ VS Code with GitHub Copilot extension
- ✅ Copilot access
- ✅ Read this guide once

### Maintaining Docs Over Time

**Team lead responsibilities:**
- Update `docs/flows/` when a flow changes (via PR)
- Review `docs/external-apis/` when external APIs have version updates
- Add new agents/prompts when the team finds repeated patterns

**Developer responsibilities:**
- Update Issue doc through all 5 phases
- Update API doc when changing an endpoint
- Never leave a code change undocumented

---

## 12. Why We Chose This Approach

### What We Researched

1. **VS Code team's `.github/` folder** — 13 instructions, 17 prompts, 11 skills, 2 agents
2. **Official VS Code Copilot documentation** — agents, skills, hooks, subagents, background agents
3. **Zoey AI Agents PM pattern** — specialist skill delegation, requirements docs, work summaries
4. **github/awesome-copilot** — 172 agents, 175 instructions, 361 skills, governance hooks, plugins
5. **GSD (Get Stuff Done)** — solo workflow with 5 phases (good inspiration, not suitable for teams)

### Why Not GSD for Teams

GSD is excellent for a solo developer. For teams it fails because:
- Shared state file → merge conflicts every day
- Phase locking assumes one person drives start to finish
- No connection to GitHub Issues (your actual tracker)
- Not an industry standard — creates onboarding overhead

### What We Kept from GSD

The **phase thinking**: Discuss → Research → Plan → Execute → Verify.
This prevents "just start coding" chaos. We kept the phases, dropped the shared state file problem.

### Our Approach Is

> **VS Code team's patterns** + **feature-level documentation** + **GSD phase structure** + **Zoey's PM skill delegation** 

Native Copilot features, used the way the teams who build Copilot use them.

Full decision log: [docs/RESEARCH-DECISIONS.md](RESEARCH-DECISIONS.md)
