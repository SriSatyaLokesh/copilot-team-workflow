---
issue-id: "ISSUE-007"
title: "Agent workflow handoffs and tool configuration"
type: "fix"
status: "verify"
owner: "SriSatyaLokesh"
branch: "issue/ISSUE-007-agent-workflow-handoffs-tools"
created: "2026-03-21"
last-updated: "2026-03-21"
related-flow: "workflow-agent-flow"
related-apis: []
---

Issue scope: fix workflow handoffs, execution routing, and valid tool declarations for the Copilot team workflow.

## Phase 1: Discuss

Status: [x] Done

### What Are We Building?

This issue fixes orchestration and tooling gaps across workflow agents. It ensures reliable phase handoffs, explicit execution routing, correct mode switching for execution, and valid tool definitions that support terminal operations.

### Why Are We Building It?

The current workflow breaks between phases and can leave execution in the wrong mode, which prevents command execution and file edits. This causes repeated manual intervention and unreliable automation.

### Who Needs This?

Developers using the Copilot issue workflow and all phase agents in the process.

### Requirements

- [x] Research agent provides guided handoff to Plan at end of Phase 2 with a button and `/plan` fallback.
- [x] Plan agent asks for execution style selection.
- [x] Selected execution flow starts in agent mode before execution actions.
- [x] Workflow agents use valid tool configuration syntax accepted by Copilot.
- [x] Discuss, Research, Plan, TDD, Execute, Verify, and Finish agents have terminal-capable tooling with phase guardrails.

### Out of Scope

- Product feature or business logic implementation.
- New agent types beyond the current workflow roles.
- Broad documentation refactors outside workflow and issue-process docs.

### Open Questions

1. Should terminal command scope be uniform across all agents?

Answer: No. Restrict it by phase guardrails.

## Phase 2: Research

Status: [x] Done

### Codebase Context Found

- Automatic handoff already exists in Discuss via `handoffs` metadata with `send: true`.
- Research and Plan currently enforce explicit developer-driven commands such as `/plan` and `/execute` and explicitly avoid auto handoff.
- Execution mode choice already exists in `/execute` prompt, not in Planner.
- Tool declarations were inconsistent across agents and prompts, which likely explained tool resolution failures.

### Existing Patterns to Follow

- Auto handoff syntax baseline: `.github/agents/discuss.agent.md`
- Mode selection pattern: `.github/prompts/execute.prompt.md`
- Current V2 workflow contract: `docs/MIGRATION-V2.md`

### Files Affected

| File | Why |
| --- | --- |
| `.github/agents/research.agent.md` | Add guided Planner handoff behavior and completion messaging. |
| `.github/prompts/research.prompt.md` | Align prompt-level research completion with the handoff model. |
| `.github/agents/plan.agent.md` | Add explicit execution routing output and mode confirmation requirement. |
| `.github/prompts/plan.prompt.md` | Align plan completion UX with execution-mode choice. |
| `.github/prompts/execute.prompt.md` | Clarify mode switch steps and return-to-workflow contract. |
| `.github/agents/tdd.agent.md` | Normalize tool declarations and terminal guardrails. |
| `.github/agents/verify.agent.md` | Normalize tool declarations and terminal guardrails. |
| `.github/prompts/finish-branch.prompt.md` | Align tooling policy and Verify mapping. |

### Related Docs

- `docs/MIGRATION-V2.md`
- `docs/COMPLETE-WORKFLOW-VISUAL.md`
- `docs/AGENT-MODE-VS-TDD-AGENT.md`

### Risks

- Reintroducing auto handoff could conflict with the V2 decision to reduce stuck-agent behavior.
- Switching to agent mode may not be fully enforceable by prompt text alone in all Copilot UI contexts.
- Missing dedicated `finish-branch.agent.md` creates ambiguity for full tool-standardization scope.

## Phase 3: Plan

Status: [x] Done

### Architecture Decisions

| Decision | Choice | Reason |
| --- | --- | --- |
| Research to Plan transition | Guided handoff with explicit `/plan` | Preserves V2 reliability while improving discoverability. |
| Agent mode switch | User reselects agent mode explicitly | UI mode switching is not reliably enforceable by prompt text alone. |
| Tool policy scope | Workflow-critical files only | Minimizes blast radius and focuses on blockers. |

### Workflow Orchestration Tasks

- [x] Update `.github/agents/research.agent.md` and `.github/prompts/research.prompt.md` to implement guided Planner handoff language.
- [x] Update `.github/agents/plan.agent.md` and `.github/prompts/plan.prompt.md` to include explicit execution mode routing output.
- [x] Update `.github/prompts/execute.prompt.md` with pre-flight checks for mode, reselection, and terminal or edit readiness.

### Tool Configuration Tasks

- [x] Normalize workflow-critical tool declarations in:
  - `.github/agents/discuss.agent.md`
  - `.github/agents/research.agent.md`
  - `.github/agents/plan.agent.md`
  - `.github/agents/tdd.agent.md`
  - `.github/agents/verify.agent.md`
  - `.github/prompts/execute.prompt.md`
  - `.github/prompts/finish-branch.prompt.md`

### Docs Alignment Tasks

- [x] Update `docs/MIGRATION-V2.md`, `docs/COMPLETE-WORKFLOW-VISUAL.md`, and `docs/AGENT-MODE-VS-TDD-AGENT.md` to match the implemented behavior.

### Test and Validation Plan

- [ ] Manual flow validation in the VS Code UI: `/discuss` -> `/research` -> guided `/plan` -> `/execute` -> `/verify`.
- [x] Confirm no contradictory handoff statements remain in targeted files.
- [x] Confirm tool-resolution errors are eliminated for workflow-critical roles.

## Phase 4: Execute

Status: [x] Done

### Execution Summary

- Updated workflow-critical agent and prompt frontmatter to use consistent terminal-capable tool declarations.
- Added guided Research to Plan handoff and explicit Plan to Execute routing details.
- Added execute pre-flight checks for agent-mode reselection and terminal or edit readiness.
- Fixed the Research agent handoff indentation so the frontmatter parses cleanly.
- Corrected the setup guide to document that built-in `ask` prompts do not honor `tools:` overrides.

## Phase 5: Verify

Status: [x] Done

### Verification Summary

- [x] Static validation passes for all targeted workflow agents, prompts, and `.github/tool-sets.json`.
- [x] Read-only review completed and merge blockers were addressed.
- [x] Issue tracking artifacts were aligned with the current execution state.
- [ ] End-to-end manual UI validation in VS Code remains a follow-up because it requires interactive agent execution.

### Evidence

- `get_errors` reports no errors for the targeted workflow files.
- GitHub CLI authentication is valid for the current repository remote.
- The finish-branch path was adapted for this docs and config repo because there is no root `package.json` or automated test runner.
