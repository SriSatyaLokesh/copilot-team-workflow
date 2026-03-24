# Model Selection Guide

This document explains which AI model is assigned to each agent, prompt, and skill, and why.

---

## Available Models

This boilerplate supports 4 model families:

| Model | Strengths | Cost | Speed | Best For |
|-------|-----------|------|-------|----------|
| **GPT-4o** | Creative tasks, natural conversation, brainstorming | Medium | Fast | Requirements gathering, documentation, API design |
| **Claude Sonnet 4** | Balanced code analysis, reliable implementation | Medium | Medium | Code generation, codebase research, TDD |
| **Claude Opus 4.5** | Deepest reasoning, complex decisions, critical gates | High | Slow | Planning, verification, code review, orchestration |
| **Claude Haiku 4** | Fast, lightweight tasks | Low | Very Fast | Status checks, quick docs sync |

---

## Model Assignment Strategy

### Agents (8 total)

| Agent | Model | Why |
|-------|-------|-----|
| **Discuss** | `gpt-4o` | Conversational, great at asking clarifying questions, gathering requirements |
| **Research** | `claude-sonnet-4-5` | Deep codebase analysis, pattern recognition, technical search |
| **Planner** | `claude-opus-4-5` | Complex planning decisions, architecture design, sequencing tasks |
| **TDD Implementer** | `claude-sonnet-4-5` | Code generation + testing, balanced speed/quality |
| **Reviewer** | `claude-opus-4-5` | Thorough quality analysis across 4 dimensions (correctness, security, performance, standards) |
| **Verify** | `claude-opus-4-5` | Critical final gate, no false positives allowed |
| **ApiBuilder** | `gpt-4o` | Creative API integration, good at understanding external APIs |
| **ParallelBuilder** | `claude-opus-4-5` | Complex orchestration, dependency analysis |

### Prompts (14 total)

Prompts delegate to agents, so they use the same model as their target agent:

| Prompt | Model | Reasoning |
|--------|-------|-----------|
| `/start-issue` | `gpt-4o` | Conversational setup, branch selection |
| `/discuss` | `gpt-4o` | Requirements gathering → Discuss agent |
| `/research` | `claude-sonnet-4-5` | Codebase exploration → Research agent |
| `/plan` | `claude-opus-4-5` | Task breakdown → Planner agent |
| `/execute` | `claude-sonnet-4-5` | TDD implementation → TDD agent |
| `/verify` | `claude-opus-4-5` | Final verification → Verify agent |
| `/debug` | `claude-sonnet-4-5` | Root cause analysis → TDD agent |
| `/add-new-api` | `gpt-4o` | Creative integration → ApiBuilder agent |
| `/receive-review` | `claude-sonnet-4-5` | Code changes → TDD agent |
| `/finish-branch` | `claude-opus-4-5` | Final gate + merge decision → Verify agent |
| `/generate-api-doc` | `gpt-4o` | **Creative documentation** (writes from scratch) |
| `/update-api-doc` | `claude-haiku-4` | **Simple sync** (updates existing fields) |
| `/status` | `claude-haiku-4` | **Fast read** (just parses Issue doc) |
| `/sync-docs` | `claude-sonnet-4-5` | **Complex analysis** (diffs code vs docs) |

### Skills (11 total)

Skills are reference documents loaded as context — they don't execute, so model selection doesn't apply. However, when a skill is invoked by an agent, it follows that agent's model.

| Skill | Used By | Effective Model |
|-------|---------|-----------------|
| `test-driven-development` | TDD Implementer | `claude-sonnet-4-5` |
| `subagent-driven-development` | Any agent orchestrating subagents | Varies |
| `receiving-code-review` | TDD Implementer | `claude-sonnet-4-5` |
| `requesting-code-review` | Developer (manual) | Varies |
| `doc-reviewer` | Reviewer | `claude-opus-4-5` |
| `documentation-writer` | Any agent writing docs | Varies (usually `gpt-4o`) |
| `playwright-*` (3 skills) | TDD Implementer | `claude-sonnet-4-5` |
| `agent-activity-logger` | All agents | N/A (logging format reference) |
| `acquire-codebase-knowledge` | Research | `claude-sonnet-4-5` |

