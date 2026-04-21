# CLAUDE.md

This project is the source workspace for the `boost` skill.

## Project Purpose

- The workspace copy is the source of truth for this skill.
- Installed copies live under `~/.codex/skills/boost/` and `~/.claude/skills/boost/`.
- Update the workspace copy first, then sync installed copies by copying, never by moving the project directory.

## Targeting Rules

- If the user asks to optimize "当前项目", "这个项目", "this project", or the current repo, the target is this project and its files.
- Do not treat the loaded `boost` skill as the target object unless the user explicitly says to optimize the skill itself.
- Do not rewrite installed copies first. Make changes in this workspace, validate them, then copy the updated files to installed locations.

## Files That Matter

- `SKILL.md` is the main skill logic, workflow contract, and runtime hooks.
- `references/` contains reusable support material and templates.
- `agents/boost-observer.md` is the read-only observation subagent for Step 3.
- `agents/openai.yaml` is Codex-side metadata, not the method itself.
- `AGENTS.md` is the Codex-side project adapter; keep host-specific rules aligned across both adapter files.

## Claude Code Execution Patterns

This section guides Claude Code when running the boost skill on any target — not just this project.

### Validation With Tools

The validation protocol in SKILL.md requires concrete checks. In Claude Code, use these tools:

| Validation need | Tool pattern |
|----------------|-------------|
| Check file content after edit | `Read` the changed file, confirm the change landed correctly |
| Compare before/after | `Bash` with `git diff` or store the original via `Read` before editing |
| Run tests | `Bash` to execute the project's test command |
| Check build | `Bash` to run build/lint/typecheck |
| Verify structure | `Grep` or `Glob` to confirm expected patterns exist |
| Measure output quality | `Bash` to run the artifact and inspect output |
| Regression check | `Bash` to run the existing test suite or fixture set |

Always capture baseline state before executing changes. At minimum, read the files you plan to change.

### Validation Examples

**After editing a prompt file:**
1. Read the changed file to confirm the edit is correct
2. If a test harness exists, run it and compare output to baseline
3. If no harness, produce one sample output and inspect it

**After editing code:**
1. Run the project's test suite or linter
2. If tests pass, check the specific behavior that should have changed
3. If tests fail, diagnose before declaring rollback

**After editing this skill itself:**
1. Count SKILL.md lines (soft limit ~300)
2. Check that frontmatter is valid
3. Walk through 2-3 eval fixtures mentally or by re-reading them
4. Verify reference links still resolve

### When Validation Is Not Runnable

If the target has no tests, no build, and no runnable output, say so explicitly. Propose instrumentation as the first optimization rather than skipping validation silently.

### Using Host Capabilities

Use subagents, worktrees, and other host capabilities when they earn their cost — see the Execution Topology table in `references/methodology.md` for trigger conditions. Default to local execution unless one of the conditions below is clearly true.

**Superpowers-style orchestration order**

- First decide whether a `Skill` should shape the workflow before any direct implementation or broad exploration.
- Start with process skills when they match the task type: brainstorming before creative design, systematic-debugging before bug fixing, test-driven-development before feature or behavior changes, requesting-code-review / verification-before-completion near completion.
- Use implementation or domain skills only after the process layer is set, and only when they materially improve execution quality for the current task.
- Escalate to an agent or subagent only after the skill path is clear and delegation has a bounded deliverable; do not use agents as a substitute for deciding the workflow.
- Keep the main thread responsible for target resolution, Stable Contract, execution choice, validation, and final keep/rollback/next decisions.

**Agent upgrade rules**

- Use a focused subagent when Step 3 evidence gathering would take more than 3 reads/searches, when the target is read-heavy enough to flood context, or when 2+ sidecar explorations can proceed independently.
- Use the built-in `Explore` subagent for broad codebase search or log/document scanning; use narrower custom subagents only when they already match the needed deliverable.
- Delegate only bounded sidecar work with a concrete deliverable. Never delegate the keep/rollback decision or the choice of which skill governs the task.
- After any subagent returns, restate target, goal, baseline, and next action before integrating its result.

