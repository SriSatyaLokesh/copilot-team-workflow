---
layout: default
parent: Learn Series
title: "Part 08 — Using This Repo as a Boilerplate"
description: "Install into any project with PowerShell or Bash, what to customise, and what works out of the box."
nav_order: 9
---

# Learn: Using This Repo as a Boilerplate


> **Note**: The `learn/` folder lives at the **project root** of this boilerplate repo. It is a standalone guide — **not copied** into your project when you install.

---

## What This Repo Is

This repository is a **ready-to-use boilerplate** for setting up GitHub Copilot for any software development team. It contains:

- 7 pre-configured agents (Discuss, Research, Planner, TDD, Reviewer, Verify, ApiBuilder)
- 5 instruction files (developer guide, API architecture, backend, frontend, testing)
- 8 prompt slash commands (discuss, plan, execute, verify, add-new-api, code-review, generate-api-doc, update-api-doc)
- 2 automation hooks (session-auto-commit, session-logger)
- 1 skill (agent-activity-logger)
- 9 documentation templates (Issue, API doc, wrapper doc, external API, flow, copilot-instructions)
- A complete learn series (this series, 9 blog-style guides)

Everything is designed to drop into an existing project with **minimal customization**.

---

## Option 1: GitHub Template (Easiest)

If this repo is public and configured as a GitHub template:

1. Click **"Use this template"** on GitHub
2. Create your new repo
3. Follow [Part 7 — Team Lead Setup](./07-team-lead-setup.md) for customization

---

## Option 2: Install into an Existing Project

### PowerShell (Windows)

```powershell
# Navigate to your project
cd your-project

# Clone the boilerplate
git clone https://github.com/SriSatyaLokesh/copilot-best-practices-for-teams.git copilot-boilerplate

# Copy .github folder and docs structure (excludes learn/ — that's a boilerplate-only guide)
Copy-Item -Recurse copilot-boilerplate\.github .\
Get-ChildItem copilot-boilerplate\docs -Exclude 'learn' | Copy-Item -Destination .\docs -Recurse -Force

# Create required local log folders (never committed)
New-Item -ItemType Directory -Force logs\copilot

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

# Copy .github folder and docs structure (excludes learn/ — that's a boilerplate-only guide)
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

### VS Code Settings

After copying, add these to your workspace `.vscode/settings.json` then **reload VS Code**:

```json
{
  "github.copilot.chat.codeGeneration.useInstructionFiles": true,
  "chat.promptFilesLocations": [".github/prompts"],
  "chat.instructionsFilesLocations": [".github/instructions"]
}
```

---

## What to Customize (The Minimum Set)

After copying, these are the files you MUST edit for your project:

### 1. `.github/copilot-instructions.md`
```markdown
# [YOUR PROJECT NAME] — GitHub Copilot Configuration

## What This Project Is
[One sentence describing your project]
Stack: [Your tech stack]

## Key Documentation
[Your relevant doc links]
```

### 2. `.github/instructions/api-architecture.instructions.md`
- Update `applyTo` file globs to match your folder structure
- Update example file paths (e.g., `src/wrappers/` → wherever your wrappers live)
- Add any project-specific architecture rules

### 3. `.github/instructions/backend.instructions.md`
- Set your actual framework (Express, Fastify, NestJS, etc.)
- Set your validation library (zod, class-validator, joi)
- Set your ORM (Prisma, TypeORM, Drizzle, Sequelize)

### 4. `docs/external-apis/` (if you have external APIs)
- Create one folder per external API
- Copy and fill in `docs/templates/external-api-doc-template.md`
- Fill in the **field mapping table** for each entity — this is the highest-value work

---

## What to Leave As-Is

These files require **no changes** and work out of the box:

| File | Why it works as-is |
|------|-------------------|
| All 7 agents | Logic is generic, not project-specific |
| All 8 prompts | Commands work for any project |
| Both hooks | Shell scripts are project-agnostic |
| All templates | Generic — fill in per use |
| `developer-guide.instructions.md` | Generic developer guide |
| `learn/` folder (boilerplate root) | The learning series, read before installing |

---

## The Files in One View

```
.github/
├── copilot-instructions.md              ← EDIT: your project overview
├── instructions/
│   ├── developer-guide.instructions.md  ← works as-is
│   ├── api-architecture.instructions.md ← EDIT: your folder paths
│   ├── backend.instructions.md          ← EDIT: your stack
│   ├── frontend.instructions.md         ← EDIT: your frontend stack
│   └── testing.instructions.md          ← EDIT: your test framework
├── agents/
│   ├── discuss.agent.md                 ← works as-is
│   ├── research.agent.md                ← works as-is
│   ├── plan.agent.md                    ← works as-is
│   ├── tdd.agent.md                     ← works as-is
│   ├── review.agent.md                  ← works as-is
│   ├── verify.agent.md                  ← works as-is
│   └── api-builder.agent.md             ← works as-is
├── prompts/
│   ├── discuss.prompt.md                ← works as-is
│   ├── plan.prompt.md                   ← works as-is
│   ├── execute.prompt.md                ← works as-is
│   ├── verify.prompt.md                 ← works as-is
│   ├── add-new-api.prompt.md            ← works as-is
│   ├── code-review.prompt.md            ← works as-is
│   ├── generate-api-doc.prompt.md       ← works as-is
│   └── update-api-doc.prompt.md         ← works as-is
├── skills/
│   └── agent-activity-logger/SKILL.md   ← works as-is
└── hooks/
    ├── session-auto-commit/             ← works as-is
    └── session-logger/                  ← works as-is

