# 10X Improvements for loop_core: Lessons from openclaw/pi-mono

> Analysis of capabilities from the pi-mono/openclaw TypeScript agentic framework that could dramatically improve loop_core's Python agentic framework.

---

## 1. Hierarchical Sub-Agent Orchestration (THE biggest gap)

loop_core agents today are **isolated peers** — they coordinate only through shared files, feeds, and DMs. pi-mono has a full parent-child agent tree with:

- **`sessions_spawn`** — fire-and-forget async dispatch with depth limits (`maxSpawnDepth`), fan-out caps (`maxChildrenPerAgent: 5`), and automatic session key namespacing
- **Mid-execution steering** — a parent can abort a child's current run and re-inject a new message into its existing context, preserving the child's prior work
- **Cascade kill** — recursive tree walk that terminates an entire sub-agent hierarchy
- **Automatic result announcement** — when a child finishes, its output is injected into the parent's session as a system message; deferred if grandchildren are still running
- **Depth-based tool policies** — leaf agents can't spawn further children, can't access memory tools; orchestrator agents get the full toolset
- **Dead-parent bubble-up** — if a parent is gone, results walk up to the grandparent

**What to build:** A `SubAgentManager` that lets loop_core agents spawn child loops (same two-phase atomic design), with depth limits, tool deny-lists by depth, cascade lifecycle management, and async result injection into the parent's `AtomicState.pending_actions`.

---

## 2. Streaming Architecture

loop_core runs synchronously — the caller gets a `LoopResult` only after the entire run completes. pi-mono streams **every event** in real-time:

- **`AssistantMessageEventStream`** — a generic async-iterable push queue (`EventStream<T, R>`) that streams 12+ event types: `text_delta`, `thinking_delta`, `toolcall_start/delta/end`, `tool_execution_start/update/end`, `message_start/end`, `turn_start/end`, `agent_end`
- **Tool partial results** — tools call an `onUpdate` callback to stream progress mid-execution (e.g., a long HTTP request streaming chunks)
- **Multi-consumer subscribe pattern** — `agent.subscribe(fn)` with `Set<listener>` and unsubscribe callbacks; multiple consumers (web UI, Slack, RPC, TUI) all subscribe independently
- **Channel-agnostic delivery** — the same event stream feeds Slack (Socket Mode), web (Lit components), RPC (JSON stdout), and CLI (terminal re-render)

**What to build:** Replace the synchronous `execute()` return with an `AgenticStream` that yields typed events per-phase. Add `onUpdate` callbacks to tools. Add a `subscribe()` pattern to `AgenticLoop` so the runtime, API, and WebSocket layers can all consume the same stream.

---

## 3. Multi-Provider LLM Abstraction

loop_core hardcodes Anthropic (and optionally OpenAI) with raw API calls. pi-mono supports **22 providers** through a clean abstraction:

- **`Model<TApi>` generic** — every model carries its `api` protocol type, `provider`, `baseUrl`, `cost`, `contextWindow`, `reasoning` capability, and input modalities
- **API registry** — `Map<api_name, StreamFunction>` dispatches to the correct wire-protocol implementation (9 protocols: Anthropic Messages, OpenAI Completions, OpenAI Responses, Bedrock Converse, Google Generative AI, Google Vertex, etc.)
- **`models.json` user config** — add any provider (local Ollama, custom proxies) with just a JSON file; supports shell command expansion for API keys (`$(cat ~/.key)`)
- **OpenAI Completions compat layer** — handles 15+ provider quirks (tool ID format, role naming, thinking block format, max_tokens field name) through a `compat` object per model
- **Fuzzy model resolution** — `claude-opus-4-6:high` parses into model + thinking level; glob patterns and substring matching for model selection

**What to build:** A `ModelRegistry` with a `Provider` protocol interface. Each provider implements `stream()` and `complete()`. Register providers at startup. Allow user-configurable `models.json` for custom endpoints. This would let loop_core agents use Claude for Phase 1 reasoning and a cheap local model for Phase 2 parameter generation — playing directly to the two-phase architecture's strength.

---

## 4. Model Failover with Circuit-Breaker Auth Profiles

loop_core has no failover. If the API call fails, the loop errors out. pi-mono has:

- **Ordered fallback chains** — config-driven list of `{provider, model}` candidates; on failure, tries the next; context overflow errors skip fallback (smaller models would fail harder)
- **Auth profile circuit breaker** — per-profile error tracking with exponential cooldown (`5^(N-1) * 60s`, max 1h); half-open probing when cooldown is near expiry; billing failures get separate longer cooldowns (`5h` base, `24h` max)
- **OAuth token rotation** — file-locked refresh with double-checked locking; 5-minute expiry buffer; fallback to parent agent's credential store for sub-agents
- **Transient HTTP retry** — single 2.5s retry for 502/503/5xx before re-running the entire fallback chain
- **Per-provider retry with server-delay parsing** — reads `Retry-After`, `x-ratelimit-reset`, body patterns like `"retryDelay": "34s"`, and falls back to exponential backoff

