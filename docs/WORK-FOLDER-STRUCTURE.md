# Work Folder Structure & GitHub CLI Integration — Complete Guide

## What Changed

This update introduces a **new work folder structure** and **GitHub CLI integration** that transforms how issues are managed in this project.

### Before (Old Structure)
```
docs/
└── issues/
    └── ISSUE-XXX-name.md  (single file with all 5 phases)
```

### After (New Structure)
```
work/
└── ISSUE-XXX-name/
    ├── plan.md    (Phases 1-3: Requirements, Research, Plan)
    └── result.md  (Phases 4-5: Execution, Verification)
```

---

## Why This Change?

### 1. Cleaner Separation of Concerns
- **plan.md** = What we're supposed to build (intent)
- **result.md** = What actually happened (reality)
- This prevents mixing "what was planned" with "what we did"

### 2. Better Git Integration
- Each issue gets its own folder
- Can add artifacts: screenshots, diagrams, scripts  
- Easier to link work folder to feature branch

### 3. GitHub Automation
- Agents now detect GitHub repos automatically
- Can create GitHub issues for tracking
- Can create PRs automatically
- Can auto-merge when CI passes
- Full GitHub CLI workflow integrated

### 4. Reduced Merge Conflicts
- Multiple devs can work on different issues simultaneously
- Each issue has its own folder — no single file bottleneck
- Parallel work is now truly parallel

---

## Core Concepts

### Work Folder Naming Convention
```
work/ISSUE-[number]-[kebab-case-description]/
```

**Examples**:
- `work/ISSUE-042-rate-limiting/`
- `work/ISSUE-001-login-authentication/`
- `work/ISSUE-123-performance-optimization/`

The number (042, 001, 123) matches:
- GitHub issue number (if created)
- Branch name: `fix/042-rate-limiting`
- Commit messages: `Resolves #42`

### File Structure

#### `plan.md` Contains:
1. **Phase 1: Requirements** (from Discuss agent)
   - What we're building
   - Why
   - Requirements checklist
   - Acceptance criteria
   - Out of scope

2. **Phase 2: Research** (from Research agent)
   - Existing patterns found
   - Files to modify
   - Dependencies
   - Risks identified

3. **Phase 3: Implementation Plan** (from Planner agent)
   - Architecture decisions
   - Task breakdown (backend, frontend, tests, docs)
   - Plan approval

#### `result.md` Contains:
1. **Phase 4: Execution** (from TDD Implementer agent)
   - Progress tracker
   - Implementation log
   - Deviations from plan
   - Commits made

2. **Phase 5: Verification** (from Verify agent)
   - Test results
   - Requirements verification
   - Code quality checks
   - PR details
   - Lessons learned

---

## GitHub CLI Integration

### Automatic Detection

All agents now check if the project is a GitHub repository:

```powershell
git config --get remote.origin.url
# If contains 'github.com' → offer automation
```

### What Gets Automated

