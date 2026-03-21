# ISSUE-007: Agent Workflow Handoffs and Tool Configuration

**Issue ID**: ISSUE-007  
**Type**: fix  
**Owner**: SriSatyaLokesh  
**Branch**: issue/ISSUE-007-agent-workflow-handoffs-tools  
**Created**: 2026-03-21  
**Status**: execute

**Related**:
- Flow: N/A
- APIs: N/A

---

## 📋 Phase 1: Requirements (Discuss)

**Status**: [x] Complete

### What Are We Building?
This issue fixes workflow orchestration gaps in Copilot phase agents used by the team process. The current flow breaks at key handoff points and causes incorrect execution mode/tool behavior. The goal is a reliable requirements-to-research-to-plan-to-execute chain with usable terminal-enabled tools.

### Why?
Developers are blocked by manual handoffs, wrong mode transitions, and invalid tool definitions that prevent terminal commands and file edits in execution phases. Fixing these improves reliability and reduces setup friction for every issue workflow.

### Who Needs This?
Team developers and AI workflow agents running Discuss, Research, Plan, Execute, Verify, and Finish phases.

### Requirements
- [ ] Research agent provides guided handoff to Plan when Phase 2 completes (button + `/plan` fallback).
- [ ] Plan agent prompts for execution style selection (TDD-driven or normal execute flow).
- [ ] Plan-to-execution handoff switches into agent mode before execution starts.
- [ ] Workflow agents use valid, consistent tool configuration syntax.
- [ ] Discuss, Research, Plan, TDD, Execute, Verify, and Finish agents can run terminal commands with phase-appropriate guardrails.

### Out of Scope
- Application feature/business logic changes.
- New agent personas beyond the current workflow set.
- Rewriting unrelated docs outside this issue workflow scope.

### Acceptance Criteria
- [ ] Research completion output includes explicit guided Plan handoff behavior.
- [ ] Plan completion output includes execution-style choice and routing confirmation.
- [ ] Routed execution agent confirms it is in agent mode before starting work.
- [ ] All targeted agent files define tools in a valid format accepted by Copilot.
- [ ] Terminal commands execute successfully in each targeted agent without tool-format errors.

### Open Questions
1. Should terminal command scope be identical for all workflow agents, or restricted per phase? → **Answer**: restricted per phase with explicit guardrails.

---

## 🔍 Phase 2: Research Findings

**Status**: [x] Complete

### Existing Patterns Found
- **Pattern**: Discuss uses explicit handoff metadata (`handoffs` with `send: true`) → **Location**: `.github/agents/discuss.agent.md` — establishes the baseline pattern for automatic cross-agent transition.
- **Pattern**: Execute prompt provides the two-mode decision (Agent Mode vs TDD Agent) → **Location**: `.github/prompts/execute.prompt.md` — existing behavior can be reused instead of inventing a third execution route.
- **Pattern**: V2 docs intentionally moved to explicit command-driven transitions (`/plan`, `/execute`, `/verify`) → **Location**: `docs/MIGRATION-V2.md` — indicates a deliberate design choice that may conflict with restoring automatic Research -> Plan handoff.
- **Pattern**: Workflow visual and guide docs define current user-driven plan/execute transitions with auto research only → **Location**: `docs/COMPLETE-WORKFLOW-VISUAL.md`, `docs/DEVELOPER-STANDARD-GUIDE.md`, `docs/AGENT-MODE-VS-TDD-AGENT.md`.

