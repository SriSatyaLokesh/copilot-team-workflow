# Docs: Issues Folder

This folder contains **one document per work item** — feature, bug fix, improvement, or task.

## Naming Convention

```
ISSUE-001-login-rate-limiting.md
ISSUE-002-order-export-to-csv.md
ISSUE-003-fix-session-timeout.md
```

- **ISSUE-XXX** — sequential number, zero-padded to 3 digits
- **short-name** — kebab-case, 2-5 words max

## Lifecycle

Each document travels through 5 phases. The `status` field in the frontmatter tracks where it is:

```
discuss → research → plan → execute → verify → done
```

## How to Create a New Issue Doc

```bash
cp docs/templates/issue-template.md docs/issues/ISSUE-XXX-your-feature.md
```

Then fill in:
1. Update the frontmatter (`issue-id`, `title`, `branch`, `developer`)
2. Work through each phase with the appropriate agent
3. Mark `status: done` when Verify passes

## Example

See [`ISSUE-001-login-rate-limiting.example.md`](./ISSUE-001-login-rate-limiting.example.md) for a fully completed example showing all 5 phases.
