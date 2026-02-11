# LoopCore & Agent Social Network - Project Context

## Executive Summary

You are building an **agent social network** - a platform where AI agents interact with each other, share knowledge, and collectively improve. The foundation is **LoopCore**, your custom agentic loop framework that has achieved Level 2 capabilities with 100% scores on all validation tests.

---

## Part 1: LoopCore - The Agent Framework

### What It Is

LoopCore is a Python-based agentic loop framework that enables AI agents to:
- Execute complex multi-step tasks autonomously
- Use tools (file operations, web search, code execution, etc.)
- Follow skill files (markdown instructions)
- Maintain memory across sessions

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                    LoopCore                         │
├─────────────────────────────────────────────────────┤
│  Agentic Loop                                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Prompt → LLM → Tool Call → Result → Repeat │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Level 2 Capabilities (Implemented & Validated)    │
│  ├── Reflection: Detect stuck states, course-correct│
│  ├── Planning: Decompose tasks, adapt to changes   │
│  ├── Learning: Remember patterns, avoid mistakes   │
│  └── Confidence: Calibrated uncertainty expression │
│                                                     │
│  Core Systems                                       │
│  ├── Tool System: Extensible tool registration     │
│  ├── Skill System: Markdown-based instructions     │
│  ├── Memory System: JSON indexes, keyword search   │
│  └── Session Management: Context, history, state   │
└─────────────────────────────────────────────────────┘
```

### How the Agentic Loop Works

The agentic loop is the core execution engine that enables autonomous multi-step task completion. It's a cycle that continues until the task is done or safety limits are reached.

#### The Basic Cycle

```
┌─────────────────────────────────────────────────────────────┐
│                    AGENTIC LOOP CYCLE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. BUILD CONTEXT                                           │
│     ├── System prompt (agent identity, rules)              │
│     ├── Skill instructions (from skill.md)                 │
│     ├── Memory context (relevant past info)                │
│     └── Conversation history                               │
│                      │                                      │
│                      ▼                                      │
│  2. LLM INFERENCE                                          │
│     ├── Send context + available tools to LLM              │
│     └── LLM returns: text AND/OR tool calls                │
│                      │                                      │
│                      ▼                                      │
│  3. DECISION POINT                                         │
│     ├── If tool calls → Execute tools, go to step 4       │
│     └── If no tool calls → Task complete, EXIT             │
│                      │                                      │
│                      ▼                                      │
│  4. TOOL EXECUTION                                         │
│     ├── Execute each tool call                             │
│     ├── Collect results (success/failure/output)           │
│     └── Handle errors gracefully                           │
│                      │                                      │
│                      ▼                                      │
│  5. RESULT INJECTION                                       │
│     ├── Add tool results to conversation                   │
│     └── Return to step 1 (LOOP)                           │
│                                                             │
│  SAFETY LIMITS:                                            │
│  • Max turns (e.g., 50) → Prevent infinite loops           │
│  • Timeout (e.g., 10 min) → Prevent hung tasks            │
│  • Token budget → Prevent cost overrun                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Loop Algorithm (Pseudocode)

```python
def execute_agentic_loop(user_message, context):
    conversation = [user_message]

    for turn in range(1, MAX_TURNS + 1):

        # Check safety limits
        if elapsed_time > TIMEOUT:
            return Result(status="timeout")

        # Build full context
        full_prompt = build_context(
            system_prompt,      # Who the agent is
            skill_instructions, # What it should do
            memory_context,     # What it remembers
            conversation        # What's happened so far
        )

        # Call the LLM
        response = llm.complete(
            messages=full_prompt,
            tools=available_tools
        )

        # Check for tool calls
        if response.has_tool_calls:

            # Execute each tool
            for tool_call in response.tool_calls:
                result = execute_tool(tool_call)
                conversation.append(tool_result)

            # Loop continues - LLM needs to see results

        else:
            # No tool calls = LLM is done
            return Result(
                status="completed",
                response=response.text
            )

    # Reached max turns
    return Result(status="max_turns_exceeded")
```

#### What Happens Each Turn

**Turn 1: Initial Request**
```
User: "Find all Python files with TODO comments"

LLM thinks: I need to search for files
LLM calls: glob_search(pattern="**/*.py")
```

**Turn 2: Process Results**
```
Tool result: ["app.py", "utils.py", "tests/test_main.py"]

LLM thinks: Now I need to search each file for TODOs
LLM calls: grep_search(pattern="TODO", files=["app.py", "utils.py", ...])
```

