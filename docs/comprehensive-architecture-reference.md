# Comprehensive Architecture Reference: pi-mono, openclaw & loop_core

> Deep investigation of all three codebases — the pi-mono coding-agent foundation,
> the openclaw orchestration layer, and the loop_core Python agentic framework.

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Pi-Mono Coding-Agent (TypeScript Foundation)](#pi-mono-coding-agent)
   - [Core Architecture & Entry Points](#1-core-architecture--entry-points)
   - [Tool System](#2-tool-system)
   - [Session Management & Branching](#3-session-management--branching)
   - [Streaming & Event System](#4-streaming--event-system)
   - [Model / LLM Abstraction](#5-model--llm-abstraction)
   - [Extension System](#6-extension-system)
   - [Compaction & Context Optimization](#7-compaction--context-optimization)
   - [Modes & I/O Layers](#8-modes--io-layers)
   - [System Prompt & Context Injection](#9-system-prompt--context-injection)
3. [OpenClaw Orchestration Layer](#openclaw-orchestration-layer)
   - [Agent Definition & Configuration](#10-agent-definition--configuration)
   - [Session Management & Routing](#11-session-management--routing)
   - [Platform Tools](#12-platform-tools)
   - [Subagent Spawning & Lifecycle](#13-subagent-spawning--lifecycle)
   - [Memory System](#14-memory-system)
   - [Inter-Agent Communication](#15-inter-agent-communication)
   - [Agent Execution & Continuation](#16-agent-execution--continuation)
   - [Extension System (Embedded)](#17-extension-system-embedded)
4. [Loop_core Python Agentic Framework](#loop_core-python-agentic-framework)
   - [Two-Phase Atomic Execution Model](#18-two-phase-atomic-execution-model)
   - [Agent & Agent Manager](#19-agent--agent-manager)
   - [Tool System](#20-tool-system)
   - [Runtime / Daemon System](#21-runtime--daemon-system)
   - [Memory System](#22-memory-system-1)
   - [Reflection System](#23-reflection-system)
   - [Planning System](#24-planning-system)
   - [Learning System](#25-learning-system)
   - [Configuration System](#26-configuration-system)
   - [API Layer](#27-api-layer)
   - [Scheduler System](#28-scheduler-system)
5. [Cross-System Comparison](#cross-system-comparison)
6. [Architecture Diagrams](#architecture-diagrams)
7. [Critical Files Reference](#critical-files-reference)

---

## System Overview

The three systems form a layered stack:

```
┌──────────────────────────────────────────────────────────────────┐
│                    loop_core (Python)                             │
│   Autonomous agents with two-phase atomic execution,             │
│   reflection, planning, learning, heartbeat runtime              │
├──────────────────────────────────────────────────────────────────┤
│                    openclaw (TypeScript)                          │
│   Multi-agent orchestration, session routing, subagent spawn,    │
│   memory search, 20+ platform tools, extension system            │
├──────────────────────────────────────────────────────────────────┤
│                    pi-mono coding-agent (TypeScript)              │
│   Foundation: 7 core tools, session persistence, streaming,      │
│   model registry, extension API, TUI/RPC/print modes             │
└──────────────────────────────────────────────────────────────────┘
```

**pi-mono** provides the coding-agent primitives — file I/O tools, session persistence, model abstraction, streaming events, and an extension API.

**openclaw** builds on top of pi-mono, adding multi-agent orchestration — subagent spawning with depth limits, inter-agent messaging, semantic memory search, platform tools (browser, image, cron, web search), and session routing across channels (Discord, Slack, Telegram).

**loop_core** is an independent Python framework with a novel two-phase atomic execution model, a daemon runtime with priority event queues, and built-in reflection/planning/learning systems.

---

## Pi-Mono Coding-Agent

### 1. Core Architecture & Entry Points

**Package:** `pi-mono/packages/coding-agent/`

**Entry Points:**
- `src/main.ts` (line 537+): CLI orchestration
- `src/cli.ts`: CLI handler
- Package bin: `"pi": "dist/cli.js"` (package.json line 11)

**Primary Class — AgentSession** (`src/core/agent-session.ts`, ~2700 lines):
```typescript
class AgentSession {
  // Core state
  private _extensionRunner: ExtensionRunner
  private _eventListeners: Map<string, Set<Function>>
  private _toolRegistry: ToolRegistry
  private _compactionAbortController: AbortController
  private _retryAbortController: AbortController
  private _bashAbortController: AbortController

  // Key methods
  prompt(text: string, options: PromptOptions): Promise<void>
  cycleModel(): void
  setThinkingLevel(level: ThinkingLevel): void
  fork(entryId: string): Promise<{ selectedText: string; cancelled: boolean }>
  navigateTree(targetId: string, options?): Promise<{ selectedText: string; cancelled: boolean }>
  compact(options?: CompactOptions): Promise<CompactionResult | undefined>
  executeBash(command: string): Promise<BashResult>
}
```

**Agent Lifecycle Flow:**
```
CLI args parsing (parseArgs)
  → createAgentSession() [SDK factory]
  → AgentSession constructor
  → _buildRuntime() [initializes extension runner + tools]
  → InteractiveMode / PrintMode / RpcMode
```

**Dependency injection** via `AgentSessionConfig`:
- `agent`, `sessionManager`, `settingsManager`, `cwd`, `resourceLoader`, `modelRegistry`

---

### 2. Tool System

**Location:** `src/core/tools/`

**Tool Registry** (`src/core/tools/index.ts`):
```typescript
// Pre-built tool collections
codingTools  = [readTool, bashTool, editTool, writeTool]
readOnlyTools = [readTool, grepTool, findTool, lsTool]
allTools     = { read, bash, edit, write, grep, find, ls }
```

**Factory functions:**
- `createCodingTools(cwd, options?)` — Read + Bash + Edit + Write
- `createReadOnlyTools(cwd, options?)` — Read + Grep + Find + Ls
- `createAllTools(cwd, options?)` — All 7 tools

**Individual Tools:**

| Tool | File | Schema | Key Features |
|------|------|--------|-------------|
| **read** | `read.ts` | `{ path, offset?, limit? }` | Image resizing (2000×2000), line truncation, remote delegation via `ReadOperations` |
| **bash** | `bash.ts` | `{ command, timeout? }` | Process spawning, output streaming to temp files, SSH delegation via `BashOperations` |
| **edit** | `edit.ts` | `{ path, oldText, newText }` | Fuzzy text matching, unified diff generation, line-ending preservation |
| **write** | `write.ts` | `{ path, content }` | Create/overwrite files |
| **grep** | `grep.ts` | `{ pattern, path?, ... }` | Content search |
| **find** | `find.ts` | `{ pattern, path?, ... }` | File discovery |
| **ls** | `ls.ts` | `{ path? }` | Directory listing |

**Tool Wrapping with Extensions** (`src/core/extensions/wrapper.ts`, lines 33–118):
```typescript
wrapToolWithExtensions<T>(tool: AgentTool, runner: ExtensionRunner): AgentTool
```
Flow: Emit `tool_call` event (can block) → Execute tool → Emit `tool_result` event (can modify) → Return result.

**Output Limits:** `DEFAULT_MAX_LINES = 500`, `DEFAULT_MAX_BYTES = 512KB` (`tools/truncate.ts`)

---

### 3. Session Management & Branching

**SessionManager** (`src/core/session-manager.ts`):
- JSONL format (one entry per line) with tree navigation (`id`/`parentId`)
- Current version: `CURRENT_SESSION_VERSION = 3`

**Entry Types:**
- `SessionMessageEntry` — Messages
- `CompactionEntry` — Context summaries
- `BranchSummaryEntry` — Branch navigation markers
- `CustomEntry` — Extension-specific data
- `CustomMessageEntry` — Extension messages in LLM context
- `ThinkingLevelChangeEntry`, `ModelChangeEntry`, `LabelEntry`, `SessionInfoEntry`

**Fork System** (agent-session.ts lines 2407–2463):
```
User: /fork
  → Extension: session_before_fork (can cancel)
  → SessionManager.forkFrom(currentPath, newCwd)
    → New JSONL file with parentSession reference
  → Extension: session_fork
```

**Tree Navigation** (agent-session.ts lines 2475–2663):
```
User: /tree → select branch
  → Extension: session_before_tree (can cancel + summarize)
  → Optional: generateBranchSummary() → LLM call
  → SessionManager.branch(newLeafId) → Update leaf pointer
  → Extension: session_tree
  → Agent context reloads from new leaf
```

---

### 4. Streaming & Event System

**Event Bus** (`src/core/event-bus.ts`):
```typescript
interface EventBus {
  emit(channel: string, data: unknown): void
  on(channel: string, handler: (data: unknown) => void): () => void
}
```
Uses Node.js EventEmitter internally with async error handling.

**Agent Events** (from `@mariozechner/pi-agent-core`):
- `agent_start` / `agent_end`
- `turn_start` / `turn_end`
- `message_start` / `message_update` / `message_end`
- `tool_execution_start` / `tool_execution_update` / `tool_execution_end`

**Extended by AgentSession:**
- `auto_compaction_start` / `auto_compaction_end`
- `auto_retry_start` / `auto_retry_end`

**Extension Events** (`src/core/extensions/types.ts`):
1. **Lifecycle:** `agent_start`, `agent_end`, `session_start`, `session_shutdown`
2. **Tool:** `tool_call`, `tool_result` (can block/modify)
3. **Message:** `message_start`, `message_end`, `context_event`
4. **Input:** `input_event` (keyboard/paste interception)
5. **Session:** `session_before_fork`, `session_before_switch`, `session_before_tree`, `session_before_compact`

---

### 5. Model / LLM Abstraction

**Model Registry** (`src/core/model-registry.ts`):
- Built-in model discovery via `@mariozechner/pi-ai`
- Custom models via `models.json`
- Provider registration system
- API key resolution with `AuthStorage`

**Model Schema:**
```typescript
ModelDefinitionSchema = {
  id, name?, api?, reasoning?, input?, cost?,
  contextWindow?, maxTokens?, headers?, compat?
}
```

**Thinking Levels:**
```typescript
THINKING_LEVELS = ["off", "minimal", "low", "medium", "high"]
THINKING_LEVELS_WITH_XHIGH = [..., "xhigh"]
```

**Agent Loop** (`agent-loop.ts`):
```typescript
agentLoop(prompts, context, config, signal?, streamFn?): EventStream
agentLoopContinue(context, config, signal?, streamFn?): EventStream
```
Transforms context via optional `convertToLlm`, supports tool calling + validation, streaming via `EventStream`.

---

### 6. Extension System

**Extension Loading** (`src/core/extensions/loader.ts`):
- Uses `@mariozechner/jiti` (TypeScript runtime)
- Virtual modules for Bun binary support

**Extension API** (`src/core/extensions/types.ts`):
```typescript
// Registration
on(event: string, handler): void
registerTool(tool: ToolDefinition): void
registerCommand(command: RegisteredCommand): void
registerShortcut(shortcut: ExtensionShortcut): void
registerProvider(name: string, config: ProviderConfig): void

// Actions
sendMessage(message: string): Promise<void>
sendUserMessage(message: string): Promise<void>
appendEntry(type: string, data?: unknown): void
setActiveTools(names: string[]): void
setModel(provider: string, modelId: string): Promise<void>
setThinkingLevel(level: ThinkingLevel): void
```

**Extension UI Context:**
```typescript
interface ExtensionUIContext {
  select(title, options, opts?): Promise<string | undefined>
  confirm(title, message, opts?): Promise<boolean>
  input(title, placeholder?, opts?): Promise<string | undefined>
  notify(message, type?): void
  setStatus(key, text): void
  setWidget(key, content, options?): void
}
```

**Extension Runner** (`src/core/extensions/runner.ts`, 620+ lines):
- `emit()` — Generic event
- `emitToolCall()` — Pre-execution tool event (can block)
- `emitToolResult()` — Post-execution tool event (can modify)

**Package System** (`src/core/package-manager.ts`):
- Supports npm, git, local paths
- CLI: `pi install`, `pi remove`, `pi update`, `pi list`

---

### 7. Compaction & Context Optimization

**Location:** `src/core/compaction/compaction.ts`

**Settings:**
```typescript
interface CompactionSettings {
  enabled?: boolean       // default: true
  reserveTokens?: number  // default: 16384
  keepRecentTokens?: number // default: 20000
}
```

**Triggers:**
1. Auto-compaction on `agent_end` if context overflow
2. Manual via `/compact` slash command
3. Extension hook via `before_compact` event

**Branch Summarization:** Generates summary of abandoned branch before tree navigation; stored as `BranchSummaryEntry`.

---

### 8. Modes & I/O Layers

| Mode | File | Description |
|------|------|-------------|
| **Interactive** | `src/modes/interactive/interactive-mode.ts` | Full TUI with keyboard/mouse, streaming UI, component-based |
| **Print** | `src/modes/print-mode.ts` | Plain text output, single input → stream response |
| **RPC** | `src/modes/rpc/` | JSON-RPC over stdin/stdout for IDE integration |

**Key Interactive Components:**
- `AssistantMessageComponent` — Rendered assistant messages
- `BashExecutionComponent` — Bash command output
- `ToolExecutionComponent` — Tool call visualization
- `UserMessageComponent` — User input editor
- `ModelSelectorComponent` — Model selection UI
- `TreeSelectorComponent` — Session tree navigation

---

### 9. System Prompt & Context Injection

**System Prompt Builder** (`src/core/system-prompt.ts`):
```typescript
interface BuildSystemPromptOptions {
  customPrompt?: string        // Replaces default
  selectedTools?: string[]     // Tools to document
  appendSystemPrompt?: string  // Text to append
  cwd?: string
  contextFiles?: Array<{ path, content }>
  skills?: Skill[]
}
```

**Default prompt structure:** Tool descriptions → Tool usage guidelines → Project context files → Skills → Date/time → Working directory.

**Context Files Discovery:**
- Global: `~/.pi/agent/AGENTS.md`
- Per-project: `.pi/AGENTS.md` (ancestors up to root)
- Priority: Project > Ancestors > Global

**Skills:** YAML frontmatter files (`name` + `description`), max 64-char name, max 1024-char description.

---

## OpenClaw Orchestration Layer

### 10. Agent Definition & Configuration

**Location:** `openclaw/src/agents/agent-scope.ts`

**Core Interface — `ResolvedAgentConfig`** (lines 17–31):
```typescript
ResolvedAgentConfig {
  name: string                     // Display name
  workspace: string                // Workspace directory
  agentDir: string                 // Agent directory
  model: string | {                // Model config
    primary: string
    fallbacks?: string[]
  }
  skills: string[]                 // Skill filter
  memorySearch: ResolvedMemorySearchConfig
  humanDelay, heartbeat            // Timing configs
  identity, groupChat              // Feature configs
  subagents: {
    maxSpawnDepth?: number         // Max nesting depth (default 1)
    maxChildrenPerAgent?: number   // Max active children (default 5)
    allowAgents?: string[]         // Allowed cross-agent targets
  }
  sandbox, tools                   // Capability configs
}
```

**Key Functions:**
- `listAgentIds(cfg)` — All configured agent IDs
- `resolveDefaultAgentId(cfg)` — Primary agent selection
- `resolveSessionAgentIds(params)` — Map session key → agent ID
- `resolveAgentConfig(cfg, agentId)` — Full resolved config
- `resolveEffectiveModelFallbacks(params)` — Merge agent + global fallbacks

---

### 11. Session Management & Routing

**Session Key Format:** `agent:{agentId}:{restOfKey}`

| Key Type | Format | Example |
|----------|--------|---------|
| **Main** | `agent:{agentId}:{mainKey}` | `agent:main:main` |
| **Subagent** | `agent:{agentId}:subagent:{parentId}:{timestamp}` | `agent:coding:subagent:main:1702500000000` |
| **Channel** | `agent:{agentId}:{channel}:{peerKind}:{peerId}` | `agent:main:discord:group:123456` |
| **Thread** | `{baseKey}:thread:{threadId}` | `agent:main:discord:channel:678:thread:900` |

**DM Scoping Options:**
- `"main"` — Single main session
- `"per-peer"` — Separate session per peer
- `"per-channel-peer"` — Channel + peer combo
- `"per-account-channel-peer"` — Full isolation

**SessionEntry** (`config/sessions/types.ts`, lines 25–104):
```typescript
SessionEntry {
  sessionId: string              // UUID
  updatedAt: number              // Timestamp
  sessionFile?: string           // Transcript path
  spawnedBy?: string             // Parent session key
  spawnDepth?: number            // 0=main, 1+=subagent
  systemSent?: boolean           // System prompt delivered
  chatType?: "direct" | "group" | "channel"
  model?: string                 // Session model override
  compactionCount?: number
  inputTokens?, outputTokens?, totalTokens?
  label?: string
  channel?: string
  groupId?: string
}
```

**Storage:** TTL-based cache (default 45s), per-agent store paths, JSON persistence.

---

### 12. Platform Tools

**Factory:** `createOpenClawTools(options)` (`openclaw-tools.ts`, lines 25–64)

#### Messaging Tools

| Tool | Purpose | Key Parameters |
|------|---------|---------------|
| **sessions_send** | Send message to another session | `sessionKey?`, `label?`, `agentId?`, `message`, `timeoutSeconds?` |
| **sessions_spawn** | Spawn background subagent | `task`, `label?`, `agentId?`, `model?`, `thinking?`, `runTimeoutSeconds?`, `cleanup` |
| **message** | Send to external channel (Discord, Slack, etc.) | `channel`, `target`, `text`, `markdown`, `buttons` |

#### Session Management Tools

| Tool | Purpose |
|------|---------|
| **session_status** | Session state + metrics (tokens, model, config) |
| **sessions_list** | List sessions matching criteria |
| **sessions_history** | Message history from session |
| **subagents** | List, kill, steer subagent runs |

#### Memory Tools

| Tool | Purpose | Key Parameters |
|------|---------|---------------|
| **memory_search** | Semantic + FTS hybrid search | `query`, `maxResults?`, `minScore?` |
| **memory_get** | Read snippet from memory file | `path`, `from?`, `lines?` |

#### Other Tools

| Tool | Purpose |
|------|---------|
| **web_search** | Internet search via provider APIs |
| **web_fetch** | Fetch & process web pages |
| **browser** | Headless browser automation |
| **image** | Image generation/processing |
| **canvas** | Drawing/graphics |
| **tts** | Text-to-speech |
| **cron** | Schedule recurring tasks |
| **nodes** | Mobile device control |
| **gateway** | Direct gateway API access |

---

### 13. Subagent Spawning & Lifecycle

**Entry Point:** `spawnSubagentDirect(params, ctx)` (`subagent-spawn.ts`, line 81)

**Validation Checks:**
1. **Depth check:** `callerDepth >= maxSpawnDepth` → FORBIDDEN (default max: 1)
2. **Active children:** `activeChildren >= maxChildren` → FORBIDDEN (default max: 5)
3. **Cross-agent auth:** `targetAgentId !== requesterAgentId` requires `allowAgents` config

**SubagentRunRecord** (`subagent-registry.ts`, lines 14–37):
```typescript
SubagentRunRecord {
  runId: string                  // Unique run ID
  childSessionKey: string        // Spawned session key
  requesterSessionKey: string    // Parent session
  task: string                   // Task description
  label?: string
  model?: string
  cleanup: "delete" | "keep"     // Cleanup policy on completion
  createdAt: number
  startedAt?: number
  endedAt?: number
  outcome?: SubagentRunOutcome
}
```

**Registry:** In-memory `subagentRuns` Map + disk persistence. Swept every ~5s, archived after 5 min. Max announce retries: 3.

**Spawn → Announce Flow:**
```
sessions_spawn tool call
  ↓
spawnSubagentDirect(params, ctx)
  ├─ Validate depth, children, cross-agent auth
  ├─ Create SessionEntry with spawnedBy, spawnDepth
  ├─ registerSubagentRun() → in-memory + disk
  └─ Return { childSessionKey, runId }
  ↓
Gateway queues background run
  ↓
runEmbeddedPiAgent({ sessionKey: childSessionKey })
  └─ Executes with parent's delivery context
  ↓
runSubagentAnnounceFlow() [async]
  ├─ Wait for completion
  ├─ Read result (runtime, tokens, success/failure)
  ├─ Send back via sessions_send to parent
  └─ Apply cleanup policy (delete/keep files)
```

**Subagent Management (subagents tool):**
- **list** — Show active/recent subagent runs
- **kill** — Terminate a running subagent
- **steer** — Abort current run, inject new message into existing context

---

### 14. Memory System

**Location:** `openclaw/src/memory/`

**Core Class:** `MemoryIndexManager` (`manager.ts`, line 41)

**Architecture:**
```typescript
MemoryIndexManager {
  provider: EmbeddingProvider            // Active embedding provider
  requestedProvider: "openai" | "local" | "gemini" | "voyage" | "auto"
  db: DatabaseSync                       // SQLite
  vector: { enabled, available, dims }   // Vector search
  fts: { enabled, available }            // Full-text search
  watcher: FSWatcher | null              // File change detection
  cache: { enabled, maxEntries? }
}
```

**Memory Sources:**
- `MEMORY.md` (main memory file)
- `memory/*.md` (per-topic files)
- Optional session transcripts (configurable)

**Search Flow:**
```
memory_search(query)
  → MemoryIndexManager.search(query, options)
    → extractKeywords(query)
    → Parallel:
       ├─ searchVector(keywords) → embedding similarity
       └─ searchKeyword(keywords) → FTS full-text
    → mergeHybridResults() with bm25RankToScore()
    → clampResultsByInjectedChars() (token budget)
    → Return MemorySearchResult[]
```

**Citations Mode:** `"on"` (always), `"off"` (never), `"auto"` (direct chats only)

**Persistence:** SQLite with tables: `chunks_vec`, `chunks_fts`, `embedding_cache`

---

### 15. Inter-Agent Communication

**Agent-to-Agent Message Flow:**
```
Agent A (sessions_send tool)
  → resolveSessionReference({ sessionKey or label })
    → Call gateway.sessions.resolve()
    → Return resolved key: agent:B:mainKey
  → Check agent-to-agent policy (tools.agentToAgent.allow)
  → runSessionsSendA2AFlow()
    → Build message context
    → Call gateway.agent() for Agent B session
    → Wait for response
    → Return status + response text
```

**Agent Step** (`agent-step.ts`, lines 33–80):
```typescript
// Execute one agent turn in another session
runAgentStep({
  sessionKey: string,
  message: string,
  extraSystemPrompt: string,
  timeoutMs: number,
  channel?, lane?, sourceSessionKey?, sourceChannel?, sourceTool?
})
```
Flow: `gateway.agent()` with `deliver: false` → `gateway.agent.wait()` → Read latest reply.

**Identity Linking:** Maps user identities across channels (same person in Slack + Discord). Scope: `identityLinks: Record<string, string[]>` (canonical → aliases).

---

### 16. Agent Execution & Continuation

**Entry Point:** `runEmbeddedPiAgent(params)` (`pi-embedded-runner/run.ts`)

**Run Lifecycle:**
1. **Init:** Load config, resolve model + auth, build system prompt, prepare transcript
2. **Per-Attempt Loop:** `runEmbeddedAttempt()` with SDK tools
3. **Error Handling:** Classify error → failover to next model → auth cooldown → retry compaction
4. **Session Update:** Model, tokens, compaction count, freshness flag

**Usage Accumulation:**
```typescript
UsageAccumulator {
  input, output, cacheRead, cacheWrite, total
  lastCacheRead, lastCacheWrite, lastInput  // Per-call values
}
```

**Result Type:**
```typescript
EmbeddedPiRunResult {
  payloads?: Array<{ text?, mediaUrl?, isError? }>
  meta: EmbeddedPiRunMeta
  didSendViaMessagingTool?: boolean
  messagingToolSentTexts?: string[]
  successfulCronAdds?: number
}
```

---

### 17. Extension System (Embedded)

**Location:** `openclaw/src/agents/pi-embedded-runner/extensions.ts`

**Built-in Extensions:**

1. **Compaction Safeguard** (`pi-extensions/compaction-safeguard.ts`):
   - Mode: `compaction.mode === "safeguard"`
   - Validates compaction doesn't exceed `maxHistoryShare`
   - Prevents excessive context summarization

2. **Context Pruning (Cache TTL)** (`pi-extensions/context-pruning/`):
   - Mode: `contextPruning.mode === "cache-ttl"`
   - For Anthropic models with prompt caching
   - Configurable TTL, max-age, batch size
   - Prunes stale tool results to save context budget

**Extension Resolution:**
- Dev: `.ts` files; Prod: `.js` files
- Extensions receive `SessionManager` instance to mutate runtime behavior

---

## Loop_core Python Agentic Framework

### 18. Two-Phase Atomic Execution Model

**File:** `loop_core/loop.py`

**Key Data Structures:**

- **`AtomicState`** (lines 1–100): Tracks `completed_steps`, `current_step_summary`, `error_context`, `tool_call_results`, `last_tool_failed`
- **`Turn`**: Per-iteration tracking — turn number, timestamp, duration_ms, tokens_used, tool_calls, llm_text, plan_step_index
- **`LoopResult`** (lines 100–403): Result container — turns, tokens, status, reflections, plan, execution_trace, journal
- **`ExecutionPlan`**: Multi-step decomposition with current_step_index, status tracking

**Execution Flow:**
```
AgenticLoop.execute()
  1. Build system prompt (system + identity + skills + memory)
  2. Call LLM via llm_client.complete_with_tools()
  3. If tool calls returned:
     → ToolRegistry.execute() for each tool (30s timeout, 100KB limit)
     → Inject ToolResult as user message (NOT system)
     → Update AtomicState with completed_steps, error_context
     → Loop to step 2
  4. If no tool calls (text response):
     → Check loop detection, reflection triggers, planning completion
     → Return LoopResult with final_response
```

**Execution Controls:**
- `max_turns`: Iteration limit (default 20)
- `timeout_seconds`: Overall time limit (default 600s)
- `enable_loop_detection`: `LoopDetector` (lines 410–521) tracking identical/repeated patterns
- `cancel_check`: Callable for external cancellation

---

### 19. Agent & Agent Manager

**Agent Class** (`loop_core/agent.py`, 1441 lines):

```python
class Agent:
    def run(self, message, session_id, conversation_history,
            additional_context, event_context, skill_id, phase2_model):
        # 1. Check user directives (memory commands)
        # 2. Load session
        # 3. Build identity_block (role, capabilities, team, workspace)
        # 4. Inject credentials, TODOs, issues into memory_prompt
        # 5. Pre-loop TODO review (for heartbeat runs)
        # 6. Execute AgenticLoop
        # 7. Post-execution: scan for facts, verify TODOs
        # 8. Save session (auto-trim if over max_turns)
        return AgentResult(status, final_response, turns, tools_called, error)
```

**AgentManager** (`loop_core/agent_manager.py`, 1089 lines):
- Central orchestrator with per-agent component caching
- `create_agent()` (line 502–660): Creates Agent with all tools registered unconditionally
- `run_agent()` (line 711–873): Entry point with rate limiting, HiveLoop integration, output saving

**Registered Tools (35+):**
- File: `FileReadTool`, `FileWriteTool` (sandboxed to agent directories)
- HTTP: `HttpCallTool`, `WebFetchTool`
- Tasks: `TaskCreateTool`, `TaskListTool`, `TaskGetTool`, `TaskUpdateTool`, `TaskDeleteTool`, `TaskTriggerTool`, `TaskRunsTool`
- Feed/Events: `FeedPostTool`, `SaveTaskStateTool`, `GetTaskStateTool`, `CreateEventTool`
- Search/Data: `WebSearchTool`, `CsvExportTool`, `SpreadsheetCreateTool`
- Content: `DocumentExtractTool`, `ImageGenerateTool`
- Communication: `SendNotificationTool`, `EmailSendTool`
- CRM: `CrmSearchTool`, `CrmWriteTool`, `TicketCreateCrmTool`, `TicketUpdateCrmTool`
- Colony: `ColonyReadTool`, `ColonyWriteTool`
- Compute: `AggregateTool`, `ComputeTool`
- Task Queue: `TodoAddTool`, `TodoListTool`, `TodoCompleteTool`, `TodoRemoveTool`, `IssueReportTool`

---

### 20. Tool System

**File:** `loop_core/tools/base.py` (400 lines)

**Architecture:**

```python
class ToolParameter:
    name, type, description, required, enum, default
    def to_schema(): # JSON Schema

class ToolDefinition:
    name, description, parameters
    def to_schema():         # OpenAI format
    def to_anthropic_schema(): # Anthropic format

class ToolResult:
    success: bool
    output: str              # Max 100KB, truncated with notice
    error: Optional[str]
    metadata: Optional[dict]

class BaseTool:
    definition: ToolDefinition
    execute(**kwargs) -> ToolResult
    set_credentials(**creds)  # Pre-configure auth

class ToolRegistry:
    # Thread pool: 4 workers max
    register(tool)
    execute(tool_name, parameters, timeout=30) -> ToolResult
    get_schemas(format="openai"|"anthropic")
```

**Execution Flow:**
```
LLM returns tool call
  → ToolRegistry.execute(tool_name, parameters)
  → Submit tool.execute(**params) to ThreadPoolExecutor
  → Wait (max 30s timeout)
  → Truncate output if > 100KB
  → Return ToolResult
  → Loop injects result as user message
```

---

### 21. Runtime / Daemon System

**File:** `loop_core/runtime.py` (1795 lines)

**Architecture:** Single daemon thread iterating every 1s over all active agents. Per-agent state with priority event queue (max 20 items). Thread pool (4 workers) for LLM calls.

**Priority Levels:**
```python
HIGH = 1      # Human messages (user waiting)
NORMAL = 2    # Webhooks, scheduled tasks
LOW = 3       # Heartbeat ticks
```

**AgentEvent:**
```python
@dataclass
class AgentEvent:
    priority: Priority
    message: str
    session_key: str
    source: str    # "human", "heartbeat", "task:{id}", "webhook:{provider}", "agent:{id}"
    skill_id: str
    status: str    # "pending_approval" | "active" | "running" | "completed" | "dropped"
    routing: OutputRouteConfig
```

**AgentState:**
```python
@dataclass
class AgentState:
    agent_id: str
    active: bool
    queue: List[AgentEvent]        # Bisect-sorted by priority
    pending_events: List[AgentEvent]  # Awaiting approval
    heartbeat_enabled: bool
    heartbeat_interval_minutes: int
    current_run: Optional[Future]
    # Metrics
    heartbeats_fired, events_processed, events_failed: int
    event_history: deque           # Last 50 completed events
```

**Main Loop:**
```
_run_loop() [every 1 second]:
  1. Write heartbeat liveness file
  2. For each active agent:
     a. Check heartbeat timer → enqueue LOW-priority events
     b. Check scheduled tasks → enqueue NORMAL-priority events
     c. If idle and queue non-empty → pop highest-priority, submit to pool
     d. If current run done → harvest result, update metrics, route response
```

**Heartbeat System:**
- Discover heartbeat skills per agent
- Deterministic stagger via `hash(agent_id)` for thundering herd prevention
- Pre-check HTTP polls (evaluate `skip_if` conditions)
- Skip if agent busy

**Queue Persistence:** Save/restore `.saved_queue.json` on graceful shutdown. Survives crashes if `persist_queue_on_stop=True`.

---

### 22. Memory System

**File:** `loop_core/memory/manager.py` (500+ lines)

**Two-Tier Architecture:**

**Tier 1 — Sessions (Conversation Persistence):**
- Stored: `data/AGENTS/{agent_id}/sessions/session_{session_id}.json`
- Max 10MB per session, max 20 completed sessions retained
- Auto-cleanup of old sessions

**Tier 2 — Long-Term Memory (Topic-Based):**
- Directory: `data/AGENTS/{agent_id}/memory/`
- `MemoryEntry`: content_id, topic_id, title, sections
- `TopicIndex`: Search index with summaries, keywords, content files
- Max 100MB per agent

**API:** Topic management (create, list, search) + Entry management (add, get, update, delete) + Session management.

---

### 23. Reflection System

**File:** `loop_core/reflection.py` (200+ lines)

**Configuration:**
```python
ReflectionConfig:
    enabled: bool
    interval_turns: int      # 0 = disabled (atomic model tracks progress)
    no_progress_turns: int   # Default 3
    max_reflections: int     # Default 2
    reflection_temperature: float  # 0.3
```

**ReflectionResult:**
- `progress_assessment`: "good" | "slow" | "stuck" | "regressing"
- `confidence_in_approach`: float 0.0–1.0
- `decision`: "continue" | "adjust" | "pivot" | "escalate" | "terminate"
- `what_worked`, `what_failed`, `blockers_identified`: Lists

**Triggers:** Interval-based, stagnation detection (N turns without progress), tool failure, approaching resource limits.

---

### 24. Planning System

**File:** `loop_core/planning.py` (200+ lines)

**Configuration:**
```python
PlanningConfig:
    enabled: bool
    min_task_complexity: int    # Word count threshold (default 10)
    max_steps: int              # Default 10
    max_turns_per_step: int     # Default 5
    auto_replan_on_block: bool
    max_replans: int            # Default 3
```

**PlanStep:**
- `step_id`, `description`, `status` (pending | in_progress | completed | skipped | blocked)
- `depends_on`, `expected_tools`, `acceptance_criteria`
- `pre_conditions`, `on_failure`

**Atomic Step Completion** (loop.py line 1031–1081):
- Uses `AtomicState.completed_steps` instead of text heuristics
- 40% keyword overlap threshold triggers step completion
- Auto-advance if exceeds `max_turns_per_step`

---

### 25. Learning System

**File:** `loop_core/learning.py` (200+ lines)

**Stored in:** `data/AGENTS/{agent_id}/memory/learning_store.json`

**Pattern Types:**

```python
ErrorPattern:
    error_type, error_signature, error_message_pattern
    resolution_strategy, resolution_steps, preventive_action
    occurrences, success_rate

SuccessPattern:
    task_type, task_keywords
    approach_summary, key_steps, tools_used, tool_sequence
    typical_turns, times_used

ToolInsight:
    tool_name
    common_parameters, common_errors, best_practices
    often_followed_by, often_preceded_by
    total_calls, success_rate, avg_execution_time_ms
```

**Learning Lifecycle:**
- Pre-execution: Retrieve relevant insights → inject into memory_prompt
- Per-tool: Track parameters, success rates, error patterns, tool sequences
- Post-execution: Learn from final result and approach taken

---

### 26. Configuration System

**File:** `loop_core/config/__init__.py`

**Hierarchy:**
- **Global:** `config/global.json` (LLM defaults, rate limits, paths, reflection/planning/learning)
- **Agent:** `data/AGENTS/{agent_id}/config.json` (can override globals)

**AgentConfig Fields:**
```python
AgentConfig:
    agent_id, name, description, role
    system_prompt
    llm: LLMConfig           # model, temperature, max_tokens
    max_turns, timeout_seconds
    enabled_tools: List[str]
    skills: Dict
    reflection, planning, learning  # Optional per-agent overrides
    heartbeat_enabled, heartbeat_interval_minutes
    phase2_model: Optional      # Different model for Phase 2
    persist_queue_on_stop: bool
```

---

### 27. API Layer

**File:** `loop_core/api/app.py` (400+ lines)

**FastAPI REST Endpoints:**

| Category | Endpoints |
|----------|-----------|
| **Agent Management** | `POST /agents/{id}/run`, `GET /agents`, `POST /agents`, `PUT /agents/{id}`, `DELETE /agents/{id}` |
| **Sessions** | `GET /agents/{id}/sessions`, `GET /sessions/{id}`, `DELETE /sessions/{id}` |
| **Skills** | `GET /skills`, `POST /skills/fetch` |
| **Memory** | `GET /memory/search` |
| **Runs** | `GET /runs`, `GET /runs/{agent_id}/{date}/{run_id}` |
| **Tasks** | `GET /tasks`, `POST /tasks`, `PUT /tasks/{id}`, `POST /tasks/{id}/trigger` |
| **Runtime** | `POST /agents/{id}/start`, `POST /agents/{id}/stop`, `GET /agents/{id}/status` |
| **Health** | `GET /health`, `GET /status` |

**Response includes:** status, response, turns, tools_called, total_tokens, duration_ms, pending_events, execution_trace, plan, reflections, journal.

---

### 28. Scheduler System

**File:** `loop_core/scheduler/scheduler.py` (300+ lines)

**Per-agent structure:** `data/AGENTS/{agent_id}/tasks/{task_id}/task.json + task.md`

**Schedule Types:**
- **interval:** Recurring every N seconds
- **cron:** Croniter expression
- **once:** One-shot (never again after first run)
- **event_only:** Manual trigger only

**Task Discovery:** Scan all `agents/tasks/` directories every 10s.

---

## Cross-System Comparison

| Capability | pi-mono | openclaw | loop_core |
|-----------|---------|----------|-----------|
| **Language** | TypeScript | TypeScript | Python |
| **Core Tools** | 7 (read, bash, edit, write, grep, find, ls) | 20+ platform tools | 35+ tools |
| **Tool Execution** | Async event stream | Async via gateway | ThreadPool (4 workers, 30s timeout) |
| **Session Format** | JSONL with tree branching | JSON with session store | JSON with topic-based memory |
| **Sub-Agents** | Fork/branch (same session) | Full parent-child tree with depth limits | Isolated peers (shared files/feeds) |
| **Streaming** | EventStream (12+ event types) | Via pi-mono EventStream | Synchronous (LoopResult on completion) |
| **Model Support** | 22+ providers via registry | Inherits pi-mono + failover | Anthropic + OpenAI (hardcoded) |
| **Memory** | Session files + compaction | Semantic + FTS hybrid search (SQLite) | Topic-based + session persistence |
| **Reflection** | None built-in | None built-in | Full system (progress, confidence, decisions) |
| **Planning** | None built-in | None built-in | Multi-step with atomic completion |
| **Learning** | None built-in | None built-in | Error/success/tool patterns |
| **Runtime** | CLI process | Multi-agent daemon | Daemon with priority event queue |
| **Scheduling** | None | Cron tool | Interval/cron/once/event scheduler |
| **Extension System** | Full API (register tools, commands, providers) | Embedded (compaction, pruning) | None |
| **Modes** | Interactive TUI / Print / RPC | Channel adapters (Discord, Slack, etc.) | FastAPI REST + WebSocket |
| **Compaction** | Auto + manual + extension hooks | Safeguard + cache-TTL extensions | Session trimming (max_turns) |

---

## Architecture Diagrams

### Pi-Mono Agent Interaction Flow
```
User Input (TUI / stdin / RPC)
    ↓
AgentSession.prompt(text, options)
    ↓
Extension: input_event → can transform input
    ↓
Agent.prompt(messages)
    ↓
Agent Loop:
  1. Transform context (convertToLlm)
  2. LLM call (streamSimple)
  3. Tool calling loop:
     ├─ Extension: tool_call → can block
     ├─ Execute tool
     └─ Extension: tool_result → can modify
  4. More turns if needed
    ↓
Extension: agent_end
    ↓
Session save (via event handler)
    ↓
Check auto-compaction / retry
    ↓
Mode renders response
```

### OpenClaw Multi-Agent Flow
```
External Message (Discord / Slack / Telegram / API)
    ↓
Channel Adapter → Session Key Resolution
    ↓
Gateway → runEmbeddedPiAgent(params)
    ├─ Load config, model, auth
    ├─ Build system prompt + memory
    ├─ Prepare transcript
    └─ Resolve extensions
    ↓
Per-Attempt Loop:
    ├─ runEmbeddedAttempt() with SDK tools
    ├─ Tool execution (may spawn subagents)
    ├─ Error? → Failover to next model
    └─ Success → Update session, route response
    ↓
If sessions_spawn called:
    ├─ Validate depth + children + auth
    ├─ Create child session
    ├─ Register SubagentRunRecord
    ├─ Queue background run
    ├─ Child executes independently
    └─ Announce result back to parent
```

### Loop_core Execution Flow
```
API Request / Runtime Event
    ↓
AgentManager.run_agent()
    ↓
Agent.run()
    ├─ Check directives & memory queries
    ├─ Load session
    ├─ Build prompts (system + identity + skills + memory)
    ├─ Inject TODOs, issues, credentials
    ├─ Load learning insights, create plan
    ↓
AgenticLoop.execute() [TWO-PHASE ATOMIC]
    ↓
Turn N:
    ├─ Call LLM with system_prompt + tools
    ├─ IF tool calls:
    │   ├─ ToolRegistry.execute(name, params) [ThreadPool, 30s, 100KB]
    │   ├─ Inject result as user message
    │   ├─ Update AtomicState
    │   ├─ Learn tool pattern
    │   └─ Check loop detection → CONTINUE
    └─ IF text response:
        ├─ Check planning (step completion, replan)
        ├─ Check reflection (progress, decision)
        ├─ Check resource limits
        └─ EXIT → LoopResult
    ↓
Post-Execution:
    ├─ Scan turn for facts (TurnScanner)
    ├─ Save session + trim
    ├─ Auto-create TODOs for failed actions
    ├─ Update heartbeat history
    └─ Return AgentResult
```

---

## Critical Files Reference

### Pi-Mono Coding-Agent

| Component | File | Key Exports |
|-----------|------|-------------|
| Core Session | `src/core/agent-session.ts` | `AgentSession` (2700+ lines) |
| Event Bus | `src/core/event-bus.ts` | `createEventBus()` |
| Session Manager | `src/core/session-manager.ts` | `SessionManager`, session entry types |
| Settings | `src/core/settings-manager.ts` | `Settings` interface |
| Model Registry | `src/core/model-registry.ts` | `ModelRegistry`, model schema |
| System Prompt | `src/core/system-prompt.ts` | `buildSystemPrompt()` |
| Tool Index | `src/core/tools/index.ts` | `codingTools`, `readOnlyTools`, `allTools` |
| Extension Runner | `src/core/extensions/runner.ts` | `ExtensionRunner` (620+ lines) |
| Extension Types | `src/core/extensions/types.ts` | `ExtensionAPI`, event types |
| Compaction | `src/core/compaction/compaction.ts` | `CompactionResult` |
| Main Entry | `src/main.ts` | CLI orchestration (537+ lines) |

### OpenClaw

| Component | File | Key Exports |
|-----------|------|-------------|
| Agent Definition | `src/agents/agent-scope.ts` | `resolveAgentConfig()`, `listAgentIds()` |
| Session Keys | `src/routing/session-key.ts` | `parseAgentSessionKey()`, key builders |
| Session Storage | `src/config/sessions/types.ts`, `store.ts` | `SessionEntry`, `loadSessionStore()` |
| Platform Tools | `src/agents/openclaw-tools.ts` | `createOpenClawTools()` |
| Subagent Spawn | `src/agents/subagent-spawn.ts` | `spawnSubagentDirect()` |
| Subagent Registry | `src/agents/subagent-registry.ts` | `registerSubagentRun()`, `SubagentRunRecord` |
| Subagent Announce | `src/agents/subagent-announce.ts` | `runSubagentAnnounceFlow()` |
| Subagent Depth | `src/agents/subagent-depth.ts` | `getSubagentDepthFromSessionStore()` |
| Memory Manager | `src/memory/manager.ts` | `MemoryIndexManager` |
| Agent Execution | `src/agents/pi-embedded-runner/run.ts` | `runEmbeddedPiAgent()` |
| Extensions | `src/agents/pi-embedded-runner/extensions.ts` | `buildEmbeddedExtensionPaths()` |
| Sessions Send | `src/agents/tools/sessions-send-tool.ts` | `createSessionsSendTool()` |
| Agent Step | `src/agents/tools/agent-step.ts` | `runAgentStep()` |

### Loop_core

| Component | Class | File |
|-----------|-------|------|
| Core Loop | `AgenticLoop`, `LoopResult`, `Turn`, `AtomicState` | `loop.py` |
| Loop Detector | `LoopDetector` | `loop.py` (lines 410–521) |
| Agent | `Agent`, `AgentResult`, `HeartbeatSummary` | `agent.py` |
| Agent Manager | `AgentManager` | `agent_manager.py` |
| Runtime | `AgentRuntime`, `AgentEvent`, `AgentState`, `Priority` | `runtime.py` |
| Tool System | `BaseTool`, `ToolRegistry`, `ToolResult`, `ToolDefinition` | `tools/base.py` |
| Memory | `MemoryManager`, `Session`, `MemoryEntry`, `TopicIndex` | `memory/manager.py` |
| Reflection | `ReflectionManager`, `ReflectionConfig`, `ReflectionResult` | `reflection.py` |
| Planning | `PlanningManager`, `PlanningConfig`, `ExecutionPlan`, `PlanStep` | `planning.py` |
| Learning | `LearningManager`, `ErrorPattern`, `SuccessPattern`, `ToolInsight` | `learning.py` |
| Configuration | `ConfigManager`, `AgentConfig`, `GlobalConfig` | `config/__init__.py` |
| API | FastAPI endpoints | `api/app.py` |
| Scheduler | `TaskScheduler`, `ScheduledTask`, `TaskSchedule` | `scheduler/scheduler.py` |
