# [Project Name] — GitHub Copilot Configuration

> [!IMPORTANT]
> **SETUP REQUIRED** — This is a template. Complete these steps before your team starts:
>
> 1. **Replace `[Project Name]`** in the heading above with your actual project name
> 2. **Replace the `Stack:` line** with your actual tech stack
> 3. **Fill in `## Conventions`** at the bottom with project-specific coding rules
> 4. **Auto-generate instructions fast**: Type `/init` in Copilot Chat — it analyzes your workspace and generates tailored instructions automatically. Paste the output into the `## Conventions` section.
> 5. **Add recommended slash commands** (`.vscode/settings.json`):
>    ```json
>    "chat.promptFilesRecommendations": {
>      "start-issue": true,
>      "debug": true
>    }
>    ```
>    This surfaces `/start-issue` and `/debug` as suggested actions every time chat opens.

> For the full beginner guide, start at [`learn/00-introduction.md`](https://srisatyalokesh.github.io/copilot-best-practices-for-teams/)

## What This Project Is
[One sentence: what does this project do?]

Stack: [e.g., Node.js + Express + PostgreSQL + Redis]

## Key Documentation
- **[Team Copilot Guide](../docs/team-copilot-guide.md)**: How we use Copilot as a team — start here
- **[Docs Architecture](../docs/docs-architecture.md)**: Where every doc lives

## Our Issue Workflow
Every work item is an **Issue** going through 5 phases:
`/discuss` → `/research` → `/plan` → `/execute` → `/verify`

## Where to Find Things
- **Flow docs**: `docs/flows/[flow-name]-flow.md`
- **API docs**: `docs/apis/[domain]/[endpoint].api.md`
- **Issue docs**: `docs/issues/issue-xxx-name.md`
- **Templates**: `docs/templates/`

## For AI Agents
1. Read the API doc from `docs/apis/` before touching endpoint code
2. Read the flow doc from `docs/flows/` for business context
3. Read the Issue doc from `docs/issues/` for what's planned in this session
4. After code changes affecting an API → run `/update-api-doc`
5. After finishing work → update Issue doc progress tracker

## Conventions (fill in per project — or run `/init` to auto-generate)
- [Your project-specific coding rules go here]