**What to build:** Wrap loop_core's `complete_json()` and `complete_with_tools()` in a `resilient_call()` that tries the primary model, detects error category (rate_limit / timeout / billing / auth / transient), applies backoff or cooldown, and falls through to configured fallback models. The circuit-breaker pattern is especially valuable for loop_core's autonomous heartbeat agents that run unattended for hours.

---

## 5. Extension / Plugin Architecture

loop_core tools are statically registered in `agent_manager.py`. pi-mono has a full plugin system:

- **Extension discovery** — scans `~/.pi/agent/extensions/`, `<project>/.pi/extensions/`, and `package.json` manifests with a `"pi"` field
- **Runtime loading via jiti** — TypeScript extensions loaded at runtime with virtual module bundling; no build step required
- **`ExtensionAPI`** — a rich API surface: `registerTool()`, `on("tool_call" | "tool_result" | "context" | "before_agent_start")`, `registerCommand()`, `registerKeybinding()`, custom TUI renderers
- **Tool middleware** — `tool_call` event can block execution; `tool_result` event can modify output; `context` event can modify the full message history before each LLM call; `before_agent_start` can replace the system prompt per-turn
- **Tool override** — register a tool with the same name as a built-in to fully replace it (enables access-control proxies, audit logging, sandboxing)
- **Hot reload** — `/reload` command reloads all extensions without restarting

**What to build:** A `PluginManager` that discovers Python files from `plugins/` directories, calls a `register(api)` factory function, and wires registered tools/hooks into the `ToolRegistry`. Add pre-execution and post-execution hook points so plugins can gate, audit, or modify tool calls. This is how loop_core could gain new integrations without touching core code.

---

## 6. Intelligent Context Compaction

loop_core's atomic design keeps context constant, but the `TurnExchange` intra-run history still grows. And session context for long-running agents accumulates. pi-mono has a sophisticated compaction system:

- **Threshold-based trigger** — fires when `contextTokens > contextWindow - reserveTokens` (default 16,384 reserve)
- **Recency-preserving cut** — walks backwards keeping the most recent ~20,000 tokens verbatim; everything before the cut point gets summarized
- **Split-turn handling** — if the cut falls mid-turn, generates a separate turn-prefix summary and merges it with the history summary
- **File operation tracking** — every compaction records which files were read/modified in the summarized region as XML metadata; carried forward across subsequent compactions so the cumulative file set is never lost
- **Iterative update** — if a prior compaction summary exists, the new summary merges with it rather than summarizing from scratch
- **Extension hook** — `session_before_compact` lets plugins provide custom compaction or cancel it entirely
- **Parallel summarization** — history and turn-prefix summaries are generated concurrently via `Promise.all()`

**What to build:** Even though the two-phase atomic design keeps Phase 1 constant, loop_core should add compaction for: (a) long session conversation histories that get loaded for identity context, (b) the `TurnExchange` journal when runs exceed ~30 turns, and (c) cross-heartbeat summaries when there are more than N prior heartbeats. The file-tracking metadata pattern is especially valuable — inject `## Files touched in prior runs` into the identity prompt.

---

## 7. Prompt Caching Strategy

loop_core sends a fresh prompt every turn with no caching. pi-mono strategically caches:

- **System prompt** — tagged with `cache_control: { type: "ephemeral" }` on every Anthropic request
- **Conversation history** — the last user message gets `cache_control`, caching everything up to the current turn
- **Configurable retention** — `"short"` (5-min), `"long"` (1-hour on api.anthropic.com), `"none"`
- **Provider-specific** — OpenAI uses `prompt_cache_key: sessionId` + `prompt_cache_retention: "24h"`; Bedrock uses `CachePointType`

**What to build:** In `complete_json()` and `complete_with_tools()`, add Anthropic `cache_control` breakpoints to: (a) the system/identity prompt block (stable across turns), and (b) the task description + AtomicState (changes per turn but has a large stable prefix). For a 30-turn run, this could cut input token costs by **80%+** since the identity/task/tool-catalog portion is identical every turn.

---

## 8. Tool Confirmation and Safety Gates

loop_core executes all tools unconditionally within the sandbox. pi-mono has layered safety:

