---
layout: default
parent: "Copilot Team Workflow"
title: "Part 01 - The .github Folder Explained"
description: "A tour of all 6 building blocks: copilot-instructions, instructions, agents, prompts, skills, and hooks."
nav_order: 2
---

# Learn: What's in the .github Folder?

[← Part 00: Getting Started](./00-introduction.md) · 📚 [Learn Series](../index.md) · [Part 02: The 5-Phase Workflow →](./02-five-phase-workflow.md)

---

## The Big Idea

The `.github/` folder is **Copilot's brain for your project**. Every file in it is a configuration that teaches Copilot something — how your project works, what your standards are, which specialist to call, what to do automatically.

You set it up once. After that, every developer on the team gets the same context automatically, every session.

---

## The 6 Building Blocks

### 1. `copilot-instructions.md` — The Project Overview

**Always loaded. Every session. Every file.**

This is the one file that is loaded into Copilot's context no matter what. It should contain:
- What the project does (one sentence)
- The tech stack
- Links to the key docs

```markdown
# MyProject — GitHub Copilot Configuration

## What This Project Is
An order management API that integrates with Microsoft Dynamics 365 CRM.
Stack: Node.js + TypeScript + Express + PostgreSQL + Redis

## Key Documentation
- [Project Overview](../docs/PROJECT.md)
- [API Architecture](../learn/03-api-architecture.md)
- [External APIs](../docs/external-apis/)
```

**Keep it short.** It loads every session — if it's 500 lines, it wastes context.

---

### 2. `instructions/` — Rules Per File Type

**Loaded automatically when you open a matching file.**

Instructions are the most powerful feature. Write a rule once, it applies everywhere automatically.

```
.github/instructions/
├── developer-guide.instructions.md   ← applyTo: "**"  (always on)
├── api-architecture.instructions.md  ← applyTo: "src/api/**,src/wrappers/**"
├── backend.instructions.md           ← applyTo: "src/**/*.ts"
├── frontend.instructions.md          ← applyTo: "src/**/*.tsx"
└── testing.instructions.md           ← applyTo: "**/*.test.*"
```

The `applyTo` field in the frontmatter tells Copilot which files trigger this instruction:

```markdown
---
applyTo: "src/wrappers/**,src/services/**"
---
# API Architecture Standards
...rules about the Controller→Service→Wrapper→Transformer pattern...
```

When a developer opens `src/wrappers/dynamics-wrapper.ts`, Copilot automatically loads the API architecture rules. **Without the developer doing anything.**

> **Real example**: The VS Code team has `typescript.instructions.md` that applies to all `.ts` files — every TypeScript suggestion automatically follows their internal standards.

---

### 3. `agents/` — Specialist Assistants

**Selected by the developer or triggered by handoff.**

Agents are like hiring a specialist for each phase of work. Each agent file defines:
- What this agent is good at
- Which tools it can use
- What it does step by step
- Which agent it hands off to next

```
.github/agents/
├── discuss.agent.md          ← Phase 1: clarifies requirements
├── research.agent.md         ← Phase 2: reads codebase before planning
├── plan.agent.md             ← Phase 3: creates task list
├── tdd.agent.md              ← Phase 4: writes tests first, then code
├── review.agent.md           ← Reviews code before PR
├── verify.agent.md           ← Phase 5: checks Issue is done
└── api-builder.agent.md      ← Specialist for external API integrations
```

Example agent frontmatter:
```markdown
---
description: 'Discuss a new Issue — defines scope and requirements'
name: Discuss
tools: ['search', 'codebase', 'editFiles']
model: 'gpt-4o'
handoffs:
  - label: Start Research →
    agent: Research
    send: true        ← auto-triggers Research without asking
---
```

The `handoffs` field creates a button in Copilot chat. `send: true` means it fires automatically.

---

### 4. `prompts/` — Slash Commands

**Triggered by the developer with `/command`.**

Prompts are reusable, parameterized instructions your team runs as slash commands. They're how you trigger a workflow with one command instead of typing a long prompt each time.

```
.github/prompts/
├── discuss.prompt.md          ← /discuss
├── plan.prompt.md             ← /plan
├── execute.prompt.md          ← /execute
├── verify.prompt.md           ← /verify
├── add-new-api.prompt.md      ← /add-new-api
├── code-review.prompt.md      ← /code-review
├── generate-api-doc.prompt.md ← /generate-api-doc
└── update-api-doc.prompt.md   ← /update-api-doc
```