#### At Issue Start (Discuss Agent):
1. ✅ **Create GitHub Issue**
   ```powershell
   gh issue create --title "fix: Rate limiting" --body "..." --label bug
   ```
   - Parses issue number (#42)
   - Uses that number for work folder and branch

2. ✅ **Create Feature Branch**
   ```bash
   git checkout -b fix/42-rate-limiting
   ```

#### During Implementation (TDD Agent):
3. ✅ **Commit with Issue Reference**
   ```bash
   git commit -m "fix: Add rate limiting

   Resolves #42"
   ```

4. ✅ **Push Branch**
   ```bash
   git push origin fix/42-rate-limiting
   ```

5. ✅ **Create Pull Request**
   ```powershell
   gh pr create --title "fix: Add rate limiting" \
     --body "Fixes #42

   Implementation summary..." \
     --base main
   ```
   - **Critical**: PR body MUST start with `Fixes #42`
   - This auto-closes the GitHub issue when PR merges

#### After Verification (Verify Agent):
6. ✅ **Auto-merge PR** (optional)
   ```powershell
   gh pr merge --squash --delete-branch --auto
   ```
   - Waits for CI to pass
   - Squashes commits into one
   - Deletes feature branch automatically
   - Closes linked GitHub issue

---

## New Workflow: Step-by-Step

### Starting an Issue

**User**:
```
@discuss I want to add rate limiting to the login endpoint
```

**Discuss Agent**:
1. Checks: `git config --get remote.origin.url`
2. Detects: `github.com` in URL
3. Asks clarifying questions about requirements
4. Proposes 2-3 approaches
5. Gets user approval
6. Asks: "🤖 Create GitHub issue for tracking? (yes/no)"
7. If yes: Creates GitHub issue #42
8. Asks: "Next issue number for work folder? (042)"
9. Creates:
   ```
   work/ISSUE-042-rate-limiting/
   ├── plan.md (Phase 1 complete)
   └── result.md (empty)
   ```
10. Asks: "Create branch `fix/42-rate-limiting`? (yes/no)"
11. If yes: `git checkout -b fix/42-rate-limiting`
12. Hands off to Research agent automatically

---

### Research Phase

**Research Agent**:
1. Reads `work/ISSUE-042-rate-limiting/plan.md` Phase 1
2. Searches codebase for existing patterns
3. Identifies files to modify
4. Documents findings in `plan.md` Phase 2
5. Marks Phase 2 as `[x] Complete`
6. Hands off to Planner

---

### Planning Phase

**User** (or autoprompt from Research):
```
@planner work/ISSUE-042-rate-limiting
```

**Planner Agent**:
1. Reads `plan.md` Phases 1 & 2
2. Verifies both are `[x] Complete`
3. Creates implementation plan (backend, frontend, tests, docs)
4. Writes to `plan.md` Phase 3
5. Gets user approval
6. Marks Phase 3 as `[x] Complete`

---

### Implementation Phase

**User**:
```
@tdd work/ISSUE-042-rate-limiting
```

**TDD Implementer Agent**:
1. Verifies: NOT on main branch
2. Reads `plan.md` all 3 phases
3. For each task:
   - Write failing test first
   - Run test (fails)
   - Write minimal code to pass
   - Run test (passes)
   - Commit
4. Fills `result.md` Phase 4 with execution notes
5. Asks: "Push branch and create PR? (yes/no)"
6. If yes:
   - `git push origin HEAD`
   - `gh pr create --title "..." --body "Fixes #42\n\n..." --base main`
   - Parses PR number (#3)
7. Asks: "Auto-merge when CI passes? (yes/no)"
8. Hands off to Verify

---

### Verification Phase

**User** (or autoprompt from TDD):
```
@verify work/ISSUE-042-rate-limiting
```

**Verify Agent**:
1. Reads `plan.md` Phase 1 (requirements)
2. Checks each requirement: ✅ met / ❌ not met
3. Runs tests: `npm test`
4. Checks for TypeScript/lint errors
5. Verifies docs updated
6. Fills `result.md` Phase 5 with verification report
7. If **✅ READY** and PR exists:
   - Asks: "Merge PR now? (yes/no)"
   - If yes: `gh pr merge --squash --delete-branch`
   - GitHub issue #42 auto-closes
8. Marks work complete

---

## File Templates

### Creating plan.md
Agents use: `docs/templates/plan-template.md`

### Creating result.md
Agents use: `docs/templates/result-template.md`

These templates are copied when Discuss agent creates the work folder.

---

## GitHub CLI Skill

All git and GitHub CLI patterns are documented in:
```
.github/skills/github-cli-workflow/SKILL.md
```

**Agents load this skill** at the start to understand:
- How to detect GitHub repos
- How to create issues, PRs
- How to handle authentication errors
- Message size limits for GitHub
- Commit message conventions
- PR auto-merge patterns

---

## Updated Agent Behaviors

### Discuss Agent
- ✅ Loads GitHub CLI skill
- ✅ Checks if GitHub repo
- ✅ Offers to create GitHub issue
- ✅ Creates `work/ISSUE-XXX-name/` folder
- ✅ Creates `plan.md` and `result.md` from templates
- ✅ Offers feature branch creation
- ✅ Logs to `logs/copilot/agent-activity.log`

### Research Agent
- ✅ Loads GitHub CLI skill
- ✅ Reads from `work/ISSUE-XXX-name/plan.md`
- ✅ Updates Phase 2 in `plan.md`
- ✅ Logs research findings

### Planner Agent
- ✅ Loads GitHub CLI skill
- ✅ Reads from `work/ISSUE-XXX-name/plan.md`
- ✅ Verifies Phases 1 & 2 complete
- ✅ Updates Phase 3 in `plan.md`
- ✅ Logs plan approval

### TDD Implementer Agent
- ✅ Loads GitHub CLI skill + TDD skill
- ✅ Reads from `work/ISSUE-XXX-name/plan.md`
- ✅ Verifies NOT on main branch
- ✅ Updates `result.md` Phase 4 during execution
- ✅ Commits with issue reference: `Resolves #42`
- ✅ Offers PR creation via GitHub CLI
- ✅ Offers auto-merge option
- ✅ Logs execution details

### Verify Agent
- ✅ Loads GitHub CLI skill
- ✅ Reads from `work/ISSUE-XXX-name/plan.md` and `result.md`
- ✅ Runs all quality checks
- ✅ Updates `result.md` Phase 5 with verification report
- ✅ Offers PR merge if all checks pass
- ✅ Logs verification outcome

---

## Migration Guide

### For New Work
Just use the new workflow — agents handle everything.

### For Existing Issues (in docs/issues/)
Two options:

**Option 1: Leave old issues as-is**
- Old issues stay in `docs/issues/`
- New work uses `work/`
- Both structures coexist

**Option 2: Migrate manually**
```bash
# For each old issue:
mkdir -p work/ISSUE-042-rate-limiting

# Split content:
# - Phases 1-3 → plan.md
# - Phases 4-5 → result.md

# Copy templates:
cp docs/templates/plan-template.md work/ISSUE-042-rate-limiting/plan.md
cp docs/templates/result-template.md work/ISSUE-042-rate-limiting/result.md

# Fill in from old issue doc
```

---

## Benefits Summary

| Benefit | How It Helps |
|---------|-------------|
| **GitHub Automation** | Create issues, PRs, merge automatically — no manual GitHub UI clicking |
| **Issue Tracking** | Every work folder links to a GitHub issue number for audit trails |
| **Parallel Work** | Multiple devs = multiple work folders, no merge conflicts |
| **Clean History** | Squash merges keep git log readable |
| **Auto-close Issues** | PRs with "Fixes #42" close linked issues automatically |
| **Cleaner Docs** | Plan vs Result separation prevents confusion |
| **Self-Contained** | Each issue folder can hold artifacts, not just text |

---

## Troubleshooting

### "gh: command not found"
**Fix**: Use full path:
```powershell
& "C:\Program Files\GitHub CLI\gh.exe" issue create ...
```

### "authentication token is missing required scopes"
**Fix**: Refresh auth:
```bash
gh auth refresh -s read:project
```

### "Review: Can not approve your own pull request"
**Normal**: You can't approve PRs you created. Skip approval or have teammate review.

### PR created but issue didn't close
**Cause**: PR body didn't start with `Fixes #42`  
**Fix**: Edit PR body to add `Fixes #42` at the top

### Work folder not created
**Check**: Templates exist at `docs/templates/plan-template.md` and `docs/templates/result-template.md`  
**Fix**: If missing, agents will fail — ensure templates are committed

---

## Quick Reference

### Start New Issue
```
@discuss I want to [feature description]
```
- Agent asks questions
- Offers GitHub issue creation
- Creates work folder
- Offers branch creation

### Continue Issue
```
@planner work/ISSUE-042-name
@tdd work/ISSUE-042-name
@verify work/ISSUE-042-name
```

### Check Status
```
@status
```
Shows current phase and next steps.

---

**Status**: ✅ All agents updated  
**Date**: March 12, 2026  
**Files Modified**:
- `.github/agents/discuss.agent.md`
- `.github/agents/research.agent.md`
- `.github/agents/plan.agent.md`
- `.github/agents/tdd.agent.md`
- `.github/agents/verify.agent.md`
- `.github/skills/github-cli-workflow/SKILL.md` (created)
- `docs/templates/plan-template.md` (created)
- `docs/templates/result-template.md` (created)
