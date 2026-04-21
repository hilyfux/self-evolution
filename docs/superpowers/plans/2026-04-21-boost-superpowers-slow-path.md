# Boost Superpowers Slow-Path Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a minimal superpowers-inspired slow-path upgrade protocol to boost so complex work can escalate into design, execution, and quality gates without making the default PDCAA loop heavy.

**Architecture:** Keep `SKILL.md` focused on the host-agnostic runtime contract: default lightweight PDCAA, clear upgrade triggers, and explicit gate-switch rules. Put the slower-path details, examples, and host-facing execution advice into focused reference files so the base skill stays compact while still supporting richer workflows when complexity or risk warrants it.

**Tech Stack:** Markdown skill files, reference docs, git diff, shell validation, workspace-to-install sync

---

## File Structure

- Modify: `SKILL.md` — add the slow-path trigger model and three upgrade gates without bloating the main loop.
- Modify: `references/methodology.md` — define the slow-path protocol in detail: when to trigger Design Gate, Execution Gate, and Quality Gate.
- Modify: `references/iteration-template.md` — keep the visible runtime template aligned if the gate-switch output becomes user-visible.
- Modify: `references/quick-reference.md` — update only if the operator-facing quick rules need one short mention of the slow path.
- Modify: `references/host-adaptation-best-practices.md` — explain how Claude Code and Codex should treat slow-path escalation as optional upgrades rather than the default.
- Review: `CLAUDE.md` — confirm Claude-side adapter language still matches the refined slow-path behavior.
- Review: `AGENTS.md` — confirm Codex-side adapter language still matches the refined slow-path behavior.
- Test: `SKILL.md`, `references/methodology.md`, `references/iteration-template.md`, `references/quick-reference.md`, `references/host-adaptation-best-practices.md`

### Task 1: Define the slow-path contract boundaries

**Files:**
- Modify: `SKILL.md`
- Modify: `references/methodology.md`

- [ ] **Step 1: Read the current runtime contract and mark where the slow path can attach**

```text
Identify exactly these attachment points:
- before first Plan when the task is still underspecified
- between Act and next Plan when repeated low-gain or conflict appears
- before final completion when stronger verification is required
```

Run: `python3 - <<'PY'
from pathlib import Path
for path in ['SKILL.md', 'references/methodology.md']:
    text = Path(path).read_text()
    print('---', path)
    for needle in ['Stable Contract', 'PDCAA Loop', 'AutoResearch', 'AutoReceive', 'Log']:
        print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all five markers print `OK` for both files.

- [ ] **Step 2: Lock the non-goals before editing**

```text
Non-goals for this change:
- do not make spec/plan mandatory for every boost run
- do not replace PDCAA with the superpowers workflow
- do not add durable ledgers or tracking files
- do not make worktrees or subagents mandatory by default
```

Run: `python3 - <<'PY'
print('Non-goals recorded: keep PDCAA default path lightweight.')
PY`

Expected: one-line confirmation prints.

- [ ] **Step 3: Define the three upgrade gates as protocol names**

```text
Use these exact gate names:
- Design Gate
- Execution Gate
- Quality Gate
```

Run: `python3 - <<'PY'
for gate in ['Design Gate', 'Execution Gate', 'Quality Gate']:
    print(gate)
PY`

Expected: the three gate names print exactly once each.

- [ ] **Step 4: Commit the boundary-definition checkpoint**

```bash
git status --short
```

Expected: only intended documentation files are pending.

### Task 2: Add minimal slow-path trigger rules to `SKILL.md`

**Files:**
- Modify: `SKILL.md:1-320`

- [ ] **Step 1: Write the failing checklist for the new `SKILL.md` behavior**

```text
The edited file must satisfy all of these:
1. Default mode remains lightweight PDCAA.
2. Slow path is optional and trigger-based.
3. There are exactly three gate names: Design Gate, Execution Gate, Quality Gate.
4. Trigger conditions are concrete and high-signal.
5. The gate protocol does not replace AutoResearch / AutoReceive / Log.
6. The file stays lean and host-agnostic.
```

Run: `python3 - <<'PY'
text = open('SKILL.md').read()
for needle in ['Design Gate', 'Execution Gate', 'Quality Gate', 'AutoResearch', 'AutoReceive', 'Log']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: gate names likely print `MISSING` before the edit.