Example prompt file:
```markdown
---
description: 'Add a new external API call — walks through the full architecture'
agent: 'ApiBuilder'
tools: ['editFiles', 'terminal', 'search']
---
You are adding a new external API call.

First, read: #docs/external-apis/${input:api-name}/README.md
Then implement: Transformer → Wrapper → Service → Controller → Tests
```

The `${input:api-name}` syntax creates an input field in the chat — Copilot asks the developer to fill it in before starting.

---

### 5. `skills/` — Bundled Knowledge Packs

**Referenced by agents. Loaded when the agent decides it needs them.**

A skill is a folder containing a `SKILL.md` — a detailed reference document that an agent can read when it needs deep context on a specific area.

```
.github/skills/
└── agent-activity-logger/SKILL.md  ← Log format for session audit trail
```

Skills are different from instructions:
- **Instructions** → auto-loaded per file type, always active
- **Skills** → an agent reads them on-demand when it needs specialized knowledge

> **VS Code team has 11 skills** including one for writing vscode extension tests, one for localization strings, one for accessibility checks.

---

### 6. `hooks/` — Scripts That Run Automatically

**Triggered by Copilot agent lifecycle events.**

Hooks let you run shell scripts at specific moments — session start, session end, every prompt submitted.

```
.github/hooks/
├── session-auto-commit/
│   ├── hooks.json         ← { "sessionEnd": [run auto-commit.sh] }
│   └── auto-commit.sh     ← git add -A && git commit && git push
└── session-logger/
    ├── hooks.json         ← { "sessionStart": ..., "sessionEnd": ..., "userPromptSubmitted": ... }
    ├── log-session-start.sh
    ├── log-session-end.sh
    └── log-prompt.sh
```

The `hooks.json` defines when each script runs:
```json
{
  "version": 1,
  "hooks": {
    "sessionEnd": [
      { "type": "command", "bash": ".github/hooks/session-auto-commit/auto-commit.sh" }
    ]
  }
}
```

Available events:

| Event | When it fires |
|:---|:---|
| `sessionStart` | runs when agent session opens |
| `sessionEnd` | runs when agent session closes |
| `userPromptSubmitted` | runs on every prompt |

---

## How It All Connects

```
Developer opens src/wrappers/dynamics-wrapper.ts
        ↓
Copilot auto-loads:
  - copilot-instructions.md (project overview)
  - developer-guide.instructions.md (always on)
  - api-architecture.instructions.md (matches src/wrappers/**)

Developer types: /add-new-api
        ↓
add-new-api.prompt.md activates → ApiBuilder agent launches
        ↓
ApiBuilder reads:
  - docs/external-apis/dynamics/ (field maps for Dynamics CRM)
  - .github/instructions/api-architecture.instructions.md (standards)
        ↓
Agent writes code → session ends

hooks/session-auto-commit/auto-commit.sh runs automatically
hooks/session-logger/log-session-end.sh logs the session
```

---

## Summary

| File/Folder | Loaded When | Purpose |
|:---|:---|:---|
| `copilot-instructions.md` | Every session | Project overview |
| `instructions/*.instructions.md` | File matches `applyTo` | Standards per file type |
| `agents/*.agent.md` | Developer selects or handoff | Specialist assistant |
| `prompts/*.prompt.md` | Developer types `/command` | Reusable workflow |
| `skills/*/SKILL.md` | Agent reads it | Deep knowledge reference |
| `hooks/*/hooks.json` | Session event fires | Automation |

**Next: The 5-Phase Issue Workflow →** [02-five-phase-workflow.md](./02-five-phase-workflow.md)

---

## Further Reading

- [VS Code Copilot — Custom Instructions](https://code.visualstudio.com/docs/copilot/copilot-customization) — full reference for `applyTo`, frontmatter, and instruction scoping
- [VS Code Copilot — Agents](https://code.visualstudio.com/docs/copilot/chat/chat-agents) — how agent files work, handoffs, and available tools
- [github/awesome-copilot](https://github.com/github/awesome-copilot) — 172 community agents and 175 instructions to explore and add to your setup
---

[← Part 00: Getting Started](./00-introduction.md) · 📚 [Learn Series](../index.md) · [Part 02: The 5-Phase Workflow →](./02-five-phase-workflow.md)
