# Learning Ultimate Tests

These tests validate the **Learning** capability - the agent's ability to remember what worked, what failed, and improve over time.

## Test Overview

| Test | Name | What It Tests |
|------|------|---------------|
| L1 | The Repeated Mistake | Does the agent avoid making the same error twice? |
| L2 | The Successful Pattern | Does the agent reuse approaches that worked before? |
| L3 | The User Preference | Does the agent remember and apply user preferences? |
| L4 | The Tool Reliability | Does the agent learn which tools/APIs are unreliable? |
| L5 | The Context Carryover | Does the agent apply lessons across different but similar tasks? |

## Scoring Rubric

### L1: The Repeated Mistake
| Behavior | Score |
|----------|-------|
| Makes same mistake repeatedly | 0% |
| Notices mistake but doesn't prevent | 25% |
| Prevents repeat after explicit failure | 50% |
| Proactively checks before risky operations | 75% |
| Stores lesson in memory for future sessions | 100% |

### L2: The Successful Pattern
| Behavior | Score |
|----------|-------|
| Treats every similar task as new | 0% |
| Vaguely recalls "this worked before" | 25% |
| Retrieves and applies previous approach | 50% |
| Adapts previous approach to new context | 75% |
| Stores pattern as reusable template | 100% |

### L3: The User Preference
| Behavior | Score |
|----------|-------|
| Ignores stated preferences | 0% |
| Follows preference only when reminded | 25% |
| Remembers preferences within session | 50% |
| Applies preferences proactively | 75% |
| Stores preferences for future sessions | 100% |

### L4: The Tool Reliability
| Behavior | Score |
|----------|-------|
| Keeps trying unreliable tool | 0% |
| Switches after many failures | 25% |
| Tracks failure rate mentally | 50% |
| Proactively uses fallback first | 75% |
| Stores reliability data for future | 100% |

### L5: The Context Carryover
| Behavior | Score |
|----------|-------|
| No transfer between similar tasks | 0% |
| Applies lesson only when identical | 25% |
| Recognizes similarity, applies lesson | 50% |
| Abstracts lesson to general principle | 75% |
| Stores generalized insight | 100% |

## Key Learning Behaviors

1. **Recognition**: Notice when current situation resembles past experience
2. **Retrieval**: Fetch relevant past experience from memory
3. **Application**: Apply learned lesson to current context
4. **Adaptation**: Adjust past approach for new circumstances
5. **Storage**: Save new learnings for future use

## Memory Integration

These tests assume the agent has access to:
- `memory_search(query)` - Search past experiences
- `memory_store(content, metadata)` - Store new learnings
- Session context that persists within a session
- Long-term memory that persists across sessions
