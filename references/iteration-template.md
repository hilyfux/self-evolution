# Iteration Template

Use this template when running a PDCAA cycle. Each cycle produces the mandatory 11-field log output.

## Stable Contract (establish once, before loop)

```text
> **Stable Contract**
> - Goal: <唯一目标>
> - Constraints: <不可违反的约束>
> - Done: <完成定义>
> - Checks: <必检项>
```

## PDCAA Cycle

### 1. Plan

```text
> **Plan:** <当前最小动作及其理由>
```

### 2. Do

```text
> **Do:** <执行了什么，结果是什么>
```

### 3. Check

```text
> **Check:**
> - Goal advancement: yes/no — <evidence>
> - Constraint violation: none / <which one>
> - New problems: none / <what>
> - Verifiable: yes/no — <how verified>
> - Future checkability: preserved / degraded — <note>
```

### 4. Align

```text
> **Align:** <drift detected: none / type + correction taken>
```

### 5. Act

```text
> **Act:** <decision> — <reason>
```

## AutoResearch (when triggered)

```text
> **AutoResearch:**
> - Trigger: <why activated>
> - Candidates: <approaches considered>
> - Recommendation: <minimum probe action>
```

## AutoReceive (when new input arrives)

```text
> **Contract Candidate:** <proposed change to which field> — <reason>
```

## Log (mandatory every cycle)

```text
round_id:         <cycle number, e.g. 1, 2, 3>
goal:             <current goal summary>
current_action:   <what was done>
action_reason:    <why this action>
execution_result: <what happened>
check_result:     <verification outcome>
align_result:     <drift signal or none>
feedback:         <any received input>
decision:         <continue/adjust/rollback/research/receive_update/complete>
next_step:        <what happens next>
log_summary:      <one-line digest of this cycle>
```

## Iteration Checkpoint (between cycles)

```text
> **[boost · Iter N]** Target: <target> | Goal: <goal> | 上轮: <result> → 本轮: <next focus>
```

## Compact Form

```text
<target>...</target>
<goal>...</goal>
<contract>...</contract>
<plan>...</plan>
<next_action>...</next_action>
```
