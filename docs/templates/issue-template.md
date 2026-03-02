---
ISSUE-id: "ISSUE-XXX"
title: "[Short descriptive title]"
type: "feature"  # feature | fix | story | task | improvement
status: "discuss"  # discuss | research | plan | execute | verify | done
owner: "[Developer Name]"
branch: "issue/ISSUE-XXX-feature-name"
created: "YYYY-MM-DD"
last-updated: "YYYY-MM-DD"
related-flow: "[flow-name]-flow"  # e.g., auth-flow
related-apis: []  # e.g., ["auth/login", "orders/create-order"]
---

# ISSUE-XXX: [Short Title]

---

## 📋 Phase 1: Discuss
*Status: [ ] Not Started | [/] In Progress | [x] Done*

### What Are We Building?
[2-3 sentences: what is this Issue? What problem does it solve?]

### Why Are We Building It?
[Business reason or user need]

### Who Needs This?
[User role or team/system that benefits]

### Requirements (Agreed with Team)
- [ ] [Requirement 1 — be specific]
- [ ] [Requirement 2]
- [ ] [Requirement 3]

### Out of Scope
- [Things we explicitly decided NOT to include]

### Open Questions
1. [Question] → Answer: [Answer or "TBD"]

---

## 🔍 Phase 2: Research
*Status: [ ] Not Started | [/] In Progress | [x] Done*

### Codebase Context Found
[What Copilot found in the codebase that's relevant]

### Existing Patterns to Follow
- **Pattern**: [Name] → **Location**: `[file path]`
- **Pattern**: [Name] → **Location**: `[file path]`

### Files That Will Be Affected
| File | Why |
|------|-----|
| `[path/to/file.ts]` | [What needs to change] |
| `[path/to/service.ts]` | [What needs to change] |

### Related Docs
- [Flow doc](../flows/[related]-flow.md)
- [API doc](../apis/[domain]/[endpoint].api.md)

---

## 📐 Phase 3: Plan
*Status: [ ] Not Started | [/] In Progress | [x] Done*

### Architecture Decisions
| Decision | Choice | Reason |
|----------|--------|--------|
| [e.g., caching strategy] | [e.g., Redis sliding window] | [why] |

### Implementation Tasks

**Backend**
- [ ] [Task 1 — e.g., Add Redis rate limit middleware]
- [ ] [Task 2 — e.g., Update login route to use rate limiter]
- [ ] [Task 3]

**Frontend**
- [ ] [Task 1]
- [ ] [Task 2]

**Infrastructure**
- [ ] [Task 1 — e.g., Add Redis env vars to production config]

**Tests**
- [ ] Unit: [what to test]
- [ ] Integration: [what to test]
- [ ] E2E: [what to test]

**Docs**
- [ ] Update `docs/apis/[domain]/[endpoint].api.md`
- [ ] Update `docs/flows/[flow].md` if flow changes

### Acceptance Criteria
- [ ] [Specific, testable criterion 1]
- [ ] [Specific, testable criterion 2]
- [ ] [Specific, testable criterion 3]

### Plan Approved By
- Developer: [Name] — [Date]
- Team Lead (if needed): [Name] — [Date]

---

## 🔨 Phase 4: Execute
*Status: [ ] Not Started | [/] In Progress | [x] Done*

### Progress Tracker
| Task | Status | Notes |
|------|--------|-------|
| [Task from plan] | ⬜ | |
| [Task from plan] | 🔄 | |
| [Task from plan] | ✅ | |

### Execution Notes
*(Copilot updates this automatically as work progresses)*

**[Date]**
- [What was done]
- [Decision made and why]
- [Any blockers or changes]

### Commits Made
- `feat: ISSUE-XXX [description]` — [commit hash]
- `test: ISSUE-XXX [description]` — [commit hash]

---

## ✅ Phase 5: Verify
*Status: [ ] Not Started | [/] In Progress | [x] Done*

### Test Results
- Unit tests: ✅ / ❌ ([X/Y passing])
- Integration tests: ✅ / ❌ ([X/Y passing])
- E2E tests: ✅ / ❌ ([X/Y passing])

### Requirements Check
| Requirement (from Phase 1) | Met? | Evidence |
|---------------------------|------|---------|
| [Requirement 1] | ✅ | [how verified] |
| [Requirement 2] | ✅ | [how verified] |

### Docs Updated
- [ ] API docs updated
- [ ] Flow docs updated (if applicable)
- [ ] Issue doc complete

### PR Created
- PR: [link]
- Reviewer: [name]

### Final Status
- [ ] Merged to main
- [ ] Issue archived as: `status: done`

---

## 📝 Developer Notes
*(Personal notes — short context for yourself and teammates)*

**[Date]**
[Free-form notes, blockers, questions, decisions]