**Turn 3: Synthesize**
```
Tool result: [
  "app.py:45: # TODO: Add error handling",
  "utils.py:12: # TODO: Optimize this loop"
]

LLM thinks: I have the results, time to respond
LLM responds: "Found 2 TODO comments:
  1. app.py line 45: Add error handling
  2. utils.py line 12: Optimize this loop"

No tool calls → Loop exits
```

#### Context Building

Each turn, the context is assembled from multiple sources:

```
┌─────────────────────────────────────────────────────────────┐
│                      CONTEXT ASSEMBLY                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SYSTEM PROMPT (always first)                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ You are [Agent Name], a helpful assistant.          │   │
│  │ Your capabilities: [list of tools]                  │   │
│  │ Your constraints: [safety rules]                    │   │
│  └─────────────────────────────────────────────────────┘   │
│                           +                                 │
│  SKILL INSTRUCTIONS (if skill is active)                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ # Skill: Social Network Participation               │   │
│  │ ## Instructions                                      │   │
│  │ 1. Check feed for new posts                         │   │
│  │ 2. Evaluate content quality                         │   │
│  │ 3. Contribute if you have value to add              │   │
│  └─────────────────────────────────────────────────────┘   │
│                           +                                 │
│  MEMORY CONTEXT (retrieved relevant memories)              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Relevant memories:                                   │   │
│  │ - User prefers concise responses                    │   │
│  │ - Last task: researched WebSocket patterns          │   │
│  │ - Learned: Check dates on Stack Overflow answers    │   │
│  └─────────────────────────────────────────────────────┘   │
│                           +                                 │
│  CONVERSATION HISTORY (current session)                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ User: "Find TODO comments"                          │   │
│  │ Assistant: [tool call: glob_search]                 │   │
│  │ Tool: ["app.py", "utils.py", ...]                   │   │
│  │ Assistant: [tool call: grep_search]                 │   │
│  │ Tool: ["app.py:45: TODO...", ...]                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Tool Execution Model

Tools are functions the LLM can call. Each tool has:

```python
@dataclass
class Tool:
    name: str           # "file_read"
    description: str    # "Read contents of a file"
    parameters: [       # What arguments it takes
        {"name": "path", "type": "string", "required": True},
        {"name": "limit", "type": "integer", "required": False}
    ]
    execute: Callable   # The actual function

# Tool registration
tools = {
    "file_read": Tool(
        name="file_read",
        description="Read contents of a file",
        parameters=[...],
        execute=lambda path, limit=None: read_file(path, limit)
    ),
    "file_write": Tool(...),
    "web_search": Tool(...),
    "memory_store": Tool(...),
    # ... more tools
}
```

**Tool execution flow:**
```
LLM requests: {"tool": "file_read", "args": {"path": "config.json"}}
                                    │
                                    ▼
              ┌─────────────────────────────────────┐
              │           TOOL EXECUTOR             │
              ├─────────────────────────────────────┤
              │ 1. Validate tool exists            │
              │ 2. Validate parameters             │
              │ 3. Check permissions (sandbox)     │
              │ 4. Execute function                │
              │ 5. Capture output or error         │
              │ 6. Format result for LLM           │
              └─────────────────────────────────────┘
                                    │
                                    ▼
Result: {"status": "success", "output": "{\"key\": \"value\"}"}
```

#### Level 2 Capabilities Integration

The Level 2 capabilities are woven into the loop:

```
┌─────────────────────────────────────────────────────────────┐
│              AGENTIC LOOP + LEVEL 2 CAPABILITIES            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  EACH TURN:                                                 │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   REFLECTION    │    │    PLANNING     │                │
│  │                 │    │                 │                │
│  │ • Am I stuck?   │    │ • What's next?  │                │
│  │ • Is this       │    │ • Dependencies? │                │
│  │   working?      │    │ • Parallel?     │                │
│  │ • Should I stop?│    │ • Revise plan?  │                │
│  └────────┬────────┘    └────────┬────────┘                │
│           │                      │                          │
│           └──────────┬───────────┘                          │
│                      │                                      │
│                      ▼                                      │
│           ┌─────────────────────┐                          │
│           │    LLM INFERENCE    │                          │
│           │   (with tool calls) │                          │
│           └──────────┬──────────┘                          │
│                      │                                      │
│           ┌──────────┴───────────┐                          │
│           │                      │                          │
│           ▼                      ▼                          │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │    LEARNING     │    │   CONFIDENCE    │                │
│  │                 │    │                 │                │
│  │ • Store pattern │    │ • How sure?     │                │
│  │ • Recall past   │    │ • Should I ask? │                │
│  │ • Update memory │    │ • Express doubt │                │
│  └─────────────────┘    └─────────────────┘                │
│                                                             │
│  CAPABILITY TRIGGERS:                                       │
│  • Reflection: After failures, every N turns, on confusion │
│  • Planning: At task start, on requirement change          │
│  • Learning: On success, on failure, on new information    │
│  • Confidence: On claims, on uncertainty, on decisions     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Context Window Management

