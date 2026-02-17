# Agentic Framework Analysis: `loop_core` vs `openclaw`

## 1. loop_core — Python Agentic Loop Framework

### Architecture

`loop_core` is a **pure Python** agentic AI framework built around a novel **two-phase atomic execution loop**. It is designed for autonomous agents that run multi-step tasks using tools.

```
loop_core/
├── loop.py              # Core engine: AgenticLoop (two-phase execution)
├── agent.py             # Agent wrapper class
├── agent_manager.py     # Orchestration / lifecycle
├── runtime.py           # Background daemon + event queue
├── config/loader.py     # Configuration system
├── tools/               # 20+ tool implementations
├── skills/              # Markdown-based behavior files
├── memory/              # Session + long-term memory
├── reflection.py        # Self-evaluation
├── planning.py          # Task decomposition
├── learning.py          # Pattern capture + reuse
├── scheduler/           # Cron/interval task scheduling
├── api/                 # FastAPI REST + WebSocket server
└── cli/                 # Command-line interface
```

### Agent Design

Each agent is a self-contained unit with its own directory:

```
data/AGENTS/{agent_id}/
├── config.json      # Model, tools, prompts, limits
├── skills/          # Private skills (markdown behavior files)
├── memory/          # Sessions + long-term memory
├── credentials.json # External service credentials
├── tasks/           # Scheduled tasks
└── runs/            # Execution history
```

Agents are configured via `AgentConfig`: system prompt, model selection, tool whitelist, max turns (default 20), timeout (default 600s), and optional reflection/planning/learning overrides.

### Execution Model — The Two-Phase Atomic Loop

This is `loop_core`'s core innovation. Instead of a traditional "send full conversation history + all tool schemas" approach, it uses **two separate LLM calls per turn**:

**Phase 1 — Reasoning (Tool Selection):**

- LLM receives a compact prompt: the task, an `AtomicState` dict, tool catalog as **names + one-line hints only** (~350 tokens vs ~1,850 for full schemas), the last tool result, and plan context.
- LLM outputs structured JSON: `analysis`, `tool` name (or null), `intent` (what the tool should do), `state_update`, and `done` flag.
- No tool schemas are sent — just names and hints.

**Phase 2 — Parameter Generation:**

- LLM receives only the `intent` string from Phase 1, the `variables` dict, and **one single tool's full schema**.
- LLM outputs a native `tool_use` block with typed parameters.

**Why this matters:** Input token usage stays **roughly constant** regardless of how many turns have executed. There is no accumulating conversation history — just the compact `AtomicState`:

```python
@dataclass
class AtomicState:
    completed_steps: List[str]    # Capped at 20
    variables: Dict[str, Any]     # Key/value store, capped at 50
    pending_actions: List[str]    # Capped at 10
    current_step: int
    error_context: Optional[str]
```

The loop runs up to `max_turns` iterations. Exit conditions: `completed`, `timeout`, `max_turns`, `error`, `loop_detected`, `escalation_needed`, `cancelled`.

### Tool System

Tools inherit from `BaseTool` with `definition` (schema) and `execute(**kwargs)` methods. The `ToolRegistry` runs tools in a `ThreadPoolExecutor` (4 workers, 30s timeout, 100KB output cap). Tools include: file I/O, HTTP, scheduling, CRM, email, web search, image generation, data aggregation, workspace, and more. Tools support **credential pre-loading** to prevent LLM hallucination of API keys.

### Memory and State

Three layers:

1. **Session Memory** — conversation persistence in JSON files (max 20 completed sessions, 10MB each)
2. **Long-Term Memory** — topic-based fact store with `UserDirectiveHandler` ("remember X"), `TurnScanner`, `SessionEndReviewer`, and `MemoryDecay`
3. **Learning Store** — captured tool patterns, error patterns, and success patterns (mostly LLM-free stats)

Cross-heartbeat context is maintained via Haiku-condensed summaries injected into Phase 1 prompts.

### Orchestration

`AgentRuntime` runs a daemon thread iterating once/second over all active agents. Each agent has a **priority event queue** (HIGH=human messages, NORMAL=webhooks/tasks, LOW=heartbeats). Only one LLM call runs per agent at a time. Agents coordinate **indirectly** through shared workspace tools, feed posts, DM notifications, and `queue_followup_event` — there is no direct agent-to-agent invocation.

### Higher-Order Capabilities

- **Reflection**: Triggers after N consecutive failures (default 3). Decisions: `continue`, `adjust`, `pivot`, `terminate`, `escalate`.
- **Planning**: Auto-triggers on complex tasks (>10 words, sequence keywords). Creates LLM-generated step plans, supports auto-replan on block (max 3 replans).
- **Learning**: Captures tool usage patterns (no LLM call), error patterns, and optionally success patterns. Insights injected into system prompt.
- **Loop Detection**: Catches identical consecutive tool calls (>3 repeats) and repeating tool-call sequences.

---

