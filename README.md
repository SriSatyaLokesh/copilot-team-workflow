# GitHub Copilot Team Workflow — Boilerplate (V2)

A production-ready `.github/` configuration that turns GitHub Copilot into a structured, consistent AI development partner for your team.

**What you get in ~3 hours of setup:**
- 8 specialist agents (Discuss, Research, Planner, TDD, Reviewer, Verify, ApiBuilder, ParallelBuilder)
- 15 slash command prompts (`/start-issue`, `/discuss`, `/research`, `/plan`, `/execute`, `/verify`, `/debug`, `/add-new-api`, `/finish-branch`, `/generate-api-doc`, `/update-api-doc`, `/receive-review`, `/status`, `/sync-docs`, `/summarize`)
- Auto-loading instructions per file type (architecture rules, commenting standards, doc-on-change)
- 11 auto-loading skills (TDD, doc-reviewer, GitHub CLI workflow, Playwright, code review patterns)
- Playwright testing skills and instructions
- Session hooks (auto-commit + structured activity log)
- 9 documentation templates + 7 codebase knowledge templates
- GitHub CLI integration (automatic issue/PR creation and merge)
- Work folder structure (`work/ISSUE-XXX-name/`) to prevent merge conflicts
- Activity logging (`logs/copilot/agent-activity.log`) for audit trail
- Dual-mode execution (Agent Mode for complex features, TDD Agent for routine work)
- A 11-part beginner-friendly learn series in `learn/` (at repo root — not copied on install)

> **What's New in V2** (March 2026): Work folder structure, GitHub CLI integration, dual-mode execution, fixed agent handoff chain, activity logging. See [MIGRATION-V2.md](./docs/MIGRATION-V2.md) for upgrade guide.