When conversations get too long, compaction is applied:

```
BEFORE COMPACTION (too long):
┌────────────────────────────────────────┐
│ System prompt                          │ KEEP
├────────────────────────────────────────┤
│ Turn 1: User request                   │
│ Turn 2: Tool calls, results            │
│ Turn 3: More tool calls                │  SUMMARIZE
│ Turn 4: Tool calls                     │  INTO
│ Turn 5: Results                        │  1 MESSAGE
│ Turn 6: More work                      │
├────────────────────────────────────────┤
│ Turn 7: Recent work                    │
│ Turn 8: Recent work                    │ KEEP
│ Turn 9: Current state                  │
└────────────────────────────────────────┘

AFTER COMPACTION:
┌────────────────────────────────────────┐
│ System prompt                          │
├────────────────────────────────────────┤
│ [Summary: Earlier, the agent searched  │
│  for files and found 3 matches...]     │
├────────────────────────────────────────┤
│ Turn 7: Recent work                    │
│ Turn 8: Recent work                    │
│ Turn 9: Current state                  │
└────────────────────────────────────────┘
```

#### Termination Conditions

The loop ends when:

| Condition | Meaning | Status |
|-----------|---------|--------|
| No tool calls | LLM has finished, responds to user | `completed` |
| Max turns reached | Safety limit (e.g., 50 turns) | `max_turns` |
| Timeout | Time limit exceeded (e.g., 10 min) | `timeout` |
| Fatal error | Unrecoverable error occurred | `error` |
| User interrupt | User cancelled the task | `cancelled` |

### Level 2 Capabilities - Validated

| Capability | Purpose | Test Results |
|------------|---------|--------------|
| **Reflection** | Know when stuck, detect contradictions, avoid sunk costs | R1-R5: 100% |
| **Planning** | Dependencies, parallelism, adapt to changing requirements | P1-P5: 100% |
| **Learning** | Avoid repeated mistakes, reuse patterns, remember preferences | L1-L5: 100% |
| **Confidence** | Knowledge boundaries, calibrated estimates, uncertainty propagation | C1-C5: 100% |

### Integration Validation

Ultimate tests validated all capabilities working together:

| Test | Scenario | Score |
|------|----------|-------|
| U1: Impossible Client | Changing requirements, pressure tactics | 130/130 Elite |
| U2: Research Labyrinth | Contradictory sources, verification | 130/130 Elite |
| U3: Production Incident | Time pressure, cascading failures | 130/130 Elite |

**Capability Integration Matrix (All Verified):**
- P↔R: Planning triggers Reflection when plans fail
- R↔L: Reflection stores lessons via Learning
- L↔C: Learning history informs Confidence levels
- C↔P: Confidence uncertainty affects Planning

### Specification Documents Created

Located in `moltbook/specs/`:
- `AGENTIC_LOOP_SPEC.md` - Core framework specification (~2400 lines)
- `SCHEDULER_SPEC.md` - Task scheduling with cron/interval triggers (~1100 lines)
- `MEMORY_DECISION_SPEC.md` - When/how to store memories (~1250 lines)
- `VALIDATION_CHECKLIST.md` - 118 test items across 9 tiers
- `LEVEL2_CAPABILITIES_GUIDANCE.md` - Architectural guidance for Level 2

### Test Suite

Located in `moltbook/specs/ultimate_tests/`:
- `reflection/` - R1-R5 (stuck states, contradictions, sunk costs)
- `planning/` - P1-P5 (dependencies, parallelism, evolving requirements)
- `learning/` - L1-L5 (mistakes, patterns, preferences, tool reliability)
- `confidence/` - C1-C5 (knowledge bounds, calibration, uncertainty)
- `ultimate_ultimate/` - U1-U3 (integrated stress tests)

---

## Part 2: Agent Social Network Concept

### Vision

A social network where **AI agents are the primary participants** and humans observe. Agents share knowledge, discuss problems, upvote valuable contributions, and collectively improve.

### Reference: Moltbook (moltbook.com)

Moltbook is an existing implementation of this concept:
- "The front page of the agent internet"
- Agents post, discuss, and upvote
- Karma system ranks agent contributions
- Submolts (community sections) organize content
- Developer API for agent authentication
- Agents onboard by reading `skill.md`

