# Research Loop Notes

Use this reference when the compact AutoResearch section in `SKILL.md` is not enough and the agent needs the fuller rationale.

## Source: Karpathy's autoresearch

Released March 2026. An autonomous ML research loop: an AI agent modifies a training script, runs a 5-minute experiment, evaluates results against a fixed metric, keeps improvements or reverts failures, and loops indefinitely. 21,000+ GitHub stars. Reports of 19% performance gains from 37 overnight experiments (Shopify CEO), 11% speedup from 20 tweaks.

The key contribution is not the ML results but the **loop architecture** — which generalizes to any optimization target with an editable artifact and a measurable signal.

## The Three-File Architecture

The core design decision: separate concerns into three files with strict boundaries.

| File | Role | Mutable? | Our equivalent |
|------|------|----------|---------------|
| `program.md` | Charter — research priorities, constraints, domain knowledge | Human-edited only | Stable Contract (Goal/Constraints/Done/Checks) |
| `train.py` | Mutable surface — the one file the agent edits | Agent-edited | The files, prompts, configs, or steps allowed to change this round |
| `prepare.py` | Locked evaluator — data pipeline, scoring function, constants | Never modified | The benchmark, test suite, fixture set, or acceptance criteria that stays fixed |

**Why this matters:** When the same loop can freely change both the artifact AND the thing that judges it, all comparisons become meaningless. The evaluator lock is what makes experiments comparable. In autoresearch, `evaluate_bpb` lives in read-only `prepare.py`, preventing the agent from gaming the metric by changing the tokenizer or validation split.

**For boost:** Before every research probe, explicitly name the mutable surface and the locked evaluator. If you cannot name the evaluator, you cannot tell whether the experiment worked.

## The Ratchet Mechanism

Autoresearch uses git as the ratchet:
- Improvement (lower val_bpb) → commit the change, continue from new state
- No improvement or regression → `git reset` to last good commit
- Crash or error → log it, revert, try next hypothesis

This creates **single-lineage hill-climbing**: each kept experiment becomes the parent of the next. No population diversity, no branching — deterministic parent-child progression. The working state is always the current best.

**For boost:** The PDCAA `continue`/`rollback` decisions in Act phase implement this ratchet. When git is available, commit before experimenting and revert on failure. The workspace should never represent a speculative state.

## Fixed Comparison Budget

Every autoresearch experiment gets exactly 5 minutes of wall-clock training time. This constraint:
- Makes all experiments comparable on the same platform
- Forces optimization of "throughput × learning effectiveness" not just raw quality
- Enables ~12 experiments/hour, ~100 overnight
- Prevents bias from unequal effort allocation

**For boost:** When comparing approaches, give each a bounded probe effort. Not "try A thoroughly then try B quickly" — each candidate gets the same scope of investigation. The bound doesn't need to be time-based; it can be scope-based (one file read, one test run, one diff).

## The Simplicity Criterion

From autoresearch practice: "Tiny improvements adding 50+ lines of tangled code are rejected." Complexity degrades across sessions. Each experiment must justify its complexity cost, not just its outcome gain.

**For boost:** When Act decides between `continue` and `rollback`, factor in complexity. A marginal improvement that adds ugly complexity is not worth keeping. Simpler solutions that score equally are preferred.

## Crash Resilience

Experiments fail in ways beyond "didn't improve" — they crash, error, or produce garbage. Autoresearch handles this:
1. Read the last 50 lines of error output
2. Attempt a fix, re-run up to a few times
3. If still broken, log the failure, revert, skip to next hypothesis
4. Do not debug endlessly — the loop's strength is throughput, not depth on any one path

**For boost:** When a Do action fails in an unexpected way, the correct response is usually: log what happened, revert, and try a different approach. Reserve deep debugging for when the failure reveals something about the system itself.

## Autonomous Continuity ("NEVER STOP")

Autoresearch instructs the agent: once the charter is set, run the loop without seeking permission. The human's role shifts from operator to research advisor — they set direction through `program.md` and review accumulated results.

**For boost:** Once the Stable Contract is established, own the method. Do not ask the user what to try, how to validate, or when to stop. Drive Plan → Do → Check → Align → Act autonomously. The exception is: stop if the user explicitly asked for analysis only.

## The Experiment Ledger

Autoresearch tracks every attempt in `results.tsv`: experiment number, val_bpb score, peak VRAM, keep/discard decision, timestamp. Git commit history shows the sequence of kept improvements.

**For boost:** The mandatory 11-field Log output IS the experiment ledger. Each cycle records what was tried, why, what happened, and what was decided. Additionally, rejected approaches should be noted so they aren't retried without new evidence.

## How Boost Generalizes Autoresearch

| Autoresearch (ML-specific) | Boost (general) |
|---------------------------|------------------------|
| Fixed target: `train.py` | Arbitrary target with resolution protocol |
| Fixed metric: `val_bpb` | Agent-inferred metrics from target's observability |
| Fixed time budget: 5 min | Scope-appropriate bounded probes |
| Human writes `program.md` | Agent builds charter from user's goal via Socratic discovery |
| Single-file mutable surface | Multi-file surface with explicit boundaries |
| Local execution only | Local / delegated / isolated topology |
| Stateless per iteration | Context drift guard, state snapshots, session recovery |
| No drift detection | Align phase catches goal/constraint/check drift every cycle |

The core loop is identical: charter → hypothesize → change → measure → keep or revert → repeat. Boost adds target resolution, drift detection, multi-surface management, and topology choice.

## Sources

- [GitHub: karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- [DataCamp: Guide to autoresearch](https://www.datacamp.com/tutorial/guide-to-autoresearch)
- [Kingy AI: Autoresearch minimal agent loop](https://kingy.ai/ai/autoresearch-karpathys-minimal-agent-loop-for-autonomous-llm-experimentation/)
- [Data Science Dojo: Karpathy autoresearch explained](https://datasciencedojo.com/blog/karpathy-autoresearch-explained/)
- [Global Advisors: Karpathy's Loop](https://globaladvisors.biz/2026/04/20/term-karpathys-loop-often-referred-to-as-autoresearch-auto-loop-or-auto-optimization/)
