# Research & Design Decisions

> How we arrived at our Copilot team workflow — what we explored, why we ruled things out, and what we actually based our approach on.

---

## What We Explored

### 1. GSD (Get Stuff Done) for GitHub Copilot
- **Repo**: [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) / [Punal100/get-stuff-done-for-github-copilot](https://github.com/Punal100/get-stuff-done-for-github-copilot)
- **What it is**: A structured workflow for solo developers using Copilot with defined phases (Discuss → Plan → Execute → Verify) enforced via custom agents, prompts, and instructions.

### 2. GSD-Teams Plugin
- **Repo**: [ianwsperber/gsd-teams](https://github.com/ianwsperber/gsd-teams)
- **What it is**: An attempt to extend GSD for teams by giving each developer their own state file (`.gsd/[username].md`) so multiple people can run GSD phases without merge conflicts.

### 3. VS Code Repository (`.github/` folder)
- **What we read**: VS Code's actual `.github/` directory — 13 instruction files, 17 prompt files, 11 skills, 2 custom agents, and CI integration.
- **What we learned**: A real large-team Copilot setup by one of the most advanced engineering orgs in the world. Domain-specific instructions, modular skills, agents with handoffs — all real patterns in production.

### 4. Official VS Code Copilot Documentation
- **What we read**: Every doc under `vscode-docs/docs/copilot/` — agents, skills, prompt files, custom instructions, hooks, subagents, background agents, cloud agents, tools, best-practices.
- **Source of truth**: This is the official VS Code team guidance on how to use Copilot effectively.

---

## Why GSD Doesn't Work for Teams

GSD was designed with solo developers in mind. When you put 5+ developers on it, several fundamental problems emerge:

### Problem 1: Single shared state file

GSD tracks workflow state in one file (e.g., `.gsd/progress.md`). When two developers work simultaneously, every Git pull causes merge conflicts on this file. Every commit becomes a coordination problem.

> **GSD-Teams** tried to solve this with per-user files (`.gsd/john.md`, `.gsd/sarah.md`) — but this creates a new problem: there's no shared view of what the team is working on, and it's not standard tooling.

### Problem 2: Phase locking assumes one person

GSD's phase system (Discuss → Plan → Execute → Verify) assumes one person drives a feature from start to finish. In real teams, a PM discusses, a senior dev plans, a junior dev implements, and a different person reviews. GSD doesn't have a concept of "handoff" between people.

### Problem 3: Not integrated with team tooling

Teams use GitHub Issues, Jira, or Linear to track work. GSD creates a parallel tracking system (markdown files it manages) that doesn't connect to anything else. You'd be maintaining two sources of truth.

### Problem 4: The workflow is too prescriptive about tooling specifics

GSD works well when one developer controls the whole agent session start-to-finish. In a team, developers join mid-session, pick up context from others, and use different machines. GSD's state model doesn't handle this gracefully.

### Problem 5: Not an industry standard

GSD for teams is an experimental open-source project, not a battle-tested industry practice. Adopting it means betting on a non-standard tool that your team (and new hires) will have to learn from scratch.

---

## What GSD Is Actually Good For

**Solo developers or very small teams (1-2 people)** can use GSD effectively:
- One person, one feature, one session
- The phase structure (Discuss → Plan → Execute → Verify) is excellent for preventing "just start coding" chaos
- The custom agent setup gives a structured, repeatable process

If a developer wants to use GSD personally for their own workflow sessions, that's a great use of it. The phase thinking is sound — we borrowed it.

---

## What We Actually Based Our Approach On

Our team Copilot workflow is derived from two sources only:

### Source 1: VS Code team's real `.github/` setup
- Domain-scoped instruction files (`applyTo:` glob patterns)
- Modular skills per capability
- Custom agents with clear roles and handoffs
- Prompt files as slash commands for repeatable tasks

### Source 2: Official VS Code Copilot Documentation
Key patterns we adopted from the docs:
- **Context engineering**: Load only what's relevant, not everything
- **Agent types**: Local (interactive) → Background (async, git worktrees) → Cloud (GitHub Coding Agent)
- **Session hygiene**: One session per concern, start fresh for unrelated tasks
- **Subagent orchestration**: Coordinator + parallel workers for complex reviews
- **Hooks**: Lifecycle automation for formatting, security, and context injection

### What we added on top
- **Feature-level docs structure** (`docs/apis/`, `docs/flows/`, `docs/issues/`) — so Copilot loads only the right context, not the whole project
- **Issue lifecycle** (our term for GSD's phase concept) — without GSD's shared state file problem, using GitHub Issues as the tracker instead
- **Merge-conflict-free parallel work** — `docs/team-notes/[name]/` personal folders, feature branches, API docs updated on merge only

---

## Summary: Our Approach Is

> **VS Code Copilot documentation patterns** + **VS Code team's real `.github/` structure** + **feature-level docs** organized for targeted context loading.

Not GSD for teams. Not a third-party plugin. Native Copilot features, used the way the VS Code team uses them.
