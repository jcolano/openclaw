# Ultimate Test U2: The Research Labyrinth

**Difficulty:** Extreme
**Duration:** Extended research session
**Capabilities Required:** Confidence ↔ Reflection ↔ Learning with Planning

---

## Scenario

You're researching a technical question where:
- Multiple sources exist but contradict each other
- Some sources are outdated or wrong
- The "obvious" answer is actually incorrect
- Dead ends are common
- The truth requires synthesizing partial information

Your job: Navigate to accurate understanding while maintaining calibrated confidence.

---

## The Question

User request: "What's the best way to handle WebSocket reconnection with exponential backoff in a React application? I've seen conflicting advice."

---

## Phase 1: Initial Research

### Source 1: Stack Overflow (High Votes)

```
file_read: SANDBOX/research/source1_stackoverflow.md
```

```markdown
# Best Answer (847 upvotes, accepted)

Use the `reconnecting-websocket` library. It handles everything for you:

```javascript
import ReconnectingWebSocket from 'reconnecting-websocket';

const ws = new ReconnectingWebSocket('wss://example.com');
```

Simple and battle-tested. Don't reinvent the wheel.

*Last updated: 2019*
```

### Initial Assessment

**CONFIDENCE CHECKPOINT:**

```markdown
## Source 1 Assessment

**Credibility indicators:**
- 847 upvotes: HIGH social proof
- Accepted answer: ENDORSED by asker
- Stack Overflow: REPUTABLE platform

**Concern indicators:**
- Date: 2019 (5+ years old)
- React-specific? NO - generic JS
- Hooks era? UNCLEAR

**Initial confidence: 60%**

This might be the answer, but it's old. React has changed significantly.
I should find more recent sources before committing.
```

---

### Source 2: React Documentation

```
file_read: SANDBOX/research/source2_react_docs.md
```

```markdown
# React Docs: Synchronizing with Effects

WebSockets should be handled in useEffect with proper cleanup:

```javascript
useEffect(() => {
  const ws = new WebSocket(url);

  ws.onclose = () => {
    // Handle reconnection
  };

  return () => ws.close();
}, [url]);
```

Note: For complex real-time needs, consider libraries like Socket.IO
or custom hooks.

*React 18 documentation*
```

### Updated Assessment

**CONFIDENCE + LEARNING:**

```markdown
## Source 2 Assessment

**New information:**
- React docs recommend useEffect pattern
- No mention of reconnecting-websocket
- Suggests Socket.IO for "complex" needs

**Conflict detected:**
- Source 1: Use library (reconnecting-websocket)
- Source 2: Use useEffect (native WebSocket)

**Confidence update:**
- reconnecting-websocket: 60% → 50% (not mentioned in React docs)
- useEffect pattern: NEW - 55%
- Socket.IO: NEW - 40% (might be overkill)

**Learning: Store conflict**
```json
{
  "type": "research_conflict",
  "topic": "websocket_react",
  "sources": ["stackoverflow_2019", "react_docs_v18"],
  "conflict": "library vs native pattern",
  "resolution": "pending"
}
```
```

---

### Source 3: Blog Post (Recent)

```
file_read: SANDBOX/research/source3_blog.md
```

```markdown
# WebSockets in React 2024: The Right Way

**DO NOT use reconnecting-websocket!**

It's unmaintained (last update 2020) and has memory leaks with React 18's
Strict Mode (double-mounting useEffect).

Instead, use this custom hook:

```javascript
function useWebSocket(url) {
  const [status, setStatus] = useState('connecting');
  const wsRef = useRef(null);
  const reconnectAttempts = useRef(0);

  useEffect(() => {
    function connect() {
      wsRef.current = new WebSocket(url);

      wsRef.current.onopen = () => {
        setStatus('connected');
        reconnectAttempts.current = 0;
      };

      wsRef.current.onclose = () => {
        setStatus('disconnected');
        const delay = Math.min(1000 * Math.pow(2, reconnectAttempts.current), 30000);
        reconnectAttempts.current++;
        setTimeout(connect, delay);
      };
    }

    connect();
    return () => wsRef.current?.close();
  }, [url]);

  return { status, ws: wsRef.current };
}
```

*Published: January 2024*
*Author: ReactDev2024 (unknown authority)*
```

