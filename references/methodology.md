# Methodology Reference

This is the detailed methodology behind the PDCAA skeleton in `SKILL.md`. Consult when a phase needs deeper context — Stable Contract design, AutoResearch triggers, AutoReceive protocol, Log structure, topology choices, and rollback heuristics.

The skeleton (Stable Contract + PDCAA loop in `SKILL.md`) is the mandatory constraint. This document provides supporting patterns.

## Stable Contract Design

The Stable Contract is the anchor that prevents drift. It must be established before any iteration begins.

### Field Guidelines

- **Goal**: One sentence. If it takes a paragraph, it's too broad — split or narrow.
- **Constraints**: Hard boundaries. Things that will not be violated even if they slow progress. Examples: "do not modify the test suite", "token budget <= X", "no breaking changes to public API".
- **Done**: Observable end state. Must be verifiable without subjective judgment. Bad: "code is cleaner". Good: "all tests pass and cyclomatic complexity <= 10".
- **Checks**: Concrete verification actions. Not "make sure it's good" but "run `npm test`, verify no regressions in fixture output, read changed file and confirm edit landed".

### Contract Modification Protocol

The Stable Contract is intentionally hard to modify. Changes require:

1. Explicit marking as `Contract Candidate`
2. Evidence that the current contract is blocking progress or incorrect
3. User confirmation OR overwhelming evidence from Check phase

Do not silently relax constraints. Do not broaden Goal without acknowledgment.

## Research System Frame

Before optimization, map the target into these roles:

- `Stable Contract` — the objective, guardrails, and stopping rule (replaces "research charter")
- `Mutable surface` — the files, prompts, workflow steps, or parameters allowed to change this round
- `Locked evaluator` — the benchmark, review basis, fixture set, or acceptance test that should not drift during the experiment
- `Experiment ledger` — the in-context record of what was tried, what happened, and what was decided (lives in Log output, not in files)
- `Topology` — whether the research runs locally, through delegated sidecars, or in isolated workspaces

If any of these are missing and the task is otherwise opaque, instrument them before making broad changes.

## Clarification Boundaries

Once the Stable Contract is established, everything operational is yours to decide. Do not ask the user about execution details.

**Ask the user (contract layer only):**

- Which object to optimize, when multiple plausible targets exist
- What outcome matters, when the user's intent is genuinely unclear
- What the real pain point is, when "improve" is too vague to act on

**Decide autonomously (never ask):**

- What metrics to use — infer from the goal and the target's observability
- What experiments to try — pick the highest-leverage candidate
- How to validate — design the check from the target's structure
- What surface to change — derive from the goal and constraints
- When to rollback vs iterate — decide from evidence

Rules:

- Never ask the user how to validate. Figure it out.
- Never ask the user what metrics to track. Infer them.
- Never ask the user what to try next. Pick the best option and execute.
- Do not ask questions whose answers are recoverable from local context.
- Do not manufacture questions if the user already supplied enough information.

## PDCAA Deep Patterns

### Plan Phase

The minimum action principle: each Plan must define the smallest action that:
- Directly serves the Goal
- Is low-risk (reversible or bounded in blast radius)
- Is verifiable (you can tell if it worked)

If you cannot define a verifiable minimum action, you lack information → trigger AutoResearch.

### Do Phase

Execute exactly what was planned. If you discover mid-execution that the plan needs modification:
- Complete the current action if it's safe to finish
- Or abort cleanly if continuing would violate constraints
- Record the deviation in the Log

### Check Phase

Five mandatory questions, every cycle:

1. **Goal advancement** — Did this action move toward Done?
2. **Constraint violation** — Did anything cross a hard boundary?
3. **New problems** — Did this introduce issues that weren't there before?
4. **Verifiable** — Can you show evidence, not just assert improvement?
5. **Future checkability** — Does the result preserve the ability to check in subsequent cycles?

If you cannot answer question 4, the action was under-instrumented. Note this and plan instrumentation next.

### Align Phase

Five drift types to detect:

| Type | Signal | Recovery |
|------|--------|----------|
| Goal drift | Target fuzzes/broadens | Re-read Goal from Contract |
| Constraint drift | Constraints silently relaxed | Re-read Constraints, undo violation |
| Check drift | Verification items skipped | Restore full Checks list |
| Done-criteria drift | Completion bar moves | Re-read Done definition |
| Local-problem hijack | Side issue displaces main objective | Clear local state, return to Goal |

