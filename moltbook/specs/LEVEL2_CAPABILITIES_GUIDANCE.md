# LoopCore Level 2 Capabilities: Architectural Guidance

**Version:** 1.0.0
**Status:** Architectural Specification
**Audience:** Senior Architects
**Purpose:** High-level design guidance for detailed specification development

---

## Document Overview

This document provides architectural guidance for four advanced cognitive capabilities to be added to LoopCore. Each section defines:

- **What** the capability does
- **Why** it matters
- **When** it should activate
- **Design principles** to follow
- **Key questions** architects must answer
- **Integration points** with existing systems
- **Success criteria** for validation

These are intentionally non-prescriptive on implementation details. Architects should use this guidance to create detailed specifications appropriate for their team and codebase.

---

# 1. REFLECTION

## 1.1 Concept

**Reflection** is the agent's ability to step back from task execution, evaluate its own progress and behavior, and make course corrections. It transforms the agent from a blind executor into a self-aware problem solver.

### Core Insight

Without reflection, an agent that starts down a wrong path will continue down that path until it hits a wall (max turns, timeout, or explicit failure). Reflection introduces metacognition - thinking about thinking.

## 1.2 Why It Matters

| Without Reflection | With Reflection |
|--------------------|-----------------|
| Repeats same failing approach | Recognizes failure pattern, tries alternative |
| No awareness of progress | Evaluates distance to goal |
| Continues until external limit | Self-terminates when stuck |
| Blind to its own mistakes | Identifies and corrects errors |
| Single strategy only | Adapts strategy based on feedback |

## 1.3 When Reflection Should Occur

Reflection is not free - it costs tokens and time. Trigger it strategically:

### Mandatory Triggers
- After N consecutive turns without measurable progress
- After any tool execution failure
- After receiving unexpected or ambiguous results
- Before executing high-risk or irreversible actions

### Optional Triggers
- At regular intervals (every N turns)
- When approaching resource limits (turns, tokens, time)
- When confidence in current approach drops
- On explicit user request

### When NOT to Reflect
- Simple, single-step tasks
- When making clear progress
- When reflection would exceed remaining budget

## 1.4 Design Principles

### Principle 1: Reflection is a Separate Cognitive Mode

Reflection should not be interleaved with task execution. It is a distinct phase where the agent temporarily stops acting and starts evaluating.

```
Execute → Execute → Execute → REFLECT → Execute → Execute → REFLECT → Complete
```

### Principle 2: Reflection Must Be Actionable

Reflection that produces only observations is wasteful. Every reflection must conclude with a decision:
- Continue current approach
- Modify current approach (how?)
- Abandon and try alternative (what?)
- Escalate to human
- Terminate with partial result

### Principle 3: Reflection Has Access to Full Context

The reflecting agent must see:
- Original goal/task
- All actions taken so far
- All results received
- Current state
- Resources remaining

### Principle 4: Reflection Should Be Honest

The reflection prompt must encourage genuine self-criticism, not self-justification. The agent should be rewarded (in prompt design) for identifying its own mistakes.

## 1.5 Key Architectural Questions

Architects must answer these before detailed design:

### Scope Questions
1. How deep should reflection go? (Surface: "Am I progressing?" vs. Deep: "Why did I choose this approach?")
2. Should reflection be agent-initiated or loop-controlled?
3. Can reflection override loop parameters (e.g., decide to exceed max_turns)?

### Implementation Questions
4. Is reflection a separate LLM call or part of the main loop?
5. Does reflection use the same model/temperature as execution?
6. How is reflection output structured? (Free text vs. structured decision)
7. Where is reflection history stored? (Memory? Session? Ephemeral?)

### Integration Questions
8. How does reflection interact with the skill system?
9. Can skills define their own reflection checkpoints?
10. Does reflection affect memory (should "I realized I was wrong" be remembered)?

### Resource Questions
11. What is the token budget for reflection?
12. How many reflections per session maximum?
13. Does reflection count against max_turns?

## 1.6 Integration Points

| Component | Integration Need |
|-----------|------------------|
| Agentic Loop | Hook for reflection trigger, pause/resume execution |
| Context Builder | Include reflection history in context |
| Memory System | Optionally persist reflection insights |
| Skill System | Allow skills to define reflection points |
| Observability | Log reflection events for debugging |

## 1.7 Success Criteria

A reflection implementation is successful when:

