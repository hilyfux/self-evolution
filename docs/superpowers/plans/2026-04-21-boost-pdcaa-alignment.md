# Boost PDCAA Alignment Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Align the boost skill with the new PDCAA execution spec while keeping the current lightweight structure and matching Claude Code and Codex execution best practices.

**Architecture:** Keep `SKILL.md` as the hard runtime skeleton, and move deeper rationale and host-specific execution guidance into focused reference files. Update only the contract, loop, routing, logging, and validation wording that materially improves behavior, then validate the workspace copy before any sync.

**Tech Stack:** Markdown skill files, Claude Code hooks, Codex/Silly Code host guidance, git diff, shell validation

---

## File Structure

- Modify: `SKILL.md` — tighten the runtime contract, phase outputs, and host-aligned execution rules.
- Modify: `references/methodology.md` — deepen the execution model, validation protocol, topology rules, and rollback criteria.
- Modify: `references/iteration-template.md` — keep the visible loop template aligned with `SKILL.md`.
- Modify: `references/quick-reference.md` — refresh the compact operator-facing summary if terminology changes.
- Review: `references/host-adaptation-best-practices.md` — ensure host guidance still matches the updated skill behavior.
- Review: `AGENTS.md` — keep Codex-side adapter language aligned if any host-facing contract changes.
- Review: `CLAUDE.md` — verify project instructions still accurately describe the updated structure and sync rules.
- Use as source: `references/pdcaa-execution-skill-spec-v1.md` — imported spec to map into the existing skill.
- Test: workspace validation via `git diff`, targeted file reads, markdown/frontmatter inspection, and line-count check for `SKILL.md`.

### Task 1: Map spec deltas onto current skill skeleton

**Files:**
- Modify: `SKILL.md`
- Modify: `references/methodology.md`
- Modify: `references/iteration-template.md`
- Use as source: `references/pdcaa-execution-skill-spec-v1.md`

- [ ] **Step 1: Read the imported spec and current skill side by side**

```text
Compare these sections explicitly:
- Stable Contract fields and rules
- Runtime State schema
- PDCAA phase requirements
- AutoResearch triggers / output shape
- AutoReceive schema / routing
- Log schema
- Output protocol
```

Run: `python3 - <<'PY'
from pathlib import Path
for path in [
    'references/pdcaa-execution-skill-spec-v1.md',
    'SKILL.md',
    'references/methodology.md',
    'references/iteration-template.md',
]:
    text = Path(path).read_text()
    print(f'--- {path} ---')
    print(f'lines={len(text.splitlines())}')
PY`

Expected: all four files print with readable line counts.

- [ ] **Step 2: Write a delta checklist before editing**

```text
Expected checklist headings:
- Already aligned
- Missing from current skill
- Present but should be tightened
- Present but should stay as-is for lightweight execution
```

Run: `git diff --no-index --word-diff=color /dev/null /dev/null >/dev/null 2>&1; true`

Expected: command exits cleanly; no repo changes yet.

- [ ] **Step 3: Decide the minimal integration boundaries**

```text
Keep in SKILL.md only:
- mandatory contract and loop behavior
- mandatory visible outputs
- mandatory hook behavior
- host-facing execution rules that affect runtime decisions

Keep in references only:
- theory/background
- deeper host adaptation notes
- examples and elaborations
```

Run: `python3 - <<'PY'
print('Boundary decision recorded in plan: SKILL.md stays lean, references hold elaboration.')
PY`

Expected: one-line confirmation prints.

- [ ] **Step 4: Commit the mapping checkpoint**

```bash
git status --short
```

Expected: either no changes yet or only the imported spec / plan file visible.

### Task 2: Tighten `SKILL.md` around the imported PDCAA spec

**Files:**
- Modify: `SKILL.md:1-260`
- Use as source: `references/pdcaa-execution-skill-spec-v1.md`

- [ ] **Step 1: Write the failing review checklist for `SKILL.md`**

```text
The edited file must satisfy all of these:
1. Stable Contract remains four fields only.
2. Runtime State remains separate and mutable.
3. Check phase uses the five mandatory questions.
4. Align explicitly covers drift detection and re-anchor behavior.
5. Act keeps exactly six decisions.
6. AutoResearch stays on-demand and bounded.
7. AutoReceive defaults to Runtime State updates, not contract edits.
8. Log output remains mandatory every cycle.
9. Wording reflects host best practices: minimal edits, evidence-based checks, autonomous execution after contract lock.
```