- **Extension `tool_call` gate** — any extension can block any tool call with a reason; first `block: true` short-circuits
- **`tool_result` modification** — extensions can sanitize, redact, or transform tool output before it reaches the LLM
- **UI confirmation dialogs** — `ctx.ui.confirm(title, message)` presents approve/deny to the user before destructive operations
- **Tool override for access control** — the example extension overrides the `read` tool to block `.env`, secrets, SSH/AWS/GPG directories
- **Docker sandbox executor** — `SandboxConfig` routes bash/read/write through `docker exec` for full OS-level isolation

**What to build:** Add a `ToolPolicy` system to loop_core's `ToolRegistry`: define per-tool `requires_confirmation: bool`, `allowed_paths: [glob]`, `blocked_patterns: [regex]`, `max_calls_per_run: int`. Add pre/post execution hooks. For autonomous agents running on heartbeats, this is critical safety infrastructure — a runaway agent shouldn't be able to send unlimited emails or make unlimited API calls.

---

## 9. Thinking/Reasoning Level Control

loop_core uses a single model configuration for all phases. pi-mono has fine-grained reasoning control:

- **5 thinking levels** — `minimal` (1024 tokens), `low` (2048), `medium` (8192), `high` (16384), `xhigh` (model-specific max)
- **Per-task routing** — compaction always uses `reasoning: "high"`; normal turns use user-configured level
- **Adaptive thinking** — Anthropic's `{ type: "adaptive" }` mode lets Claude decide its own effort level per turn
- **Provider-specific mapping** — Anthropic uses budget tokens; OpenAI uses `reasoning_effort` string; Google uses `thinkingLevel` enum

**What to build:** Make loop_core's two-phase design thinking-aware. Phase 1 (reasoning/planning) should use `high` thinking. Phase 2 (parameter generation) should use `minimal` or `low`. Reflection should use `medium`. Planning should use `high`. This plays directly to loop_core's strength — each phase can independently optimize its reasoning budget.

---

## 10. Session Branching and Navigation

pi-mono stores session entries in a **tree structure** (each entry has `id` and `parentId`):

- Users can navigate to any point in the conversation tree and branch from there
- `getBranch()` returns the linear path from root to current leaf
- When navigating to a different branch, `generateBranchSummary()` summarizes the abandoned branch and injects it so the LLM knows what was explored
- This enables "what-if" exploration without losing prior work

**What to build:** For loop_core's planning system, add plan-branch support: when `replan()` fires, preserve the abandoned plan branch as a summary in the new plan's context. This prevents the agent from repeating failed approaches. The `PlanningManager` already tracks step history — adding branching metadata would let the LLM see "I tried approach X, it failed because Y, now trying Z."

---

## 11. Parse Error Resilience

loop_core's `complete_json()` expects well-formed JSON from the LLM. If the LLM returns malformed JSON, the turn fails. pi-mono has:

- **`partial-json` library** — parses incomplete JSON during streaming, always returns a valid object
- **Silent SSE skip** — malformed SSE lines are `continue`d, never crash the stream
- **Validation with coercion** — AJV with `coerceTypes: true` auto-converts `"5"` to `5`
- **Role-ordering conflict recovery** — detects corrupted session state and resets
- **Session corruption detection** — provider-specific regex patterns trigger transcript delete + fresh session

**What to build:** Wrap loop_core's `json.loads()` in a resilient parser that: (a) strips markdown code fences, (b) tries partial JSON parsing on failure, (c) attempts regex extraction of JSON from mixed text, (d) on total failure, injects the raw response as `error_context` and retries the turn instead of aborting the loop. For Phase 2 tool_use parsing, add type coercion (string-to-number, string-to-boolean).

---

## 12. Cost Tracking and Optimization

loop_core tracks tokens via HiveLoop but doesn't track cost. pi-mono has:

- **Per-message cost calculation** — `calculateCost(model, usage)` using per-model pricing ($/million tokens for input, output, cache read, cache write)
- **Cost attached to every response** — `usage.cost.total` on every `AssistantMessage`
- **Cost analysis tooling** — scripts that aggregate per-provider costs by day from session files
- **Auto-generated pricing** — fetched from OpenRouter and Vercel AI Gateway APIs, always current

**What to build:** Add a `CostTracker` to loop_core that: (a) records per-turn cost using model pricing, (b) tracks cumulative cost per run and per agent, (c) enforces per-run and per-agent cost budgets (critical for autonomous agents), (d) reports cost in `LoopResult` and HiveLoop telemetry. This directly enables cost-aware model routing — use the cheap model when `pending_actions` are simple, expensive model when they're complex.

---

## 13. Dynamic Tool Set Management

loop_core gives every agent every tool in its registry, every turn. pi-mono has:

- **`setActiveToolsByName()`** — dynamically change which tools the LLM sees mid-session
- **Extension-controlled tool sets** — extensions can read/write the active tool list, enabling context-dependent tool availability
- **System prompt rebuilds on tool change** — when active tools change, the system prompt is regenerated to only describe available tools
- **Persistent tool-group configs** — stored as session entries, restored across context switches