### Files to Modify
| File | Change Needed |
|------|---------------|
| `.github/agents/research.agent.md` | Add/restore Planner handoff behavior and align Next Step text with the selected handoff strategy. |
| `.github/prompts/research.prompt.md` | Ensure Planner handoff behavior is explicit and consistent with agent-level handoff policy. |
| `.github/agents/plan.agent.md` | Add explicit execution-style choice output contract and agent-mode transition guidance before implementation starts. |
| `.github/prompts/plan.prompt.md` | Align planner completion UX with execution mode selection and concrete route (`/execute` with mode choice). |
| `.github/prompts/execute.prompt.md` | Clarify how agent mode is invoked and how return-to-workflow happens; remove ambiguity about where mode switch occurs. |
| `.github/agents/discuss.agent.md` | Reuse as canonical handoff syntax reference; update only if tool naming normalization requires it. |
| `.github/agents/tdd.agent.md` | Normalize tool declaration format and terminal guardrail language with final tool policy. |
| `.github/agents/verify.agent.md` | Normalize tool declaration format; ensure terminal capability naming matches repo standard. |
| `.github/prompts/finish-branch.prompt.md` | This is currently prompt-level (not separate agent file); align tool policy and document mapping to Verify agent role. |
| `.github/prompts/start-issue.prompt.md` | Optional follow-up: align baseline and tool wording if global tool format normalization is applied. |

### Dependencies
- GitHub CLI availability/authentication (`gh`) for tracker and PR actions.
- VS Code Copilot custom agent/prompt schema behavior for `tools` and `handoffs` fields.
- Existing workflow docs that describe V2 behavior and must remain consistent after any workflow change.

### Risks Identified
- ⚠️ **Product decision conflict**: `docs/MIGRATION-V2.md` explicitly says Research/Plan/TDD auto-handoffs were removed to avoid stuck sessions. Reintroducing auto handoff may regress that known problem. **Mitigation**: decide whether to keep explicit `/plan` command and implement a guided handoff button only, or re-enable fully automatic send with fallback text.
- ⚠️ **Mode-switch limitation**: prompts can instruct/prepare agent mode but may not force UI mode changes in all Copilot surfaces. **Mitigation**: encode deterministic user steps and validation text ("confirm agent mode active") instead of assuming programmatic mode switch.
- ⚠️ **Tool token inconsistency**: repository currently mixes multiple tool naming styles (`terminal`, `execute`, `edit/editFiles`, `read/readFile`, `vscode`). **Mitigation**: define one canonical allowlist format and apply it consistently to targeted workflow files.
- ⚠️ **Agent inventory mismatch**: there is no dedicated `finish-branch.agent.md`; finish branch currently maps to prompt with `agent: 'Verify'`. **Mitigation**: either keep prompt-only mapping and document it, or create a dedicated agent file in a separate follow-up issue.
- ⚠️ **Docs drift risk**: multiple docs currently describe manual transitions by design. **Mitigation**: update affected docs in same change set when behavior changes.

---

## 📐 Phase 3: Implementation Plan

**Status**: [x] Complete

### Architecture Decisions
| Decision | Choice | Reason |
|----------|--------|--------|
| Research -> Plan transition | Guided handoff button/message, explicit `/plan` run required | Preserves V2 reliability model while improving discoverability and reducing missed next-step confusion |
| Plan -> Execute mode switch | Explicit "reselect agent mode" operational step, plus confirmation prompt | Prompt files cannot guarantee programmatic UI mode switch in all surfaces; deterministic operator steps are reliable |
| Tool policy scope | Workflow execution-critical set only | Limits blast radius and avoids unnecessary edits to unrelated agents/prompts |
| Finish-branch coverage | Treat finish-branch as Verify-mapped prompt, not new agent file | Current repository has no dedicated finish-branch agent file |
| Tool declaration normalization | Canonical token set shared across targeted files | Removes mixed token naming that likely causes tool-resolution failures |

### Task Breakdown

#### Backend Tasks
- [x] Define canonical workflow tool schema and allowed terminal guardrails for targeted scope in planning notes and apply consistently to:
	- `.github/agents/discuss.agent.md`
	- `.github/agents/research.agent.md`
	- `.github/agents/plan.agent.md`
	- `.github/agents/tdd.agent.md`
	- `.github/agents/verify.agent.md`
	- `.github/prompts/execute.prompt.md`
	- `.github/prompts/finish-branch.prompt.md`
