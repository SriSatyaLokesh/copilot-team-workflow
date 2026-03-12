---
layout: default
parent: "Copilot Team Workflow"
title: "Part 07 - Team Lead Setup Guide"
description: "First-time setup (3-4 hours), customisation steps, ongoing maintenance, and scaling to more developers."
nav_order: 8
---

# Learn: Team Lead Setup Guide


---

## Your Role as Team Lead

As a tech lead, your job is to set up the system once and keep it healthy.

After setup, developers should be able to:
- Start a Copilot session with zero configuration
- Get architecture rules automatically based on which file they open
- Use slash commands for every repeatable task
- Know exactly where to find and write documentation

---

## First-Time Setup (3–4 hours)

### Step 1: Copy the .github folder (30 min)

Follow the install script in [Part 08 — Boilerplate Setup](./08-boilerplate-setup.md) for the full copy-paste commands (PowerShell or Bash). In short:

```bash
# Navigate to your project
git clone https://github.com/SriSatyaLokesh/copilot-team-workflow.git copilot-boilerplate
cp -r copilot-boilerplate/.github ./
cp -r copilot-boilerplate/docs ./
rm -rf copilot-boilerplate
git add .github/ docs/
```

Or use this repo as a template via GitHub: **"Use this template" → Create new repository**.

### Step 2: Update copilot-instructions.md (20 min)

Open `.github/copilot-instructions.md` and complete the **SETUP REQUIRED** checklist at the top of the file:

1. Replace `[Project Name]` in the heading
2. Replace the `Stack:` line with your actual stack
3. Fill in `## Conventions` with project-specific rules
4. **Set your primary branch** — add this line to the `## Conventions` section:
   ```
   Primary branch: dev
   ```
   Replace `dev` with whatever branch your team cuts feature branches from (`main`, `develop`, `staging`, etc.). This is the line `/start-issue` reads every time a developer starts new work.

> **If you skip Step 4**, each developer will be asked "What is your team's primary branch?" the first time they run `/start-issue` and it will be written automatically. Setting it upfront means no one is ever asked.

**Fastest way for Step 3**: Type `/init` in Copilot Chat — it analyzes your workspace and generates tailored `## Conventions` content automatically.

```markdown
# [Your Project Name] — GitHub Copilot Configuration

## What This Project Is
[One sentence: what does this project do?]

Stack: [Node.js + TypeScript + Express + PostgreSQL | Next.js + Prisma | etc.]

## Key Documentation
- [Project Overview](https://github.com/SriSatyaLokesh/copilot-team-workflow/blob/main/docs/project.md)
- [API Architecture](../learn/03-api-architecture.md)
- [External APIs](https://github.com/SriSatyaLokesh/copilot-team-workflow/blob/main/docs/external-apis/)
- [Active Work Folders](https://github.com/SriSatyaLokesh/copilot-team-workflow/tree/main/work) (gitignored — local only)
- [Activity Logs](https://github.com/SriSatyaLokesh/copilot-team-workflow/tree/main/logs/copilot) (gitignored — local only)
- [Example Issue Docs](https://github.com/SriSatyaLokesh/copilot-team-workflow/tree/main/docs/issues) (templates and examples)

## Conventions
Primary branch: dev
[Other project-specific coding rules here]
```

### Step 3: Customize the instructions (1 hour)

Update each instruction file for your project's stack:

**`.github/instructions/backend.instructions.md`**
```markdown
---
applyTo: "src/**/*.ts"
---
# Backend Standards

## Framework
Express 4.x with TypeScript. Use async/await, never callbacks.

## Validation
Use zod for all request validation.

## Error handling
All errors must extend AppError: src/utils/errors.ts
```

**`.github/instructions/api-architecture.instructions.md`**
- Update the file paths in examples to match your project structure
- Add any project-specific rules (e.g., "All services must extend BaseService")

### Step 4: Document your external APIs (1–2 hours)

This is the most impactful setup step.

For each external API your project calls:
1. Create `docs/external-apis/[api-name]/`
2. Copy `docs/templates/external-api-doc-template.md` → `README.md`
3. Fill in: auth, base URL, rate limits, gotchas
4. Create one file per entity: `[entity].api.md`
5. **Fill in the field mapping table** — this is what prevents Copilot from guessing wrong field names

```bash
# Example for Dynamics
mkdir -p docs/external-apis/dynamics
cp docs/templates/external-api-doc-template.md docs/external-apis/dynamics/README.md
cp docs/templates/external-api-doc-template.md docs/external-apis/dynamics/accounts.api.md
# Edit each file...
```

### Step 5: Document your business flows (30 min)

