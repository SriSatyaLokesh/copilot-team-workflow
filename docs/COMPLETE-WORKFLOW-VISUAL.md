# Complete Workflow with Execution Modes — Visual Guide

## The Full Journey

```
┌─────────────────────────────────────────────────────────────────┐
│  /start-issue                                                    │
│  ├─ Confirm primary branch (main/dev/staging)                   │
│  ├─ Check current branch                                        │
│  ├─ Offer: Stay or create new branch                           │
│  └─ If new: git checkout -b fix/042-description                │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  /discuss OR @discuss                                            │
│  PHASE 1: Requirements                                           │
│  ├─ Ask clarifying questions (one at a time)                    │
│  ├─ Propose 2-3 approaches                                      │
│  ├─ Get requirement approval                                    │
│  ├─ Detect GitHub repo (optional)                              │
│  ├─ Create GitHub issue #42 (optional)                         │
│  ├─ Create work/ISSUE-042-name/ folder                         │
│  ├─ Create plan.md (Phase 1 complete)                          │
│  ├─ Create result.md (empty)                                   │
│  └─ Auto hand off to Research                                  │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  @research (automatic after /discuss)                            │
│  PHASE 2: Codebase Exploration                                  │
│  ├─ Search for existing patterns                               │
│  ├─ Find files to modify                                       │
│  ├─ Identify dependencies and risks                            │
│  ├─ Update plan.md Phase 2                                     │
│  └─ Guided handoff: Open Planner button or /plan              │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  /plan OR @planner work/ISSUE-042-name                          │
│  PHASE 3: Implementation Plan                                   │
│  ├─ Review requirements and research                           │
│  ├─ Ask 2-3 clarifying questions                               │
│  ├─ Create task breakdown:                                     │
│  │  ├─ Backend tasks                                           │
│  │  ├─ Frontend tasks                                          │
│  │  ├─ Infrastructure                                          │
│  │  ├─ Tests                                                   │
│  │  └─ Documentation                                           │
│  ├─ Document architecture decisions                            │
│  ├─ Get plan approval                                          │
│  ├─ Route execution: choose Mode A or Mode B                   │
│  └─ Update plan.md Phase 3 (COMPLETE)                          │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  /execute work/ISSUE-042-name                                   │
│  PHASE 4: Choose Execution Mode                                 │
│                                                                  │
│  ┌──────────────────┐           ┌──────────────────┐           │
│  │  MODE A          │    OR     │  MODE B          │           │
│  │  Agent Mode      │           │  TDD Agent       │           │
│  └────────┬─────────┘           └────────┬─────────┘           │
│           │                               │                     │
└───────────┼───────────────────────────────┼─────────────────────┘
            │                               │
            ▼                               ▼
┌───────────────────────────┐  ┌────────────────────────────────┐
│ MODE A: Agent Mode        │  │ MODE B: TDD Agent              │
│ (Flexible Implementation) │  │ (Disciplined TDD)              │
├───────────────────────────┤  ├────────────────────────────────┤
│ 1. Generate context       │  │ 1. @tdd work/ISSUE-042-name    │
│    bundle:                │  │                                │
│    - Requirements         │  │ 2. For each task:              │
│    - Research findings    │  │    ├─ RED: Write failing test  │
│    - Implementation tasks │  │    ├─ Run test (fails)         │
│    - Quality standards    │  │    ├─ GREEN: Minimal code      │
│                           │  │    ├─ Run test (passes)        │
│ 2. Show bundle to dev     │  │    ├─ REFACTOR: Clean up       │
│                           │  │    └─ Auto commit              │
│ 3. Dev copies → New chat  │  │                                │
│    and reselects Agent    │  │                                │
│    Mode before pasting    │  │                                │
│                           │  │ 3. Every 3 tasks:              │
│ 4. Agent implements:      │  │    ├─ Request code review      │
│    ├─ Multi-file editing  │  │    ├─ Address Critical issues  │
│    ├─ Can ask questions   │  │    └─ Continue next batch      │
│    ├─ Updates result.md   │  │                                │
│    └─ Commits as needed   │  │ 4. Update result.md Phase 4    │
│                           │  │                                │
│ 5. Agent says:            │  │ 5. Run quality checks:         │
│    "Ready for /verify"    │  │    ├─ npm test                 │
│                           │  │    ├─ npx tsc --noEmit         │
│ 6. Dev returns to         │  │    └─ npm run lint             │
│    workflow               │  │                                │
│                           │  │ 6. Offer PR creation           │
└───────────┬───────────────┘  └────────────┬───────────────────┘
            │                               │
            └───────────┬───────────────────┘
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│  /verify work/ISSUE-042-name                                    │
│  PHASE 5: Quality Gate                                          │
│  ├─ Check Phase 1 requirements (all met?)                       │
│  ├─ Run test suite (all passing?)                              │
│  ├─ Check TypeScript errors (none?)                            │
│  ├─ Check lint errors (none?)                                  │
│  ├─ Verify docs updated                                        │
│  ├─ Update result.md Phase 5                                   │
│  │                                                              │
│  ├─ If all ✅ GREEN:                                            │
│  │  ├─ Detect GitHub repo                                      │
│  │  ├─ Check if PR exists                                      │
│  │  ├─ If not: Offer to create PR with "Fixes #42"            │
│  │  └─ Offer auto-merge when CI passes                        │
│  │                                                              │
│  └─ If any ❌ RED: List issues, require fixes before merge      │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  GitHub Integration (if detected)                               │
│  ├─ Create PR: gh pr create --title "..." --body "Fixes #42..." │
│  ├─ Auto-merge: gh pr merge --squash --delete-branch --auto    │
│  ├─ PR merges → GitHub issue #42 auto-closes                   │
│  └─ Feature branch deleted automatically                        │
└─────────────────────────────────────────────────────────────────┘
                 │
                 ▼
              ✅ DONE
```

