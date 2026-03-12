# Work Folder Structure

This folder contains **active work items** organized by issue.

## Structure

Each issue gets its own folder:
```
work/
├── ISSUE-001-login-authentication/
│   ├── plan.md      # Requirements, Research, Implementation Plan
│   └── result.md    # Execution notes, Test results, Verification
│
├── ISSUE-042-rate-limiting/
│   ├── plan.md
│   └── result.md
│
└── ISSUE-123-performance-optimization/
    ├── plan.md
    └── result.md
```

## Naming Convention

```
work/ISSUE-[number]-[kebab-case-description]/
```

**Examples**:
- `ISSUE-001-login-authentication`
- `ISSUE-042-rate-limiting`
- `ISSUE-123-performance-optimization`

The number should match:
- GitHub issue number (if using GitHub)
- Branch name: `fix/042-rate-limiting`
- Commit messages: `Resolves #42`

## File Contents

### plan.md
Contains **what we're supposed to build**:
- Phase 1: Requirements (from Discuss agent)
- Phase 2: Research findings (from Research agent)
- Phase 3: Implementation plan (from Planner agent)

### result.md
Contains **what actually happened**:
- Phase 4: Execution notes (from TDD Implementer agent)
- Phase 5: Verification report (from Verify agent)

## Workflow

1. **Start**: `@discuss [feature description]`
   - Creates folder
   - Fills `plan.md` Phase 1

2. **Research**: `@research work/ISSUE-XXX-name`
   - Updates `plan.md` Phase 2

3. **Plan**: `@planner work/ISSUE-XXX-name`
   - Updates `plan.md` Phase 3

4. **Implement**: `@tdd work/ISSUE-XXX-name`
   - Updates `result.md` Phase 4

5. **Verify**: `@verify work/ISSUE-XXX-name`
   - Updates `result.md` Phase 5

6. **Done**: PR merged, GitHub issue closed, work complete

## Templates

New work folders are created from:
- `docs/templates/plan-template.md`
- `docs/templates/result-template.md`

## GitHub Integration

If this is a GitHub repository, agents will offer to:
- Create GitHub issues for tracking
- Create feature branches automatically
- Create PRs with proper linking (`Fixes #XX`)
- Auto-merge when CI passes

## Notes

- Each folder can contain artifacts: screenshots, diagrams, scripts
- Multiple developers can work on different issues simultaneously
- No merge conflicts — each issue is isolated
- Completed work stays here for reference

---

For full documentation, see:
- [WORK-FOLDER-STRUCTURE.md](../docs/WORK-FOLDER-STRUCTURE.md)
- [WORKFLOW-INTEGRATION-SUMMARY.md](../docs/WORKFLOW-INTEGRATION-SUMMARY.md)