Align is mandatory every cycle. "No drift detected" is a valid output but must be stated explicitly.
If drift is detected, perform Re-anchor before the next Plan: re-read the Stable Contract, clear invalid local state, restore checks, and regenerate the minimum action.

### Act Phase

Six possible decisions:

- `continue` — next step is obvious, proceed
- `adjust` — right direction, refine approach
- `rollback` — undo the change, it didn't work
- `research` — path unclear, activate AutoResearch
- `receive_update` — integrate new external input via AutoReceive
- `complete` — Done criteria met with evidence

## AutoResearch Protocol

AutoResearch is a slow path derived from Karpathy's autoresearch pattern. It activates only when the fast path (Plan → immediate action) is blocked. See `references/research-loop-notes.md` for the full theoretical grounding.

### Trigger conditions

1. **Path unclear** — cannot define a verifiable minimum action
2. **Conflicting approaches** — 2+ viable paths with unclear tradeoffs
3. **Consecutive low-gain** — 2+ cycles with minimal progress
4. **Contradictory results** — evidence conflicts with expectations
5. **Repeated rejection** — actions denied or rolled back consecutively

### Process

1. Name the mutable surface and the locked evaluator for this probe
2. List candidate approaches (max 3)
3. Compare each against Stable Contract (which best serves Goal within Constraints?)
4. Propose a minimum probe — the smallest experiment that distinguishes candidates, with a bounded effort comparable across candidates
5. Return to Plan with the probe action

### Suggested schema

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

### Constraints

AutoResearch does NOT:
- Replace the PDCAA loop
- Make keep/rollback decisions
- Modify the Stable Contract
- Execute changes directly

After each probe, the result goes through the normal Check → Align → Act path. The ratchet property applies: keep improvements, revert failures, never accumulate half-tested changes.

## AutoReceive Protocol

AutoReceive is the always-on listener, grounded in cybernetic feedback theory. It classifies, routes, and integrates incoming signals while protecting the Stable Contract boundary. See `references/feedback-loop-notes.md` for the full theoretical grounding.

### Routing by feedback type

| Feedback type | Default response | Integration timing |
|--------------|-----------------|-------------------|
| Errors, constraint violations | Negative feedback — correct toward Stable Contract | Immediate (this cycle) |
| User corrections ("no, I meant...") | Negative feedback — correct toward user intent | Immediate (this cycle) |
| Validation successes, confirmations | Positive feedback — reinforce current direction | This cycle's Act decision |
| New observations from tools | Record in Runtime State | Deferred (next Plan) |
| New requirements or constraints | Record in Runtime State first, then propose as Contract Candidate if needed | Escalated (after explicit marking) |
| Evidence that Done criteria are wrong | Propose as Contract Candidate | Escalated (after explicit marking) |

### Boundary protection

Default: update Runtime State only. Crossing into Stable Contract requires explicit escalation via Contract Candidate.
New requirements and new constraints should enter as feedback first; they only cross the boundary after explicit escalation.

Format:
```
> **Contract Candidate:** <field>: <proposed new value> — <evidence/reason>
```

Apply only when:
- User explicitly confirms, OR
- Evidence is overwhelming AND the current contract is demonstrably blocking correct behavior

### Signal vs. noise

Not every signal warrants action. Before routing as Immediate, ask: "If I act on this and it turns out to be noise, what's the cost?" High-cost-if-wrong signals should be classified as Deferred until confirmed by a second observation.

## Slow-Path Gate Protocol

The gates below are slow-path upgrades around PDCAA, not a replacement for PDCAA. They are optional and evidence-triggered.

### Design Gate

Use this gate when the task is too ambiguous for a trustworthy first Plan.

Typical triggers:
- ambiguous scope
- multiple subsystems or moving parts hidden inside one request
- broad multi-file change where boundaries are still unclear

What it changes:
- escalate into spec/plan-oriented clarification before broad execution
- tighten scope, file surface, and validation basis before returning to PDCAA

What it does not change:
- does not replace the Stable Contract
- does not bypass Check, Align, or Log
- does not make design docs mandatory for every run

### Execution Gate