**What to build:** Add a `ToolPolicy` per plan step in loop_core's `PlanningManager`: each `PlanStep` declares `expected_tools`, and the loop activates only those tools for that step. This reduces Phase 1 tool catalog size (already compact, but fewer tools = less ambiguity for the LLM) and prevents the agent from using tools inappropriate to the current step (e.g., don't offer `email_send` during a data-gathering step).

---

## 14. Structured Error Sanitization

loop_core passes raw error messages to `error_context` in `AtomicState`. pi-mono has:

- **Error pattern classification** — 6 categories (rate_limit, overloaded, timeout, billing, auth, format) with regex patterns covering all major providers
- **User-facing sanitization** — raw API JSON, Cloudflare HTML error pages, and stack traces are all rewritten to friendly messages
- **`FailoverError` typed wrapper** — every error gets a `reason` field enabling programmatic handling

**What to build:** Add an `ErrorClassifier` to loop_core that categorizes tool errors and LLM errors into actionable types. Instead of dumping "HTTPError: 429 Too Many Requests" into `error_context`, inject "Rate limited — will retry in 30s" and trigger the appropriate recovery. The `LearningManager` already captures `ErrorPattern` — the classifier feeds it structured data instead of raw strings.

---

## 15. WebSocket / RPC Mode for Embedding

loop_core has a REST API (FastAPI). pi-mono additionally has:

- **RPC mode** — JSON lines over stdin/stdout; every `AgentSessionEvent` serialized as JSON; commands arrive as JSON on stdin. This enables embedding the agent in any host process (IDE, Electron app, another agent)
- **WebSocket streaming** — `api/ws.py` exists in loop_core but pi-mono's implementation is richer with the full event type system
- **Proxy stream** — browser clients that can't call LLMs directly get an SSE relay via `/api/stream`

**What to build:** Add an RPC mode to loop_core: read JSON commands from stdin, emit typed events to stdout. This would let loop_core agents be embedded in VS Code extensions, web apps, or other Python processes without the HTTP overhead. The existing WebSocket in `api/ws.py` should be enhanced to stream per-turn events (phase1_decision, tool_execution, phase2_result) instead of just final results.

---

## Priority Ranking (by 10X impact)

| Priority | Capability | Why It's 10X |
|---|---|---|
| **1** | Sub-agent orchestration | Turns single-agent into multi-agent; enables task decomposition, parallel work, and specialization |
| **2** | Model failover + circuit breaker | Autonomous agents can't afford to die on a 429; this is table-stakes for reliability |
| **3** | Multi-provider abstraction | Unlocks cost optimization (cheap Phase 2), local models, and provider independence |
| **4** | Prompt caching | 80%+ cost reduction on the two-phase atomic design's biggest weakness (repeated prompt overhead) |
| **5** | Streaming architecture | Enables real-time UIs, long-running task visibility, and interactive steering |
| **6** | Extension/plugin system | Enables community-contributed tools without forking core; biggest velocity multiplier |
| **7** | Thinking level control | Phase 1 high / Phase 2 minimal = better reasoning at lower cost |
| **8** | Tool safety gates | Critical for autonomous agents; prevents runaway behavior on heartbeat runs |
| **9** | Cost tracking + budgets | Enables cost-aware routing and prevents surprise bills from autonomous agents |
| **10** | Intelligent compaction | Enables arbitrarily long sessions and cross-heartbeat context growth |
| **11** | Parse error resilience | Prevents single malformed LLM response from killing an entire autonomous run |
| **12** | Dynamic tool sets | Less ambiguity per turn = fewer wrong tool selections = faster task completion |
| **13** | Session branching | Better replanning through explicit "what I already tried" context |
| **14** | Error classification | Feeds learning system structured data; enables smarter recovery |
| **15** | RPC/embedding mode | Opens loop_core to IDE and app embedding use cases |

---

## The 10X Combination

The combination of **#1 + #2 + #3 + #4** alone would transform loop_core from "a single autonomous agent" into "a self-healing swarm of specialized agents that costs 80% less to run."

- **Sub-agent orchestration** lets a planner agent decompose work and delegate to specialist agents
- **Model failover** means autonomous agents survive API outages and rate limits
- **Multi-provider abstraction** lets Phase 1 use Claude Opus for reasoning while Phase 2 uses a cheap local model for parameter generation
- **Prompt caching** eliminates the repeated token cost of the identity/task/tool-catalog block that's identical every turn

Together, these four capabilities transform loop_core from a reliable single-agent framework into a production-grade autonomous agent swarm.
