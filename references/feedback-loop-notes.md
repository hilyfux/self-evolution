# Feedback Loop Notes

Use this reference when the compact AutoReceive section in `SKILL.md` is not enough and the agent needs the fuller cybernetic rationale.

## Source: Cybernetic Feedback Theory

AutoReceive is grounded in three layers of feedback theory:
1. **First-order cybernetics** (Wiener, 1948) — negative/positive feedback, homeostasis, signal filtering
2. **Ashby's Law of Requisite Variety** (1956) — the controller must match the complexity of its disturbances
3. **Second-order cybernetics** (von Foerster, 1974) — the observer as part of the system, reflexivity

These are not metaphors. They are the operational principles that determine whether AutoReceive works or fails.

## Negative Feedback (Stabilizing)

Wiener's core insight: intelligent behavior emerges from continuous comparison between desired state and observed state, with corrective action when they diverge. The anti-aircraft predictor didn't need to predict exactly — it corrected continuously based on deviation.

**For AutoReceive:** The Stable Contract (Goal/Constraints/Done/Checks) is the desired state. When Check reveals deviation — constraint violated, goal not advancing, new problem introduced — negative feedback kicks in. The corrective path flows through Align (detect drift) → Act (rollback or adjust) → Plan (new minimum action).

**Key rule:** Negative feedback is the DEFAULT. Most feedback is stabilizing — it pulls the system back toward the Stable Contract. Errors, violations, user corrections, and regressions all trigger correction.

## Positive Feedback (Amplifying)

Not all feedback is corrective. When Check confirms progress — goal advancing, no violations, result verifiable — that's a positive signal. It reinforces the current direction.

**For AutoReceive:** Validation successes, user confirmations ("yes that's right"), and improving metrics are positive feedback. They mean: continue this direction, increase confidence in the current approach.

**Danger:** Uncontrolled positive feedback creates runaway behavior. In optimization, this manifests as over-committing to a local optimum. The Align phase exists specifically to catch this — "are we drifting because we're reinforcing a direction without questioning it?"

## Homeostasis and the Stable Contract

A homeostatic system maintains its setpoint through dynamic feedback. A thermostat keeps temperature stable not by being rigid but by constantly sensing and adjusting.

**For AutoReceive:** The Stable Contract IS the homeostatic setpoint. AutoReceive's primary job is protecting this setpoint. Feedback flows into Runtime State by default; crossing into Stable Contract territory requires explicit escalation (Contract Candidate). This asymmetry is intentional: the setpoint should be hard to move, because moving it invalidates all prior comparisons.

**When the setpoint should move:** When evidence is overwhelming that the current contract is blocking correct behavior — not just inconvenient, but wrong. This maps to the Contract Candidate protocol in SKILL.md.

## Signal vs. Noise (Wiener Filter)

Wiener's wartime work on separating signals from noise led to the Wiener filter: a mathematical framework for extracting useful information from noisy data. The principle: not everything you observe warrants action.

**For AutoReceive:** Feedback classification must distinguish:
- **Actionable signals** — errors with clear causes, user corrections, constraint violations, verified regressions
- **Noise** — ambiguous observations, inconclusive metrics, low-confidence hunches, one-off anomalies

The correct response to noise is: record it in `observations`, do not trigger plan changes. "Interesting but unclear" is a valid and important classification. Acting on noise is worse than ignoring it — it introduces random perturbation.

**Practical test:** Before routing a signal as Immediate, ask: "If I act on this and it turns out to be noise, what's the cost?" If the cost is high, classify as Deferred and gather more evidence first.

## Requisite Variety (Ashby's Law)

Ashby's Law: "Only variety can destroy variety." A controller must have response capabilities that match or exceed the complexity of the disturbances it faces. If the environment is more complex than the controller, the environment overwhelms it.

**For AutoReceive:** The classification schema (type × priority × source) must be rich enough to handle the variety of incoming signals. If everything is bucketed as "high/low priority", nuanced feedback gets misrouted:
- A new constraint from the user (high priority, escalated) ≠ a compile error (high priority, immediate)
- A tool timing out once (low signal) ≠ a tool timing out every cycle (pattern = high signal)

The current schema provides 7 types × 3 priorities × 3 sources = 63 possible classifications. This is sufficient for typical optimization loops. If it proves inadequate for a specific domain, expand the schema — don't force-fit signals into wrong categories.

## Reflexivity (Second-Order Cybernetics)

Von Foerster's key move: the observer is part of the observed system. "First-order cybernetics is the cybernetics of observed systems. Second-order cybernetics is the cybernetics of observing systems."

**For AutoReceive:** The system's own outputs are valid feedback inputs:
- Check results → feedback about whether the approach is working
- Align drift signals → feedback about whether the system is maintaining discipline
- Log patterns → feedback about whether the loop itself is healthy (e.g., consecutive rollbacks suggest a deeper problem)
- A pattern of `adjust` decisions across cycles → the system is struggling; consider triggering AutoResearch

This is the `source: self` classification in the schema. Most feedback systems only handle external input (user, system). AutoReceive explicitly processes self-generated signals because the optimization loop is itself part of the system being observed.

**Practical application:** After 3+ cycles, scan the Log for patterns. Are decisions always `continue`? That might mean the probes are too conservative. Are decisions always `adjust`? That might mean the plan is systematically wrong. The meta-pattern matters.

## How These Principles Connect

```
Stable Contract (setpoint)
    ↑ negative feedback (correction)
    ↑ positive feedback (reinforcement)
    ↑ signal filtering (Wiener: separate signal from noise)
    ↑ variety matching (Ashby: schema fits disturbance complexity)
    ↑ reflexivity (von Foerster: system observes itself)
    ↑
AutoReceive (classification → routing → integration)
    ↑
Incoming signals (user, system, self)
```

The stack works bottom-up: signals arrive, get classified with enough variety to be useful, filtered for noise, routed as negative (corrective) or positive (reinforcing) feedback, and applied against the homeostatic setpoint. The system also observes its own processing, making the feedback loop recursive.

## Sources

- Wiener, N. (1948). *Cybernetics: Or Control and Communication in the Animal and the Machine*. MIT Press.
- Ashby, W.R. (1956). *An Introduction to Cybernetics*. Chapman & Hall.
- von Foerster, H. (1974). *Cybernetics of Cybernetics*. University of Illinois.
- [Nature: Return of cybernetics](https://www.nature.com/articles/s42256-019-0100-x)
- [Medium: 6 Key Ideas of 2nd Order Cybernetics](https://medium.com/intuitionmachine/6-key-ideas-of-2nd-order-cybernetics-explained-83c5692a1a77)
- [Businessballs: Ashby's Law](https://www.businessballs.com/strategy-innovation/ashbys-law-of-requisite-variety/)
- [Medium: Wiener and the Birth of Cybernetics](https://medium.com/@RaghavKrishna25/norbert-wiener-and-the-birth-of-cybernetics-the-science-of-feedback-de7c411debca)
