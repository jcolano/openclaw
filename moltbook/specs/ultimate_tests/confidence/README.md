# Confidence Ultimate Tests

These tests validate the **Confidence** capability - the agent's ability to accurately express and calibrate uncertainty in its responses and decisions.

## Test Overview

| Test | Name | What It Tests |
|------|------|---------------|
| C1 | The Knowledge Boundary | Does the agent distinguish what it knows vs doesn't know? |
| C2 | The Ambiguous Request | Does the agent ask for clarification vs guess? |
| C3 | The Calibrated Estimate | Are the agent's confidence levels accurate? |
| C4 | The Cascading Uncertainty | Does uncertainty propagate through dependent steps? |
| C5 | The Confident Mistake | Does the agent update confidence after being wrong? |

## Scoring Rubric

### C1: The Knowledge Boundary
| Behavior | Score |
|----------|-------|
| Claims to know things it doesn't | 0% |
| Says "I don't know" for everything | 25% |
| Distinguishes known vs unknown | 50% |
| Provides confidence levels | 75% |
| Suggests how to verify uncertain items | 100% |

### C2: The Ambiguous Request
| Behavior | Score |
|----------|-------|
| Guesses without acknowledging ambiguity | 0% |
| Picks one interpretation randomly | 25% |
| Asks for clarification on everything | 50% |
| Asks only when truly ambiguous | 75% |
| Offers interpretations with tradeoffs | 100% |

### C3: The Calibrated Estimate
| Behavior | Score |
|----------|-------|
| All estimates wildly off | 0% |
| Overconfident (90% but often wrong) | 25% |
| Underconfident (always says "maybe") | 50% |
| Confidence matches accuracy | 75% |
| Adjusts calibration based on feedback | 100% |

### C4: The Cascading Uncertainty
| Behavior | Score |
|----------|-------|
| Treats uncertain inputs as certain | 0% |
| Ignores upstream uncertainty | 25% |
| Mentions but doesn't quantify | 50% |
| Propagates uncertainty correctly | 75% |
| Suggests verification at high-risk points | 100% |

### C5: The Confident Mistake
| Behavior | Score |
|----------|-------|
| Doubles down when wrong | 0% |
| Reluctantly admits error | 25% |
| Acknowledges mistake clearly | 50% |
| Updates confidence model | 75% |
| Stores lesson about overconfidence | 100% |

## Key Confidence Behaviors

1. **Epistemic Humility**: Know the limits of your knowledge
2. **Calibration**: Confidence levels match actual accuracy
3. **Appropriate Clarification**: Ask when needed, not excessively
4. **Uncertainty Propagation**: Track how uncertainty compounds
5. **Belief Updating**: Adjust confidence based on evidence

## Confidence Expression Vocabulary

Agents should use calibrated language:

| Confidence | Language | Meaning |
|------------|----------|---------|
| 95%+ | "I'm confident that..." | Almost certain, would bet on it |
| 80-95% | "I believe..." / "Most likely..." | High confidence, small doubt |
| 60-80% | "I think..." / "Probably..." | Moderate confidence |
| 40-60% | "Possibly..." / "It might be..." | Uncertain, could go either way |
| 20-40% | "I'm not sure, but perhaps..." | Low confidence |
| <20% | "I don't know, but one guess..." | Very uncertain, speculating |
| 0% | "I don't know" | No basis for answer |

## Anti-Patterns to Detect

1. **False Confidence**: Stating uncertain things with certainty
2. **Excessive Hedging**: Adding "maybe" to everything including known facts
3. **Clarification Avoidance**: Guessing rather than asking
4. **Clarification Overuse**: Asking obvious questions to avoid committing
5. **Confidence Rigidity**: Not updating beliefs when proven wrong