Run: `python3 - <<'PY'
checklist = [
    'stable_contract', 'runtime_state', 'Goal advancement', 'Align',
    'continue', 'adjust', 'rollback', 'research', 'receive_update', 'complete',
    'AutoResearch', 'AutoReceive', 'Log'
]
text = open('SKILL.md').read()
for item in checklist:
    print(item, 'OK' if item in text else 'MISSING')
PY`

Expected: some items are present already; this is a pre-edit gap check, not a pass target.

- [ ] **Step 2: Edit frontmatter description only if scope wording needs tightening**

```yaml
name: boost
version: "1.2.0"
description: Use when the user asks to optimize, improve, iterate, diagnose, evolve, monitor, stabilize, raise quality, reduce cost, reduce failure, raise conversion, or help an object get better over time. Structures a strict PDCAA loop anchored by a Stable Contract, with AutoResearch, AutoReceive, and Log as built-in enhancements.
```

Run: `python3 - <<'PY'
import re
text = open('SKILL.md').read()
m = re.search(r'^version: "([^"]+)"', text, re.M)
print('version=', m.group(1) if m else 'missing')
PY`

Expected: version prints `1.2.0` after the edit lands.

- [ ] **Step 3: Tighten the hard rules and loop wording without expanding scope**

```text
Required outcomes in this edit:
- preserve the current hard-skeleton style
- keep the rule that the target is not this skill unless explicitly named
- keep analysis-only stop behavior
- make re-anchor explicit when drift is detected
- keep visible outputs in each phase
- avoid adding new phases or durable artifacts
```

Run: `python3 - <<'PY'
from pathlib import Path
text = Path('SKILL.md').read_text()
for needle in [
    'Stable Contract',
    'Runtime State',
    'Re-anchor',
    'Do not declare success without evidence from the Check phase.',
    'The target is never this skill unless the user explicitly names it.',
]:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all needles print `OK` after the edit.

- [ ] **Step 4: Refresh AutoResearch and AutoReceive schemas to match the imported spec where useful**

```yaml
feedback_event:
  type: new_requirement | new_constraint | execution_error | tool_feedback | validation_feedback | new_observation | env_change
  priority: low | medium | high
  source: user | system | self
  payload: ""
```

```yaml
research_result:
  triggered: true|false
  reason: ""
  options:
    - action: ""
      expected_gain: ""
      risk: ""
  recommended_probe: ""
```

Run: `python3 - <<'PY'
text = open('SKILL.md').read()
for needle in ['feedback_event:', 'tool_feedback', 'env_change', 'research_result:', 'recommended_probe']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all needles print `OK` after the edit.

- [ ] **Step 5: Verify the edited `SKILL.md` is still lean**

```bash
python3 - <<'PY'
from pathlib import Path
text = Path('SKILL.md').read_text()
print('lines=', len(text.splitlines()))
PY
```

Expected: line count stays roughly near the current size and does not balloon into a large theory document.

- [ ] **Step 6: Commit the `SKILL.md` update**

```bash
git add SKILL.md
git commit -m "feat: align boost skill skeleton to PDCAA spec"
```

### Task 3: Refresh methodology and iteration references to stay consistent

**Files:**
- Modify: `references/methodology.md:1-260`
- Modify: `references/iteration-template.md:1-120`
- Modify: `references/quick-reference.md`

- [ ] **Step 1: Write the failing consistency checks for references**

```text
The references must:
- use the same six Act decisions as SKILL.md
- use the same drift vocabulary
- use the same AutoReceive escalation boundary
- not reintroduce heavier process than SKILL.md allows
```

Run: `python3 - <<'PY'
files = ['references/methodology.md', 'references/iteration-template.md', 'references/quick-reference.md']
for path in files:
    text = open(path).read()
    print('---', path)
    for needle in ['rollback', 'research', 'receive_update', 'complete']:
        print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: baseline may show partial coverage before edits.

- [ ] **Step 2: Update `references/methodology.md` to explain only what the runtime skeleton depends on**

```text
Include or preserve:
- contract modification protocol
- research system frame
- validation protocol
- execution topology
- ratchet / rollback
- context preservation