## 2. openclaw — TypeScript Multi-Channel AI Gateway

### Architecture

`openclaw` is a **TypeScript/Node.js** application that serves as a multi-channel AI gateway. It bridges users across messaging platforms (Telegram, Discord, Slack, Signal, iMessage, WhatsApp, web) to AI models, with a built-in agent execution system.

```
openclaw/src/
├── agents/                    # Agent execution engine
│   ├── pi-embedded-runner/    # Core execution loop (retry, auth, compaction)
│   │   ├── run.ts             # Outer retry loop
│   │   └── run/attempt.ts     # Single execution attempt
│   ├── openclaw-tools.ts      # Tool assembly
│   ├── pi-tools.ts            # Tool composition pipeline
│   ├── subagent-spawn.ts      # Sub-agent spawning
│   ├── subagent-registry.ts   # Sub-agent lifecycle tracking
│   ├── subagent-depth.ts      # Depth limiting
│   ├── agent-scope.ts         # Agent config resolution
│   └── tools/                 # 18+ tool implementations
├── gateway/                   # WebSocket server + JSON-RPC
├── sessions/                  # Session key management
├── channels/                  # Channel adapters (Telegram, Discord, etc.)
└── providers/                 # Extended LLM provider configs
```

It relies heavily on `@mariozechner/pi-coding-agent` and `@mariozechner/pi-ai` as its underlying SDK for LLM streaming and session management.

### Agent Design

Agents are defined in a config file under `agents.list` with properties like: `name`, `workspace`, `model`, `skills`, `identity`, `heartbeat`, `subagents`, `sandbox`, and `tools`. Sessions are tracked via structured key strings: `agent:<agentId>:<type>:<id>` (e.g., `agent:myagent:subagent:<uuid>`).

Session state is persisted as **JSONL files** on disk containing the full message transcript. The SDK manages read/write with advisory file locks.

### Execution Model — Three-Layer Loop

**Layer 1 — Outer Retry Loop** (`run.ts`):

- Enqueues the run into session-specific and global lanes to prevent concurrent execution
- Resolves the model and auth profile
- **Auth profile rotation**: cycles through multiple API keys on rate-limit/auth failures
- Handles context overflow by triggering compaction and retrying
- Handles thinking-level downgrades and retries

**Layer 2 — Single Attempt** (`attempt.ts`):

- Builds the full tool set via `createOpenClawCodingTools()`
- Constructs the system prompt (workspace notes, skills, heartbeat, tool hints)
- Opens the session via `SessionManager.open()` + `createAgentSession()`
- Sanitizes history (validates turn ordering for Anthropic/Gemini, limits turns, repairs orphaned tool pairs)
- Calls `activeSession.prompt(effectivePrompt)` — the actual LLM call
- Subscribes to streaming events (message chunks, tool executions, lifecycle events)

**Layer 3 — Internal Agent Loop** (inside `pi-coding-agent` SDK):

- Streams the LLM response
- Detects tool call blocks, executes tool functions, feeds results back
- Continues until `stopReason !== "tool_calls"`

This is a **traditional conversation-accumulating approach** — the full message history grows with each turn, with compaction as a safety valve when context overflows.

### Tool System

Tools are assembled in a multi-step pipeline:

1. Base coding tools from the SDK (read, write, edit)
2. Exec tool (bash execution with optional sandbox)
3. OpenClaw-specific tools: browser, canvas, cron, message, TTS, gateway, web search/fetch, image, sub-agent management
4. Plugin tools from extensions
5. **Policy pipeline**: tools pass through 8+ layers of policy filters (profile, provider, global, agent, group, sandbox, subagent)
6. **Tool wrapping**: parameter normalization, hook injection, abort signal support

### Sub-Agent System

Sub-agents are a **first-class feature** with deep integration:

- **Spawning**: `sessions_spawn` tool creates child agent sessions with depth limiting (default max depth: 1, max 5 children per agent)
- **Registry**: Tracks all sub-agent runs with lifecycle events (start, end, error), persisted to disk, survives restarts
- **Management**: `subagents` tool lets the parent LLM list, kill (with cascade to descendants), or steer (interrupt + redirect) sub-agents
- **Depth tracking**: Computed from session key structure (`:subagent:` segment count) and stored in session metadata
- **Cross-agent authorization**: `allowAgents` whitelist controls which agents can spawn which

### Memory and State

- **Session transcripts**: Full message history in JSONL files, managed by the SDK's `SessionManager`
- **Context management**: Auto-compaction when context overflows, oversized tool result truncation, history turn limiting
- **Session write locks**: Advisory locks prevent corruption from concurrent access
- **No separate long-term memory system**: State lives in the session transcript; cross-session memory depends on skills/workspace files

### LLM Provider Support

Extensive multi-provider support:

- **Built-in**: Anthropic (Claude), OpenAI, Google/Gemini, AWS Bedrock
- **Extended**: Ollama (local), Together AI, HuggingFace, Venice, MiniMax, Cloudflare AI Gateway, GitHub Copilot, Qwen Portal
- **Custom**: Any provider with OpenAI-compatible API via config
- **Auth rotation**: Multi-account load balancing across API keys for high-throughput scenarios

---

## 3. Comparative Analysis

| Dimension | **loop_core** | **openclaw** |
|---|---|---|
| **Language** | Python | TypeScript (Node.js/Bun) |
| **Primary Purpose** | Autonomous agent framework | Multi-channel AI gateway with embedded agents |
| **Execution Model** | Two-phase atomic loop (constant token cost) | Traditional conversation-accumulating loop (growing context) |
| **Token Efficiency** | High — Phase 1 sees only state dict + tool hints; Phase 2 sees one schema | Lower — full history sent each turn; relies on compaction when overflow |
| **LLM Calls Per Turn** | 2 (Phase 1: reasoning, Phase 2: parameters) | 1 (single call with full tool schemas + history) |
| **State Representation** | Compact `AtomicState` dict (completed_steps, variables, pending_actions) | Full message transcript (JSONL) |
| **Tool Schema Delivery** | Phase 1: names only; Phase 2: single tool schema | All tool schemas sent every call |
| **Sub-Agent Model** | Indirect — agents coordinate via shared workspace, events, feeds | Direct — first-class sub-agent spawning with depth limits, registry, kill/steer |
| **Multi-Agent Coordination** | Event queue + workspace tools (loosely coupled) | Parent-child tree with lifecycle tracking (tightly coupled) |
| **Memory System** | 3-layer: session + long-term topics + learning store | Session transcripts only; no dedicated long-term memory |
| **Reflection/Planning** | Built-in: self-evaluation, task decomposition, auto-replan | Not built-in (delegated to LLM's native reasoning) |
| **Learning** | Pattern capture across executions (tool stats, error patterns) | None — each session starts fresh |
| **LLM Providers** | Anthropic only (via external `llm_client`) | 10+ providers (Anthropic, OpenAI, Gemini, Bedrock, Ollama, etc.) |
| **Channel Support** | API + CLI | Telegram, Discord, Slack, Signal, iMessage, WhatsApp, web, etc. |
| **Skill System** | Markdown behavior files injected into system prompt | Markdown-based skills (similar concept) |
| **Auth Management** | Per-agent credentials file | Auth profile rotation with multi-account load balancing |
| **Observability** | HiveLoop integration via `contextvars` | Cache tracing, payload logging, event subscriptions |
| **Scheduling** | Built-in cron/interval/event scheduler | Cron tool available to agents |
| **Loop Detection** | Explicit detector (consecutive repeats + pattern sequences) | Hook-based detection during tool wrapping |

### Key Architectural Differences

#### 1. Token Economics (Most Significant Difference)

`loop_core`'s two-phase atomic approach is fundamentally different from `openclaw`'s traditional approach. In `loop_core`, whether an agent is on turn 1 or turn 20, the input tokens are roughly the same because only the compact `AtomicState` and last tool result are sent. In `openclaw`, the context grows with every turn until compaction is triggered — a reactive rather than proactive strategy.

#### 2. Agent Autonomy vs. Gateway Connectivity

`loop_core` is designed for **autonomous background agents** — they run on heartbeat timers, have long-term memory, learn from past executions, and self-reflect. `openclaw` is designed for **interactive user-facing sessions** — it excels at connecting users across channels to AI models, with sub-agents as a delegation mechanism during a conversation.

#### 3. Multi-Agent Philosophy

`loop_core` uses **loose coupling**: agents are peers that communicate through a shared workspace (feeds, DMs, events). There is no hierarchy. `openclaw` uses **tight coupling**: a clear parent-child tree with depth limits, lifecycle tracking, cascade kills, and steering — the parent actively manages its children.

#### 4. Cognitive Architecture

`loop_core` has built-in **reflection** (self-evaluation on failure), **planning** (task decomposition with auto-replan), and **learning** (cross-execution pattern capture). These are explicit modules layered on top of the execution loop. `openclaw` relies entirely on the LLM's native reasoning capabilities — it provides no additional cognitive scaffolding beyond what the model itself does.

#### 5. Breadth vs. Depth

`openclaw` has much broader **integration surface**: 10+ LLM providers, 8+ messaging channels, plugin system, sandbox support, auth rotation. `loop_core` goes deeper on **agent intelligence**: memory decay, topic-based knowledge, learning stores, heartbeat summaries, and atomic state management.

### When to Use Which

- **loop_core**: Best for autonomous, long-running agents that need to manage state efficiently, learn over time, and operate with predictable token costs. Ideal for business automation, scheduled tasks, and agents that run continuously.

- **openclaw**: Best for interactive, user-facing AI applications that need multi-channel delivery, sub-agent delegation, broad LLM provider support, and real-time streaming. Ideal for chatbots, coding assistants, and multi-platform AI services.
