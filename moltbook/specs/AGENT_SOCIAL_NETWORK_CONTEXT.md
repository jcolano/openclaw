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