docs/
├── templates/                           ← works as-is (copy-paste starters)
│   ├── issue-template.md
│   ├── api-doc-template.md
│   ├── api-wrapper-doc-template.md
│   ├── external-api-doc-template.md
│   ├── flow-doc-template.md
│   └── copilot-instructions-template.md
├── external-apis/                       ← CREATE: one folder per external API
├── issues/                              ← FILL: one doc per Issue
├── apis/                                ← FILL: one doc per endpoint
└── flows/                               ← FILL: one doc per user journey

learn/                                   ← standalone guide (not copied on install)
├── 00-introduction.md
├── 01-github-folder-explained.md
├── 02-five-phase-workflow.md
├── 03-api-architecture.md
├── 04-documenting-external-apis.md
├── 05-daily-workflow.md
├── 06-hooks-and-automation.md
├── 07-team-lead-setup.md
└── 08-boilerplate-setup.md              ← this file
```

---

## Estimated Setup Time

| Task | Time |
|------|------|
| Copy .github folder | 5 min |
| Update copilot-instructions.md | 15 min |
| Customize instruction files | 45 min |
| Document first external API | 60 min |
| Install and test hooks | 20 min |
| Brief the team | 30 min |
| **Total** | **~3 hours** |

---

## What You Get After Setup

✅ Every Copilot session loads architecture rules automatically based on the file open
✅ Every developer follows the same 5-phase workflow
✅ Correct field names used in every external API call (from docs)
✅ Every session is logged with timestamps and key decisions
✅ Every work item has a full paper trail (discuss → research → plan → execute → verify)
✅ New developers onboard in 30 minutes by reading the `learn/` series on the boilerplate repo

---

## Continue the Series

You've set up the system — now learn how to keep your code quality high with TDD.

**Next: Test-Driven Development (TDD) →** [09-test-driven-development.md](./09-test-driven-development.md)

You now know:
1. What every file in `.github/` does and why it exists
2. How the 5-phase Issue workflow prevents "just start coding" chaos
3. The architecture rules that keep every layer doing one job
4. How to document external APIs so Copilot uses the right field names
5. What your daily workflow looks like in practice
6. How hooks automate session management
7. How to set this up as a team lead
8. How to use this repo as a boilerplate for any project

**Share this series**: `learn/00-introduction.md` is the entry point for your whole team — link them to the boilerplate repo directly.

---

## Further Reading

- [GitHub Copilot Documentation](https://code.visualstudio.com/docs/copilot/overview) — everything covered in this series, with the official VS Code perspective
- [github/awesome-copilot](https://github.com/github/awesome-copilot) — community-contributed agents, instructions, and skills ready to copy into your setup
- [VS Code team's `.github/` folder](https://github.com/microsoft/vscode/tree/main/.github) — the gold standard: 13 instructions, 17 prompts, 11 skills in production
---

[← Part 07: Team Lead Setup](./07-team-lead-setup.md) · 📚 [Learn Series](../index.md) · [Part 09: TDD →](./09-test-driven-development.md)
