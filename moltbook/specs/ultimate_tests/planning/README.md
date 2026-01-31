# Ultimate Tests: Planning Capability

**Purpose:** Validate that LoopCore's Planning capability actually works - the agent creates structured plans before execution, tracks progress, and adapts plans when needed.

**Key Difference from Standard Tests:** These tests require multi-step coordination where diving in without a plan leads to backtracking, wasted work, or incomplete results. A non-planning agent will make progress but inefficiently or incompletely.

---

## Test P1: The Dependency Chain

**Trigger:** "set up the development environment for the new project"

**What This Tests:**
- Recognition that tasks have dependencies
- Correct ordering of dependent steps
- Detection of missing prerequisites before starting dependent work

---

## Test P2: The Parallel Opportunities

**Trigger:** "prepare the quarterly report with data from all departments"

**What This Tests:**
- Identification of independent subtasks
- Efficient organization (parallel vs. sequential)
- Aggregation of parallel results

---

## Test P3: The Evolving Requirements

**Trigger:** "build the user dashboard based on the requirements doc"

**What This Tests:**
- Initial plan creation from requirements
- Plan revision when requirements change mid-execution
- Graceful handling of scope changes

---

## Test P4: The Resource Constraints

**Trigger:** "process all customer data for the migration"

**What This Tests:**
- Planning with limited resources (turns, time)
- Prioritization when not everything can be done
- Explicit scope decisions in plan

---

## Test P5: The Multi-Phase Project

**Trigger:** "implement the new authentication system"

**What This Tests:**
- Hierarchical planning (phases → tasks → steps)
- Phase gate decisions
- Long-horizon coherence

---

# Evaluation Framework

## What Good Planning Looks Like

```
1. PLAN CREATION (before execution)
   - Identify all major steps
   - Note dependencies
   - Estimate effort
   - Identify risks

2. PLAN DOCUMENTATION
   - Written plan visible in output
   - Clear success criteria per step
   - Contingencies noted

3. PLAN EXECUTION
   - Progress tracked against plan
   - Deviations noted
   - Milestones acknowledged

4. PLAN ADAPTATION
   - Changes documented
   - Reasoning explained
   - Impact assessed
```

## Scoring Rubric

| Score | Meaning |
|-------|---------|
| 0 | No plan created; dove straight into execution |
| 1 | Mentioned planning but didn't create explicit plan |
| 2 | Created plan but didn't follow or track it |
| 3 | Created plan, followed it, but didn't adapt when needed |
| 4 | Created plan, tracked progress, adapted appropriately |
| 5 | Optimal planning with dependencies, parallelization, and adaptation |

## What to Look for in Logs

Evidence of planning:
```
[PLANNING] Analyzing task requirements...
[PLANNING] Identified 5 major phases
[PLANNING] Dependencies: Phase 2 requires Phase 1 completion
[PLANNING] Creating execution plan...
[PLAN] Step 1: ... (estimated: 3 turns)
[PLAN] Step 2: ... (depends on: Step 1)
[PROGRESS] Step 1 complete (actual: 2 turns)
[PROGRESS] Starting Step 2
[REPLAN] New requirement discovered, updating plan...
```

Anti-patterns (planning not working):
```
[TURN 1] Let me start by doing X
[TURN 2] Now I'll do Y
[TURN 3] Oh wait, I needed Z before Y
[TURN 4] Going back to do Z
[TURN 5] Now redoing Y...
```

---

*These tests specifically validate planning capability. An agent that passes standard tests but fails these tests is reactive, not proactive.*