---

## Key Decision Points

### Decision 1: New Branch or Stay? (`/start-issue`)
- **Stay on current**: Already on right branch, continuing work
- **Create new**: Starting fresh work, needs clean branch from primary

### Decision 2: Create GitHub Issue? (`/discuss`)
- **Yes**: Creates issue #42, uses number for folder/branch/PR linking
- **No**: Manual tracking only, uses sequential numbering

### Decision 3: Which Execution Mode? (`/execute`)

| Choose Agent Mode When | Choose TDD Agent When |
|------------------------|----------------------|
| Complex feature (10+ tasks) | Simple feature (3-5 tasks) |
| Unfamiliar codebase | High-risk code |
| Need flexibility | Learning TDD |
| Tight deadline | Want discipline |
| Conversational coding | Automatic commits |

### Decision 4: Create PR? (`/verify`)
- **Yes**: Auto-creates with "Fixes #42", offers auto-merge
- **No**: Manual PR creation later

---

## Agent Mode Deep Dive

### What Happens When You Choose Agent Mode

```
/execute work/ISSUE-042-name
    │
    ▼
"Choose execution mode: A) Agent Mode or B) TDD Agent?"
    │
    ├─ Choose A (Agent Mode)
    │
    ▼
Prompt generates context bundle:
┌────────────────────────────────────────┐
│ # Implementation Task                  │
│                                        │
│ Work Folder: work/ISSUE-042-name      │
│ Branch: fix/42-rate-limiting          │
│ GitHub Issue: #42                      │
│                                        │
│ ## Requirements                        │
│ [Full Phase 1 from plan.md]           │
│                                        │
│ ## Research Context                    │
│ [Full Phase 2 from plan.md]           │
│                                        │
│ ## Implementation Plan                 │
│ [Full Phase 3 from plan.md]           │
│                                        │
│ ## Your Task                           │
│ Implement all tasks using TDD         │
│ [Detailed instructions]                │
│                                        │
│ ## When Complete                       │
│ 1. Update result.md Phase 4            │
│ 2. Run npm test                        │
│ 3. Say "Ready for /verify"             │
└────────────────────────────────────────┘
    │
    ▼
Prompt shows bundle and says:
"📋 Context bundle prepared.
 
 Next steps:
 1. Open new chat (Ctrl+L)
 2. Paste bundle above
 3. Let agent implement
 4. When done, run /verify"
    │
    ▼
Developer opens new chat → Pastes bundle
    │
    ▼
Agent Mode implements:
├─ Reads full context
├─ Asks clarifying questions if needed
├─ Edits multiple files
├─ Writes tests (guided by bundle)
├─ Runs tests
├─ Commits logically
├─ Updates result.md Phase 4
└─ Says: "Ready for /verify"
    │
    ▼
Developer returns to workflow:
    /verify work/ISSUE-042-name
```

---

## TDD Agent Deep Dive

### What Happens When You Choose TDD Agent

```
/execute work/ISSUE-042-name
    │
    ▼
"Choose execution mode: A) Agent Mode or B) TDD Agent?"
    │
    ├─ Choose B (TDD Agent)
    │
    ▼
Prompt invokes: @tdd work/ISSUE-042-name
    │
    ▼
TDD Implementer Agent:
├─ Verifies NOT on main branch
├─ Reads plan.md Phases 1-3
├─ Confirms all phases complete
│
├─ Task 1:
│  ├─ 🔴 RED: Write failing test
│  ├─ Run: npm test (MUST FAIL)
│  ├─ 🟢 GREEN: Write minimal code
│  ├─ Run: npm test (MUST PASS)
│  ├─ 🔵 REFACTOR: Clean up
│  ├─ Run: npm test (MUST PASS)
│  └─ Commit: "feat: implemented task 1"
│
├─ Task 2: [same Red-Green-Refactor]
├─ Task 3: [same Red-Green-Refactor]
│
├─ After 3 tasks: Call Reviewer agent
│  ├─ Reviewer checks code quality
│  ├─ 🔴 Critical → Fix immediately
│  ├─ 🟡 Warning → Note for later
│  └─ 🔵 Suggestion → Document
│
├─ Continue next batch of 3...
│
└─ All tasks complete:
   ├─ Update result.md Phase 4
   ├─ Run: npm test (final check)
   ├─ Offer PR creation
   └─ Hand off to /verify
```

---

## Comparison Table