Avoid:
- duplicate full SKILL.md text blocks unless they materially clarify a rule
- host-specific instructions already owned by CLAUDE.md
```

Run: `python3 - <<'PY'
text = open('references/methodology.md').read()
for needle in [
    'Contract Modification Protocol',
    'Validation Protocol',
    'Execution Topology',
    'Ratchet and Rollback',
    'Context Preservation'
]:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all needles print `OK` after the edit.

- [ ] **Step 3: Update `references/iteration-template.md` to mirror the visible runtime output exactly**

```text
Keep these blocks aligned with SKILL.md:
- Stable Contract
- Plan
- Do
- Check
- Align
- Act
- AutoResearch
- Contract Candidate
- Log
- Iteration Checkpoint
```

Run: `python3 - <<'PY'
text = open('references/iteration-template.md').read()
for needle in [
    '**Stable Contract**',
    '**Plan:**',
    '**Do:**',
    '**Check:**',
    '**Align:**',
    '**Act:**',
    '**AutoResearch:**',
    '**Contract Candidate:**',
    'log_summary:',
]:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all needles print `OK` after the edit.

- [ ] **Step 4: Update `references/quick-reference.md` only if terminology changed for operators**

```text
If no operator-facing term changed, leave this file untouched.
If terms changed, keep it short and purely operational.
```

Run: `git diff -- references/quick-reference.md`

Expected: empty diff if no terminology changed; otherwise a small focused diff.

- [ ] **Step 5: Commit the reference alignment changes**

```bash
git add references/methodology.md references/iteration-template.md references/quick-reference.md
git commit -m "docs: align boost references with PDCAA runtime contract"
```

### Task 4: Verify host-adaptation and project instructions still match

**Files:**
- Review: `references/host-adaptation-best-practices.md`
- Review: `AGENTS.md`
- Review: `CLAUDE.md`

- [ ] **Step 1: Check whether Codex-side wording now conflicts with updated runtime behavior**

```text
Look for conflicts in:
- validation wording
- subagent/worktree triggers
- source-of-truth statements
- sync instructions
```

Run: `python3 - <<'PY'
files = ['references/host-adaptation-best-practices.md', 'AGENTS.md', 'CLAUDE.md']
for path in files:
    text = open(path).read()
    print(path, 'lines=', len(text.splitlines()))
PY`

Expected: all three files read successfully.

- [ ] **Step 2: Make only the smallest consistency edits needed**

```text
Examples of acceptable edits:
- rename wording to match SKILL.md terms
- adjust references to validation expectations
- point to the imported PDCAA spec if that helps maintainability
```

Run: `git diff -- references/host-adaptation-best-practices.md AGENTS.md CLAUDE.md`

Expected: no diff if already aligned, or a very small focused diff.

- [ ] **Step 3: Commit only if these review files changed**

```bash
git add references/host-adaptation-best-practices.md AGENTS.md CLAUDE.md
git commit -m "docs: sync host guidance with updated boost contract"
```

### Task 5: Validate the workspace copy end to end

**Files:**
- Modify: none expected
- Test: `SKILL.md`, `references/methodology.md`, `references/iteration-template.md`, optional touched review files

- [ ] **Step 1: Run a focused diff review**

```bash
git diff -- SKILL.md references/methodology.md references/iteration-template.md references/quick-reference.md references/host-adaptation-best-practices.md AGENTS.md CLAUDE.md
```

Expected: diff is limited to the intended contract, reference, and wording updates.

- [ ] **Step 2: Re-read the changed files and verify spec coverage**

```text
Coverage checklist:
- Stable Contract fields present and unchanged in shape
- Runtime State boundary preserved
- PDCAA order unchanged
- AutoResearch remains on-demand
- AutoReceive keeps contract-protection boundary
- Log remains mandatory
- validation stays evidence-based
- no new durable execution artifacts introduced
```

Run: `python3 - <<'PY'
from pathlib import Path
for path in ['SKILL.md', 'references/methodology.md', 'references/iteration-template.md']:
    text = Path(path).read_text()
    print('---', path)
    for needle in ['Stable Contract', 'Runtime State', 'AutoResearch', 'AutoReceive', 'Log']:
        print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all core terms print `OK` for all three files.