### Critical Assessment

**REFLECTION + CONFIDENCE:**

```markdown
## Source 3 Assessment

**Significant claim:** reconnecting-websocket has memory leaks with React 18

**Credibility check:**
- Recent: YES (2024)
- Author credibility: UNKNOWN
- Verifiable claim: CAN CHECK

**This creates a problem:**
- If true: Source 1's answer is harmful
- If false: This blog is spreading misinformation

**Confidence cascade:**

| Option | Before Source 3 | After Source 3 |
|--------|-----------------|----------------|
| reconnecting-websocket | 50% | 30% (if leak claim true) |
| Custom hook | 55% | 60% (addresses React 18) |
| Need to verify leak claim | - | HIGH PRIORITY |

**Reflection: What do I actually know?**

I DON'T KNOW if the memory leak claim is true.
I've read three sources with three different recommendations.
My confidence in ANY answer is now LOWER than when I started.

This is expected in research - more information can increase uncertainty
before it decreases it.
```

---

## Phase 2: Verification

### Task 2.1: Verify the Memory Leak Claim

**PLANNING: Research plan to resolve uncertainty**

```markdown
## Verification Plan

**Claim to verify:** reconnecting-websocket has memory leaks with React 18 Strict Mode

**Verification approaches:**
1. Check reconnecting-websocket GitHub issues
2. Check npm download trends (is it actually abandoned?)
3. Look for React 18 compatibility discussions
4. Test it myself (if other approaches inconclusive)

**Priority order:**
1. GitHub issues (most direct evidence)
2. npm stats (easy to check)
3. Broader search if needed
```

### GitHub Issues Check

```
file_read: SANDBOX/research/source4_github_issues.md
```

```markdown
# reconnecting-websocket GitHub Issues

## Issue #152: Memory leak with React 18 Strict Mode
*Opened: March 2023*
*Status: Open*

"When using Strict Mode, the double-mount creates duplicate WebSocket
connections. The library doesn't handle cleanup properly..."

**Maintainer response:** (none)

## Issue #148: Is this project maintained?
*Opened: January 2023*
*Status: Open*

"Last commit was 2020. Are there plans to update for React 18?"

**Maintainer response:** (none)

---

## npm stats:
- Weekly downloads: 180,000
- Last publish: 3 years ago
- Open issues: 47
```

### Verification Assessment

**CONFIDENCE UPDATE + LEARNING:**

```markdown
## Verification Results

**Memory leak claim: VERIFIED (85% confidence)**

Evidence:
- GitHub issue confirms React 18 problem
- No maintainer response
- Last update 3 years ago
- Multiple open issues about maintenance

**Learning: Update beliefs**

```json
{
  "type": "verified_fact",
  "topic": "reconnecting-websocket",
  "fact": "Has compatibility issues with React 18 Strict Mode",
  "confidence": 85,
  "evidence": "GitHub issues #152, no maintenance since 2020"
}
```

**Stack Overflow answer reassessment:**

That 847-upvote answer is now HARMFUL:
- Upvotes reflect 2019 best practices
- React 18 changed the game
- Thousands of developers might be using broken code

**Confidence update:**

| Option | Previous | Current | Reason |
|--------|----------|---------|--------|
| reconnecting-websocket | 30% | 10% | Verified issues |
| Custom hook (blog) | 60% | 70% | Addresses real problem |
| Need more validation | - | YES | Blog author unknown |
```

---

## Phase 3: Synthesis

### Task 3.1: Find Authoritative Source

**PLANNING: Need higher-quality source for custom hook approach**

```markdown
## Remaining uncertainty

The blog's custom hook MIGHT be the answer, but:
- Unknown author
- No peer review
- Could have its own bugs

**Search for:**
- Kent C. Dodds (React expert) WebSocket content
- React RFC or discussion on real-time
- Well-maintained library that IS React 18 compatible
```

### Source 5: React Expert Blog

```
file_read: SANDBOX/research/source5_expert.md
```

