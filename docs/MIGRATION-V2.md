# Migration Guide: V1 → V2 (March 2026)

**Major improvements in this release:**
- ✅ **Work folder structure** (`work/ISSUE-XXX-name/` replaces `docs/issues/ISSUE-XXX.md`)
- ✅ **GitHub CLI integration** (automatic issue/PR creation)
- ✅ **Dual-mode execution** (Agent Mode vs TDD Agent)
- ✅ **Activity logging** (`logs/copilot/agent-activity.log`)
- ✅ **Fixed agent handoff chain** (auto Discuss→Research + guided Research→Plan)
- ✅ **11 skills auto-loaded** (TDD, doc-reviewer, GitHub CLI, etc.)

---

## Breaking Changes

### 1. Issue Documentation Structure

**Old (V1)**:
```
docs/issues/ISSUE-042-login-rate-limiting.md   ← Single file with all 5 phases
```

**New (V2)**:
```
work/ISSUE-042-login-rate-limiting/
  ├── plan.md      ← Intent: Phases 1-3 (Discuss, Research, Plan)
  └── result.md    ← Reality: Phases 4-5 (Execute, Verify)
```

**Why?** 
- Prevents merge conflicts when multiple developers work on different issues
- Separates "what we want to build" from "what we built"
- Easier for agents to read specific sections without parsing a large file

**Migration**:
```powershell
# Create work folder structure
New-Item -ItemType Directory -Force work/

# For each existing issue doc, split into plan.md and result.md
# (Manual — templates available at docs/templates/plan-template.md)
```

---

### 2. Agent Handoff Chain

**Old (V1)**:
```
@Discuss → @Research → Click button → @Planner → Click button → @TDD
                           ⬆️ Manual                  ⬆️ Manual
```

**New (V2)**:
```
@Discuss → @Research → /plan → /execute (Mode A/B) → /verify
           (auto)      (guided/user) (user choice)   (user)
```

**Why?** 
- Full auto-handoffs past Research were causing stuck-session failure modes
- `/execute` prompt offers choice between Agent Mode and TDD Agent (was bypassed before)
- Guided transitions plus explicit commands (`/plan`, `/execute`, `/verify`) are clearer and more predictable

**What changed**:
- Research agent now uses guided Planner handoff (button + `/plan` fallback), without forced send
- Planner agent no longer has `handoffs: [agent: TDD]` — completes with "Run `/execute`" message
- User must explicitly run `/plan`, `/execute`, and `/verify` prompts

---

### 3. Execution Modes

**Old (V1)**:
```
/execute → @TDD agent (only option)
```

**New (V2)**:
```
/execute → Choose:
  Mode A: Agent Mode (copy context to new chat, implement, return)
  Mode B: TDD Agent (strict Red-Green-Refactor in same chat)
```

**Why?**
- Agent Mode leverages Copilot's full workspace awareness for complex features
- TDD Agent provides strict discipline for routine work
- Flexibility for different feature types

**When to use which**:
- **Agent Mode**: Complex features, refactoring, large changes across many files
- **TDD Agent**: Small features, bug fixes, well-defined tasks

---

### 4. GitHub CLI Integration

**Old (V1)**:
```
Manual: Developer creates GitHub issue, PR, and merges via web UI
```

**New (V2)**:
```
Automatic:
  - /start-issue → offers to create GitHub issue
  - @TDD completes → offers to create PR
  - @Verify passes → offers to merge PR
```

**Why?**
- Reduces context switching (stay in VS Code)
- Links GitHub issue to work folder automatically
- Tracks PR status and CI results

**Requirements**:
- GitHub CLI (`gh`) installed and authenticated
- Working in a GitHub repository

---

### 5. Activity Logging

**Old (V1)**:
```
No logging — only git commits track progress
```

**New (V2)**:
```
logs/copilot/agent-activity.log ← JSON Lines format, one entry per phase
```

**Why?**
- Audit trail for what agents did
- Debug workflow issues
- Team visibility into progress

**Log entry example**:
```json
{
  "timestamp": "2026-03-12T10:30:00Z",
  "issueId": "ISSUE-042",
  "phase": "discuss",
  "agent": "Discuss",
  "status": "complete",
  "summary": "Rate limiting for login endpoint defined",
  "workFolder": "work/ISSUE-042-login-rate-limiting/",
  "nextPhase": "research"
}
```

---

## New Commands

