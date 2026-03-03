---
layout: default
parent: "Copilot Team Workflow"
title: "Part 06 - Hooks and Automation"
description: "Session auto-commit, session logger, and agent activity log — what runs automatically."
nav_order: 7
---

# Learn: Hooks — What Runs Automatically


---

## What Are Hooks?

Hooks are shell scripts that Copilot runs automatically at specific moments during an agent session — without the developer having to do anything.

Think of them as event listeners for your AI sessions.

**Available events** (VS Code uses **PascalCase**):

| Event | When it fires |
|:---|:---|
| `SessionStart` | Copilot agent session opens |
| `Stop` | Copilot agent session closes |
| `UserPromptSubmit` | Developer sends any prompt |
| `PreToolUse` | Before any tool is invoked |
| `PostToolUse` | After any tool completes |

This repo includes two pre-built hooks:
1. **session-auto-commit** — saves your work automatically when a session ends
2. **session-logger** — creates an audit trail of every session

---

## How Hooks Work

Each hook is a folder in `.github/hooks/` with:
- `hooks.json` — declares which events trigger which scripts
- One or more shell scripts
- `README.md` — explains what the hook does

```
.github/hooks/
├── session-auto-commit/
│   ├── hooks.json
│   └── auto-commit.sh
└── session-logger/
    ├── hooks.json
    ├── log-session-start.sh
    ├── log-session-end.sh
    └── log-prompt.sh
```

The `hooks.json` format (VS Code PascalCase — `Stop` is the session-end event):
```json
{
  "hooks": {
    "Stop": [
      {
        "type": "command",
        "command": ".github/hooks/session-auto-commit/auto-commit.sh",
        "windows": "powershell -ExecutionPolicy Bypass -File .github/hooks/session-auto-commit/auto-commit.ps1",
        "timeout": 30
      }
    ]
  }
}
```

> **Windows**: The `windows` property overrides `command` on Windows — add a `.ps1` equivalent for each `.sh` script.

> Hooks only activate once the `.github/hooks/` folder is committed to the **default branch** of the repo.

---

## Hook 1: Session Auto-Commit

**Problem it solves**: A background Copilot agent session runs for 30 minutes, writes code, then the session ends. If the developer doesn't commit manually, all that work is uncommitted.

**What it does**:
1. Checks if there are uncommitted changes
2. `git add -A`
3. `git commit --no-verify -m "auto-commit: 2026-03-02 21:30:00 UTC"`
4. `git push`

**File**: `.github/hooks/session-auto-commit/auto-commit.sh`

**To skip it** (e.g., you're mid-refactor and not ready to commit):
```bash
export SKIP_AUTO_COMMIT=true
```

**To install in your project**:
```bash
cp -r .github/hooks/session-auto-commit /your-project/.github/hooks/
chmod +x .github/hooks/session-auto-commit/auto-commit.sh
git add .github/hooks/ && git commit -m "chore: add auto-commit hook" && git push
```

---

## Hook 2: Session Logger

**Problem it solves**: No visibility into how developers are using Copilot. When something goes wrong or a bad pattern appears in code, there's no trail to understand how Copilot sessions contributed.

**What it does**:

| Event | Script | Output file |
|:---|:---|:---|
| `sessionStart` | `log-session-start.sh` | `logs/copilot/session.log` |
| `sessionEnd` | `log-session-end.sh` | `logs/copilot/session.log` |
| `userPromptSubmitted` | `log-prompt.sh` | `logs/copilot/prompts.log` |

**Log format** (JSON Lines):
```json
{"timestamp":"2026-03-02T16:00:00Z","event":"sessionStart","cwd":"/workspace/project"}
{"timestamp":"2026-03-02T16:01:00Z","event":"userPromptSubmitted","level":"INFO"}
{"timestamp":"2026-03-02T16:45:00Z","event":"sessionEnd"}
```

> **camelCase vs PascalCase**: The hook event names in `hooks.json` use PascalCase (`SessionStart`, `Stop`, `UserPromptSubmit`) — these are the names VS Code recognizes. The log labels in the output above (`sessionStart`, `sessionEnd`, `userPromptSubmitted`) are camelCase because the shell scripts write them independently. They are two separate things: the trigger name (PascalCase, for VS Code) and the log label (camelCase, written by the script).

**Privacy**: Full prompt text is **never** logged. Only event metadata (timestamp, event type).

**Important**: Add `logs/` to `.gitignore`. These files are local only — never committed.

```bash
echo "logs/" >> .gitignore
```

**To install**:
```bash
cp -r .github/hooks/session-logger /your-project/.github/hooks/
chmod +x .github/hooks/session-logger/*.sh
mkdir -p logs/copilot
echo "logs/" >> .gitignore
git add .github/hooks/ .gitignore && git commit -m "chore: add session-logger hook" && git push
```

Requires `jq` installed:
- macOS: `brew install jq`
- Ubuntu: `sudo apt install jq`
- Windows (WSL): `sudo apt install jq`

---

## The Agent Activity Log

Beyond session events, each phase agent writes a structured log entry to `logs/copilot/agent-activity.log` when it completes work.

This is written by the agents themselves (using the `editFiles` tool), not by a hook.

**Example entries**:
```json
{"timestamp":"2026-03-02T16:00:00Z","issueId":"issue-042","phase":"discuss","status":"complete","summary":"Agreed: rate limiting after 5 failures, 15-min window, admin exempt","decisions":["Window: 15 min","Admin role exempt","Per-email, not per-IP"]}
{"timestamp":"2026-03-02T16:30:00Z","issueId":"issue-042","phase":"plan","taskCount":"6","decisions":["Use existing Redis client from otp-rate-limit.ts"]}
{"timestamp":"2026-03-02T17:15:00Z","issueId":"issue-042","phase":"verify","testResults":{"unit":{"passed":12,"failed":0}},"requirementsMet":"3/3","verdict":"ready"}
```

**Querying the log**:
```bash
# All phases for issue-042
grep "issue-042" logs/copilot/agent-activity.log | jq .

# All issues with verdict "not-ready"
cat logs/copilot/agent-activity.log | jq 'select(.verdict == "not-ready")'

# Today's work (replace date)
cat logs/copilot/agent-activity.log | jq 'select(.timestamp | startswith("2026-03-02"))'
```

---

## Summary

| Hook | Trigger | What it does |
|:---|:---|:---|
| `session-auto-commit` | Session ends | Commits + pushes uncommitted changes |
| `session-logger` | Start/end/prompt | JSON audit trail in `logs/copilot/` |
| Agent logging | Phase complete | Structured decision log in `logs/copilot/agent-activity.log` |

**Next: Team Lead Setup Guide →** [07-team-lead-setup.md](./07-team-lead-setup.md)

---

## Further Reading

- [jq documentation](https://jqlang.org/manual/) — the JSON processor used to query agent activity logs
- [VS Code Copilot Hooks](https://code.visualstudio.com/docs/copilot/copilot-customization#_using-hooks) — official hooks documentation including all available events
- [Git hooks vs Copilot hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) — Git hooks run on git commands; Copilot hooks run on AI session events — two different systems
---

[← Part 05: Daily Developer Workflow](./05-daily-workflow.md) · 📚 [Learn Series](/) · [Part 07: Team Lead Setup →](./07-team-lead-setup.md)
