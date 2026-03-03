---
layout: default
parent: "Copilot Team Workflow"
title: "Part 10 - Subagent-Driven Development"
description: "How to use specialized subagents for parallel tasks and automated quality reviews."
nav_order: 11
---

# Learn: Subagent-Driven Development

[← Part 09: Test-Driven Development (TDD)](./09-test-driven-development.md) · 📚 [Learn Series](./index.md)

---

## The Big Idea

When a task gets complex, one single LLM context can get "polluted" or overwhelmed. **Subagent-Driven Development** solves this by dispatching fresh, specialized agents for each sub-task.

Instead of one agent doing everything, you use a **Controller agent** that manages several specialized workers.

---

## The Roles

| Agent | Responsibility |
|:---|:---|
| **Controller** | Reads the plan, breaks it down, and dispatches subagents. |
| **Implementer** | Follows TDD to write code and tests for a specific task. |
| **Spec Reviewer** | Checks if the implementer actually met the requirements in the spec. |
| **Code Reviewer** | Audits the code for quality, performance, and best practices. |

---

## Receiving Code Review

When a subagent (or a human) gives you feedback, follow the **Technical Rigor** rule:

1.  **Verify First**: Don't say "You're right!" immediately. Check the code.
2.  **No Performative Agreement**: Avoid "Great point!" or "Thanks for catching that." Just state the technical fix or push back with reasoning.
3.  **One at a Time**: Fix items individually and verify with tests.

This ensures you aren't just blindly following suggestions that might break other parts of your system.

---

## Why Use Subagents?

1. **Isolation**: If a subagent gets stuck or hallucinating, it doesn't break your entire session. You just kill it and start a fresh one.
2. **Specialization**: A Reviewer agent is grounded in review rules, while the Implementer is grounded in TDD and coding.
3. **Speed**: Subagents can work on independent tasks without dragging along the entire history of the project.
4. **Verification**: Having an autonomous "Spec Reviewer" catch issues before they reach you (the human) saves time.

---

## How to Use Subagents in Practice

When you have an implementation plan with 3+ independent tasks in Phase 4:

1. **Open Copilot Chat** → select the **Agent** tab
2. **Select the `ParallelBuilder` agent** from the agent picker
3. **Type** (or use the argument-hint placeholder):
   ```
   Execute Phase 4 from #docs/issues/ISSUE-XXX-name.md
   ```
4. The ParallelBuilder agent will:
   - Read the Phase 3 task list
   - Identify which tasks are truly independent (no shared files)
   - Dispatch each independent task to a fresh TDD Implementer subagent
   - Wait for each to complete
   - Run the full test suite across all results
   - Present a combined summary

> **One-session rule still applies**: the ParallelBuilder session is the controller. Each subagent gets its own isolated context so they don't pollute each other.

> **Red flag**: If two tasks modify the same file, ParallelBuilder will flag them as dependent and execute them sequentially, not in parallel. This is expected — it prevents merge conflicts.

---

## The Two-Stage Review

In this workflow, every task goes through two gates before it's marked "Done":

1.  **Stage 1: Spec Compliance** — Does the code do what the plan said it should do? (No more, no less).
2.  **Stage 2: Code Quality** — Is the code clean, documented, and following patterns?

If a reviewer finds an issue, it sends it back to the **Implementer subagent** to fix it before you ever see it.

---

## When to Use Subagents

- **Complex Feature Builds**: When you have an implementation plan with 5+ independent tasks.
- **Large Refactors**: When changing multiple files that need to be tested in isolation.
- **Automated PR prep**: Using subagents to verify requirements and quality before submission.

**Red Flag**: Don't use subagents for tiny fixes. The overhead of starting them is only worth it for modular tasks in an implementation plan.

---

## Summary

Subagent-Driven Development turns coding into a **systematic process** where the AI self-corrects through review loops. By the time you review the final result, it has already passed two AI quality gates.

**You have finished the expanded 10-part series!**

---

[← Part 09: Test-Driven Development (TDD)](./09-test-driven-development.md) · 📚 [Learn Series](./index.md)
