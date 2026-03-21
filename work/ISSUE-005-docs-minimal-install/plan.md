# ISSUE-005 — Docs: Ship Minimal Docs Payload During Boilerplate Install

> **GitHub Issue**: [#5](https://github.com/SriSatyaLokesh/copilot-team-workflow/issues/5)
> **Branch**: `feature/005-docs-minimal-install`
> **Created**: 2026-03-21

---

## Phase 1 — Requirements [x] Complete

### Problem
Installing this boilerplate currently copies too many docs into downstream projects. Teams only need starter documentation scaffolding, not boilerplate-specific reference material.

### Goal
Ship a **minimal docs payload** during installation:
- Keep essential structure and templates
- Avoid copying boilerplate narrative/reference docs into consumer repos

### Requirements
- [ ] Fresh install copies only allowlisted docs paths
- [ ] `docs/templates/**` is always copied
- [ ] Boilerplate-only docs (long-form guides, research writeups, migration notes) are NOT copied into target projects
- [ ] README and learn guide show accurate install behavior
- [ ] Migration/cleanup guidance is documented for existing adopters

### Acceptance Criteria
1. A fresh install creates the correct minimal `docs/` structure in the consumer repo
2. `docs/templates/` is always present post-install
3. No boilerplate-internal docs (e.g., research writeups) are copied
4. Install instructions (PowerShell + Bash) reflect the new behavior exactly
5. A "Copied vs Not Copied" reference table exists in the README or learn guide

### Out of Scope
- Rewriting the full docs architecture
- Changing agent workflow behavior
- Implementing project-specific docs generation

---

## Phase 2 — Research [x] Complete

### Files to Modify

| File | What changes |
|:-----|:-------------|
| `README.md` | Update PowerShell + Bash install snippets (lines ~37–71) |
| `learn/08-boilerplate-setup.md` | Update PowerShell + Bash install snippets (lines ~50–100) |
| `docs/README.md` | Remove dangling reference to `DEVELOPER-STANDARD-GUIDE.md` (last line) |

> ⚠️ **Sync risk**: `README.md` and `learn/08-boilerplate-setup.md` contain near-identical install snippets. They must always be updated together or they drift.

---

### Current Install Behavior (what ships today)

**PowerShell:**
```powershell
Get-ChildItem copilot-boilerplate\docs -Exclude 'learn' | Copy-Item -Destination .\docs -Recurse -Force
```
**Bash:**
```bash
rsync -av --exclude='learn/' copilot-boilerplate/docs/ ./docs/ 2>/dev/null || { cp -r copilot-boilerplate/docs ./ && rm -rf docs/learn; }
```
Both copy **everything in `docs/`** except `learn/`. This ships 13 boilerplate-only markdown files to consumer projects.

---

### Docs Inventory — Copied vs Not Copied

| Path | Currently shipped | Should ship | Classification |
|:-----|:-----------------:|:-----------:|:---------------|
| `docs/README.md` | ✅ | ✅ | Folder index — needed |
| `docs/project.md` | ✅ | ✅ | Consumer fills this in |
| `docs/templates/` (11 files) | ✅ | ✅ | Core value — always copy |
| `docs/codebase/` (7 files) | ✅ | ✅ | Team fills in — needed |
| `docs/apis/README.md` | ✅ | ✅ | Folder guide — needed |
| `docs/apis/auth/login.api.example.md` | ✅ | ✅ | Reference example |
| `docs/flows/README.md` | ✅ | ✅ | Folder guide — needed |
| `docs/flows/order-flow.example.md` | ✅ | ✅ | Reference example |
| `docs/external-apis/README.md` | ✅ | ✅ | Folder guide — needed |
| `docs/external-apis/dynamics/accounts.api.example.md` | ✅ | ✅ | Reference example |
| `docs/issues/README.md` | ✅ | ✅ | Folder guide — needed |
| `docs/issues/ISSUE-001-login-rate-limiting.example.md` | ✅ | ✅ | Reference example |
| `docs/decisions/` (empty) | ✅ | ✅ | Empty folder needed |
| `docs/team-notes/` (empty) | ✅ | ✅ | Empty folder needed |
| `docs/AGENT-MODE-VS-TDD-AGENT.md` | ✅ | ❌ | Boilerplate-internal |
| `docs/blog-best-way-to-use-copilot.md` | ✅ | ❌ | Boilerplate blog |
| `docs/COMPLETE-WORKFLOW-VISUAL.md` | ✅ | ❌ | Boilerplate reference |
| `docs/CONTRIBUTING.md` | ✅ | ❌ | Boilerplate-internal |
| `docs/copilot-team-setup-guide.md` | ✅ | ❌ | Boilerplate guide |
| `docs/DEVELOPER-STANDARD-GUIDE.md` | ✅ | ❌ | Boilerplate guide |
| `docs/MIGRATION-V2.md` | ✅ | ❌ | Boilerplate upgrade notes |
| `docs/MODEL-SELECTION.md` | ✅ | ❌ | Boilerplate-internal |
| `docs/RESEARCH-DECISIONS.md` | ✅ | ❌ | Boilerplate research |
| `docs/team-copilot-guide.md` | ✅ | ❌ | Boilerplate guide |
| `docs/vscode-team-copilot-research.md` | ✅ | ❌ | Boilerplate research |
| `docs/WHATS-NEW-V2.md` | ✅ | ❌ | Boilerplate changelog |
| `docs/WORK-FOLDER-STRUCTURE.md` | ✅ | ❌ | Boilerplate-internal |

**13 files should NOT be copied.** All are flat UPPERCASE `.md` files at the root of `docs/`.

---

### Recommended Implementation Approach: Denylist

A **denylist (exclude-list)** is preferred over an allowlist because:
- New files added to needed subdirectories (e.g., a new codebase template) are copied automatically without updating install scripts.
- Less brittle — the boilerplate evolves but install scripts stay stable.
- The pattern is: copy everything in `docs/` except `learn/` (already excluded) AND the 13 boilerplate-only flat files.

**PowerShell pattern:**
```powershell
$excludeDocs = @('AGENT-MODE-VS-TDD-AGENT.md','blog-best-way-to-use-copilot.md','COMPLETE-WORKFLOW-VISUAL.md','CONTRIBUTING.md','copilot-team-setup-guide.md','DEVELOPER-STANDARD-GUIDE.md','MIGRATION-V2.md','MODEL-SELECTION.md','RESEARCH-DECISIONS.md','team-copilot-guide.md','vscode-team-copilot-research.md','WHATS-NEW-V2.md','WORK-FOLDER-STRUCTURE.md')
Get-ChildItem copilot-boilerplate\docs -Exclude ('learn' + $excludeDocs) | Copy-Item -Destination .\docs -Recurse -Force
```

**Bash (rsync) pattern:**
```bash
rsync -av \
  --exclude='learn/' \
  --exclude='AGENT-MODE-VS-TDD-AGENT.md' \
  --exclude='blog-best-way-to-use-copilot.md' \
  # ... (all 13 files)
  copilot-boilerplate/docs/ ./docs/
```

> ⚠️ **Rsync fallback risk**: The current Bash snippet has a fallback `{ cp -r ... && rm -rf docs/learn; }` for systems without rsync. This fallback needs to be updated to also remove the excluded files, OR the fallback should be replaced with a simpler explicit exclude loop.

---

### Risks

1. **Sync drift**: Two files contain install snippets. Must update both in the same commit.
2. **Rsync fallback**: The `cp -r + rm -rf` fallback is harder to denylist — needs careful handling.
3. **`docs/README.md` dangling link**: Last line references `DEVELOPER-STANDARD-GUIDE.md` which will no longer be present post-install — must be removed.
4. **No validation**: No test to confirm post-install tree matches expectation. Plan task 4 addresses this with a documented expected tree (not a script, per scope).

---

### Related Docs
- `learn/08-boilerplate-setup.md` — primary consumer-facing install guide
- `README.md` — install section (lines ~30–90)

---

## Phase 3 — Plan [x] Complete

### Architecture Decision
Use a **denylist** (named file excludes) rather than an allowlist. This means copy everything in `docs/` except `learn/` (already excluded) AND the 13 identified boilerplate-only flat `.md` files at the `docs/` root. Rationale: new templates or subfolders added to the boilerplate in future are copied automatically without updating install scripts.

### Tasks

- [x] **Task 1** — Fix `docs/README.md`: remove dangling link to `DEVELOPER-STANDARD-GUIDE.md` (last line of file)

- [x] **Task 2** — Update PowerShell snippet in `README.md`: replace single `Get-ChildItem` copy line with denylist `$excludeDocs` array + updated `Get-ChildItem -Exclude` call excluding all 13 boilerplate-only files

- [x] **Task 3** — Update Bash snippet in `README.md`: replace `rsync` line with explicit `--exclude` flags for all 13 files; update the `cp -r` fallback to individually remove each excluded file instead of only removing `docs/learn`

- [x] **Task 4** — Update PowerShell snippet in `learn/08-boilerplate-setup.md`: identical change to Task 2 (keep in sync)

- [x] **Task 5** — Update Bash snippet in `learn/08-boilerplate-setup.md`: identical change to Task 3 (keep in sync)

- [x] **Task 6** — Add "Copied vs Not Copied" reference table to `README.md` (after or within the Installation section)

- [x] **Task 7** — Add "Copied vs Not Copied" reference table + migration cleanup section to `learn/08-boilerplate-setup.md`

### Test Plan
No code logic — no unit tests required. Validation is manual:
- Follow the install snippet in a fresh temp directory and confirm only expected paths exist
- Confirm none of the 13 excluded files appear in the copied output
- Expected post-install `docs/` tree is documented inline in the guide (Task 7)

### Acceptance Criteria
- [ ] Fresh install copies only non-boilerplate-specific docs paths
- [ ] `docs/templates/**` is always copied
- [ ] All 13 boilerplate-only flat `.md` files are NOT copied into consumer projects
- [ ] `README.md` and `learn/08-boilerplate-setup.md` show identical, accurate install snippets
- [ ] "Copied vs Not Copied" table exists in at least one of those files
- [ ] Migration cleanup instructions exist for existing adopters

### Open Questions
None.

---

## Phase 4 — Execute [x] Complete

### Changes Made
- `docs/README.md` — removed dangling link to `DEVELOPER-STANDARD-GUIDE.md`
- `README.md` — updated PowerShell + Bash install snippets with 13-file denylist; added "Copied vs Not Copied" table
- `learn/08-boilerplate-setup.md` — updated PowerShell + Bash install snippets with 13-file denylist; added "Copied vs Not Copied" table + migration cleanup section

---

## Phase 5 — Verify [ ] Pending

> Verification report goes here after `/verify` runs.