---

## Decision Rules

### Use GPT-4o when:
- ✅ Task involves conversation (requirements, brainstorming)
- ✅ Creative documentation needed
- ✅ Designing new APIs or integrations
- ✅ Explaining concepts to humans

### Use Claude Sonnet 4 when:
- ✅ Writing production code
- ✅ Analyzing existing code
- ✅ TDD implementation (tests + code)
- ✅ Codebase research
- ✅ Complex doc syncing (analyzing diffs)

### Use Claude Opus 4.5 when:
- ✅ Planning critical architecture
- ✅ Final verification before merge
- ✅ Comprehensive code review
- ✅ Orchestrating parallel tasks
- ✅ High-stakes decision gates

### Use Claude Haiku 4 when:
- ✅ Simple status checks
- ✅ Quick text parsing
- ✅ Lightweight doc updates
- ✅ Fast iteration on well-defined tasks

---

## Cost Optimization Tips

1. **Don't use Opus for exploratory work** — use Sonnet for research, then Opus for final decisions
2. **Use Haiku for status checks** — reading Issue docs is cheap, no need for expensive models
3. **Use GPT-4o for docs** — better at creative writing than Claude
4. **Use Sonnet for implementation** — best balance of code quality and speed

---

## Customizing Models

To change a model for an agent or prompt, edit the `model:` field in the frontmatter:

```yaml
---
description: 'Agent description'
agent: 'AgentName'
tools: ['tool1', 'tool2']
model: 'Exact model label from the VS Code model picker'  # optional
---
```

Important:
- The value must match the exact model label shown in your VS Code model picker.
- If you are not sure of the exact label, omit the `model:` field and let the current picker selection apply.
- Model availability depends on your Copilot plan, feature flags, and enabled providers.

**Restart VS Code** after changing models to reload the agents.

---

## When to Use Gemini 2.0 Pro

Gemini excels at:
- **Multimodal tasks** (images, diagrams, UI screenshots)
- **Web search integration** (live documentation lookup)
- **Exploring unfamiliar codebases** (with web context)

Consider using Gemini for:
- Research agent (when external docs are needed)
- Documentation writer (when referencing online examples)
- API builder (when integrating with poorly-documented APIs)

To enable Gemini for an agent:
```yaml
model: 'gemini-2.0-pro'
```

---

## Model Performance Matrix

| Task Type | Recommended Model | Alternative | Avoid |
|-----------|------------------|-------------|-------|
| Requirements gathering | GPT-4o | Sonnet | Haiku (too fast, misses nuance) |
| Codebase analysis | Sonnet | Opus (slower but deeper) | Haiku (too shallow) |
| Planning | Opus | Sonnet | Haiku (misses dependencies) |
| Code generation | Sonnet | GPT-4o | Haiku (quality issues) |
| Code review | Opus | Sonnet | Haiku (misses issues) |
| Final verification | Opus | — | Anything else (too risky) |
| Documentation | GPT-4o | Sonnet | — |
| Status checks | Haiku | Sonnet | Opus (overkill) |

---

## Telemetry & Monitoring

Track which models are most effective for your team:

```bash
# Count model usage in activity log
cat logs/copilot/agent-activity.log | jq -r '.agent' | sort | uniq -c

# Track which phases take longest
cat logs/copilot/agent-activity.log | jq -r 'select(.status=="complete") | "\(.phase) \(.agent)"' | sort | uniq -c
```

If a particular model consistently produces low-quality output for your project, adjust the assignments in this guide and update the agent/prompt files.

---

## Future Model Support

As new models become available:
1. Add them to the "Available Models" table above
2. Test on non-critical agents first (Research, Documentation)
3. Compare quality with existing models on 3-5 real Issues
4. Update this guide with your findings
5. Roll out to critical agents (Plan, Verify) only after validation

---

**Last updated**: 2026-03-03
**Review frequency**: Quarterly (new models release often)
