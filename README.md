# boost

`boost` is an agent skill for iterative optimization using a strict PDCAA loop (Plan-Do-Check-Align-Act) anchored by a Stable Contract, instead of one-shot advice.

## Description

A cross-host agent skill that drives optimization through a Stable Contract (Goal/Constraints/Done/Checks) and disciplined PDCAA cycles with mandatory log output, drift detection, and evidence-based decisions.

It is designed for tasks like:

- improving prompts, workflows, agents, services, or repositories
- diagnosing why a system is underperforming
- choosing one concrete next intervention instead of listing vague options
- iterating with validation and visible state checkpoints

The current version is written to work across both Claude Code and Codex, with extra attention to Claude Code execution discipline and context recovery.

## Core Idea

The skill treats the thing being improved as a target object and drives it through a strict loop:

1. Establish a **Stable Contract** (Goal, Constraints, Done, Checks) before any iteration.
2. **Plan** — define the current minimum action.
3. **Do** — execute exactly what was planned.
4. **Check** — verify with evidence against 5 mandatory questions.
5. **Align** — detect drift across 5 dimensions; correct if found.
6. **Act** — decide: continue / adjust / rollback / research / complete.
7. Emit a visible iteration checkpoint and loop back to Plan, or stop.

Built-in enhancements: **AutoResearch** (triggered on path uncertainty), **AutoReceive** (handles incoming feedback), and **Log** (11-field mandatory output every cycle).

## Repository Layout

- `SKILL.md`
  The main cross-host skill definition. Keep this lean and operational.

- `CLAUDE.md`
  Claude Code-specific project adapter and execution guidance.

- `AGENTS.md`
  Codex-specific project adapter and repository working rules.

- `agents/openai.yaml`
  Codex metadata used for discovery and UI rendering.

- `references/claude-clarification-prompts.md`
  Claude-friendly confirmation wording, tagged state packets, and few-shot examples.

- `references/iteration-template.md`
  Full working template for a single optimization pass.

- `references/iteration-state-snapshot.md`
  Compact reloadable state for long or noisy threads.

- `references/eval-fixtures.md`
  Regression fixtures for trigger behavior, clarification, iteration, and host-specific behavior.

- `references/methodology.md`
  Full methodology: research frames, topology, autonomous iteration, validation protocol, ratchet/rollback patterns.

- `references/research-loop-notes.md`
  Karpathy's autoresearch: three-file architecture, ratchet, fixed budget, simplicity criterion.

- `references/feedback-loop-notes.md`
  Cybernetic feedback theory: Wiener, Ashby, von Foerster applied to AutoReceive.

- `references/skill-best-practices.md`
  Skill design, host adaptation, heuristics, and failure modes.

- `references/evolution-metrics.md`
  Metric layers, evaluation design, and evidence hierarchy.

- `references/host-adaptation-best-practices.md`
  Claude Code and Codex host adaptation guidance.

- `references/quick-reference.md`
  Minimal system prompt and quick start checklist.

- `agents/boost-observer.md`
  Read-only baseline observation subagent for Plan phase evidence gathering.

## Install

### Claude Code

Clone or copy this directory to:

```bash
~/.claude/skills/boost/
```

Claude Code reads `SKILL.md` as the main skill definition and uses `CLAUDE.md` as project guidance when you work on this repository.

### Codex

Clone or copy this directory to:

```bash
~/.codex/skills/boost/
```

Codex uses both `SKILL.md` and `agents/openai.yaml`.

## Usage

Example requests:

- `Use boost to improve this prompt chain without increasing token cost.`
- `用 boost 优化这个 workflow，先确认目标和目标，再做一轮可执行优化。`
- `Treat the boost skill itself as the target object and improve its Claude Code behavior.`

Typical output structure per cycle:

```text
> **Stable Contract**
> - Goal: ...
> - Constraints: ...
> - Done: ...
> - Checks: ...

> **Plan:** ...
> **Do:** ...
> **Check:** Goal advancement / Constraint violation / New problems / Verifiable / Future checkability
> **Align:** drift detected: none / type + correction
> **Act:** continue / adjust / rollback / research / complete — reason

round_id / goal / current_action / action_reason / execution_result / check_result / align_result / feedback / decision / next_step / log_summary

> **[boost · Iter N]** Target: ... | Goal: ... | 上轮: ... → 本轮: ...
```

Boost stays local by default, but upgrades topology when it earns its cost:

- use a focused subagent when evidence gathering would otherwise flood the main thread context
- use an isolated worktree when the experiment is broad, rollback-sensitive, or likely to destabilize the workspace
- keep Stable Contract, final Check, Act decision, and state snapshot in the main thread

## Development Notes

- The workspace copy is the source of truth.
- Installed copies under `~/.claude/skills/` and `~/.codex/skills/` should be updated by copying from the workspace.
- Backup files such as `*.bak` and `*.bak.*` are intentionally ignored by git.
- Keep `SKILL.md` short. Push extra rationale, examples, and host-specific tactics into `references/`, `CLAUDE.md`, or `AGENTS.md`.

## Testing

This repository is maintained by behavior-driven iteration rather than unit tests alone.

Use these checks when changing the skill:

- verify Stable Contract is established before any PDCAA cycle
- verify target is resolved correctly (project vs. skill)
- verify all 5 PDCAA phases execute in order with visible output
- verify 11-field Log is emitted every cycle
- verify Align phase detects drift when scope expands
- verify the skill does not accidentally self-modify unless explicitly asked
- verify rollback path is available and exercised when evidence is negative

The main regression reference is:

- `references/eval-fixtures.md`

## Git Hygiene

This repository ignores backup files and Claude Code working directories through `.gitignore`:

```text
*.bak
*.bak.*
.claude/
.playwright/
```

Version is tracked in `SKILL.md` frontmatter (`version` field). Git tags mark significant releases.
