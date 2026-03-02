# [Project Name] — GitHub Copilot Configuration

> For the full team workflow guide, see [docs/team-copilot-guide.md](../docs/team-copilot-guide.md)

## What This Project Is
[One sentence: what does this project do?]

Stack: [e.g., Node.js + Express + PostgreSQL + Redis]

## Key Documentation
- **[Team Copilot Guide](../docs/team-copilot-guide.md)**: How we use Copilot as a team — start here
- **[Docs Architecture](../docs/docs-architecture.md)**: Where every doc lives
- **[Research & Decisions](../docs/RESEARCH-DECISIONS.md)**: Why we chose this approach
- **[VS Code Research](../docs/vscode-team-copilot-research.md)**: Source research from VS Code team

## Our Workflow
Every work item is an **Issue** that goes through 5 phases via Copilot:

| Phase | Slash Command | Agent |
|-------|--------------|-------|
| 1. Discuss | `/discuss` | Discuss |
| 2. Research | `/research` | Research |
| 3. Plan | `/plan` | Planner |
| 4. Execute | `/execute` | TDD |
| 5. Verify | `/verify` | Verify |

## Where to Find Things
- **Flow docs**: `docs/flows/[flow-name]-flow.md`
- **API docs**: `docs/apis/[domain]/[endpoint].api.md`
- **Issue docs**: `docs/issues/ISSUE-XXX-name.md`
- **Templates**: `docs/templates/`

## For AI Agents Working in This Project
1. Read the relevant API doc from `docs/apis/` before touching endpoint code
2. Read the flow doc from `docs/flows/` for business context
3. Read the Issue doc from `docs/issues/` for what's planned
4. Follow conventions in `docs/CONTRIBUTING.md`
5. After making code changes that affect an API: run `/update-api-doc`
6. After completing Issue work: update the Issue doc progress tracker