Use this gate when the work is clear enough to execute, but the default local path is no longer the safest or cheapest.

Typical triggers:
- rollback-sensitive or destabilizing changes
- competing approaches worth isolating
- context-heavy sidecar work better handled with worktree/subagent/topology escalation

What it changes:
- upgrades execution topology: worktree, delegated exploration, or bounded sidecar execution
- keeps the main thread responsible for contract ownership, final Check, and Act decisions

What it does not change:
- does not make worktrees or subagents mandatory by default
- does not allow delegated agents to own keep/rollback decisions
- does not replace AutoResearch when the path itself is unclear

### Quality Gate

Use this gate when the work is nearly done, but the cost of a weak finish is too high.

Typical triggers:
- before declaring complete on broad changes
- risky or user-visible changes
- cases where stronger verification/review is justified before completion

What it changes:
- escalates verification/review before the final `complete` decision
- may require stronger before/after evidence, focused review, or completion checks

What it does not change:
- does not add mandatory ceremony to every cycle
- does not lower the requirement for evidence in the normal Check phase
- does not replace the final Act decision inside PDCAA

## Validation Protocol

After every Do phase, before declaring any decision in Act:

1. **State what you will check** — name the specific file, output, behavior, metric, or test
2. **Run the check** — execute a concrete verification action
3. **Compare against baseline** — state before/after side by side
4. **Check guardrails** — verify nothing protected got worse
5. **Declare with evidence** — the Act decision must reference specific evidence

Never skip validation because the change "looks right." Never batch multiple unvalidated changes.

## Execution Topology

Use the lightest topology that fits. Trigger heavier ones by condition, not by habit.

| Condition | Topology | Why |
|-----------|----------|-----|
| Single-scope edit, next step obvious | `local` | No overhead |
| 2+ independent exploration directions | `delegated` (parallel subagents) | Explore in parallel, converge in main thread |
| Read-heavy evidence gathering that would flood context | `delegated` (Explore subagent) | Preserve main-thread context quality |
| Risky/broad change, needs cheap rollback | `isolated-worktree` | Main workspace stays clean |
| Trying two competing approaches side by side | `delegated` + `isolated-worktree` | Compare without pollution |

Rules:
- Main thread always owns: Stable Contract, final Check, Act decision, state snapshot.
- Subagents get: narrow scope, clear deliverable, bounded tools. Not "help me optimize."
- Worktrees get: one experiment path. Merge back only after Check passes.

## Ratchet and Rollback

Adopt the ratchet property: the working state always represents the current best. Failed experiments do not accumulate.

When git is available, use it naturally: commit before experimenting, revert on failure.

After each Do + Check, declare in Act:
- `continue` — improved, commit stays
- `adjust` — right direction, keep and refine
- `rollback` — hurt the target or no gain, revert
- `research` — inconclusive, revert and probe differently

Simplicity criterion: all else being equal, simpler is better. A marginal improvement that adds ugly complexity is not worth keeping.

## Autonomous Iteration Contract

Once the Stable Contract is established, drive the full loop autonomously: Plan, Do, Check, Align, Act, iterate. Do not stop to ask the user what to try, how to measure, or how to validate. The user gave you the Goal; you own the method.

Exception: stop if the user explicitly asked for analysis only.

## Lightweight Execution

This is a thinking methodology, not infrastructure. Keep it weightless:

- Do not create tracking files, ledger files, or planning documents. Use conversation context + Log output.
- If a temporary file is unavoidable, put it in `/tmp`. Delete when done.
- Do not leave behind any artifact the user didn't ask for.
- The skill's footprint after execution should be zero, except for the changes the user wanted.

## Context Preservation

Long tasks cause context loss. The PDCAA structure with mandatory Log output is the primary defense.

**Mandatory reload checkpoints** — re-read the skill and rebuild state at these moments:

1. Before each new PDCAA cycle — emit the visible state checkpoint
2. After any large tool output that may have pushed the method out of context
3. After returning from a delegated subagent
4. When the agent notices it is acting without the Stable Contract frame

**Drift signals** — if any appear, trigger Align immediately:
- Target fuzzes or broadens without explicit re-charter
- Agent proposes changes without stating which experiment they belong to
- Multiple cycles pass without a Check phase
- User corrects what is being optimized