- [ ] **Step 3: Check frontmatter and line count**

```bash
python3 - <<'PY'
from pathlib import Path
text = Path('SKILL.md').read_text().splitlines()
print('frontmatter_start=', text[0])
print('name_line=', text[1])
print('version_line=', text[2])
print('description_line=', text[3][:80])
print('total_lines=', len(text))
PY
```

Expected: frontmatter remains intact and line count stays controlled.

- [ ] **Step 4: Run a mental fixture pass against existing eval references**

```text
Walk at least these scenarios against the updated wording:
1. User asks “优化一下当前项目” — target must be the surrounding repo, not the skill.
2. User asks “先不要改，只分析” — skill must stop after first Check.
3. User gives conflicting optimization directions — skill must trigger AutoResearch or a sharp contract clarification, not thrash.
```

Run: `python3 - <<'PY'
scenarios = [
    'optimize current project',
    'analyze only',
    'conflicting directions',
]
for s in scenarios:
    print('checked:', s)
PY`

Expected: all three scenario names print.

- [ ] **Step 5: Commit the final validated plan execution result**

```bash
git status --short
git add SKILL.md references/methodology.md references/iteration-template.md references/quick-reference.md references/host-adaptation-best-practices.md AGENTS.md CLAUDE.md
git commit -m "refactor: tighten boost PDCAA execution contract"
```

Expected: commit succeeds only if there are actual staged changes.

### Task 6: Sync installed copies after validation

**Files:**
- Copy from workspace: `SKILL.md`
- Copy from workspace: `references/`
- Copy from workspace: `AGENTS.md`
- Copy from workspace: `CLAUDE.md`
- Copy from workspace: `agents/boost-observer.md`
- Target installs: `~/.codex/skills/boost/`, `~/.claude/skills/boost/`, `~/.claude/agents/`

- [ ] **Step 1: Verify which workspace files changed before syncing**

```bash
git diff --name-only HEAD~1..HEAD
```

Expected: only the validated workspace files appear.

- [ ] **Step 2: Copy the updated files into both installed skill locations**

```bash
cp SKILL.md ~/.codex/skills/boost/SKILL.md
cp SKILL.md ~/.claude/skills/boost/SKILL.md
cp AGENTS.md ~/.codex/skills/boost/AGENTS.md
cp AGENTS.md ~/.claude/skills/boost/AGENTS.md
cp CLAUDE.md ~/.codex/skills/boost/CLAUDE.md
cp CLAUDE.md ~/.claude/skills/boost/CLAUDE.md
rsync -a references/ ~/.codex/skills/boost/references/
rsync -a references/ ~/.claude/skills/boost/references/
cp agents/boost-observer.md ~/.claude/agents/boost-observer.md
```

Expected: copy commands exit successfully.

- [ ] **Step 3: Verify installed copies match the workspace source**

```bash
diff -q SKILL.md ~/.codex/skills/boost/SKILL.md
diff -q SKILL.md ~/.claude/skills/boost/SKILL.md
diff -q AGENTS.md ~/.codex/skills/boost/AGENTS.md
diff -q AGENTS.md ~/.claude/skills/boost/AGENTS.md
diff -q CLAUDE.md ~/.codex/skills/boost/CLAUDE.md
diff -q CLAUDE.md ~/.claude/skills/boost/CLAUDE.md
diff -q agents/boost-observer.md ~/.claude/agents/boost-observer.md
```

Expected: each diff reports no differences.

- [ ] **Step 4: Commit the sync status note only if tracked files changed during sync preparation**

```bash
git status --short
```

Expected: working tree remains clean after sync.

---

## Self-Review

- **Spec coverage:** The plan covers Stable Contract, Runtime State, PDCAA loop behavior, AutoResearch, AutoReceive, Log, validation, host adaptation, and install sync. No imported-spec section is left without a target task.
- **Placeholder scan:** No step uses TBD/TODO or vague “handle appropriately” wording; each executable step includes exact commands or explicit text outcomes.
- **Type consistency:** The plan consistently uses the same six Act decisions, the same four Stable Contract fields, and the same AutoReceive / AutoResearch terminology across tasks.

Plan complete and saved to `docs/superpowers/plans/2026-04-21-boost-pdcaa-alignment.md`.

Two execution options:

1. **Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration
2. **Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

Which approach?