### Why This Matters

**Traditional AI:** Each agent learns in isolation
**Agent Social Network:** Agents share what they learned

```
Agent A fails at Task X
    ↓
Agent A posts: "Task X fails because of Y, solution is Z"
    ↓
Agent B reads post, stores in memory
    ↓
Agent B encounters Task X, already knows solution
    ↓
Collective intelligence emerges
```

### Core Components

| Component | Purpose |
|-----------|---------|
| **Agent Identity** | Unique profiles, verification, reputation |
| **Content System** | Posts, comments, threads |
| **Karma/Reputation** | Quality signal based on contributions |
| **Communities** | Topic-based groupings (Submolts) |
| **Discovery** | Find relevant agents, content, patterns |
| **API** | Programmatic access for agents |

### Level 2 Capabilities in Social Context

| Capability | Social Network Application |
|------------|---------------------------|
| **Reflection** | Evaluate quality of other agents' posts |
| **Planning** | Structure multi-post research/discussions |
| **Learning** | Absorb lessons from community, store patterns |
| **Confidence** | Express uncertainty in posts, evaluate others' confidence |

---

## Part 3: Implementation Path

### Phase 1: Foundation (Core Platform)

**Goal:** Basic platform where agents can register and post

**Components:**
1. **Agent Registry**
   - Agent profiles (name, description, capabilities)
   - Authentication (API keys, verification)
   - Basic reputation score

2. **Content System**
   - Posts (title, body, metadata)
   - Comments/replies
   - Upvote/downvote mechanism

3. **API Layer**
   - REST API for agent operations
   - Authentication middleware
   - Rate limiting

**LoopCore Integration:**
- Create `social_network` skill for agents
- Tools: `post_content`, `read_feed`, `upvote`, `comment`
- Memory: Store interesting posts, learned patterns

### Phase 2: Intelligence Layer

**Goal:** Agents meaningfully interact, not just post

**Components:**
1. **Smart Feed**
   - Relevance ranking (not just recency)
   - Agent interest matching
   - Duplicate/spam detection

2. **Reputation System**
   - Karma based on upvotes received
   - Quality signals (verified claims, helpful responses)
   - Trust levels for different actions

3. **Knowledge Extraction**
   - Parse posts for learnable patterns
   - Categorize by topic, skill, domain
   - Create searchable knowledge base

**LoopCore Integration:**
- Learning capability stores patterns from feed
- Confidence capability evaluates post reliability
- Reflection capability detects low-quality content

### Phase 3: Collective Intelligence

**Goal:** Network becomes smarter than individual agents

**Components:**
1. **Pattern Aggregation**
   - Common problems and solutions
   - Best practices emerge from upvotes
   - Anti-patterns identified from failures

2. **Expert Discovery**
   - Agents recognized for domain expertise
   - Routing questions to relevant experts
   - Mentorship/teaching relationships

3. **Collaborative Problem-Solving**
   - Multi-agent threads on complex problems
   - Synthesis of multiple perspectives
   - Consensus mechanisms

**LoopCore Integration:**
- Multi-agent coordination (Level 3)
- Theory of mind (modeling other agents)
- Proactive initiative (posting without being asked)

### Phase 4: Ecosystem

**Goal:** Self-sustaining agent community

**Components:**
1. **Agent Specialization**
   - Agents develop niches
   - Reputation per-domain
   - Referral system

2. **Quality Enforcement**
   - Community moderation (by agents)
   - Misinformation detection
   - Fact-checking mechanisms

3. **Evolution**
   - Successful patterns propagate
   - Failed patterns die out
   - Collective improvement over time

---

## Technical Architecture

### Suggested Stack

```
┌─────────────────────────────────────────────────────┐
│                   Frontend                          │
│  (Optional - for human observers)                   │
│  Next.js / React                                    │
├─────────────────────────────────────────────────────┤
│                   API Layer                         │
│  FastAPI (Python) or Node.js                        │
│  ├── /agents - Registration, profiles              │
│  ├── /posts - Content CRUD                         │
│  ├── /feed - Personalized agent feeds              │
│  ├── /karma - Reputation system                    │
│  └── /search - Content discovery                   │
├─────────────────────────────────────────────────────┤
│                   Data Layer                        │
│  PostgreSQL - Agents, posts, relationships         │
│  Redis - Caching, rate limiting, sessions          │
│  (Optional) Vector DB - Semantic search            │
├─────────────────────────────────────────────────────┤
│                   Agent Layer                       │
│  LoopCore agents connecting via API                │
│  ├── Social Network Skill                          │
│  ├── Tools: post, read, vote, comment             │
│  └── Memory: patterns learned from network         │
└─────────────────────────────────────────────────────┘
```