- [ ] Agent detects when it's stuck (no progress over N turns)
- [ ] Agent can articulate what's not working
- [ ] Agent proposes concrete alternatives
- [ ] Agent follows through on reflection decisions
- [ ] Reflection improves task completion rate on complex tasks
- [ ] Reflection does not significantly slow down simple tasks

## 1.8 Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| Reflect every turn | Wastes resources, slows execution |
| Reflection without decision | Observes but doesn't act |
| Reflection ignores its own advice | Reflects then continues same approach |
| Infinite reflection loop | Reflects on reflection on reflection... |
| Reflection as excuse | Uses reflection to justify giving up too early |

---

# 2. PLANNING

## 2.1 Concept

**Planning** is the agent's ability to decompose a complex task into a structured sequence of steps before beginning execution. It transforms reactive behavior ("what should I do next?") into proactive behavior ("here's my complete approach").

### Core Insight

Humans don't write a novel one sentence at a time without knowing where it's going. Neither should agents tackle complex tasks without a plan. Planning enables coherent multi-step execution and provides a framework for progress measurement.

## 2.2 Why It Matters

| Without Planning | With Planning |
|------------------|---------------|
| Each turn decides next step | All steps known upfront |
| No progress measurement | Clear milestone tracking |
| Easy to lose coherence | Plan maintains direction |
| Redundant exploration | Efficient execution path |
| Hard to estimate completion | Can predict remaining effort |

## 2.3 When Planning Should Occur

### Tasks That Need Planning
- Multi-step workflows (5+ steps)
- Tasks with dependencies between steps
- Tasks with multiple possible approaches
- Tasks requiring resource allocation
- Tasks with explicit deliverables

### Tasks That Don't Need Planning
- Simple Q&A
- Single-tool operations
- Well-defined atomic tasks
- Tasks where planning overhead exceeds execution

### Planning Triggers
- Task complexity exceeds threshold
- User explicitly requests a plan
- Skill file specifies planning phase
- Task involves irreversible actions

## 2.4 Design Principles

### Principle 1: Plans Are Hypotheses, Not Commitments

A plan is the agent's best guess at the right approach given current information. Plans must be revisable as new information emerges.

```
Plan → Execute Step 1 → New Info → Revise Plan → Execute Step 2 → ...
```

### Principle 2: Plans Should Be Hierarchical

Complex tasks need multi-level plans:
- **Strategic**: Overall approach and major phases
- **Tactical**: Steps within each phase
- **Operational**: Specific tool calls and parameters

Not every task needs all levels.

### Principle 3: Plans Must Be Executable

A plan item like "solve the problem" is useless. Plans must decompose to actionable steps that map to available tools and capabilities.

### Principle 4: Plans Should Include Contingencies

Good plans anticipate failure:
- What if this API is unavailable?
- What if the file doesn't exist?
- What if the data is malformed?

### Principle 5: Plans Are Visible

Plans should be inspectable by users and available to the agent throughout execution. Hidden plans are undebuggable plans.

## 2.5 Key Architectural Questions

### Structure Questions
1. What is the plan data structure? (Linear list? DAG? Tree?)
2. How are dependencies between steps represented?
3. How are contingencies/branches represented?
4. What metadata does each step carry? (Estimated effort? Required tools? Success criteria?)

### Creation Questions
5. Is planning a separate LLM call or integrated with first turn?
6. How detailed should initial plan be?
7. Should agent explain its planning rationale?
8. How long can planning take before it's "analysis paralysis"?

### Execution Questions
9. How strictly must agent follow the plan?
10. When can agent deviate from plan without replanning?
11. How is plan progress tracked?
12. What triggers a replan vs. minor adjustment?

### Persistence Questions
13. Are plans persisted to disk/memory?
14. Can plans be resumed after session interruption?
15. Are successful plans remembered for similar future tasks?

## 2.6 Integration Points

| Component | Integration Need |
|-----------|------------------|
| Agentic Loop | Plan-aware execution mode, progress tracking |
| Reflection | Reflect on plan validity, trigger replanning |
| Memory System | Store successful plans for reuse |
| Skill System | Skills may include plan templates |
| Context Builder | Include current plan and progress |
| Observability | Log plan creation, revisions, completion |

## 2.7 Success Criteria

A planning implementation is successful when:

- [ ] Agent creates coherent multi-step plans
- [ ] Plans are specific enough to execute
- [ ] Agent tracks progress against plan
- [ ] Agent detects when plan is failing
- [ ] Agent can revise plans mid-execution
- [ ] Planning improves completion rate on complex tasks
- [ ] Simple tasks bypass planning overhead

## 2.8 Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| Plan everything | Wastes time on simple tasks |
| Over-detailed plans | Brittle, breaks on first surprise |
| Under-detailed plans | Not actionable, no better than no plan |
| Rigid plan adherence | Ignores new information |
| Constant replanning | Never makes progress |
| Plan without execution | All planning, no doing |

---

# 3. LEARNING

## 3.1 Concept

**Learning** is the agent's ability to improve performance over time by remembering what worked, what didn't, and why. It transforms isolated sessions into cumulative experience.

### Core Insight

An agent that makes the same mistake twice hasn't learned. An agent that avoids a mistake because it remembers failing that way before has learned. Learning is memory with purpose.

## 3.2 Why It Matters

| Without Learning | With Learning |
|------------------|---------------|
| Same mistake repeatedly | Mistake made once, avoided thereafter |
| Each session starts fresh | Sessions build on past experience |
| No improvement over time | Performance improves with use |
| Generic approaches only | Approaches tuned to context |
| Blind to patterns | Recognizes and exploits patterns |

## 3.3 What Should Be Learned

### Learn From Failures
- What was attempted?
- Why did it fail?
- What would have worked instead?
- What warning signs existed?

### Learn From Successes
- What approach worked?
- Why did it work?
- What were the key factors?
- Is this generalizable?

### Learn From Feedback
- User corrections
- User preferences
- User domain knowledge
- Explicit "remember this" instructions

### Don't Learn (Blindly)
- One-off anomalies
- Context-specific details that don't generalize
- Stale information that will change
- Sensitive information that shouldn't persist

## 3.4 Design Principles

### Principle 1: Learning Must Generalize

Learning "API X was down on Tuesday" is not useful. Learning "APIs may be unavailable; always have a fallback" is useful. Learning must extract patterns, not memorize instances.

### Principle 2: Learning Must Be Retrievable

Learned knowledge that can't be found when needed provides no value. Learning requires not just storage but retrieval at the right moment.

```
New Task → Search Relevant Learnings → Inject into Context → Execute with Wisdom
```

### Principle 3: Learning Must Decay

Not all learnings remain valid forever. APIs change. Best practices evolve. Stale learnings can be worse than no learnings. Build in decay or validation.

### Principle 4: Learning Must Be Attributable

When the agent says "I learned not to do X," it should be traceable to why and when. This enables debugging and override when learnings are wrong.

### Principle 5: Learning Should Be Auditable

Users should be able to see what the agent has learned, correct wrong learnings, and add explicit learnings.

## 3.5 Key Architectural Questions

### What to Learn
1. What events trigger learning capture?
2. How is a "failure" defined programmatically?
3. How is a "success" defined?
4. How specific vs. general should learnings be?

### How to Store
5. Where do learnings live? (Memory system? Separate store?)
6. What's the schema for a learning entry?
7. How are learnings indexed for retrieval?
8. Is there a size/count limit on learnings?

### How to Retrieve
9. When are learnings searched for relevance?
10. How is relevance determined? (Keywords? Semantic?)
11. How many learnings are injected per task?
12. Where in the prompt do learnings appear?

### How to Maintain
13. How do learnings decay or expire?
14. Can learnings be updated/refined?
15. How are contradictory learnings resolved?
16. Can users CRUD learnings directly?

## 3.6 Learning Categories

Consider categorizing learnings for better retrieval:

| Category | Example | Retrieval Trigger |
|----------|---------|-------------------|
| Tool Usage | "file_write needs absolute paths" | When tool is about to be used |
| API Behavior | "This API returns 429 after 100 calls/min" | When API is about to be called |
| Domain Facts | "User's company uses Python 3.11" | When domain is mentioned |
| User Preferences | "User prefers concise responses" | Always |
| Failure Patterns | "Parsing HTML with regex fails" | When similar approach attempted |
| Success Patterns | "Breaking task into <5 steps works better" | During planning |

## 3.7 Integration Points

| Component | Integration Need |
|-----------|------------------|
| Memory System | Learnings as special memory type |
| Context Builder | Inject relevant learnings |
| Reflection | Reflection triggers learning capture |
| Skill System | Skills may have associated learnings |
| User Directives | "Remember this" creates learning |
| Observability | Log learning creation and retrieval |