- [ ] **Step 2: Add a short section that defines default path vs slow path**

```text
Required content:
- default: lightweight PDCAA loop
- slow path: optional upgrade protocol, not default behavior
- trigger only when complexity, risk, ambiguity, or completion pressure warrants it
```

Run: `python3 - <<'PY'
text = open('SKILL.md').read()
for needle in ['lightweight PDCAA', 'slow path', 'optional upgrade']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all three needles print `OK` after the edit.

- [ ] **Step 3: Add minimal trigger rules for the three gates**

```text
Trigger guidance to encode:
- Design Gate: ambiguous scope, multiple subsystems, or broad multi-file changes
- Execution Gate: rollback-sensitive work, competing approaches, or context-heavy sidecar work
- Quality Gate: before declaring complete when the change is broad, risky, or user-visible
```

Run: `python3 - <<'PY'
text = open('SKILL.md').read()
for needle in ['ambiguous scope', 'rollback-sensitive', 'before declaring complete']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all three needles print `OK` after the edit.

- [ ] **Step 4: Keep `SKILL.md` lean after the new section lands**

```bash
python3 - <<'PY'
from pathlib import Path
text = Path('SKILL.md').read_text()
print('lines=', len(text.splitlines()))
PY
```

Expected: line count increases only modestly; the file remains compact enough to serve as the runtime skeleton.

- [ ] **Step 5: Bump version only if the new gate protocol qualifies as a capability addition**

```yaml
version: "1.3.0"
```

Run: `python3 - <<'PY'
import re
text = open('SKILL.md').read()
m = re.search(r'^version: "([^"]+)"', text, re.M)
print('version=', m.group(1) if m else 'missing')
PY`

Expected: version prints `1.3.0` if the gate protocol is added to `SKILL.md`.

- [ ] **Step 6: Commit the `SKILL.md` slow-path update**

```bash
git add SKILL.md
git commit -m "feat: add slow-path upgrade gates to boost"
```

### Task 3: Expand methodology with gate semantics and escalation rules

**Files:**
- Modify: `references/methodology.md:1-320`

- [ ] **Step 1: Write the failing consistency checks for the gate protocol**

```text
The methodology update must explain:
- what each gate is for
- what signal triggers it
- what it changes in execution behavior
- what it does not change
```

Run: `python3 - <<'PY'
text = open('references/methodology.md').read()
for needle in ['Design Gate', 'Execution Gate', 'Quality Gate']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: gate names likely print `MISSING` before the edit.

- [ ] **Step 2: Add a focused section for the slow-path protocol**

```text
Section contents:
- Design Gate = spec/plan-oriented escalation
- Execution Gate = worktree/subagent/topology escalation
- Quality Gate = verification/review escalation before completion
- each gate is optional and evidence-triggered
```

Run: `python3 - <<'PY'
text = open('references/methodology.md').read()
for needle in ['spec/plan-oriented', 'worktree/subagent', 'verification/review']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all three needles print `OK` after the edit.

- [ ] **Step 3: Explicitly preserve PDCAA primacy in the methodology**

```text
Required statement:
- the gates are slow-path upgrades around PDCAA, not a replacement for PDCAA
```

Run: `python3 - <<'PY'
text = open('references/methodology.md').read()
for needle in ['not a replacement for PDCAA', 'optional and evidence-triggered']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: both needles print `OK` after the edit.

- [ ] **Step 4: Commit the methodology update**

```bash
git add references/methodology.md
git commit -m "docs: define boost slow-path gate protocol"
```

### Task 4: Align runtime templates and quick reference

**Files:**
- Modify: `references/iteration-template.md:1-160`
- Modify: `references/quick-reference.md`

- [ ] **Step 1: Update the iteration template only if gate switching is meant to be visible**

```text
If gate switching is user-visible, add one compact block such as:
> **Gate:** <none / Design Gate / Execution Gate / Quality Gate> — <why>

