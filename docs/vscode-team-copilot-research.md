# VS Code Team Copilot Usage — Research Report

> **Research Date:** 2026-03-02  
> **Source:** Local clone of `microsoft/vscode` at `temp_repos/vscode/` (depth=1)  
> **Researcher:** Agent 2 — VS Code Copilot Patterns

---

## Summary

The VS Code team (100+ developers) uses GitHub Copilot extensively. This is not ad-hoc — they have a **structured, deeply layered customization system** baked directly into their `.github/` folder. The key insight for a smaller team: they solve the "large team Copilot" problem through **context segmentation**, not a single monolithic instructions file.

---

## 1. What Copilot Customization Files They Use

### Primary Instructions: `.github/copilot-instructions.md`
**File found:** ✅ `.github/copilot-instructions.md` (9,057 bytes)

This is their **global architecture + coding standards** file. It does NOT contain workflow steps — it contains:
- **Project architecture overview** — exact folder paths and what lives where (`src/vs/base/`, `src/vs/platform/`, etc.)
- **MANDATORY build validation rule** — "Always check the `VS Code - Build` watch task output before declaring work complete"
- **Coding guidelines** — tabs not spaces, PascalCase types, camelCase functions, no `any`, JSDoc comments
- **Code quality rules** — how to handle disposables, when to use which service, no duplicated code
- **Test patterns** — snapshot assertions over multiple precise ones, no tests in wrong suite

**Key Learning:** Their copilot-instructions.md is a **living, authoritative coding contract** — not a tutorial. Every rule is prescriptive ("Use PascalCase", "NEVER run tests if there are compilation errors").

### Multi-Model Support: `AGENTS.md` + `CLAUDE.md` (implicit)
**AGENTS.md found:** ✅ Root level  
Content: Just a pointer to `.github/copilot-instructions.md` — they consolidate into one authoritative file and reference it from all model-specific files.

---

## 2. Domain-Specific Instructions (The Secret Weapon)

**Found:** ✅ 13 domain-specific `.instructions.md` files in `.github/instructions/`

This is their core "context segmentation" strategy — instead of one massive file, each domain gets its own instruction file that Copilot loads when working in that area:

| File | Domain |
|------|---------|
| `accessibility.instructions.md` | Accessibility guidelines for UI components |
| `ai-customization.instructions.md` | Rules for building Copilot/AI customizations |
| `api-version.instructions.md` | API versioning protocols |
| `chat.instructions.md` | Chat feature implementation patterns |
| `disposable.instructions.md` | IDisposable lifecycle management |
| `interactive.instructions.md` | Interactive Notebooks subsystem |
| `kusto.instructions.md` | Data query patterns (KQL) |
| `learnings.instructions.md` | Accumulated gotchas and best practices |
| `notebook.instructions.md` | Notebook rendering architecture |
| `observables.instructions.md` | Observable/event patterns |
| `sessions.instructions.md` | Agent Sessions subsystem |
| `telemetry.instructions.md` | Telemetry event naming and patterns |
| `tree-widgets.instructions.md` | Tree/List widget component guidelines |

**Key Learning for a Smaller Team:** You don't need 13 files to start. Begin with 3–5 domain instructions covering your core areas (e.g., `api.instructions.md`, `ui.instructions.md`, `database.instructions.md`). Copilot automatically loads the relevant one based on your CWD.

---

## 3. Reusable Skills Library

**Found:** ✅ 11 skills in `.github/skills/`

| Skill | Purpose |
|-------|---------|
| `accessibility/` | Audit and fix accessibility issues |
| `agent-sessions-layout/` | Work with the Sessions panel layout |
| `author-contributions/` | Find who wrote what code |
| `azure-pipelines/` | Understand/modify CI pipelines |
| `component-fixtures/` | Use component-level test fixtures |
| `fix-errors/` | Fix TypeScript/compile errors |
| `hygiene/` | Code hygiene and linting |
| `memory-leak-audit/` | Find and fix memory leaks |
| `sessions/` | Sessions feature subsystem |
| `tool-rename-deprecation/` | Handle API deprecations |
| `update-screenshots/` | Update test screenshots |

**Key Learning:** Skills encode **team knowledge** that would otherwise live in someone's head. The `memory-leak-audit` skill is a perfect example — no one needs to mentor on disposable patterns; Copilot just knows.

---

## 4. Custom Agents for Specialized Roles

**Found:** ✅ 2 custom agents in `.github/agents/`

### `data.md` — Data Analyst Agent
- **Tools:** Kusto Explorer MCP, Azure MCP, telemetry extensions
- **Role:** Answer telemetry questions by running real KQL queries against Azure Data Explorer
- **Key pattern:** Uses MCP servers to give Copilot direct data access — eliminates the "ask someone on the data team" bottleneck