## 3.8 Success Criteria

A learning implementation is successful when:

- [ ] Agent captures learnings from failures
- [ ] Agent retrieves relevant learnings for new tasks
- [ ] Retrieved learnings influence agent behavior
- [ ] Same mistake is not repeated after learning
- [ ] Learnings generalize across similar situations
- [ ] Stale/wrong learnings can be identified and removed
- [ ] Performance improves measurably over time

## 3.9 Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| Learn everything | Noise drowns signal |
| Learn nothing | No improvement over time |
| Over-generalize | "All APIs fail" from one failure |
| Under-generalize | Learning too specific to reuse |
| Never forget | Stale learnings mislead |
| Unattributed learnings | Can't debug or correct |

---

# 4. CONFIDENCE

## 4.1 Concept

**Confidence** is the agent's ability to assess and communicate its certainty (or uncertainty) about its outputs. It transforms assertions into calibrated claims.

### Core Insight

An agent that says "The answer is X" when it's sure and "The answer is X" when it's guessing is dangerous. Users can't tell when to trust it. Confidence calibration makes uncertainty visible and actionable.

## 4.2 Why It Matters

| Without Confidence | With Confidence |
|--------------------|-----------------|
| All assertions equal | Certainty is explicit |
| Users can't assess reliability | Users know when to verify |
| Guesses look like facts | Guesses are labeled |
| No escalation signal | Low confidence triggers escalation |
| Overconfidence on hallucinations | Uncertainty on unknowns |

## 4.3 Sources of Uncertainty

### Epistemic Uncertainty (Knowledge Gaps)
- Information not in training data
- Information requires real-time data
- Information is domain-specific
- Information requires computation/reasoning the model can't do

### Aleatoric Uncertainty (Inherent Randomness)
- Questions with multiple valid answers
- Predictions about future events
- Subjective judgments
- Ambiguous questions

### Contextual Uncertainty
- Insufficient context provided
- Contradictory information in context
- Context may be outdated
- Tool results may be unreliable

## 4.4 Design Principles

### Principle 1: Confidence Is About the Claim, Not the Agent

"I am 80% confident" is about a specific claim, not general capability. Different claims in the same response may have different confidence levels.

### Principle 2: Confidence Should Be Calibrated

An agent that says 90% confidence should be right 90% of the time on such claims. Calibration can be measured and improved.

### Principle 3: Low Confidence Is Not Failure

Saying "I don't know" or "I'm uncertain" is valuable output. It's better than a confident wrong answer.

### Principle 4: Confidence Should Trigger Actions

Confidence below threshold should trigger:
- Request for more information
- Escalation to human
- Caveats in response
- Alternative suggestions

### Principle 5: Confidence Is Communicable

Users should understand what confidence levels mean. "73% confident" is less useful than "Fairly confident - based on [sources], but you should verify [X]."

## 4.5 Key Architectural Questions

### Representation
1. How is confidence represented? (Numeric? Categorical? Explanation?)
2. Is confidence per-response or per-claim?
3. What's the confidence scale? (0-1? Low/Med/High? Percentage?)
4. Should confidence include reasoning?

### Generation
5. How is confidence calculated/generated?
6. Is confidence from the same LLM call or separate?
7. What signals inform confidence? (Source reliability? Reasoning chain? Hedging language?)
8. How is confidence calibrated over time?

### Thresholds
9. What confidence threshold triggers escalation?
10. What threshold triggers caveats?
11. Are thresholds configurable per use case?
12. What happens below minimum acceptable confidence?

### Communication
13. How is confidence communicated to users?
14. Do all responses include confidence, or only uncertain ones?
15. How verbose is confidence explanation?
16. Should confidence be in response or metadata?

## 4.6 Confidence Levels

Consider a structured confidence framework:

| Level | Numeric | Meaning | Agent Behavior |
|-------|---------|---------|----------------|
| **Certain** | 0.95+ | Verified fact or direct observation | Assert directly |
| **High** | 0.80-0.94 | Strong evidence, reliable sources | Assert with minimal caveat |
| **Moderate** | 0.60-0.79 | Good evidence, some uncertainty | Assert with caveats |
| **Low** | 0.40-0.59 | Limited evidence, plausible guess | Suggest, note uncertainty |
| **Speculative** | 0.20-0.39 | Educated guess, weak basis | Offer as possibility only |
| **Unknown** | <0.20 | No basis for claim | Decline or escalate |