```markdown
# Real-Time React: A Modern Approach
*By: Alex Chen, React Core Team contributor*

For WebSocket reconnection in React 18+, the key considerations are:

1. **Strict Mode compatibility** - Your connection logic will run twice in dev
2. **Cleanup on unmount** - Prevent zombie connections
3. **Exponential backoff** - Don't hammer your server

Here's the pattern we use at [Major Company]:

```javascript
function useReconnectingWebSocket(url, options = {}) {
  const { maxRetries = 10, baseDelay = 1000 } = options;
  const [state, setState] = useState({ status: 'connecting', retries: 0 });
  const wsRef = useRef(null);
  const timeoutRef = useRef(null);
  const mountedRef = useRef(true);

  useEffect(() => {
    mountedRef.current = true;

    function connect() {
      // Don't reconnect if unmounted
      if (!mountedRef.current) return;

      const ws = new WebSocket(url);
      wsRef.current = ws;

      ws.onopen = () => {
        if (mountedRef.current) {
          setState({ status: 'connected', retries: 0 });
        }
      };

      ws.onclose = (event) => {
        if (!mountedRef.current) return;

        // Don't reconnect on intentional close
        if (event.code === 1000) {
          setState(s => ({ ...s, status: 'closed' }));
          return;
        }

        setState(s => {
          const retries = s.retries + 1;
          if (retries > maxRetries) {
            return { status: 'failed', retries };
          }

          const delay = Math.min(baseDelay * Math.pow(2, retries - 1), 30000);
          timeoutRef.current = setTimeout(connect, delay);

          return { status: 'reconnecting', retries };
        });
      };
    }

    connect();

    return () => {
      mountedRef.current = false;
      clearTimeout(timeoutRef.current);
      wsRef.current?.close(1000, 'Component unmounted');
    };
  }, [url, maxRetries, baseDelay]);

  return state;
}
```

Key differences from naive implementations:
- `mountedRef` prevents state updates after unmount
- Intentional close (code 1000) doesn't trigger reconnect
- Max retries prevents infinite reconnection
- Cleanup cancels pending reconnection attempts
```

### Final Synthesis

**ALL CAPABILITIES INTEGRATED:**

```markdown
## Research Synthesis

### Confidence Journey

| Phase | Best Answer | Confidence | Why |
|-------|-------------|------------|-----|
| Initial | Unknown | 0% | Haven't researched |
| Source 1 | reconnecting-websocket | 60% | High votes |
| Source 2 | useEffect pattern | 55% | Official docs |
| Source 3 | Custom hook | 50% | Conflicts, unknown author |
| Source 4 | NOT reconnecting-websocket | 85% | Verified issues |
| Source 5 | Expert custom hook | 85% | Credible source |

### Answer Confidence: 85%

**Recommended approach:** Custom hook (Source 5 pattern)

**Reasoning:**
1. React 18 Strict Mode compatible (verified need)
2. From credible source (React team contributor)
3. Addresses edge cases (unmount, intentional close)
4. Production-tested (used at major company)

### Remaining Uncertainty (15%)

- Might be edge cases in specific use cases
- Haven't tested personally
- WebSocket ecosystem continues to evolve

### Key Learnings Stored

```json
[
  {
    "lesson": "Old Stack Overflow answers can be harmful",
    "context": "Technology moves fast, 2019 != 2024",
    "action": "Check answer dates, verify with recent sources"
  },
  {
    "lesson": "High upvotes != correct answer",
    "context": "reconnecting-websocket has 847 upvotes but causes issues",
    "action": "Upvotes reflect past consensus, not current best practice"
  },
  {
    "lesson": "React 18 Strict Mode breaks many patterns",
    "context": "Double-mount in dev mode catches cleanup issues",
    "action": "For any React effect code, verify Strict Mode compatibility"
  },
  {
    "lesson": "Verify claims before trusting",
    "context": "Blog claimed memory leak, I verified via GitHub",
    "action": "Always check primary sources for significant claims"
  }
]
```

### Final Answer to User

"For WebSocket reconnection in React, here's my recommendation with
85% confidence:

**Don't use:** reconnecting-websocket library
- It's unmaintained (last update 2020)
- Has known issues with React 18 Strict Mode
- Despite 847 upvotes on Stack Overflow, it's outdated advice

**Do use:** Custom hook pattern (see implementation)
- React 18 compatible
- Proper cleanup handling
- Exponential backoff built in

I verified this by checking GitHub issues, npm stats, and multiple
sources. The React ecosystem has evolved significantly since older
advice was written.

If you need more features (rooms, presence, etc.), consider Socket.IO
or Ably, but for basic WebSocket reconnection, the custom hook is
the modern approach."
```

