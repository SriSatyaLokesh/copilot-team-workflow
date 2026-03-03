---
description: 'Use when starting any new work item, feature, fix, or investigation — when a developer says "start a new issue", "begin new work", "I want to work on X", or when on main/master branch without a feature branch. Creates an isolated feature branch, verifies clean test baseline, then starts the discussion phase. Required before writing any code.'
agent: 'Discuss'
tools: ['terminal', 'editFiles']
---
# Start Issue — Branch First, Then Define

Before any requirements discussion or code, we create an isolated workspace.

**Issue description**: ${input:issue-description:What are you building? (e.g., "add rate limiting to login endpoint")}
**Issue ID**: ${input:issue-id:Issue ID (e.g., ISSUE-042)}

## Step 1 — Verify you are not on main

Run in terminal:
```bash
git branch --show-current
```

> If the output is `main` or `master`, continue to Step 2.
> If already on a feature branch, skip to Step 4.

## Step 2 — Pull latest main

```bash
git pull origin main
```

## Step 3 — Create and switch to a feature branch

```bash
git checkout -b issue/${input:issue-id:Issue ID}-${input:issue-slug:Short slug (e.g., login-rate-limiting)}
```

Confirm the branch was created:
```bash
git branch --show-current
```

Expected output: `issue/ISSUE-042-login-rate-limiting`

## Step 4 — Verify clean baseline (tests must pass before working)

```bash
npm test
```

> **If tests fail:** Report the failures. Ask the developer: "Existing tests are failing on this branch. Should we fix them before starting, or is this expected?"
> **Do NOT proceed** with new work on a red baseline without explicit developer approval.

> **If tests pass:** Announce: "✅ Branch is clean. X tests passing. Ready to define requirements."

## Step 5 — Begin requirements discussion

Now that we have an isolated branch with a clean baseline, begin the Discuss phase.

Ask: "Tell me about what you want to build with **${input:issue-description}**."

Create the Issue doc at: `docs/issues/${input:issue-id}-${input:issue-slug}.md`

---

## Step 6 — Switch to the Discuss agent

Branch created. Issue doc initialized. Baseline verified.

> ✅ **Your next step**: In the Copilot Chat panel, **select the `Discuss` agent** and describe what you want to build. The Discuss agent will guide you through defining requirements before any code is written.