For each major user journey (auth flow, order flow, etc.):
```bash
cp docs/templates/flow-doc-template.md docs/flows/auth-flow.md
# Edit with the actual flow steps
```

Flows are what Copilot reads when a developer is working on a feature that touches multiple parts of the system.

### Step 6: Install hooks (15 min)

```bash
chmod +x .github/hooks/session-auto-commit/auto-commit.sh
chmod +x .github/hooks/session-logger/*.sh
mkdir -p logs/copilot
echo "logs/" >> .gitignore
```

### Step 7: Configure VS Code workspace settings (5 min)

Create or update `.vscode/settings.json` in the project:

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

`chat.promptFilesRecommendations` surfaces `/start-issue` as a suggested action every time a developer opens a new chat — so they can't accidentally skip to coding without going through the workflow.

### Step 8: Brief the team (30 min)

Run a 30-minute session covering:
1. Read `learn/00-introduction.md` together
2. Demo: start a new Issue using `/start-issue` — show how it reads the primary branch from `copilot-instructions.md` and offers to create a fresh branch or stay on the current one
3. Show where docs live and how to find the right template
4. Explain the two rules: **always run `/start-issue` first** and **always read the Issue doc when resuming work**
5. Show the `argument-hint` in action — when selecting an agent in chat, Copilot shows placeholder text for what to type

---

## Ongoing Maintenance

### When an external API changes version

Update the relevant `docs/external-apis/[api-name]/` docs:
- New fields added → add rows to the field mapping table
- Fields deprecated → mark them `[DEPRECATED]` + note the replacement
- Auth method changed → update the README
- Rate limits changed → update the README

Commit the doc update in a separate commit from any code changes.

### When a new team member joins

1. Point them to `learn/` in this boilerplate repo — they read the series in order
2. Pair with them on their first Issue through all 5 phases
3. After that, they're independent

### When you want to add a new agent or prompt

Use the templates in `docs/templates/`:
- Agents: copy an existing agent from `.github/agents/`, modify for the new role
- Prompts: create a `.prompt.md` with the right frontmatter

### When the workflow isn't working for the team

The most common problems and fixes:

| Problem | Cause | Fix |
|:---|:---|:---|
| Copilot not following architecture rules | Instructions not scoped to right files | Check `applyTo` glob in the instruction frontmatter |
| Agent giving wrong field names | External API doc not filled in | Document the entity in `docs/external-apis/` |
| Context lost between sessions | Developer not reading work folder | Make it a team habit: every session starts with "Read work/ISSUE-XXX/plan.md and result.md" |
| Sessions not auto-committing | Hook not installed or not on default branch | Check `.github/hooks/` is committed to main |
| `/start-issue` keeps asking for primary branch | `Primary branch:` line missing from `copilot-instructions.md` | Add `Primary branch: dev` (or your branch) to the `## Conventions` section |

---

## What Developers Are Responsible For

Tech leads own the system. Developers own the content.

| Developer responsibility | What "done" looks like |
|:---|:---|
| Update Issue doc through all 5 phases | All 5 sections filled, `status: done` |
| Update API doc when endpoint changes | `/update-api-doc` run, doc committed with code |
| Use the external API doc before wrapper work | `/add-new-api` prompt used, doc existed first |
| Code + docs in same commit | No commit with code change but no doc update |

---

## Scaling to More Developers

For a team of 5–10 developers working on 3–5 Issues simultaneously:

1. **Branch naming**: `<type>/XXX-kebab-name` — consistent and searchable (e.g., `feat/042-login-rate-limiting`)
2. **Issue doc per developer**: own your Issue doc — don't edit others
3. **Team notes folder**: `docs/team-notes/[your-name]/` — personal scratch space, no conflicts
4. **External API docs**: team lead owns these — reviewed before wrapper code is written
5. **Agent activity log**: `logs/copilot/agent-activity.log` — local, not committed

**Next: Using This Repo as a Boilerplate →** [08-boilerplate-setup.md](./08-boilerplate-setup.md)

---

## Further Reading

- [GitHub Copilot for Business](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/about-github-copilot-in-your-organization) — how to manage Copilot access, policies, and audit logs as a team lead
- [VS Code Workspace Settings](https://code.visualstudio.com/docs/getstarted/settings#_workspace-settings) — how ``.vscode/settings.json`` workspace settings work and why they override user settings
- [Glob patterns](https://code.visualstudio.com/docs/editor/glob-patterns) — the syntax used in ``applyTo`` fields in instruction files
---

[← Part 06: Hooks and Automation](./06-hooks-and-automation.md) · 📚 [Learn Series](/) · [Part 08: Using as Boilerplate →](./08-boilerplate-setup.md)