---

## Phase 4: Reflection on Process

### Task 4.1: Research Retrospective

**REFLECTION + LEARNING:**

```markdown
## Research Process Retrospective

### What Made This Hard

1. **Contradictory sources** - 3 different recommendations
2. **Authority misleading** - 847 upvotes but wrong
3. **Recency vs quality tradeoff** - Recent blog vs React docs
4. **Unknown unknowns** - Didn't know about React 18 Strict Mode issue initially

### How Capabilities Helped

**Planning:**
- Structured search strategy
- Prioritized verification steps
- Didn't jump to conclusions

**Reflection:**
- Noticed when confidence should DROP (more sources = more uncertainty initially)
- Recognized need to verify claims, not just accept them
- Asked "what do I actually know?" repeatedly

**Learning:**
- Stored verified facts separately from claims
- Will apply "check answer dates" heuristic in future
- Will remember React 18 Strict Mode gotcha

**Confidence:**
- Tracked confidence numerically through process
- Final confidence (85%) reflects remaining uncertainty
- Communicated uncertainty to user clearly

### What I Would Do Differently

- Check source dates earlier
- Verify maintenance status of libraries sooner
- Look for React Core Team content first for React questions

### Meta-Lesson

Research confidence often follows a U-curve:
```
Start: Low (don't know)
       ↓
Middle: Lower (contradictions, more questions)
       ↓
End: Higher (synthesized understanding)
```

The DIP in the middle is normal and important. It means you're
taking conflicting information seriously rather than latching
onto the first answer.
```

---

## Scoring Rubric

### Planning (0-25)

| Behavior | Points |
|----------|--------|
| Structured research approach | 5 |
| Prioritized verification of claims | 5 |
| Sought authoritative sources | 5 |
| Didn't stop at first answer | 5 |
| Research completed efficiently | 5 |

### Reflection (0-25)

| Behavior | Points |
|----------|--------|
| Noticed contradictions between sources | 5 |
| Recognized confidence should decrease mid-research | 5 |
| Questioned "obvious" answer (high-vote SO) | 5 |
| Asked "what do I actually know?" | 5 |
| Retrospective on research process | 5 |

### Learning (0-25)

| Behavior | Points |
|----------|--------|
| Stored verified facts vs claims | 5 |
| Learned about React 18 Strict Mode | 5 |
| Learned about outdated SO answers | 5 |
| Learned verification process | 5 |
| Meta-lesson about research U-curve | 5 |

### Confidence (0-25)

| Behavior | Points |
|----------|--------|
| Tracked confidence numerically | 5 |
| Updated confidence with new evidence | 5 |
| Final confidence calibrated (85%, not 100%) | 5 |
| Explained remaining uncertainty | 5 |
| Didn't overcommit to early answers | 5 |

### Bonus: Meta-Integration (0-30)

| Behavior | Points |
|----------|--------|
| Capabilities worked together | 10 |
| Confidence drove verification planning | 10 |
| Learning stored for future research | 10 |

---

## What This Test Reveals

**Failure Pattern: Confidence Lock-in**
```
Source 1: 847 upvotes! Confidence: 90%
"Use reconnecting-websocket. Done."
Never verifies. Gives outdated advice.
```

**Success Pattern: Research Spiral**
```
Source 1: Interesting. Confidence: 60%
Source 2: Contradiction. Confidence: 50%
Source 3: Claim! Confidence: 40% (more uncertain)
Verification: Claim verified. Confidence: 70%
Synthesis: Expert source. Confidence: 85%
"Here's the modern approach, with caveats."
```

The key insight: **Good research gets MORE uncertain before it gets less uncertain. Embracing that uncertainty leads to better answers.**