### `demonstrate.md` — QA Demonstration Agent
- **Tools:** `vscode-playwright-mcp/` (browser automation), GitHub MCP
- **Role:** Automatically explore and demonstrate UI changes from PRs, captures screenshots/recordings
- **Key pattern:** Uses Playwright MCP to visually verify and showcase feature changes, attached to the PR

**Key Learning:** These agents are **specialized roles** — not everyone needs to run them. A small team can start with just one agent (e.g., a design-system agent or a backend API agent).

---

## 5. Prompt Files as Team-Shared Commands

**Found:** ✅ 17 prompt files in `.github/prompts/`

| Prompt | Purpose |
|--------|---------|
| `implement.prompt.md` | Standard implementation workflow |
| `plan-fast.prompt.md` | Quick planning (less research) |
| `plan-deep.prompt.md` | Deep planning with full research |
| `fix-error.prompt.md` | Structured error fix flow |
| `fixIssueNo.prompt.md` | Fix a GitHub issue by number |
| `find-duplicates.prompt.md` | Find duplicate code |
| `doc-comments.prompt.md` | Generate documentation comments |
| `migrate.prompt.md` | Migrate from old API to new |
| `no-any.prompt.md` | Remove TypeScript `any` types |
| `micro-perf.prompt.md` | Micro-performance optimization |
| `component.prompt.md` | Create a new component |
| `build-champ.prompt.md` | Build system champion tasks |
| `codenotify.prompt.md` | CODENOTIFY integration |
| `find-issue.prompt.md` | Find related issues |
| `issue-grouping.prompt.md` | Group and organize issues |
| `setup-environment.prompt.md` | Dev environment setup |
| `update-instructions.prompt.md` | Update instruction files |

**Key Learning:** Prompt files are **team-level consistency tools**. Instead of everyone prompting differently, `@plan-deep` or `@implement` gives every developer the same structured, high-quality Copilot interaction.

---

## 6. CI/CD Integration

**Found:** ✅ `copilot-setup-steps.yml` in `.github/workflows/`

The VS Code team has a `copilot-setup-steps.yml` workflow — this is GitHub's standard way of setting up the environment for Copilot-driven coding agents (Copilot Workspace / GitHub Copilot Agent Mode). It installs dependencies so Copilot can run commands (like building and testing) during agent sessions.

They also have:
- `check-clean-git-state.sh` — enforces no dirty state before merges
- `no-engineering-system-changes.yml` — guards build system from unauthorized changes
- `screenshot-test.yml` — automated screenshot regression tests

**Key Learning:** The `copilot-setup-steps.yml` workflow is the **bridge between Copilot and CI** — it lets Copilot agents run tests and builds in the cloud, not just locally.

---

## 7. Hooks — Runtime Behavior Control

**Found:** ✅ `.github/hooks/hooks.json`

The VS Code team uses hooks to control Copilot agent behavior at runtime. This is advanced usage — hooks can intercept tool calls before they execute (`preToolUse`) or after (`postToolUse`) to enforce policies.

---

## 8. Lessons for a Team of 5–15 Developers

| VS Code Practice | Team of 5–15 Adaptation |
|---|---|
| Global `copilot-instructions.md` with architecture + coding rules | ✅ Adopt immediately. Start with: architecture overview, tech stack, coding conventions |
| Domain `.instructions.md` files | Start with 3–5 for your core domains (e.g., API, UI, DB) |
| 17+ prompt files | Start with 4–5 core prompts: `implement`, `plan`, `fix-error`, `review` |
| 11 skills | Identify your top 3 team knowledge pain points and encode as skills |
| AGENTS.md pointing to copilot-instructions.md | ✅ Adopt immediately — zero cost, enables all AI models |
| `copilot-setup-steps.yml` CI workflow | Adopt when using GitHub Copilot Workspace or agent sessions |
| Custom agents (`data`, `demonstrate`) | Build role-specific agents for your team's unique expertise gaps |
| CODEOWNERS + CODENOTIFY | ✅ Adopt — reduces review bottlenecks for large teams |

---

## 9. What They DON'T Do

- ❌ No GSD-style planning files (`.gsd/`, `.planning/`) — they rely on issues and GitHub Projects
- ❌ No per-feature context isolation (no `STATE.md` per feature) — they use domain instructions files instead
- ❌ No "consolidate" or "share" commands — they rely on PRs for state transitions
- ❌ No explicit "parallel work" orchestration — their contribution model (separate `workbench/contrib/` folders per feature) provides natural isolation

**Core VS Code insight:** Their strategy is **"Copilot as a very well informed team member"** — load the architecture, coding rules, and domain knowledge into Copilot's context, then get out of the way. They don't script workflows; they encode knowledge.
