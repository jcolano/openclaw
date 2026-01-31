# Ultimate Test Skills for LoopCore Validation

**Purpose:** These 10 complex skills are designed to validate that your LoopCore implementation is truly agentic - not just a chatbot with tools.

**How to Use:**
1. Pick a skill the system has never seen
2. Load it as a skill file (or inject into the system)
3. Trigger it with a simple message
4. Walk away for 10 minutes
5. Return and evaluate: Did it complete correctly without intervention?

---

## The 10 Ultimate Tests

| # | Skill | Trigger | Complexity | Duration |
|---|-------|---------|------------|----------|
| 01 | [Market Research](./01_market_research.md) | "research [TOPIC] market" | High | 5-10 min |
| 02 | [Code Review](./02_code_review.md) | "review code in [DIR]" | Medium-High | 3-8 min |
| 03 | [Meeting Prep](./03_meeting_prep.md) | "prepare for meeting with [NAME]" | Medium | 5-8 min |
| 04 | [Data Pipeline](./04_data_pipeline.md) | "run data pipeline from [SOURCE]" | High | 3-7 min |
| 05 | [Incident Response](./05_incident_response.md) | "incident [SERVICE]" | High | 2-5 min |
| 06 | [Content Publishing](./06_content_publishing.md) | "publish [CONTENT] to [PLATFORMS]" | Medium-High | 5-10 min |
| 07 | [Research Synthesis](./07_research_synthesis.md) | "synthesize research on [TOPIC]" | Very High | 8-15 min |
| 08 | [Customer Onboarding](./08_customer_onboarding.md) | "onboard customer [NAME]" | Medium-High | 5-10 min |
| 09 | [Project Setup](./09_project_setup.md) | "create project [NAME]" | Medium | 3-7 min |
| 10 | [Competitive Intel](./10_competitive_intel.md) | "analyze competitor [COMPANY]" | Very High | 8-15 min |

---

## What Each Skill Tests

### Core Agentic Capabilities

| Capability | Tested By |
|------------|-----------|
| Multi-step execution | All skills (15+ steps each) |
| Conditional logic | 01, 02, 04, 05, 08 |
| Loops (FOR EACH) | 01, 02, 06, 07, 10 |
| Error handling | All skills have error handling sections |
| Memory operations | All skills include "Remember" steps |
| File operations | All skills write to OUTPUT/ |
| API calls | 01, 04, 05, 06, 08 |
| Web fetching | 01, 03, 07, 10 |
| Decision points | 04, 05, 06, 10 |

### Skill Difficulty Ranking

```
Difficulty
    │
    │  ██████████████████████████  07 Research Synthesis
    │  ██████████████████████████  10 Competitive Intel
    │  ████████████████████        01 Market Research
    │  ████████████████████        04 Data Pipeline
    │  ████████████████████        05 Incident Response
    │  ██████████████████          06 Content Publishing
    │  ██████████████████          08 Customer Onboarding
    │  ████████████████            02 Code Review
    │  ████████████████            03 Meeting Prep
    │  ██████████████              09 Project Setup
    │
    └───────────────────────────────────────────────────
```

---

## Evaluation Criteria

For each skill test, evaluate:

### Completion
- [ ] All 15+ steps attempted
- [ ] Output files created as specified
- [ ] Memory updated as instructed
- [ ] User received final summary

### Quality
- [ ] Conditional logic followed correctly
- [ ] Loops executed for all items
- [ ] Error handling worked (if errors occurred)
- [ ] Output is coherent and useful

### Autonomy
- [ ] No human intervention required
- [ ] Agent recovered from errors
- [ ] Agent made reasonable decisions at decision points

### Scoring

| Score | Meaning |
|-------|---------|
| 10/10 | Perfect execution - truly agentic |
| 8-9/10 | Minor issues but completed autonomously |
| 6-7/10 | Partial completion, some intervention needed |
| 4-5/10 | Significant issues, frequent intervention |
| 0-3/10 | Failed - not agentic |

---

## Suggested Test Order

**Start simple, increase complexity:**

1. **09 Project Setup** - Structured, predictable
2. **02 Code Review** - File-heavy, some analysis
3. **03 Meeting Prep** - Web research, synthesis
4. **08 Customer Onboarding** - Workflow, multi-step
5. **04 Data Pipeline** - Data manipulation, validation
6. **06 Content Publishing** - Multi-platform, adaptation
7. **01 Market Research** - Research, analysis, reporting
8. **05 Incident Response** - Time-sensitive, dynamic
9. **10 Competitive Intel** - Deep research, SWOT
10. **07 Research Synthesis** - Maximum complexity

---

## Common Failure Modes

Watch for these issues:

| Failure Mode | Indicates |
|--------------|-----------|
| Stops after 2-3 steps | Loop not continuing |
| Skips conditional branches | Conditional logic broken |
| Doesn't read skill file | Skill system not truly autonomous |
| Forgets earlier context | Context management issues |
| Crashes on first error | Error handling missing |
| Creates wrong file structure | Instructions not followed |
| Memory not updated | Memory integration broken |

---

## Passing the Ultimate Test

> **A skill passes the Ultimate Test if:**
>
> 1. It completes all major steps (15+)
> 2. It handles errors gracefully
> 3. It produces useful output
> 4. It requires zero human intervention
> 5. A human reviewing the output would find it valuable

If your system passes 8/10 skills with a score of 8+ each, you have built a truly agentic system.

---

*Generated for LoopCore validation - January 2026*
