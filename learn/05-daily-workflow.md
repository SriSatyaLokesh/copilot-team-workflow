---
layout: default
parent: "Copilot Team Workflow"
title: "Part 05 — Daily Developer Workflow"
description: "Start an issue, resume work, work on external APIs, and prepare a PR — the full day in practice."
nav_order: 6
---

# Learn: Your Daily Developer Workflow


---

## The Problem with Ad-Hoc Copilot Use

Without a workflow, every developer has a different Copilot habit:
- Some describe requirements in every prompt (slow)
- Some forget to add tests
- Some commit AI-generated code without reviewing
- Context is lost at the end of every session

This guide shows the **single workflow every developer follows** — beginner or senior.

---

## Starting a New Issue

### Morning: A new task comes in

```
Product: "We need to add rate limiting to the login endpoint"
```

**Step 1**: Create a branch

```bash
git checkout -b issue/issue-042-login-rate-limiting
```

**Step 2**: Open Copilot Chat. Select **"Discuss"** agent.

> If you're starting a new feature, bug, or task, always start with the **Discuss** agent.
> It helps define the requirements and creates your Issue doc.

**Step 3**: Describe the task

```
"We need to add rate limiting to the login endpoint. 
Max 5 attempts per 15 minutes per email address."
```

**Step 4**: The **Discuss** agent will define the requirements.

The Discuss agent will ask you focused questions. Answer them — it takes about 10 minutes to get clear, agreed requirements.

**Step 5**: Discuss finishes → **Research** auto-starts

You'll see Copilot scanning the codebase. It will find relevant files and existing patterns.

**Step 6**: Review research findings, click **"Create Plan →"**

The Planner creates a task checklist. Review each item — if anything looks wrong or is missing, say so now.

**Step 7**: Approve the plan, click **"Start Implementation (TDD) →"**

The TDD agent takes over. It writes tests first, then implementation, committing as it goes.

**Step 8**: Implementation finishes → run `/verify`

The Verify agent checks all requirements, tests, and docs. It gives you a pass/fail report.

**Step 9**: Fix anything the Verify agent caught, then create the PR.

---

## Resuming Work the Next Day

The most important habit: **tell Copilot to read the Issue doc FIRST**.

```
"Continue issue-042. Read #docs/issues/issue-042-login-rate-limiting.md first."
```

Copilot reads:
- Phase 1 requirements (what was agreed)
- Phase 2 research (what files are involved)
- Phase 3 plan (what tasks remain)
- Phase 4 progress (what's done, what's next)

No re-explaining. No "as we discussed" context setting. Just pick up from where you left off.

---

## Working on an External API

This is the most specific workflow — external API work has a required prep step.

**Before writing any code, open the external API doc**:

```
"Read #docs/external-apis/dynamics/accounts.api.md"
```

Then run `/add-new-api`:

```
/add-new-api
→ Which external API? dynamics
→ Which entity? accounts
→ What operation? get by ID
```

The ApiBuilder agent reads the external API doc → writes transformer → writes wrapper → writes service → writes controller → writes tests.

> **Rule**: Never open a wrapper file without first telling Copilot to read the external API doc.

---

## Before Committing

Run these manually before every `git commit`:

```bash
npm test                # all tests must pass
npm run lint            # no lint errors
npx tsc --noEmit        # no TypeScript errors
```

If you changed an endpoint:

```
/update-api-doc
```

Copilot reads the git diff and updates `docs/apis/` for you.

**Commit format**:

```
issue-042 feat: add rate limiting to login endpoint

Updated:
- docs/apis/auth/login.api.md (added 429 response, rate limit rules)
- docs/issues/issue-042-login-rate-limiting.md (Phase 4 progress)
```

Code and docs in the **same commit**, always.

---

### Step 5 — Requesting Code Review (`/review`)

**Goal**: Catch architectural or performance issues before they reach your lead.

**What to do**:
Before you create a PR, run the `/review` command. This dispatches a specialized `code-reviewer` agent that audits your diff against the requirements in the Issue doc.

**How to act on feedback**:
- **Critical**: Fix immediately.
- **Important**: Fix before merging.
- **Minor**: Note for a future issue.

---

### Step 6 — Creating the PR

1. Run `/verify` — get the verification report
2. Fix every ❌ item the Verify agent found
3. Ask Copilot: **"Write a PR description for issue-xxx from the issue doc"** — paste the output into GitHub
4. Request a teammate to review

The reviewer runs `/code-review` on the PR files for a systematic multi-dimension review.

---

## Switching Between Issues

**One Copilot session per Issue.** When you switch Issues, start a fresh chat session.

```
# Finishing issue-042
/verify → passed → open PR → done for now

# Starting issue-043
[New Copilot Chat session]
"Continue issue-043. Read #docs/issues/issue-043-export-orders.md first."
```

Mixing issues in one session causes context bleed — Copilot starts mixing requirements from different Issues.

---

## Quick Command Reference

| Situation | Command |
|-----------|---------|
| Starting a new Issue | Select Discuss agent → describe task |
| Resuming an issue | "Read #docs/issues/issue-xxx.md first" |
| Planning code | `/plan` |
| Implementing | `/execute` |
| Adding external API | `/add-new-api` |
| Reviewing code | `/code-review` |
| Before PR | `/verify` → ask Copilot for PR description |
| Changed endpoint | `/update-api-doc` |
| New endpoint | `/generate-api-doc` |

---

## The Daily Checklist

```
Morning:
□ git pull (get latest from team)
□ Check open Issues in docs/issues/ — what's your priority today?
□ Open Copilot → read Issue doc → start where you left off

During work:
□ One session per Issue
□ Tests before implementation (TDD agent handles this)
□ Commit small: one test + one implementation per commit
□ Changed an external API call → update the external API doc first

End of day:
□ npm test — all green?
□ git push
□ Update Phase 4 progress tracker in Issue doc manually if needed
□ (Session auto-commit hook runs automatically when session ends)
```

---

## What You Do NOT Need to Do

❌ Configure Copilot before each session (instructions load automatically)
❌ Explain your architecture in every prompt (instructions carry it)
❌ Remember external API field names (external API docs carry them)
❌ Write PR descriptions manually (ask Copilot "Write a PR description from the Issue doc")
❌ Manually commit at session end (session-auto-commit hook does it)

**Next: Hooks and Automation →** [06-hooks-and-automation.md](./06-hooks-and-automation.md)

---

## Further Reading

- [Git branching strategies](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) — the branch-per-issue pattern this workflow uses
- [Conventional Commits](https://www.conventionalcommits.org/) — the ``issue-042 feat:`` commit format explained
- [VS Code Copilot — Slash Commands](https://code.visualstudio.com/docs/copilot/copilot-chat#_slash-commands) — how prompt files become slash commands
---

[← Part 04: Documenting External APIs](./04-documenting-external-apis.md) · 📚 [Learn Series](../index.md) · [Part 06: Hooks and Automation →](./06-hooks-and-automation.md)
