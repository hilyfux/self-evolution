# Iteration State Snapshot

Use this reference when the conversation has become long, noisy, or branched and the agent needs to recover the active optimization state without rereading the entire transcript.

## Purpose

The snapshot is not a summary of the whole conversation. It is a minimal reloadable state for continuing the current PDCAA cycle safely.

Use it to preserve:

- what is being optimized (target)
- the Stable Contract (goal, constraints, done, checks)
- what the current PDCAA phase is
- what happens next
- what is still unresolved

## Snapshot Format

```text
Target:
Goal:
Constraints:
Done:
Checks:
Current Phase: Plan / Do / Check / Align / Act
Baseline:
Last Decision:
Next Action:
Open Questions:
```

## Writing Rules

- Keep each line short and operational.
- Prefer explicit nouns over pronouns.
- Prefer the current confirmed state over historical detail.
- If something is unknown, write `Unknown` rather than guessing.
- Replace stale state instead of appending to it forever.

## Mandatory Reload Checkpoints

Do not wait for drift to happen. Proactively reload at these moments:

| Checkpoint | Action |
|-----------|--------|
| Before each new PDCAA cycle | Re-read snapshot. If any field is unclear, re-open `SKILL.md`. |
| After large tool output | Write a one-line state anchor before proceeding. |
| After subagent returns | Restate the Stable Contract before integrating results. |
| After context compaction | Full reload: re-open `SKILL.md`, re-read snapshot, rebuild state. |

## When to Refresh the Snapshot

Refresh when:

- any Stable Contract field changes
- a PDCAA cycle completes (Act decision made)
- the user corrects a misunderstanding
- the agent returns from a subagent or worktree
- AutoReceive proposes a Contract Candidate that is accepted

## Recovery Protocol

When drift is detected (editing without knowing which experiment, target fuzzing, skipping Check phase):

1. STOP the current action.
2. Re-open `SKILL.md`.
3. Re-open this reference.
4. Restate the latest valid snapshot.
5. Run Align phase explicitly.
6. Continue only after the frame is coherent again.

## Example

```text
Target: Agent workflow prompt chain
Goal: Raise answer quality without increasing token cost
Constraints: Do not modify evaluation scripts; token budget <= current baseline
Done: Quality score improves by >=10% on fixture set with no cost increase
Checks: Run fixture set, compare scores, verify token count
Current Phase: Do
Baseline: Quality score 72/100, avg 1.2k tokens per response
Last Decision: continue — first probe showed 5% improvement
Next Action: Apply refined system prompt variant B and re-run fixtures
Open Questions: Whether context window usage affects quality independently
```
