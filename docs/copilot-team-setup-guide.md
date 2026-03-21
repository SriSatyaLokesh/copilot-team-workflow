# GitHub Copilot Team Workflow Setup Guide

> A comprehensive, opinionated guide to setting up GitHub Copilot for a team of 5+ developers,
> based on VS Code official Copilot documentation and the Atomic GSD Hybrid methodology.

---

## Table of Contents

1. [Mental Model: The 7 Customization Levers](#1-mental-model-the-7-customization-levers)
2. [Repository Structure](#2-repository-structure)
3. [Custom Instructions (The Foundation)](#3-custom-instructions-the-foundation)
4. [Prompt Files (Slash Commands)](#4-prompt-files-slash-commands)
5. [Custom Agents (Specialized Personas)](#5-custom-agents-specialized-personas)
6. [Agent Skills (Reusable Capabilities)](#6-agent-skills-reusable-capabilities)
7. [Hooks (Automation & Enforcement)](#7-hooks-automation--enforcement)
8. [Tool Sets](#8-tool-sets)
9. [MCP Servers](#9-mcp-servers)
10. [The Team Development Workflow](#10-the-team-development-workflow)
11. [Agent Types: Choosing the Right One](#11-agent-types-choosing-the-right-one)
12. [Subagent Orchestration Patterns](#12-subagent-orchestration-patterns)
13. [Context Engineering for Teams](#13-context-engineering-for-teams)
14. [Feature Development: End-to-End Walkthrough](#14-feature-development-end-to-end-walkthrough)
15. [Scaling: 10-15 Features Simultaneously](#15-scaling-10-15-features-simultaneously)
16. [Best Practices & Anti-Patterns](#16-best-practices--anti-patterns)
17. [Quick Reference Cheat Sheet](#17-quick-reference-cheat-sheet)

---

## 1. Mental Model: The 7 Customization Levers

Copilot in VS Code has exactly 7 ways to customize its behavior. Think of them as layers:

```
┌─────────────────────────────────────────────────────────────┐
│  MCP Servers        →  External tools (DBs, APIs, CLIs)      │
│  Tool Sets          →  Grouped tools for quick selection     │
│  Hooks              →  Automated enforcement at lifecycle    │
│  Agent Skills       →  Domain capabilities (e.g. testing)   │
│  Custom Agents      →  Personas (Planner, Reviewer, etc.)   │
│  Prompt Files       →  Reusable slash commands (/review)    │
│  Custom Instructions→  Always-on project rules & standards  │
└─────────────────────────────────────────────────────────────┘
            ↑ More specific / on-demand
            ↓ More general / always applied
```

**Decision rule:**
- If it should always apply → **Custom Instructions**
- If it's a recurring task → **Prompt File**
- If it needs a persona/role → **Custom Agent**
- If it needs scripts/resources → **Skill**
- If it needs enforcement → **Hook**
- If it connects external systems → **MCP Server**

---

## 2. Repository Structure

For a team of 5+ developers with multiple features, use this layout:

```
my-project/
├── .github/
│   ├── copilot-instructions.md      # Global always-on instructions
│   ├── instructions/                # Domain-specific instructions
│   │   ├── frontend.instructions.md
│   │   ├── backend.instructions.md
│   │   ├── testing.instructions.md
│   │   └── database.instructions.md
│   ├── agents/                      # Custom agents
│   │   ├── plan.agent.md
│   │   ├── implement.agent.md
│   │   ├── review.agent.md
│   │   └── tdd.agent.md
│   ├── prompts/                     # Slash commands
│   │   ├── new-feature.prompt.md
│   │   ├── code-review.prompt.md
│   │   ├── write-tests.prompt.md
│   │   ├── fix-bug.prompt.md
│   │   └── create-pr-description.prompt.md
│   ├── skills/                      # Reusable capabilities
│   │   ├── api-patterns/
│   │   │   └── SKILL.md
│   │   ├── testing-strategy/
│   │   │   └── SKILL.md
│   │   └── deployment/
│   │       └── SKILL.md
│   └── hooks/                       # Lifecycle automation
│       ├── formatting.json
│       ├── security.json
│       └── audit.json
├── .claude/                         # Claude Code compatibility (optional)
│   └── agents/                      # Same agents, Claude format
├── docs/
│   ├── ARCHITECTURE.md
│   ├── PROJECT.md
│   ├── CONTRIBUTING.md
│   └── features/                    # Live feature plans
│       ├── auth-plan.md
│       └── payments-plan.md
├── CLAUDE.md                        # Entry point for Claude/AI agents
└── .github/copilot-instructions.md  # Entry point for Copilot
```

---

## 3. Custom Instructions (The Foundation)

### What Instructions Are For

Custom instructions are **always automatically included** in every chat interaction.
Use them for: coding standards, architectural decisions, environment context, team conventions.

> **Do NOT put** everything here. Keep it concise. If something only applies to a specific domain or task, use domain-specific instruction files or prompt files instead.

### File Types

| File | Pattern | When Applied |
|------|---------|--------------|
| `.github/copilot-instructions.md` | Always | Every chat interaction |
| `.github/instructions/*.instructions.md` | Configurable via `applyTo` | When glob pattern matches active file |

### Setting Up Your Global Instructions

`.github/copilot-instructions.md`:

```markdown
# [Project Name] — Copilot Context

## Project Overview
- **Type**: [e.g., B2B SaaS, E-commerce platform, CLI tool]
- **Stack**: [e.g., Next.js 15 + Node.js + PostgreSQL + Redis]
- **Architecture**: [e.g., Monorepo with shared packages]

## Key Documentation
- [Product Vision](../docs/PROJECT.md): What we're building and why
- [System Architecture](../docs/ARCHITECTURE.md): How everything fits together
- [Contributing Guidelines](../docs/CONTRIBUTING.md): Our team practices

## Non-Obvious Conventions
- We use [convention X] because [reason]
- All API routes must [rule]
- Avoid [pattern] — we prefer [alternative] instead

## Environment
- Node 20, TypeScript strict mode
- Testing: Vitest + Playwright
- CI: GitHub Actions

> If you find incomplete or conflicting documentation, suggest updates.
```

### Domain-Specific Instructions

Create one file per domain. Use the `applyTo` frontmatter to scope them:

`.github/instructions/frontend.instructions.md`:

```markdown
---
applyTo: "src/components/**,src/pages/**,src/app/**"
---
# Frontend Standards

- Use React Server Components by default
- Client components must be in `*.client.tsx` files
- Use our design system from `@/components/ui/*` — never raw DOM elements
- All forms must use `react-hook-form` with `zod` validation
- A11y: all interactive elements need ARIA labels
```

`.github/instructions/backend.instructions.md`:

```markdown
---
applyTo: "src/api/**,src/services/**,src/db/**"
---
# Backend Standards

- All endpoints must validate input with zod
- Use the repository pattern — never query DB directly in route handlers
- Use structured logging with `logger.info({ context }, message)`
- Never return raw DB errors to clients
- Rate limiting required on all public endpoints
```

`.github/instructions/testing.instructions.md`:

```markdown
---
applyTo: "**/*.test.ts,**/*.spec.ts,tests/**"
---
# Testing Standards

- Unit tests: Vitest, co-located with source files
- Integration tests: in `tests/integration/`
- E2E tests: Playwright, in `tests/e2e/`
- Use `describe` blocks to group related tests
- Mock external services, never hit real APIs in tests
- Coverage target: 80% for business logic
```

---

## 4. Prompt Files (Slash Commands)

Prompt files let team members invoke standardized workflows with `/command` syntax.
They are **manually invoked** (unlike instructions which are always-on).

### File Format

```markdown
---
description: 'What this prompt does'
agent: 'agent'       # ask | agent | plan | <custom-agent-name>
tools: [execute, read, edit, search]
---
# Instructions for the AI
Your detailed instructions here...

Reference docs: [API Guidelines](../docs/api-guidelines.md)
```

If a prompt targets a custom agent, omit `tools:` unless you intentionally want the prompt to override that custom agent's tool restrictions.
Built-in prompt agents such as `ask` do not honor a `tools:` override in frontmatter.

### Essential Team Prompts

**`.github/prompts/new-feature.prompt.md`** — Start any feature:

```markdown
---
description: 'Create an implementation plan for a new feature'
agent: 'plan'
tools: [read, search]
---
Create a detailed implementation plan for the feature described below.

Use the plan template at [plan-template.md](../docs/plan-template.md).

Before generating the plan:
1. Search the codebase for existing patterns related to this feature
2. Identify any potential conflicts with existing code
3. Ask me 2-3 clarifying questions if requirements are ambiguous

Feature request: ${input:feature:Describe the feature to implement}
```

**`.github/prompts/code-review.prompt.md`** — Review current changes:

```markdown
---
description: 'Review the current file or selection for quality and issues'
agent: 'ask'
---
Review this code: [${fileBasename}](${file})

Check for:
1. Logic correctness and edge cases
2. Security issues (injection, hardcoded secrets, missing auth)
3. Performance concerns
4. Missing tests
5. Adherence to our [coding standards](../.github/copilot-instructions.md)

${selection}

Provide a prioritized list of issues with severity: critical / warning / suggestion.
```

**`.github/prompts/write-tests.prompt.md`** — Generate tests:

```markdown
---
description: 'Generate tests for the current file'
agent: 'agent'
tools: ['search', 'editFiles', 'terminal']
---
Generate comprehensive tests for [${fileBasename}](${file}).

- Framework: ${input:framework:vitest or playwright}
- Place test file at: ${fileDirname}/${fileBasenameNoExtension}.test.ts
- Follow our [testing standards](../.github/instructions/testing.instructions.md)
- Cover: happy paths, error cases, edge cases, boundary conditions
- After writing tests, run them and fix any failures

${selection}
```

**`.github/prompts/fix-bug.prompt.md`** — Systematic bug fixing:

```markdown
---
description: 'Debug and fix a bug systematically'
agent: 'agent'
tools: ['search', 'codebase', 'terminal', 'editFiles']
---
Fix the following bug systematically:

1. **Understand**: Read the relevant code to understand the current behavior
2. **Reproduce**: Identify how to reproduce the issue
3. **Root cause**: Find the actual root cause, not just symptoms
4. **Fix**: Make the minimal change needed to fix it
5. **Verify**: Run relevant tests to confirm the fix

Bug description: ${input:bug:Describe the bug}
```

**`.github/prompts/create-pr-description.prompt.md`** — PR descriptions:

```markdown
---
description: 'Generate a comprehensive PR description from recent changes'
agent: 'ask'
---
Generate a pull request description for the changes in this branch.

Format:
## What changed
[Brief summary of the changes]

## Why
[The problem this solves / feature this adds]

## How to test
[Step-by-step testing instructions]

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or migration notes added)
```

### Using Prompt Files

```
/new-feature       → Opens the feature planning workflow
/code-review       → Reviews current file
/write-tests       → Generates tests for current file
/fix-bug           → Systematic bug debugging
/create-pr-description → Generates PR description
```

---

## 5. Custom Agents (Specialized Personas)

Custom agents define **specialized roles** with specific tools and instructions.
They are stored in `.github/agents/*.agent.md`.

### The Core Team Agents

**`.github/agents/plan.agent.md`** — The Planner:

```markdown
---
description: 'Architect and planner — creates implementation plans without making code changes'
name: Planner
tools: [read, search]
model: ['GPT-5', 'Claude Sonnet 4.5']
---
# Planning Agent

You are an architect and strategist. Your job is to create implementation plans — NOT to write code.

## Your Workflow
1. **Gather context**: Search the codebase to understand existing patterns
2. **Clarify requirements**: Ask 2-3 questions if anything is ambiguous
3. **Draft plan**: Use the [plan template](../docs/plan-template.md)
4. **Review with user**: Present the plan and iterate based on feedback

## Plan Structure
Every plan must include:
- **Overview**: What we're building and why
- **Architecture**: How it fits into the existing system
- **Tasks**: Ordered checklist of implementation steps
- **Testing**: What tests need to be written
- **Open questions**: Anything that needs clarification

Never make code edits. Only read, analyze, and plan.
```

**`.github/agents/tdd.agent.md`** — TDD Implementer:

```markdown
---
description: 'Test-driven implementation agent — writes failing tests first, then code'
name: TDD Implementer
tools: [execute, read, edit, search]
model: ['GPT-5', 'Claude Sonnet 4.5']
---
# TDD Implementation Agent

You implement features using strict test-driven development.

## TDD Cycle (repeat until all tasks complete)
1. **Red**: Write a failing test that defines expected behavior
2. **Green**: Write the minimal code to make the test pass
3. **Refactor**: Improve code quality while keeping tests green

## Rules
- ALWAYS write tests first — never implementation first
- Run tests after each change: `npm test`
- Commit after each passing cycle: `git commit -m "feat: [specific thing]"`
- If a test is hard to write, the design may be wrong — reconsider

## Success Criteria
- All planned tasks completed with tests
- Full test suite passing
- No skipped or commented-out tests
```

**`.github/agents/review.agent.md`** — Code Reviewer:

```markdown
---
description: 'Code reviewer — checks quality, security, and alignment with standards'
name: Reviewer
tools: ['search', 'codebase', 'usages', 'problems']
model: ['Claude Opus 4.5', 'GPT-4o']
---
# Code Review Agent

You are a senior engineer performing a thorough code review.
Read only — do NOT make changes.

## Review Dimensions
Run these checks in parallel using subagents:

1. **Correctness**: Logic errors, edge cases, off-by-ones, null handling
2. **Security**: Injection risks, unvalidated inputs, exposed secrets, missing auth
3. **Performance**: N+1 queries, synchronous blocking, unnecessary re-renders
4. **Maintainability**: Naming clarity, code duplication, complexity
5. **Standards**: Alignment with [project conventions](../.github/copilot-instructions.md)

## Output Format
Prioritized list of findings:
- 🔴 CRITICAL: Must fix before merging
- 🟡 WARNING: Should fix, potential issues
- 🔵 SUGGESTION: Nice to have improvements
- ✅ GOOD: Highlight what's done well
```

**`.github/agents/implement.agent.md`** — Standard Implementer:

```markdown
---
description: 'Standard implementation agent — implements plans with quality focus'
name: Implementer
tools: [execute, read, edit, search]
model: ['GPT-5', 'Claude Sonnet 4.5']
---
# Implementation Agent

You implement features from implementation plans. You prioritize quality and following team patterns.

## Process
1. Read the plan thoroughly before touching any code
2. Follow the existing code patterns in the codebase
3. For each task: implement → run tests → verify → commit
4. Run the full test suite before declaring done
5. Keep commits atomic: one logical change per commit

## Quality Gate
Before marking each task ✅:
- Code is tested (existing tests still pass + new tests added)
- No lint errors
- No TypeScript errors
- Follows team conventions
```

### Agent Handoffs

The handoff system creates a guided workflow:

```
User types a feature request
         ↓
    /new-feature (or Planner agent)
         ↓
    Plan is created and reviewed
         ↓
  Developer runs `/execute`
         ↓
    /execute prompts mode selection
         ↓
  Code is implemented and tested
         ↓
    /code-review (or Reviewer agent)
         ↓
    Findings addressed
         ↓
    /create-pr-description → PR created
```

---

## 6. Agent Skills (Reusable Capabilities)

Skills are **domain-specific knowledge** that Copilot can load on-demand.
Skills differ from instructions: they can include scripts, examples, and are load-on-demand (not always-on).

### How Skills Work

```
Level 1: Copilot always knows skill names & descriptions (reads frontmatter)
Level 2: When relevant, loads SKILL.md body into context
Level 3: Accesses additional files (scripts, examples) only when needed
```

### Creating a Skill

Each skill is a folder in `.github/skills/` with a `SKILL.md`:

```markdown
---
name: api-patterns
description: Guide for creating REST API endpoints in this project. Use when asked to create, modify, or review API endpoints or routes.
---

# API Patterns for [Project Name]

## File Structure
- Route handlers: `src/api/[resource]/route.ts`
- Business logic: `src/services/[resource].service.ts`
- Data access: `src/repositories/[resource].repository.ts`

## Creating an Endpoint

### Step 1: Define the schema
```typescript
// src/api/users/schema.ts
export const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2)
})
```

### Step 2: Create the route handler
[See example](./examples/route-example.ts)

## Authentication
All endpoints must use: `import { requireAuth } from '@/middleware/auth'`

## Error Handling
Return errors using: `import { apiError } from '@/utils/errors'`
```

### Essential Skills for a Team

```
.github/skills/
├── api-patterns/         # How to create/modify API endpoints
│   ├── SKILL.md
│   └── examples/
│       └── route-example.ts
├── testing-strategy/     # Testing approach and patterns
│   ├── SKILL.md
│   └── templates/
│       └── unit-test-template.ts
├── database-patterns/    # DB schema, migrations, queries
│   └── SKILL.md
├── state-management/     # Frontend state patterns
│   └── SKILL.md
└── deployment/           # How to deploy and what to check
    └── SKILL.md
```

### Using Skills

```
/api-patterns           → Load API patterns skill manually
/testing-strategy       → Load testing approach

# Or automatically: Copilot loads them when relevant
"Create a new user registration endpoint" 
→ Copilot auto-loads api-patterns skill
```

---

## 7. Hooks (Automation & Enforcement)

Hooks execute shell commands at key lifecycle points. They enforce team standards automatically.

### The 8 Hook Events

| Event | When | Use For |
|-------|------|---------|
| `SessionStart` | New session begins | Inject project context |
| `UserPromptSubmit` | User submits prompt | Audit/log requests |
| `PreToolUse` | Before any tool runs | Block dangerous ops |
| `PostToolUse` | After tool completes | Auto-format, run linters |
| `PreCompact` | Before context compacted | Save state |
| `SubagentStart` | Subagent spawned | Track nested agents |
| `SubagentStop` | Subagent completes | Aggregate results |
| `Stop` | Session ends | Cleanup, reports |

### Recommended Team Hooks

**`.github/hooks/formatting.json`** — Auto-format after edits:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "type": "command",
        "command": "bash .github/hooks/scripts/format-changed.sh",
        "windows": "powershell -File .github/hooks/scripts/format-changed.ps1",
        "timeout": 30
      }
    ]
  }
}
```

**`.github/hooks/scripts/format-changed.sh`**:

```bash
#!/bin/bash
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')

if [ "$TOOL_NAME" = "editFiles" ] || [ "$TOOL_NAME" = "createFile" ]; then
  # Run prettier on changed files
  FILES=$(echo "$INPUT" | jq -r '.tool_input.files[]? // .tool_input.path // empty')
  for FILE in $FILES; do
    if [ -f "$FILE" ]; then
      npx prettier --write "$FILE" 2>/dev/null
    fi
  done
fi

echo '{"continue":true}'
```

**`.github/hooks/security.json`** — Block dangerous commands:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "type": "command",
        "command": "bash .github/hooks/scripts/security-check.sh",
        "timeout": 5
      }
    ]
  }
}
```

**`.github/hooks/scripts/security-check.sh`**:

```bash
#!/bin/bash
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')
TOOL_INPUT=$(echo "$INPUT" | jq -r '.tool_input')

if [ "$TOOL_NAME" = "runTerminalCommand" ]; then
  CMD=$(echo "$TOOL_INPUT" | jq -r '.command // empty')
  
  # Block destructive operations
  if echo "$CMD" | grep -qiE '(rm\s+-rf|DROP\s+TABLE|DELETE\s+FROM\s+\w+\s*;|format\s+[a-z]:)'; then
    echo '{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"Destructive command blocked by team security policy — manual execution required"}}'
    exit 0
  fi
fi

echo '{"continue":true}'
```

**`.github/hooks/context.json`** — Inject project context at session start:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "type": "command",
        "command": "bash .github/hooks/scripts/inject-context.sh",
        "timeout": 10
      }
    ]
  }
}
```

**`.github/hooks/scripts/inject-context.sh`**:

```bash
#!/bin/bash
PROJECT=$(cat package.json 2>/dev/null | jq -r '.name + " v" + .version' || echo "Unknown")
BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
OPEN_ISSUES=$(git log --oneline -5 2>/dev/null | head -5 || echo "none")

cat <<EOF
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Project: $PROJECT | Branch: $BRANCH | Recent commits: $OPEN_ISSUES"
  }
}
EOF
```

---

## 8. Tool Sets

Group related tools for quick selection in prompts and agents:

**`.github/tool-sets.json`** (create via `Chat: Configure Tool Sets`):

```json
{
  "read-only": {
    "tools": ["read", "search"],
    "description": "Read-only analysis tools",
    "icon": "book"
  },
  "write-only": {
    "tools": ["edit"],
    "description": "File editing tools",
    "icon": "pencil"
  },
  "full-dev": {
    "tools": ["execute", "read", "edit", "search"],
    "description": "Full development workflow",
    "icon": "tools"
  }
}
```

Use in agents or prompts: `tools: ['read-only']`

---

## 9. MCP Servers

MCP extends Copilot with tools from external systems.
Configure in `.vscode/mcp.json` or user-level settings.

### Recommended MCP Servers for Teams

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${env:GITHUB_TOKEN}" }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": { "DATABASE_URL": "${env:DATABASE_URL}" }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
    }
  }
}
```

### GitHub MCP Usage

With the GitHub MCP server, you can do:

```
"Create an implementation plan for issue #42"
→ Agent fetches issue details, comments, labels automatically

"What issues are related to authentication?"
→ Searches GitHub issues with real context

"Create a PR for the feature I just implemented"
→ Reads branch, diffs, creates PR with description
```

---

## 10. The Team Development Workflow

### Standard Feature Lifecycle

Every feature follows this workflow:

```
1. PLAN   → Create implementation plan (Planner agent)
2. ASSIGN → Save plan, assign to developer
3. BUILD  → Implement from plan (TDD or Standard agent)
4. REVIEW → Code review (Reviewer agent)
5. MERGE  → Create PR description, merge
```

### Step-by-Step

**Step 1: Start a feature**

```
# In VS Code Chat, select "Planner" agent, then:
"Add user authentication with email/password including
 register, login, logout, and password reset"

# Or use the prompt:
/new-feature Add user authentication...
```

The Planner will:
- Search codebase for existing auth patterns
- Ask clarifying questions
- Generate a plan using your template
- Show "Start Implementation" handoff button

**Step 2: Save the plan**

```
# In chat, ask:
"Save this plan to docs/features/auth-plan.md"
```

**Step 3: Implement**

```
# Run `/execute`
# Or start fresh with:
"Run /execute for #docs/features/auth-plan.md and choose Mode A (Agent Mode) or Mode B (TDD Agent)"
```

**Step 4: Review**

```
# When implementation is done:
/code-review

# Or select Reviewer agent and ask:
"Review the changes I just made for the auth feature"
```

**Step 5: Create PR**

```
/create-pr-description
```

---

## 11. Agent Types: Choosing the Right One

Copilot offers 4 types of agents. Choose based on your task:

| Agent Type | Location | Best For | Key Characteristics |
|-----------|---------|---------|---------------------|
| **Local Agent** | VS Code Editor | Interactive development | Full editor context, requires your attention |
| **Background Agent** | Local machine (CLI) | Longer tasks while you work | Git worktrees isolate changes, runs locally |
| **Coding Agent** | GitHub cloud | Well-defined features, issues | Creates PRs, runs in GitHub Actions, async |
| **Cloud Agent** | GitHub | Team collaboration tasks | Access to GitHub context, async |

### When to Use Each

**Local (Interactive) Agent:**
- Exploring codebase: "How does our auth middleware work?"
- Quick fixes: inline, targeted code changes
- Planning sessions with back-and-forth discussion
- Anything needing your immediate attention

**Background Agent (Copilot CLI):**
- "Implement the plan in auth-plan.md" (while you work on something else)
- Larger, well-scoped implementation tasks
- Tasks where you've already done the planning
- Multiple features in parallel (one per background session)

**GitHub Coding Agent:**
- Assign GitHub issues directly: `@copilot`
- Features specified in detailed issue descriptions
- Bug fixes with clear reproduction steps
- Background work that continues even when VS Code is closed

### Parallel Development with Background Agents

```
Developer 1: Local session → Planning feature A
Developer 2: Background session → Implementing feature B (from plan)
Developer 3: Another background session → Bug fixing issue #23
AI Coding Agent: Working on issue #41 (assigned on GitHub)
```

---

## 12. Subagent Orchestration Patterns

Subagents let a coordinator agent delegate to specialized workers in parallel.

### Pattern 1: Coordinator + Workers

```markdown
# feature-builder.agent.md
---
name: Feature Builder
tools: ['agent', 'edit']
agents: ['Planner', 'Implementer', 'Reviewer']
---
You are a feature development coordinator. For each feature:
1. Use the Planner to create an implementation plan
2. Use the Implementer to write the code
3. Use the Reviewer to check for issues
4. If reviewer finds critical issues, use Implementer to fix them
Iterate until the implementation passes review.
```

### Pattern 2: Parallel Analysis

```markdown
# In chat or a prompt file:
Analyze this PR for issues. Run these reviews in parallel using subagents:
1. Security: injection risks, auth gaps, exposed data
2. Performance: N+1 queries, blocking operations, cache misses
3. Correctness: logic errors, edge cases, type safety
4. Architecture: consistency with codebase patterns

Then synthesize findings into a prioritized list.
```

### Pattern 3: Research Before Implementation

```
"Use a subagent to research the best WebSocket libraries for our Node.js
stack — compare socket.io, ws, and uWebSockets.js for our use case.
Then use the findings to implement real-time notifications."
```

### When to Use Subagents

- **Research** before implementation (keep main context clean)
- **Parallel analysis** of different concerns (security, perf, correctness)
- **Context isolation** for exploratory work
- **Specialized tools** per subtask (read-only researcher, write-capable implementer)

---

## 13. Context Engineering for Teams

Context is everything. Here's how to manage it for a team.

### Context Hierarchy

```
Global (applies always)
  └── .github/copilot-instructions.md → project-wide standards

Domain (applies to matching files)
  └── .github/instructions/*.instructions.md → frontend/backend/testing rules

Feature (applies on-demand)
  └── docs/features/my-feature-plan.md → reference in prompt with #file
  └── .github/agents/*.agent.md → specialized persona
  └── .github/skills/*/SKILL.md → domain knowledge
```

### Session Management

**Rule: One session per concern**
- Planning session → just planning
- Implementation session → just this feature
- Review session → just this PR
- Don't mix planning + implementation in one session

**Rule: Start fresh for unrelated tasks**

```
# Wrong ❌
"Now that auth is done, let's also add dark mode and fix the billing bug"

# Right ✅
Finish auth session → Close → New session for dark mode
Finish auth session → Close → New session for billing bug
```

**Rule: Use subagents for research**

```
# Wrong ❌
"What are the best practices for WebSocket auth?" 
(adds to context, pollutes implementation session)

# Right ✅
"Use a subagent to research WebSocket auth patterns — return only a recommendation"
(isolated, only the final answer comes back)
```

### Managing Long-Running Features

For features spanning multiple sessions:

```markdown
# docs/features/auth-plan.md (your living document)
---
title: User Authentication
status: in-progress  
last-updated: 2026-03-01
---

## Tasks
- [x] Database schema for users table
- [x] Registration endpoint
- [/] Login endpoint (in progress)
- [ ] Password reset flow
- [ ] Session management

## Current State
Implementation started on register endpoint. Next: login with JWT.

## Key Decisions Made
- Using JWTs (not sessions) because [reason]
- Password hashing: bcrypt with 10 rounds
```

Start each new session:
```
"Continue implementing the auth feature. Current state is in #docs/features/auth-plan.md"
```

---

## 14. Feature Development: End-to-End Walkthrough

Let's walk through building a "User Notifications" feature with a team.

### Day 1: Planning

**Developer (in VS Code chat, Planner agent selected):**

```
"We need to add real-time notifications. Users should see:
- New message notifications
- Task assignment alerts  
- System announcements
They need to mark as read and view history."
```

**Planner agent will:**
1. Search codebase for existing notification patterns
2. Check API patterns skill for conventions
3. Ask: "Should notifications persist after page refresh? Backend tech preference?"
4. Generate `docs/features/notifications-plan.md`

**Save the plan and share with team by committing it.**

### Day 2: Parallel Implementation

**Developer A works on backend** (starts background session with Implementer agent):

```
"Implement the backend for notifications: 
- Socket.io setup
- Notification service and repository  
- REST API endpoints
Use plan at #docs/features/notifications-plan.md backend tasks"
```

**Developer B works on frontend** (starts another background session):

```
"Implement the frontend for notifications UI:
- Notification bell component
- Notification center panel
- Real-time updates via WebSocket
Use plan at #docs/features/notifications-plan.md frontend tasks"
```

Both background sessions run in separate Git worktrees, no conflicts.

### Day 3: Integration + Review

**Developer C does review** (Reviewer agent):

```
"Review the changes on the notifications feature branch.
Run security, performance, and correctness reviews in parallel."
```

**Fixing review findings** (Implementer agent):

```
"Fix these review findings: [paste findings]
Prioritize the 🔴 CRITICAL items first."
```

### Day 4: Merge

```
/create-pr-description
```

PR is created with full context. Assign Copilot coding agent for any last-minute issues:
```
@copilot Fix the failing test in notifications.test.ts — the mock for socket.io is incorrect
```

---

## 15. Scaling: 10-15 Features Simultaneously

For large teams with many features in flight:

### Feature Namespacing

Organize features in the docs:

```
docs/features/
├── sprint-2/
│   ├── auth-plan.md           (✅ Done)
│   ├── notifications-plan.md  (🔄 In Progress)
│   └── dark-mode-plan.md      (📋 Planned)
└── sprint-3/
    ├── billing-plan.md        (📋 Planned)
    └── api-v2-plan.md         (📋 Planned)
```

### Domain-Based Instructions

Create more specific instructions files as features multiply:

```
.github/instructions/
├── auth.instructions.md            # Specific to auth module
├── notifications.instructions.md   # Specific to notifications
├── billing.instructions.md         # Billing-specific patterns
└── api-v2.instructions.md          # API v2 migration rules
```

### Team Agent Specialization

Different developers can specialize as different agents:

```
.github/agents/
├── backend-specialist.agent.md    # Deep backend expert
├── frontend-specialist.agent.md   # React/Next.js expert
├── api-designer.agent.md          # API contract design
└── security-auditor.agent.md      # Security-focused review
```

### Coding Agent for Batch Work

For clearly defined, well-documented features, use the Copilot coding agent:

```
# Create a GitHub issue with detailed spec:
Title: Implement dark mode toggle
Body: [Full spec from docs/features/dark-mode-plan.md]

# Assign to @copilot
# Agent creates PR, implements, runs tests
# Developers review the PR
```

This lets a team have 5+ features being worked on simultaneously:
- 3 developers actively planning/reviewing
- 4 background sessions implementing
- 3 GitHub coding agent sessions working on clear issues

### Managing Context Across Features

Prevent confusion by being explicit:

```
# Always start implementation sessions with:
"I'm working on the NOTIFICATIONS feature.
Ignore any code related to auth, billing, or other unrelated systems.
The relevant code is in src/notifications/* and the plan is #docs/features/notifications-plan.md"
```

---

## 16. Best Practices & Anti-Patterns

### ✅ Do This

**Context**
- Keep `copilot-instructions.md` concise (under 100 lines). Link to other docs.
- Scope domain-specific instructions with `applyTo` patterns
- Use subagents for research to keep main context clean
- Start new sessions for different features/concerns
- Save implementation plans to files for cross-session continuity

**Planning**
- Use the Planner agent before any implementation
- Get the plan reviewed by a teammate before implementing
- Answer the planner's clarifying questions — don't skip
- Keep plans updated as decisions change

**Implementation**
- Implement from plans, not from memory
- Use TDD — write tests first
- Run tests after every change
- Commit frequently (after each passing TDD cycle)

**Quality**
- Run the Reviewer agent before every PR
- Use hooks to auto-enforce formatting
- Use hooks to block dangerous commands
- Pin models in agents for consistency

### ❌ Avoid This

**Context Pollution**
- Don't put everything in `copilot-instructions.md`
- Don't mix unrelated tasks in one session
- Don't ask Copilot to explore alternatives in an implementation session
- Don't keep appending context — start fresh when context gets stale

**Planning Skips**
- Don't start implementing without a plan for anything > 2 hours of work
- Don't let Copilot decide the architecture — that's your team's job
- Don't accept the first plan without review

**Quality Gaps**
- Don't commit Copilot code without reviewing it
- Don't skip the reviewer agent because "it looks fine"
- Don't ignore security findings
- Don't hardcode credentials even in test files

**Token Waste**
- Don't enable 128 tools at once — use tool sets
- Don't ask Copilot to re-explain code it just wrote — start fresh
- Don't use the cloud coding agent for complex, ambiguous features — use local/background first

---

## 17. Quick Reference Cheat Sheet

### File Locations

| What | Where |
|------|-------|
| Global instructions | `.github/copilot-instructions.md` |
| Domain instructions | `.github/instructions/*.instructions.md` |
| Agents | `.github/agents/*.agent.md` |
| Slash commands | `.github/prompts/*.prompt.md` |
| Skills | `.github/skills/*/SKILL.md` |
| Hooks | `.github/hooks/*.json` |
| Feature plans | `docs/features/*.md` |

### Slash Commands Reference

| Command | What It Does |
|---------|-------------|
| `/init` | Auto-generates starter `copilot-instructions.md` |
| `/instructions` | Manage instruction files |
| `/prompts` | Manage prompt files |
| `/agents` | Manage custom agents |
| `/skills` | Manage agent skills |
| `/hooks` | Configure hooks |
| `/new-feature` | Start feature planning workflow |
| `/code-review` | Review current file |
| `/write-tests` | Generate tests |
| `/fix-bug` | Systematic bug fix |

### Context References in Chat

| Reference | What It Does |
|-----------|-------------|
| `#file-name.ts` | Include specific file |
| `#folder-name` | Include folder contents |
| `#symbol-name` | Include specific function/class |
| `#selection` | Include selected text |
| `#codebase` | Search whole codebase |
| `#changes` | Include current git changes |
| `#problems` | Include current errors |
| `#terminal` | Include terminal output |
| `#fetch URL` | Fetch web page content |
| `#githubRepo owner/repo` | Include GitHub repo context |

### Model Selection Guide

| Task | Recommended Model |
|------|-----------------|
| Planning / Architecture | Claude Opus / GPT-4o (best reasoning) |
| Implementation | Claude Sonnet / GPT-4o |
| Code review | Claude Opus / GPT-4o |
| Quick completions | Claude Haiku / GPT-4o Mini (faster) |
| Security analysis | Claude Opus (best at nuance) |

### Agent Type Decision Tree

```
Is the task well-defined with a clear spec?
  ├─ YES: Can it wait / be done async?
  │         ├─ YES: GitHub Coding Agent (assign @copilot on issue)
  │         └─ NO: Background Agent (while you work on something else)
  └─ NO: Local Agent (interactive planning first)
              ↓
         After planning:
              ↓
         Background Agent for implementation
```

---

*This guide is a living document. Update it as your team learns what works best.*
*Built from: VS Code official Copilot docs + Atomic GSD Hybrid methodology*
