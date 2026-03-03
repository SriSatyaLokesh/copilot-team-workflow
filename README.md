# GitHub Copilot Team Workflow — Boilerplate

A production-ready `.github/` configuration that turns GitHub Copilot into a structured, consistent AI development partner for your team.

**What you get in ~3 hours of setup:**
- 7 specialist agents (Discuss, Research, Planner, TDD, Reviewer, Verify, ApiBuilder)
- 8 slash command prompts (`/discuss`, `/plan`, `/execute`, `/verify`, `/add-new-api`, `/code-review`, `/generate-api-doc`, `/update-api-doc`)
- Auto-loading instructions per file type (architecture rules, commenting standards, doc-on-change)
- Playwright testing skills and instructions
- Session hooks (auto-commit + structured activity log)
- 9 documentation templates (Issue, API, flow, external-API)
- A 11-part beginner-friendly learn series in `learn/` (at repo root — not copied on install)

> **Learn more**: Start with [`learn/00-introduction.md`](./learn/00-introduction.md) or read it as a website: **[srisatyalokesh.github.io/copilot-best-practices-for-teams](https://srisatyalokesh.github.io/copilot-best-practices-for-teams)**

---

## 🚀 Installation

### PowerShell (Windows)

```powershell
# Navigate to your project
cd your-project

# Clone the boilerplate
git clone https://github.com/SriSatyaLokesh/copilot-best-practices-for-teams.git copilot-boilerplate

# Copy .github folder and docs structure (excludes learn/ — that stays in the boilerplate repo)
Copy-Item -Recurse copilot-boilerplate\.github .\
Get-ChildItem copilot-boilerplate\docs -Exclude 'learn' | Copy-Item -Destination .\docs -Recurse -Force

# Create required local log folders (never committed)
New-Item -ItemType Directory -Force logs\copilot

# Make hook scripts executable (if using WSL/Git Bash)
# git update-index --chmod=+x .github/hooks/session-auto-commit/auto-commit.sh
# git update-index --chmod=+x .github/hooks/session-logger/*.sh

# Add logs folder to .gitignore
Add-Content .gitignore "`nlogs/"

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
git clone https://github.com/SriSatyaLokesh/copilot-best-practices-for-teams.git copilot-boilerplate

# Copy .github folder and docs structure (excludes learn/ — that stays in the boilerplate repo)
cp -r copilot-boilerplate/.github ./
rsync -av --exclude='learn/' copilot-boilerplate/docs/ ./docs/ 2>/dev/null || { cp -r copilot-boilerplate/docs ./ && rm -rf docs/learn; }

# Create required local log folders (never committed)
mkdir -p logs/copilot

# Make hook scripts executable
chmod +x .github/hooks/session-auto-commit/auto-commit.sh
chmod +x .github/hooks/session-logger/*.sh

# Add logs folder to .gitignore
echo "logs/" >> .gitignore

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
    "debug": true
  }
}
```

> **`chat.promptFilesRecommendations`**: Surfaces `/start-issue` and `/debug` as suggested actions when opening a new chat session — ensures developers always start from the correct entry point.

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
├── prompts/                             ← 12 slash commands
│   ├── start-issue.prompt.md            ← /start-issue (Entry point — always start here)
│   ├── discuss.prompt.md                ← /discuss (branch gate enforced)
│   ├── research.prompt.md               ← /research
│   ├── plan.prompt.md                   ← /plan
│   ├── execute.prompt.md                ← /execute
│   ├── verify.prompt.md                 ← /verify
│   ├── debug.prompt.md                  ← /debug
│   ├── add-new-api.prompt.md            ← /add-new-api
│   ├── code-review.prompt.md            ← /code-review
│   ├── receive-review.prompt.md         ← /receive-review
│   ├── generate-api-doc.prompt.md       ← /generate-api-doc
│   └── update-api-doc.prompt.md         ← /update-api-doc
├── skills/                              ← 10 auto-loading knowledge packs
│   ├── agent-activity-logger/SKILL.md   ← Log format reference
│   ├── doc-reviewer/SKILL.md            ← Brutal doc review (Critical/Major/Minor)
│   ├── documentation-writer/SKILL.md    ← Diátaxis-guided doc creation
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

learn/                               ← 10-part beginner guide (standalone, not copied on install)
docs/
├── templates/                           ← Copy-paste starters for every doc type
├── external-apis/                       ← Fill in: one folder per external API
├── issues/                              ← Fill in: one doc per work item
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