## 4.7 Integration Points

| Component | Integration Need |
|-----------|------------------|
| Agentic Loop | Confidence-aware response generation |
| Planning | Confidence in plan steps, contingencies for uncertain steps |
| Reflection | Reflect on confidence calibration |
| Learning | Learn from overconfident mistakes |
| Memory | Memory recall confidence (old memories less certain) |
| Escalation | Low confidence triggers human escalation |
| API/Response | Include confidence in response structure |

## 4.8 Success Criteria

A confidence implementation is successful when:

- [ ] Agent expresses uncertainty when uncertain
- [ ] Agent expresses certainty when certain
- [ ] Confidence levels are calibrated (90% confident = 90% correct)
- [ ] Low confidence triggers appropriate actions (caveats, escalation)
- [ ] Users can understand and act on confidence levels
- [ ] Agent says "I don't know" rather than hallucinating
- [ ] Confidence improves trust and usability

## 4.9 Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| Always confident | Hides uncertainty, breeds distrust |
| Always uncertain | Useless, never commits |
| Uncalibrated confidence | Numbers meaningless |
| Confidence without explanation | Users can't evaluate |
| Confidence as excuse | "Low confidence" to avoid hard questions |
| Binary confidence | Loses nuance of calibration |

---

# CROSS-CUTTING CONCERNS

## How These Capabilities Interact

```
                    ┌─────────────┐
                    │   PLANNING  │
                    │  (before)   │
                    └──────┬──────┘
                           │
                           ▼
          ┌────────────────────────────────┐
          │         EXECUTION LOOP         │
          │                                │
          │  ┌──────────┐    ┌──────────┐  │
          │  │ LEARNING │◄───│REFLECTION│  │
          │  │(retrieve)│    │(evaluate)│  │
          │  └────┬─────┘    └────┬─────┘  │
          │       │               │        │
          │       ▼               ▼        │
          │  ┌─────────────────────────┐   │
          │  │       EXECUTE          │   │
          │  │   (with CONFIDENCE)    │   │
          │  └─────────────────────────┘   │
          │              │                 │
          │              ▼                 │
          │       ┌──────────┐             │
          │       │ LEARNING │             │
          │       │ (store)  │             │
          │       └──────────┘             │
          └────────────────────────────────┘
```

## Interaction Matrix

| | Planning | Reflection | Learning | Confidence |
|---|----------|------------|----------|------------|
| **Planning** | - | Reflection triggers replan | Past plans inform new plans | Low-confidence steps need contingencies |
| **Reflection** | Reflect on plan validity | - | Reflection insights become learnings | Confidence in reflection conclusions |
| **Learning** | Learn good planning patterns | Learn reflection triggers | - | Learn confidence calibration |
| **Confidence** | Plan confidence (will this work?) | Confidence in reflection accuracy | Confidence in learnings (how old? how relevant?) | - |

## Implementation Order Recommendation

Based on dependencies and value:

1. **Reflection** (First)
   - Foundation for self-improvement
   - No hard dependencies
   - Enables learning capture

2. **Planning** (Second)
   - Benefits from reflection (replan)
   - Enables structured execution
   - Provides framework for progress measurement

3. **Learning** (Third)
   - Captures insights from reflection
   - Improves planning over time
   - Requires retrieval infrastructure

4. **Confidence** (Fourth)
   - Applies to all outputs including reflection, planning
   - Requires calibration data (needs history)
   - Most valuable with other capabilities in place

---

# ARCHITECTURAL CHECKLIST

Before detailed specification, architects should confirm:

## For All Capabilities
- [ ] Token budget allocated
- [ ] Trigger conditions defined
- [ ] Integration points identified
- [ ] Data structures designed
- [ ] Persistence strategy chosen
- [ ] Observability requirements met
- [ ] Test strategy defined
- [ ] Rollback/disable strategy defined

## Capability-Specific
- [ ] Reflection: Decision framework defined
- [ ] Planning: Plan schema defined
- [ ] Learning: Generalization rules defined
- [ ] Confidence: Calibration strategy defined

---

*This document provides guidance. Architects should create detailed specifications with implementation specifics appropriate for their team, codebase, and use cases.*