| Feature | Agent Mode | TDD Agent |
|---------|-----------|-----------|
| **Speed** | ⚡⚡⚡ Fast | ⚡⚡ Moderate |
| **Flexibility** | ✅ High (can adapt) | ⚠️ Low (follows plan) |
| **TDD Enforcement** | ⚠️ Self-enforced | ✅ Strictly enforced |
| **Commits** | 👤 Manual (developer decides) | 🤖 Auto (after each task) |
| **Code Review** | 📝 At end | 📝 Every 3 tasks |
| **Context** | 🌍 Full workspace | 📋 Plan-focused |
| **Questions** | 💬 Can ask mid-implementation | ❌ No questions |
| **Multi-file** | ✅ Natural | ✅ Supported |
| **Learning Curve** | ⚡ Easy | 📚 Requires TDD knowledge |
| **Best For** | Production work | Training & high-risk code |

---

## Complete Workflow Summary

### Phase Mapping

| Phase | Command | Agent | Output | Time |
|-------|---------|-------|--------|------|
| **0. Setup** | `/start-issue` | None (git only) | Feature branch created | 2 min |
| **1. Requirements** | `/discuss` | Discuss | `work/ISSUE-XXX/plan.md` Phase 1 ✅ | 10-20 min |
| **2. Research** | (auto) | Research | `work/ISSUE-XXX/plan.md` Phase 2 ✅ | 5-10 min |
| **3. Planning** | `/plan` | Planner | `work/ISSUE-XXX/plan.md` Phase 3 ✅ | 15-30 min |
| **4. Execution** | `/execute` | Agent Mode OR TDD Agent | `work/ISSUE-XXX/result.md` Phase 4 ✅ | 1-8 hours |
| **5. Verification** | `/verify` | Verify | `work/ISSUE-XXX/result.md` Phase 5 ✅ | 5-10 min |
| **6. Merge** | (auto) | GitHub CLI | PR merged, issue closed | 1 min |

**Total**: 2-10 hours depending on feature complexity

---

## Real-World Example

### Feature: Add Rate Limiting to Login Endpoint

```
Day 1, 9:00 AM — User runs:
    /start-issue
    Issue: "Add rate limiting to login endpoint"
    Creates branch: fix/42-rate-limiting
    
Day 1, 9:05 AM — Discuss auto-starts:
    @discuss prompts questions:
    - How many attempts before lockout? (5)
    - Lockout duration? (15 minutes)
    - Should admins be exempt? (Yes)
    
    Creates: work/ISSUE-042-rate-limiting/
    ├─ plan.md (Phase 1 ✅)
    └─ result.md (empty)
    
    Offers: Create GitHub issue? (Yes)
    → Creates GitHub issue #42
    
Day 1, 9:25 AM — Research auto-runs:
    Finds: Redis rate limiter pattern in middleware/
    Updates: plan.md Phase 2 ✅
    
Day 1, 9:35 AM — User runs:
    /plan work/ISSUE-042-rate-limiting
    
    Planner creates task breakdown:
    - Task 1: Create rate limiter middleware
    - Task 2: Apply to login route
    - Task 3: Add admin exemption check
    - Task 4: Write integration tests
    - Task 5: Update API docs
    
    Updates: plan.md Phase 3 ✅
    
Day 1, 10:00 AM — User runs:
    /execute work/ISSUE-042-rate-limiting
    
    Chooses: Mode A (Agent Mode)
    
    Copies context bundle → Opens new chat → Pastes
    
Day 1, 10:05 AM — Agent Mode works:
    - Implements rate limiter middleware
    - Applies to routes
    - Adds admin check
    - Writes tests
    - Updates result.md Phase 4
    
Day 1, 11:30 AM — Agent says:
    "Implementation complete. Ready for /verify"
    
Day 1, 11:35 AM — User runs:
    /verify work/ISSUE-042-rate-limiting
    
    Checks:
    ✅ All requirements met
    ✅ Tests passing (18/18)
    ✅ No TypeScript errors
    ✅ No lint errors
    ✅ Docs updated
    
    Offers: Create PR? (Yes)
    → gh pr create --title "fix: Add rate limiting" --body "Fixes #42..."
    → PR #3 created
    
    Offers: Auto-merge? (Yes)
    → gh pr merge 3 --squash --delete-branch --auto
    
Day 1, 11:40 AM — CI passes, PR merges:
    ✅ Code merged to main
    ✅ Issue #42 auto-closed
    ✅ Branch fix/42-rate-limiting deleted
    
    DONE! 🎉
```

**Total time**: ~2.5 hours from start to merged PR

---

## Quick Reference Commands

```bash
# Start workflow
/start-issue
# Answer prompts, creates branch and work folder

# Execution (after planning complete)
/execute work/ISSUE-042-name
# Choose: Agent Mode (flexible) or TDD Agent (disciplined)

# Verification
/verify work/ISSUE-042-name
# Runs quality checks, offers PR creation

# Check status anytime
/status
# Shows current phase and next step
```

---

**Last Updated**: March 12, 2026  
**Status**: Production Ready  
**Next**: Try it! Run `/start-issue` and follow this guide.