**Worktree upgrade rules**

- Use a worktree when the planned experiment is broad, rollback-sensitive, or likely to destabilize the current workspace.
- Keep the main workspace as source of truth; merge or copy back only after the isolated experiment validates.
- Do not require a worktree for small, single-scope edits where local rollback is cheap.

Key principles:

- Subagents get narrow scope and a concrete deliverable. Never delegate the keep/rollback decision.
- Worktrees are for risky experiments that could destabilize the workspace. Merge only after validation.
- After any delegation returns, restate the state snapshot before acting on results.

### Context Preservation

If you notice yourself editing without knowing which experiment the edit belongs to — STOP. Re-read `SKILL.md`, rebuild state, then continue. This is the #1 failure mode in long sessions.

## Skill Hooks

Hooks live in SKILL.md frontmatter and use `python3` instead of raw bash. This avoids `$` (bash variable syntax) in the YAML, which Codex's template engine misinterprets as JavaScript variable references ("Cannot access '$' before initialization"). Python commands are functionally identical and cross-platform safe.

## Claude Code Working Style

When editing this skill project specifically:

- Keep `SKILL.md` lean. Move Claude-specific or optional guidance to `CLAUDE.md` or `references/`.
- Use the staged confirmation flow: target, goal, surface, validation, execution.
- Add surface confirmation when scope may drift.
- Ask the smallest number of clarification questions needed to unblock progress.
- When the thread becomes long or noisy, re-open `SKILL.md` and rebuild state from `references/iteration-state-snapshot.md`.
- Prefer compact XML-like tags (`<target>`, `<goal>`, `<surface>`, `<validation>`, `<next_action>`) when context is dense.
- Prefer a few concrete examples over long abstract explanation.
- Stay local by default. Use `Explore` for read-heavy evidence gathering. Use `Plan` when sequencing is the hard part.
- Prefer focused subagents with narrow responsibilities. Trigger worktrees only for broad/risky changes.
- Keep the workspace copy as source of truth; integrate validated results back explicitly.
- Avoid over-aggressive trigger wording that causes newer Claude models to overtrigger.
- Use examples and fixtures to stabilize ambiguous behavior.

## Claude Fast Path

When invoking this skill in Claude Code, prefer this compact structure over long prose:

```text
<target>...</target>
<goal>...</goal>
<surface>...</surface>
<validation>...</validation>
<next_action>...</next_action>
```

If the request is ambiguous, ask at most the minimum next question needed to fill one of those fields.

## Versioning Rule

Version lives in `SKILL.md` frontmatter `version` field — single source of truth.

Format: `major.minor.patch`

| Level | When to bump | Examples |
|-------|-------------|----------|
| **major** | Stable Contract schema 变更、PDCAA 循环结构变更、Log 字段不兼容变更 | 删除 Align 阶段、重命名 Stable Contract 字段 |
| **minor** | 新增能力、新 reference 文件、新漂移类型、新 AutoResearch 触发条件 | 添加 Runtime State 节、添加 feedback_event schema |
| **patch** | 措辞修正、同步修复、eval fixture 补充、typo | 对齐 spec 字段名、修正输出格式 |

Rules:
- 每次编辑 SKILL.md 后必须评估是否需要 bump。
- Bump 时同步更新 frontmatter `version` 字段，不需要单独的 changelog 文件 — git log 即 changelog。
- 发布重要版本时打 git tag：`git tag v<version>`。
- 安装副本同步时必须携带最新版本号。

## Sync Rule

After editing this project:

1. Validate the workspace copy.
2. Copy updated files into `~/.codex/skills/boost/`.
3. Copy updated files into `~/.claude/skills/boost/`.
4. Copy `agents/boost-observer.md` into `~/.claude/agents/`.
5. Verify the installed copies match the workspace source.

Never move the project directory during installation or sync.
