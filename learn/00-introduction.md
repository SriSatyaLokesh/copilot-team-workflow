---
layout: default
parent: "Copilot Team Workflow"
title: "Part 00 - Getting Started"
description: "Install GitHub Copilot, set up VS Code, enable customization, and install the boilerplate."
nav_order: 1
---

# Learn: GitHub Copilot for Teams — Getting Started


---

## What Is GitHub Copilot?

GitHub Copilot is an AI assistant built into VS Code. Most developers know it as **autocomplete** — it suggests the next line of code as you type. But it's also a full **chat assistant** and can run as an **agent** that plans, writes, tests, and reviews code on your behalf.

This series focuses on using Copilot as an **agent** — a structured AI workflow partner that knows your project and follows your team's standards.

---

## Step 1 — Get a Copilot Plan

You need a GitHub Copilot subscription. Go to:

👉 **[github.com/features/copilot](https://github.com/features/copilot)**

| Plan | Who it's for | Price |
|:---|:---|:---|
| **Copilot Free** | Individuals (limited requests/month) | Free |
| **Copilot Individual** | Solo developers (unlimited) | $10/month |
| **Copilot Business** | Teams (centrally managed) | $19/user/month |
| **Copilot Enterprise** | Large orgs (custom policies) | $39/user/month |

> For most teams: **Copilot Business** gives each developer unlimited access and lets a team lead manage settings centrally.

---

## Step 2 — Install VS Code

If you don't have VS Code:

👉 **[code.visualstudio.com](https://code.visualstudio.com)**

Download and install for your OS (Windows / macOS / Linux). It's free.

---

## Step 3 — Install the GitHub Copilot Extension

1. Open VS Code
2. Click the **Extensions** icon in the left sidebar (or press `Ctrl+Shift+X` / `Cmd+Shift+X`)
3. Search for **"GitHub Copilot"**
4. Click **Install** on the extension by GitHub

You'll see two extensions — install both:
- **GitHub Copilot** — the inline autocomplete
- **GitHub Copilot Chat** — the chat panel and agent mode

5. After installing, VS Code will ask you to **sign in with GitHub** — sign in with the account that has your Copilot subscription

---

## Step 4 — Open Copilot Chat

Once signed in, you'll see a **chat icon** in the left sidebar (looks like a speech bubble with a star).

Click it. You now have access to:
- **Ask** — general coding questions
- **Edit** — make changes to files
- **Agent** — run a full AI agent session

> **Where you'll spend most time in this system**: the **Agent** tab, where you select an agent (like `Discuss` or `Planner`) and it guides your work through phases.

---

## Step 5 — Enable Copilot's Customization Features

By default, VS Code doesn't automatically load your `.github/instructions/` files. You need to turn this on **once per project** by adding these settings to your workspace `.vscode/settings.json`:

```json
{
  "github.copilot.chat.codeGeneration.useInstructionFiles": true,
  "chat.promptFilesLocations": [".github/prompts"],
  "chat.instructionsFilesLocations": [".github/instructions"]
}
```

**How to do this**:
1. In VS Code, press `Ctrl+Shift+P` (or `Cmd+Shift+P`)
2. Type **"Open Workspace Settings (JSON)"** and press Enter
3. Paste the JSON above inside the `{}` brackets
4. Save the file
5. **Reload VS Code** (`Ctrl+Shift+P` → "Reload Window")

> After reload, Copilot will automatically load the right instructions based on which file you have open — without you doing anything else.

---

## Step 6 — Install This Boilerplate

The `learn/` folder you're reading now stays in this boilerplate repo as a reference guide. When you install into your project, only `.github/` and `docs/` (templates, examples) are copied — **`learn/` is excluded**. See [Part 08 — Boilerplate Setup](./08-boilerplate-setup.md) for full instructions, or use the quick install:

**Browse as a website**: **[srisatyalokesh.github.io/copilot-team-workflow](https://srisatyalokesh.github.io/copilot-team-workflow)**

### PowerShell (Windows)
```powershell
cd your-project
git clone https://github.com/SriSatyaLokesh/copilot-team-workflow.git copilot-boilerplate
Copy-Item -Recurse copilot-boilerplate\.github .\
Get-ChildItem copilot-boilerplate\docs -Exclude 'learn' | Copy-Item -Destination .\docs -Recurse -Force
New-Item -ItemType Directory -Force logs\copilot
Add-Content .gitignore "`nlogs/"
Remove-Item -Recurse -Force copilot-boilerplate
git add .github/ docs/ .gitignore
git commit -m "chore: add Copilot team workflow boilerplate"
git push
```

### Bash (Linux / macOS)
```bash
cd your-project
git clone https://github.com/SriSatyaLokesh/copilot-team-workflow.git copilot-boilerplate
cp -r copilot-boilerplate/.github ./
rsync -av --exclude='learn/' copilot-boilerplate/docs/ ./docs/ 2>/dev/null || { cp -r copilot-boilerplate/docs ./ && rm -rf docs/learn; }
mkdir -p logs/copilot
chmod +x .github/hooks/session-auto-commit/auto-commit.sh
chmod +x .github/hooks/session-logger/*.sh
echo "logs/" >> .gitignore
rm -rf copilot-boilerplate
git add .github/ docs/ .gitignore
git commit -m "chore: add Copilot team workflow boilerplate"
git push
```

> Reload VS Code after committing. Copilot will now load the project instructions automatically.

---

## What You Can Do Now

Open Copilot Chat → select the **Agent** tab → choose **"Discuss"** → describe a task.

That's it. The Discuss agent will guide you through the first phase of the workflow — Discuss → Research → Plan → Execute → Verify — step by step.

---

## The Problem This Solves

Every team using Copilot hits the same wall:

> *"Copilot doesn't know our architecture — it keeps suggesting the wrong pattern."*
> *"Every developer uses Copilot differently — no consistency."*
> *"Context gets lost between sessions — we repeat ourselves every day."*
> *"We don't know what decisions the AI made during a session."*

This system solves all of those by teaching Copilot your project once — and having every developer follow the same structured workflow.

---

## Who This Is For

| Role | What you get |
|:---|:---|
| **Developer (new to Copilot)** | Clear step-by-step workflow, no guessing |
| **Developer (experienced)** | Architecture-aware Copilot, no repeated context setting |
| **Tech Lead** | Coordination system for parallel work across the team |
| **New hire** | A documented project where Copilot teaches you as you code |

---

## The Full Series

| Part | Topic | Read time |
|:---|:---|:---| 
| **00** | Getting Started — Installation & Setup (this file) | 5 min |
| **01** | [What's in the .github folder?](./01-github-folder-explained.md) | 8 min |
| **02** | [The 5-Phase Issue Workflow](./02-five-phase-workflow.md) | 10 min |
| **03** | [API Architecture — The Layer Rules](./03-api-architecture.md) | 12 min |
| **04** | [Documenting External APIs](./04-documenting-external-apis.md) | 10 min |
| **05** | [Your Daily Developer Workflow](./05-daily-workflow.md) | 8 min |
| **06** | [Hooks — What Runs Automatically](./06-hooks-and-automation.md) | 6 min |
| **07** | [Customizing for your project — what to fill in](./07-team-lead-setup.md) | 10 min |
| **08** | [Using This Repo as a Boilerplate](./08-boilerplate-setup.md) | 5 min |
| **09** | [Test-Driven Development (TDD)](./09-test-driven-development.md) | 10 min |
| **10** | [Subagent-Driven Development](./10-subagent-driven-development.md) | 8 min |

**Total read time: ~90 minutes**

**Continue to Part 01 →** [01-github-folder-explained.md](./01-github-folder-explained.md)

---

## Further Reading

- [GitHub Copilot Documentation](https://code.visualstudio.com/docs/copilot/overview) — full reference for agents, instructions, prompts, and customization
- [GitHub Copilot Plans & Pricing](https://github.com/features/copilot) — current plan details
- [github/awesome-copilot](https://github.com/github/awesome-copilot) — community collection of agents, instructions, and skills to add to your setup
- [Get Stuff Done for Copilot](https://github.com/Punal100/get-stuff-done-for-github-copilot) — the solo-developer alternative to this system
---

📚 [Learn Series](/) · [Part 01: .github Folder Explained →](./01-github-folder-explained.md)