| Command | Purpose | When to use |
|---------|---------|-------------|
| `/start-issue` | Create branch + work folder + define requirements | **Always start here** |
| `/plan` | Create implementation plan | After research completes |
| `/execute` | Choose execution mode (Agent/TDD) | After plan approved |
| `/verify` | Check requirements before PR | After implementation |
| `/status` | Check current phase and next step | When lost or resuming work |

---

## Skill System (New in V2)

**11 auto-loading skills** (loaded when mentioned in conversation):

| Skill | When it loads |
|-------|---------------|
| `test-driven-development` | When TDD, testing, or Red-Green-Refactor mentioned |
| `subagent-driven-development` | When executing plan, multiple tasks, or subagent dispatch mentioned |
| `doc-reviewer` | When doc review, README audit, or markdown lint mentioned |
| `documentation-writer` | When creating docs, tutorials, API docs, or guides |
| `github-cli-workflow` | When GitHub issue, PR, or gh command mentioned |
| `agent-activity-logger` | When activity log, agent logging, or log format mentioned |
| `playwright-automation-fill-in-form` | When form filling, web automation, or Playwright form mentioned |
| `playwright-explore-website` | When website exploration, test planning, or UI review mentioned |
| `playwright-generate-test` | When Playwright test, e2e test, or browser test mentioned |
| `receiving-code-review` | When code review feedback, PR comments, or reviewer suggestions received |
| `requesting-code-review` | When requesting review, checking quality, or before merge |

**How they work**:
- Agents mention skill keywords in conversation → skill auto-loads
- Provides deep context without bloating agent definitions
- Skills are modular (add more as needed)

---

## Migration Checklist

For teams upgrading from V1:

### Phase 1: Backup
- [ ] `git commit -m "backup: before v2 migration"`
- [ ] Copy existing `docs/issues/` to `docs/issues.backup/`

### Phase 2: Install V2
- [ ] Pull latest boilerplate changes
- [ ] Copy updated `.github/` folder
- [ ] Create `work/` folder at repo root
- [ ] Create `logs/` folder at repo root
- [ ] Add `logs/` to `.gitignore`

### Phase 3: Update Configuration
- [ ] Review `.github/copilot-instructions.md` (set primary branch)
- [ ] Update VS Code settings (add `chat.promptFilesRecommendations`)
- [ ] Reload VS Code window

### Phase 4: Migrate Active Issues
- [ ] For each open issue in `docs/issues/`:
  - Create `work/ISSUE-XXX-name/` folder
  - Split into `plan.md` (Phases 1-3) and `result.md` (Phases 4-5)
  - Push to branch

### Phase 5: Test Workflow
- [ ] Run `/start-issue` with a small test feature
- [ ] Verify GitHub CLI integration (if applicable)
- [ ] Verify `/plan`, `/execute`, `/verify` flow
- [ ] Check `logs/copilot/agent-activity.log` is created

### Phase 6: Team Training
- [ ] Share updated learn guides (learn/ folder)
- [ ] Demo the new workflow in team meeting
- [ ] Update team onboarding docs

---

## Rollback Plan

If V2 causes issues:

1. **Revert `.github/` folder**:
   ```bash
   git checkout HEAD~1 .github/
   ```

2. **Keep using `docs/issues/` structure** (still works, just not recommended)

3. **Report issues**: [GitHub Issues](https://github.com/SriSatyaLokesh/copilot-team-workflow/issues)

---

## FAQ

**Q: Do I have to use the work folder structure?**  
A: No, the old `docs/issues/` structure still works. But work folders prevent merge conflicts and are recommended for teams.

**Q: Can I skip Agent Mode and always use TDD Agent?**  
A: Yes! Choose Mode B every time. Agent Mode is optional.

**Q: What if I don't have GitHub CLI installed?**  
A: The workflow still works — you just won't get automatic issue/PR creation. Agents will skip those steps.

**Q: Do I need to migrate all old issue docs?**  
A: No. Only migrate active issues. Completed issues can stay in `docs/issues/` for historical reference.

**Q: Will this break my existing workflow scripts?**  
A: Check any scripts that hardcode `docs/issues/` paths. Update them to use `work/` instead.

---

**Migration support**: [GitHub Discussions](https://github.com/SriSatyaLokesh/copilot-team-workflow/discussions)  
**Bug reports**: [GitHub Issues](https://github.com/SriSatyaLokesh/copilot-team-workflow/issues)