> **Learn more**: Start with [`learn/00-introduction.md`](./learn/00-introduction.md) or read it as a website: **[srisatyalokesh.github.io/copilot-team-workflow](https://srisatyalokesh.github.io/copilot-team-workflow)**

---

## 🚀 Installation

### PowerShell (Windows)

```powershell
# Navigate to your project
cd your-project

# Clone the boilerplate
git clone https://github.com/SriSatyaLokesh/copilot-team-workflow.git copilot-boilerplate

# Copy .github folder and docs structure (excludes learn/ — that stays in the boilerplate repo)
Copy-Item -Recurse copilot-boilerplate\.github .\
Get-ChildItem copilot-boilerplate\docs -Exclude 'learn' | Copy-Item -Destination .\docs -Recurse -Force

# Create required local folders (logs and work folders never committed)
New-Item -ItemType Directory -Force logs\copilot, work

# Make hook scripts executable (if using WSL/Git Bash)
# git update-index --chmod=+x .github/hooks/session-auto-commit/auto-commit.sh
# git update-index --chmod=+x .github/hooks/session-logger/*.sh

# Add logs folder to .gitignore
Add-Content .gitignore "`nlogs/`nwork/"

# Clean up
Remove-Item -Recurse -Force copilot-boilerplate

# Commit
git add .github/ docs/ .gitignore
git commit -m "chore: add Copilot team workflow boilerplate"
git push
```

### Bash (Linux / macOS)

```bash
# Navigate to your project
cd your-project

# Clone the boilerplate
git clone https://github.com/SriSatyaLokesh/copilot-team-workflow.git copilot-boilerplate

# Copy .github folder and docs structure (excludes learn/ — that stays in the boilerplate repo)
cp -r copilot-boilerplate/.github ./
rsync -av --exclude='learn/' copilot-boilerplate/docs/ ./docs/ 2>/dev/null || { cp -r copilot-boilerplate/docs ./ && rm -rf docs/learn; }

# Create required local folders (logs and work folders never committed)
mkdir -p logs/copilot work

# Make hook scripts executable
chmod +x .github/hooks/session-auto-commit/auto-commit.sh
chmod +x .github/hooks/session-logger/*.sh

# Add logs and work folders to .gitignore
echo -e "logs/\nwork/" >> .gitignore

# Clean up
rm -rf copilot-boilerplate

# Commit
git add .github/ docs/ .gitignore
git commit -m "chore: add Copilot team workflow boilerplate"
git push
```

---

## ⚙️ VS Code Settings

After copying, enable Copilot customization features in your **workspace** `.vscode/settings.json`:

```json
{
  "github.copilot.chat.codeGeneration.useInstructionFiles": true,
  "chat.promptFilesLocations": [".github/prompts"],
  "chat.instructionsFilesLocations": [".github/instructions"],
  "chat.promptFilesRecommendations": {
    "start-issue": true,
    "status": true,
    "debug": true
  }
}
```

> **`chat.promptFilesRecommendations`**: Surfaces `/start-issue`, `/status`, and `/debug` as suggested actions when opening a new chat session — ensures developers always start from the correct entry point and can quickly check their progress.

Then **reload VS Code** to pick up the agents, instructions, and prompts.

> 💡 **Auto-generate project instructions**: Type `/init` in Copilot Chat to have Copilot analyze your workspace and generate tailored instructions. Paste the output into `.github/copilot-instructions.md` under `## Conventions`.

---

## ✏️ What to Customize

After installation, edit these 4 files for your project:

| File | What to change |
|------|----------------|
| `.github/copilot-instructions.md` | Your project name, tech stack, key doc links |
| `.github/instructions/api-architecture.instructions.md` | Your folder paths (`src/controllers/`, `src/wrappers/`, etc.) |
| `.github/instructions/backend.instructions.md` | Your framework, ORM, validation library |
| `.github/instructions/frontend.instructions.md` | Your frontend stack (React, Next.js, Vue, etc.) |
| `.github/skills/acquire-codebase-knowledge/SKILL.md` | Initial codebase mapping questions and standards |

> 📘 **Model Selection**: All agents and prompts come with optimized model assignments (GPT-4o, Claude Sonnet/Opus, Haiku). See [`docs/MODEL-SELECTION.md`](./docs/MODEL-SELECTION.md) for the rationale and how to customize.

Everything else works out of the box.

---

## 📁 What's Included

```
.github/
├── copilot-instructions.md              ← Edit: your project overview + SETUP REQUIRED block
├── tool-sets.json                       ← Pre-defined tool groups (read-only, full-dev, review)
├── instructions/
│   ├── developer-guide.instructions.md  ← Always loaded — workflow overview
│   ├── api-architecture.instructions.md ← Loaded on src/wrappers/**, src/services/**
│   ├── backend.instructions.md          ← Loaded on src/**/*.ts
│   ├── frontend.instructions.md         ← Loaded on src/**/*.tsx
│   ├── testing.instructions.md          ← Loaded on **/*.test.*
│   ├── playwright-typescript.instructions.md         ← Playwright test standards
│   ├── self-explanatory-code-commenting.instructions.md ← Comment WHY not WHAT
│   ├── update-docs-on-code-change.instructions.md   ← Doc sync on code change
│   └── parallel-agents.instructions.md  ← When and how to use parallel agents
├── agents/                              ← 8 specialist agents
│   ├── discuss.agent.md                 ← Phase 1: requirements (argument-hint ✅)
│   ├── research.agent.md                ← Phase 2: codebase exploration (argument-hint ✅)
│   ├── plan.agent.md                    ← Phase 3: task breakdown (argument-hint ✅)
│   ├── tdd.agent.md                     ← Phase 4: TDD implementer (argument-hint ✅)
│   ├── review.agent.md                  ← Code quality reviewer
│   ├── verify.agent.md                  ← Phase 5: done-check (argument-hint ✅)
│   ├── api-builder.agent.md             ← External API integrations (argument-hint ✅)
│   └── parallel-builder.agent.md        ← Orchestrator: dispatches independent tasks in parallel
├── prompts/                             ← 15 slash commands
│   ├── start-issue.prompt.md            ← /start-issue (Entry point — always start here)
│   ├── discuss.prompt.md                ← /discuss (define requirements)
│   ├── research.prompt.md               ← /research (find existing patterns)
│   ├── plan.prompt.md                   ← /plan (create implementation tasks)
│   ├── execute.prompt.md                ← /execute (dual-mode: Agent Mode or TDD Agent)
│   ├── verify.prompt.md                 ← /verify (check completeness)
│   ├── debug.prompt.md                  ← /debug (troubleshooting)
│   ├── add-new-api.prompt.md            ← /add-new-api (external API integration)
│   ├── receive-review.prompt.md         ← /receive-review (handle feedback)
│   ├── finish-branch.prompt.md          ← /finish-branch (merge/PR/discard)
│   ├── generate-api-doc.prompt.md       ← /generate-api-doc (new API endpoint doc)
│   ├── update-api-doc.prompt.md         ← /update-api-doc (sync existing doc)
│   ├── status.prompt.md                 ← /status (check current phase)
│   ├── summarize.prompt.md              ← /summarize (save session context)
│   └── sync-docs.prompt.md              ← /sync-docs (bulk doc updates)
├── workflows/                           ← Specialist workflows
│   └── acquire-codebase-knowledge.md    ← Factual codebase mapping procedure
├── skills/                              ← 11 auto-loading knowledge packs
│   ├── acquire-codebase-knowledge/SKILL.md ← Factual codebase mapping (Arch, Stack, etc.)
│   ├── agent-activity-logger/SKILL.md   ← Log format reference
│   ├── doc-reviewer/SKILL.md            ← Brutal doc review (Critical/Major/Minor)
│   ├── documentation-writer/SKILL.md    ← Diátaxis-guided doc creation
│   ├── github-cli-workflow/SKILL.md     ← GitHub issue/PR automation patterns
│   ├── playwright-automation-fill-in-form/SKILL.md
│   ├── playwright-explore-website/SKILL.md
│   ├── playwright-generate-test/SKILL.md
│   ├── receiving-code-review/SKILL.md   ← Evaluate-before-implement pattern
│   ├── requesting-code-review/SKILL.md  ← Subagent review dispatch
│   ├── subagent-driven-development/SKILL.md ← Per-task subagent + 2-stage review
│   └── test-driven-development/SKILL.md ← Iron Law TDD + Red-Green-Refactor
└── hooks/
    ├── session-auto-commit/             ← Auto-commit on session end (Stop)
    └── session-logger/                  ← JSON audit log (SessionStart/Stop/UserPromptSubmit)

learn/                               ← 11-part beginner guide (standalone, not copied on install)
work/                                ← Active issue work folders (gitignored, local only)
logs/                                ← Activity logs (gitignored, local only)
docs/
├── MIGRATION-V2.md                      ← Upgrade guide from V1 to V2
├── MODEL-SELECTION.md                   ← Model assignment rationale (GPT-4o, Claude, Haiku)
├── templates/                           ← Copy-paste starters for every doc type
│   ├── plan-template.md                 ← Template for work/ISSUE-XXX/plan.md
│   ├── result-template.md               ← Template for work/ISSUE-XXX/result.md
│   └── ...
├── codebase/                            ← Factual codebase knowledge (Arch, Stack, etc.)
├── external-apis/                       ← Fill in: one folder per external API
├── issues/                              ← (Deprecated in V2) Example issue docs only
├── apis/                                ← Fill in: one doc per endpoint
└── flows/                               ← Fill in: one doc per user journey
```

---

## 📖 Learn the System

The `learn/` folder at the root of this repo is a **standalone guide** — share it with your team or read it before installing. It is **not copied** when you install the boilerplate into your project.

| Part | Topic |
|------|-------|
| [00](./learn/00-introduction.md) | Getting Started — Installation & Setup |
| [01](./learn/01-github-folder-explained.md) | .github folder — all 6 building blocks explained |
| [02](./learn/02-five-phase-workflow.md) | 5-Phase Issue workflow (Discuss → Verify) |
| [03](./learn/03-api-architecture.md) | API Architecture — the layer rules |
| [04](./learn/04-documenting-external-apis.md) | Documenting external APIs before writing wrappers |
| [05](./learn/05-daily-workflow.md) | Day-to-day developer workflow |
| [06](./learn/06-hooks-and-automation.md) | Hooks — what runs automatically |
| [07](./learn/07-team-lead-setup.md) | Team lead setup guide |
| [08](./learn/08-boilerplate-setup.md) | Using this repo as a boilerplate |
| [09](./learn/09-test-driven-development.md) | Test-Driven Development (TDD) |
| [10](./learn/10-subagent-driven-development.md) | Subagent-Driven Development |

---

## Requirements

- GitHub Copilot (Individual, Business, or Enterprise plan)
- VS Code with the GitHub Copilot extension
- `jq` installed for session-logger hook (`brew install jq` / `sudo apt install jq`)
- `bash` 4+ for hook scripts (Windows: use Git Bash or WSL)

---

## 👤 Building Solo? Use This Instead

This boilerplate is **designed for teams** — structured phases, documentation standards, and multi-developer coordination.

If you're a **solo developer building your own app**, we recommend:

### [Get Stuff Done for GitHub Copilot](https://github.com/Punal100/get-stuff-done-for-github-copilot)

A lightweight Plan → Execute → Verify workflow optimized for individual developers who need to move fast with minimal ceremony. Start there, and migrate to this boilerplate when your team grows.

---

## 📚 References & Further Reading

This boilerplate was built by studying how professional teams configure Copilot. These are the primary sources:

| Resource | What you'll learn |
|----------|------------------|
| [github/awesome-copilot](https://github.com/github/awesome-copilot) | Community collection of 172 agents, 175 instructions, 361 skills, hooks, and plugins. Browse this to find more tools to add to your setup. |
| [VS Code Copilot Documentation](https://code.visualstudio.com/docs/copilot/overview) | Official docs — agents, instructions, prompt files, customization, and all Copilot features explained in depth. |
| [VS Code repository .github/](https://github.com/microsoft/vscode/tree/main/.github) | How the team that builds VS Code uses Copilot internally — 13 instructions, 17 prompts, 11 skills in production. The gold standard for real-world usage. |