- [x] Update `.github/agents/research.agent.md` to replace hard stop wording with guided handoff behavior to Planner while still requiring explicit `/plan` command.
- [x] Update `.github/prompts/research.prompt.md` to mirror guided handoff semantics and ensure Phase 2 output tells users exactly how to continue.
- [x] Update `.github/agents/plan.agent.md` completion section to require execution-mode choice output and include explicit reselection-to-agent-mode instructions before `/execute`.
- [x] Update `.github/prompts/plan.prompt.md` to align with planner output contract and present mode-aware transition details that map cleanly to `/execute`.
- [x] Update `.github/prompts/execute.prompt.md` to include a deterministic pre-flight checklist:
	- confirm selected mode,
	- confirm agent reselected to agent mode when Mode A is chosen,
	- confirm ability to edit files and run terminal commands before implementation.
- [x] Update `.github/prompts/finish-branch.prompt.md` tool declaration and wording to align with workflow-critical tool policy and Verify-role mapping.
- [x] Validate no conflicting handoff behavior remains in targeted files via repo search (`handoff`, `/plan`, `/execute`, `Do NOT automatically hand off`, `tools:`).

#### Frontend Tasks
- [ ] Not applicable (workflow/prompt/agent configuration only).

#### Infrastructure
- [ ] No runtime infrastructure changes.
- [ ] Keep changes on branch `issue/ISSUE-007-agent-workflow-handoffs-tools` and maintain issue traceability to #7.

#### Tests
- [ ] **Unit**: N/A (prompt/agent configuration).
- [ ] **Integration**: Manual workflow integration checks in VS Code Copilot:
	- Discuss -> Research auto handoff still works.
	- Research completion presents guided Plan handoff text/button behavior.
	- Planner output includes mode choice and agent-reselection step.
	- Execute flow validates mode pre-flight and terminal/edit capability expectations.
- [ ] **E2E**: Dry-run one issue lifecycle through `/start-issue` -> `/discuss` -> `/research` -> `/plan` -> `/execute` (stop before code changes) to confirm transitions and messaging.

#### Documentation
- [x] Update workflow docs to match final behavior decisions:
	- `docs/MIGRATION-V2.md`
	- `docs/COMPLETE-WORKFLOW-VISUAL.md`
	- `docs/AGENT-MODE-VS-TDD-AGENT.md`
	- `docs/DEVELOPER-STANDARD-GUIDE.md` (if references differ after edits)
- [ ] Ensure issue tracking docs stay in sync:
	- `docs/issues/ISSUE-007-agent-workflow-handoffs-tools.md` Phase 3 summary
	- `work/ISSUE-007-agent-workflow-handoffs-tools/result.md` placeholders ready for Phase 4

### Plan Approved By
- **Developer**: SriSatyaLokesh — 2026-03-21
- **Team Lead**: [Name] — [Date] (if required)

### Acceptance Criteria (Implementation Phase)
- [ ] Research completion provides guided Plan handoff UX and no contradictory "do not hand off" instruction in targeted research files.
- [ ] Planner completion output includes explicit execution-mode choice and reselection-to-agent-mode step for Mode A.
- [ ] Execute prompt contains explicit pre-flight checks for mode selection, agent reselection, and tool capability expectations.
- [ ] Workflow-critical targeted files use a consistent valid tool declaration style and support terminal access policy for their phase responsibilities.
- [ ] Workflow docs and issue docs reflect the final behavior, with no documented contradiction between migration notes and actual prompts/agents.

### Open Questions
1. Canonical tool token set: keep existing repo style where currently working, or enforce one strict token vocabulary across all targeted files? -> **TBD for execute phase implementation spike**.

---

## Notes
- This plan is created in Phase 3 and referenced during execution
- Changes during implementation should be noted in `result.md`
- Keep this as the source of truth for "what was supposed to happen"
