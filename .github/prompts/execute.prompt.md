---
description: 'Use when executing an approved implementation plan from an Issue doc — when a developer says "execute the plan", "start implementing", or "begin the tasks". Requires Phase 3 (Plan) to be complete. Handles both single-agent sequential tasks and signals when parallel dispatching is needed.'
agent: 'TDD Implementer'
tools: ['editFiles', 'terminal', 'search', 'codebase', 'problems']
model: 'claude-sonnet-4-5'
---
# Execute Implementation Plan

**Issue doc**: ${input:ISSUE-doc:Path to Issue doc (e.g., docs/issues/ISSUE-042-name.md)}

## Step 1 — Load and Review Plan

Read Phase 3 of the Issue doc. Review critically:
- Are there gaps or ambiguities in any task?
- Are task dependencies clear?
- Are verification commands specified?

If concerns exist: raise them before starting. If clear: proceed.

## Step 2 — Choose Execution Mode

**Read all tasks first, then decide:**

| Mode | When to use |
|:---|:---|
| **Sequential (default)** | Tasks depend on each other, or fewer than 3 tasks total |
| **Parallel dispatch** | 3+ tasks that are fully independent (different files, different subsystems, no shared state) |

> **Parallel example:** Task A (auth service), Task B (notifications), Task C (analytics logging) — these touch completely different code, run them in parallel.
>
> **Sequential example:** Task 1 = create DB schema, Task 2 = write service using schema — Task 2 depends on Task 1. Must be sequential.
>
> See [parallel-agents guide](../instructions/parallel-agents.instructions.md) for the full pattern.

## Step 3 — Execute in Batches of 3

For each batch of 3 tasks (sequential mode):

1. **For each task:**
   - 🔴 **RED**: Write the failing test first — run `npm test`, confirm it fails
   - 🟢 **GREEN**: Write minimal code to pass — run `npm test`, confirm it passes
   - 🔵 **REFACTOR**: Clean up — run `npm test`, confirm still green
   - **Commit**: `git commit -m "feat: [task description]"`
   - Update Phase 4 progress in Issue doc

2. **After every 3 tasks — checkpoint review:**
   ```
   Batch complete. Requesting code review before continuing.
   ```
   Invoke the Reviewer agent: *"Review the latest 3 tasks. Check for correctness, security, and standards alignment."*

3. **Address review findings:**
   - 🔴 Critical → fix before next batch starts
   - 🟡 Warning → fix before `/verify`
   - 🔵 Suggestion → note in Issue doc

4. Continue next batch.

## Step 4 — Final Quality Gate

After all tasks and reviews:

```bash
npm test               # All tests must pass
npx tsc --noEmit       # No TypeScript errors
npm run lint           # No lint errors
```

Then say: *"All tasks complete. Run `/verify` to check completeness before creating a PR."*
