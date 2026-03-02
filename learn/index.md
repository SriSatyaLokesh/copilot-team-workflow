---
layout: home
title: "Copilot Team Workflow"
nav_order: 1
has_children: true
description: "A complete guide for software teams to use GitHub Copilot with structure, consistency, and discipline."
permalink: /
---

# GitHub Copilot Team Workflow — Learn Series

A **11-part, blog-style guide** for software teams who want to use GitHub Copilot as a structured development partner — not just autocomplete.

By the end of this series you will have:
- Copilot that knows your architecture and loads the right rules per file automatically
- A 5-phase workflow (Discuss → Research → Plan → Execute → Verify) every developer follows consistently
- External API docs that stop Copilot from guessing wrong field names
- Session hooks that auto-commit and log every agent session
- A boilerplate ready to drop into any project

---

## The Series

| Part | Title | What you learn | Read time |
|:---|:---|:---|:---|
| [00](./00-introduction.md) | **Getting Started** | Install Copilot, VS Code, extensions, enable customization | 5 min |
| [01](./01-github-folder-explained.md) | **.github Folder Explained** | Instructions, agents, prompts, skills, hooks — all 6 blocks | 8 min |
| [02](./02-five-phase-workflow.md) | **The 5-Phase Workflow** | Discuss → Research → Plan → Execute → Verify, with examples | 10 min |
| [03](./03-api-architecture.md) | **API Architecture** | Controller → Service → Wrapper → Transformer layer rules | 12 min |
| [04](./04-documenting-external-apis.md) | **Documenting External APIs** | Field mapping tables, auth docs, why this prevents runtime crashes | 10 min |
| [05](./05-daily-workflow.md) | **Daily Developer Workflow** | Start issue, resume work, external API integration, before PR | 8 min |
| [06](./06-hooks-and-automation.md) | **Hooks & Automation** | Auto-commit hook, session logger, agent activity log | 6 min |
| [07](./07-team-lead-setup.md) | **Team Lead Setup Guide** | First-time setup, customisation, ongoing maintenance | 10 min |
| [08](./08-boilerplate-setup.md) | **Using This as a Boilerplate** | Install into any project, what to customise, what works out of the box | 5 min |
| [09](./09-test-driven-development.md) | **Test-Driven Development (TDD)** | How to follow the Red-Green-Refactor ritual and avoid traps | 10 min |
| [10](./10-subagent-driven-development.md) | **Subagent-Driven Development** | Use specialized agents for parallel tasks and quality review | 8 min |

**Total read time: ~90 minutes**

---

## Who This Is For

| Role | What you get |
|:---|:---|
| **Developer new to Copilot** | Clear step-by-step workflow — no guessing |
| **Experienced developer** | Architecture-aware Copilot, no repeated context setting |
| **Tech Lead** | A system to coordinate parallel work across your team |
| **New hire** | A documented project where Copilot teaches you as you code |

---

## Quick Links

- 📥 **[Installation →](./08-boilerplate-setup.md)** — copy this into your project in 5 minutes
- 📖 **[Start Reading →](./00-introduction.md)** — begin at Part 00
- 🌟 **[GitHub Repo →](https://github.com/SriSatyaLokesh/copilot-best-practices-for-teams)** — browse the code

---

## References

- [VS Code Copilot Documentation](https://code.visualstudio.com/docs/copilot/overview)
- [github/awesome-copilot](https://github.com/github/awesome-copilot)
- [VS Code team's .github/ folder](https://github.com/microsoft/vscode/tree/main/.github)