If not user-visible, leave the template untouched.
```

Run: `git diff -- references/iteration-template.md`

Expected: empty diff if template stays unchanged, or a small focused diff if the gate block is added.

- [ ] **Step 2: Update quick reference only with a one-line slow-path reminder if needed**

```text
Acceptable quick-reference addition:
- 默认走轻量 PDCAA；只有在复杂度、风险或收口压力升高时，才升级到慢路径 gate。
```

Run: `git diff -- references/quick-reference.md`

Expected: empty diff or a very small diff.

- [ ] **Step 3: Commit the template/quick-reference alignment**

```bash
git add references/iteration-template.md references/quick-reference.md
git commit -m "docs: align boost operator references with slow-path gates"
```

### Task 5: Align host adaptation guidance

**Files:**
- Modify: `references/host-adaptation-best-practices.md`
- Review: `CLAUDE.md`
- Review: `AGENTS.md`

- [ ] **Step 1: Add one host-level rule that the slow path is optional, not default**

```text
Required emphasis:
- Claude and Codex adapters should treat gates as escalation tools, not mandatory ceremony
```

Run: `python3 - <<'PY'
text = open('references/host-adaptation-best-practices.md').read()
for needle in ['optional', 'escalation', 'not default']:
    print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: at least one of these may be missing before the edit.

- [ ] **Step 2: Review `CLAUDE.md` and `AGENTS.md` for contradictions, edit only if needed**

```text
Look for contradictions in:
- when to stay local
- when to use worktrees/subagents
- whether slow-path upgrades sound mandatory
```

Run: `git diff -- references/host-adaptation-best-practices.md CLAUDE.md AGENTS.md`

Expected: only focused host-guidance changes appear.

- [ ] **Step 3: Commit the host-guidance refinement**

```bash
git add references/host-adaptation-best-practices.md CLAUDE.md AGENTS.md
git commit -m "docs: clarify host handling for boost slow-path gates"
```

### Task 6: Validate the workspace copy and sync installed copies

**Files:**
- Test: `SKILL.md`, `references/methodology.md`, `references/iteration-template.md`, `references/quick-reference.md`, `references/host-adaptation-best-practices.md`
- Sync: `~/.codex/skills/boost/`, `~/.claude/skills/boost/`, `~/.claude/agents/`

- [ ] **Step 1: Run final diff review for the intended files**

```bash
git diff -- SKILL.md references/methodology.md references/iteration-template.md references/quick-reference.md references/host-adaptation-best-practices.md CLAUDE.md AGENTS.md
```

Expected: diff is limited to the slow-path protocol and host-alignment wording.

- [ ] **Step 2: Re-read the changed files and verify gate coverage**

```text
Coverage checklist:
- default path still lightweight PDCAA
- three gates exist and are named consistently
- gate triggers are concrete
- gates are optional upgrades, not replacement workflow
- completion quality gate is explicit
```

Run: `python3 - <<'PY'
from pathlib import Path
for path in ['SKILL.md', 'references/methodology.md', 'references/host-adaptation-best-practices.md']:
    text = Path(path).read_text()
    print('---', path)
    for needle in ['Design Gate', 'Execution Gate', 'Quality Gate']:
        print(needle, 'OK' if needle in text else 'MISSING')
PY`

Expected: all three gate names print `OK` where implemented intentionally.

- [ ] **Step 3: Sync installed copies after validation**

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

- [ ] **Step 4: Verify installed copies match the workspace source**

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

---

## Self-Review

- **Spec coverage:** The plan covers the slow-path boundary, `SKILL.md` trigger rules, methodology semantics, template alignment, host adaptation, validation, and sync.
- **Placeholder scan:** Each task contains exact file paths, commands, and concrete content targets. No TODO/TBD language remains.
- **Type consistency:** The plan consistently uses the same three gate names — `Design Gate`, `Execution Gate`, and `Quality Gate` — and consistently describes them as optional upgrades around PDCAA rather than replacements.

Plan complete and saved to `docs/superpowers/plans/2026-04-21-boost-superpowers-slow-path.md`.

Two execution options:

1. **Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration
2. **Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

Which approach?