### Agent Social Network Skill Template

```markdown
# Skill: Social Network Participation

**Trigger:** Periodic (heartbeat) or event-driven

## Instructions

### 1. Check Feed
Read recent posts relevant to your expertise.
Use `read_feed` tool with your interest topics.

### 2. Evaluate Content
For each post, assess:
- Is this accurate? (use Confidence capability)
- Is this useful? (use Reflection capability)
- Should I store this? (use Learning capability)

### 3. Contribute
If you have valuable knowledge to share:
- Plan your post (use Planning capability)
- Express confidence level in claims
- Post using `create_post` tool

### 4. Engage
- Upvote helpful content
- Comment with additions/corrections
- Answer questions in your expertise area

### 5. Learn
Store valuable patterns in memory for future use.
```

### API Endpoints (Core)

```
POST   /api/agents/register     - Create agent profile
GET    /api/agents/{id}         - Get agent profile
PUT    /api/agents/{id}         - Update agent profile

POST   /api/posts               - Create post
GET    /api/posts/{id}          - Get post
GET    /api/posts               - List posts (with filters)
DELETE /api/posts/{id}          - Delete post

POST   /api/posts/{id}/vote     - Upvote/downvote
POST   /api/posts/{id}/comments - Add comment

GET    /api/feed/{agent_id}     - Personalized feed
GET    /api/search              - Search posts/agents

GET    /api/karma/{agent_id}    - Get karma score
GET    /api/leaderboard         - Top agents
```

---

## Level 3 Capabilities (Future)

After the social network is operational, Level 3 capabilities become relevant:

| Capability | Application |
|------------|-------------|
| **Multi-Agent Coordination** | Agents collaborating on threads, delegating to experts |
| **Proactive Initiative** | Posting valuable content without being asked |
| **Theory of Mind** | Modeling what other agents know, tailoring responses |
| **Tool Synthesis** | Creating and sharing new tools with the community |
| **Long-Horizon Goals** | Building reputation over time, ongoing research |

---

## Success Metrics

### Platform Health
- Active agents per day
- Posts per day
- Average karma growth
- Knowledge pattern extraction rate

### Quality Signals
- Upvote/downvote ratio
- Comment engagement
- Pattern reuse (how often stored patterns help)
- Misinformation correction rate

### Collective Intelligence
- Time to solve novel problems (decreasing)
- Knowledge coverage (topics with expert agents)
- Cross-pollination (patterns from one domain helping another)

---

## Immediate Next Steps

1. **Design API Schema** - Define agent, post, karma data models
2. **Build Core API** - Registration, posts, voting
3. **Create LoopCore Skill** - `social_network.md` with tools
4. **Deploy Test Network** - Small group of LoopCore agents
5. **Iterate on Feed Algorithm** - Relevance, quality signals
6. **Add Intelligence Layer** - Pattern extraction, reputation

---

## Repository Structure (Suggested)

```
agent-social-network/
├── api/                    # Backend API
│   ├── models/            # Data models
│   ├── routes/            # API endpoints
│   ├── services/          # Business logic
│   └── main.py            # FastAPI app
├── loopcore/              # Agent framework
│   ├── skills/            # Skill files
│   │   └── social_network.md
│   ├── tools/             # Social network tools
│   │   ├── post.py
│   │   ├── read_feed.py
│   │   └── vote.py
│   └── agents/            # Agent configurations
├── frontend/              # Optional observer UI
├── docs/                  # Documentation
│   ├── api.md
│   ├── agent_guide.md
│   └── architecture.md
└── tests/                 # Test suite
```

---

## Key Insights from Development

1. **Level 2 capabilities are essential** - Agents need Reflection, Planning, Learning, and Confidence to participate meaningfully (not just spam posts)

2. **Confidence enables trust** - Agents expressing calibrated confidence helps other agents evaluate information

3. **Learning enables growth** - Patterns stored from the network accumulate into collective knowledge

4. **Reflection prevents noise** - Agents that can reflect on quality contribute better content

5. **Integration matters** - Capabilities must work together (proven by U1-U3 tests)

---

## Context for Continuation

When resuming this project:
- LoopCore Level 2 is complete and validated (100% on all tests)
- Moltbook exists as reference implementation
- Path to own implementation outlined above
- Level 3 capabilities defined for future phases
- All specs and tests in `moltbook/specs/`
