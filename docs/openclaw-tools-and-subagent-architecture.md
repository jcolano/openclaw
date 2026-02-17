# OpenClaw Complete Tool Catalog & Sub-Agent Architecture

> A comprehensive reference covering every tool available in openclaw/pi-mono,
> how sub-agents are triggered, and what loop_core can learn from the design.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Layer 1: pi-mono Core Tools (7)](#layer-1-pi-mono-core-tools)
3. [Layer 2: openclaw Platform Tools (20)](#layer-2-openclaw-platform-tools)
4. [Layer 3: Extension Tools (25+)](#layer-3-extension-tools)
5. [Layer 4: pi-mono Example Extensions (11)](#layer-4-pi-mono-example-extensions)
6. [Layer 5: Web UI Tools (3)](#layer-5-web-ui-tools)
7. [Sub-Agent Architecture Deep Dive](#sub-agent-architecture-deep-dive)
8. [When and Why Sub-Agents Are Triggered](#when-and-why-sub-agents-are-triggered)
9. [Key Takeaways for loop_core](#key-takeaways-for-loop_core)

---

## Architecture Overview

OpenClaw has a **layered tool architecture**. Each layer adds capabilities on top of the previous:

```
┌─────────────────────────────────────────────────────────┐
│  Layer 5: Web UI Tools (artifacts, js_repl, extract)    │  Browser-only
├─────────────────────────────────────────────────────────┤
│  Layer 4: pi-mono Example Extensions (subagent, ssh...) │  Optional, user-installed
├─────────────────────────────────────────────────────────┤
│  Layer 3: openclaw Extensions (feishu, memory, voice..) │  Plugin-based, per-deployment
├─────────────────────────────────────────────────────────┤
│  Layer 2: openclaw Platform Tools (sessions, browser..) │  Server-side orchestration
├─────────────────────────────────────────────────────────┤
│  Layer 1: pi-mono Core Tools (read, bash, edit, write..)│  Foundation - always present
└─────────────────────────────────────────────────────────┘
```

The LLM sees **all active tools as a flat list** — it doesn't know which layer they come from. Tool selection is entirely the LLM's decision, guided by tool descriptions and the system prompt.

---

## Layer 1: pi-mono Core Tools

These 7 tools form the foundation. Every pi-mono agent has access to some or all of them.

**Source:** `pi-mono/packages/coding-agent/src/core/tools/`

### Tool Groups (Presets)

| Group | Tools | Use Case |
|-------|-------|----------|
| `codingTools` | read, bash, edit, write | Full coding access |
| `readOnlyTools` | read, grep, find, ls | Exploration without modification |
| `allTools` | all 7 | Maximum capability |

---

### `read`

| Field | Value |
|-------|-------|
| **Name** | `read` |
| **Description** | Read file contents. Supports text files and images (jpg, png, gif, webp). Images sent as attachments. Text truncated to 2000 lines or 50KB. Use offset/limit for large files. |
| **Parameters** | `path` (string, required), `offset` (number, optional, 1-indexed), `limit` (number, optional) |
| **Modifies state** | No |

**Key behaviors:**
- Images auto-resized to 2000x2000 max, returned as base64
- Truncation appends actionable notice: `[Showing lines X-Y of Z. Use offset=N to continue.]`
- Pluggable via `ReadOperations` interface for remote/SSH use
- Handles macOS path quirks (NFD normalization, curly quotes)

---

### `bash`

| Field | Value |
|-------|-------|
| **Name** | `bash` |
| **Description** | Execute a bash command. Returns stdout and stderr. Output truncated to last 2000 lines or 50KB. If truncated, full output saved to temp file. |
| **Parameters** | `command` (string, required), `timeout` (number, optional, seconds) |
| **Modifies state** | Yes (can modify filesystem, run processes) |

**Key behaviors:**
- **Tail-truncated** (keeps last N lines — shows errors, not headers)
- Overflow written to `/tmp/pi-bash-<hex>.log`
- Process tree killed on abort (SIGTERM → SIGKILL)
- Streaming progress via `onUpdate` callback
- Pluggable `BashSpawnHook` can mutate command/cwd/env before execution

---

### `edit`

| Field | Value |
|-------|-------|
| **Name** | `edit` |
| **Description** | Edit a file by replacing exact text. The oldText must match exactly (including whitespace). |
| **Parameters** | `path` (string, required), `oldText` (string, required), `newText` (string, required) |
| **Modifies state** | Yes |

**Key behaviors:**
- Fuzzy matching fallback: normalizes trailing whitespace, smart quotes, Unicode dashes/spaces
- Rejects if `oldText` matches more than once (must be unique)
- Preserves BOM and original line endings (CRLF vs LF)
- Returns unified diff on success

---

### `write`

| Field | Value |
|-------|-------|
| **Name** | `write` |
| **Description** | Write content to a file. Creates the file if it doesn't exist, overwrites if it does. Automatically creates parent directories. |
| **Parameters** | `path` (string, required), `content` (string, required) |
| **Modifies state** | Yes |

---

### `grep`

| Field | Value |
|-------|-------|
| **Name** | `grep` |
| **Description** | Search file contents for a pattern. Returns matching lines with paths and line numbers. Respects .gitignore. Truncated to 100 matches or 50KB. |
| **Parameters** | `pattern` (string, required), `path` (optional), `glob` (optional), `ignoreCase` (bool), `literal` (bool), `context` (number), `limit` (number, default 100) |
| **Modifies state** | No |

**Key behaviors:**
- Backed by ripgrep (`rg`), auto-downloaded if missing
- JSON output parsing, process killed once limit reached
- Long lines truncated to 500 chars

---

### `find`

| Field | Value |
|-------|-------|
| **Name** | `find` |
| **Description** | Search for files by glob pattern. Returns matching paths relative to search directory. Respects .gitignore. Truncated to 1000 results or 50KB. |
| **Parameters** | `pattern` (string, required), `path` (optional), `limit` (number, default 1000) |
| **Modifies state** | No |

**Key behaviors:**
- Backed by `fd` binary, auto-downloaded if missing
- Auto-discovers nested `.gitignore` files

---

### `ls`

| Field | Value |
|-------|-------|
| **Name** | `ls` |
| **Description** | List directory contents. Returns entries sorted alphabetically, with '/' suffix for directories. Includes dotfiles. Truncated to 500 entries or 50KB. |
| **Parameters** | `path` (optional), `limit` (number, default 500) |
| **Modifies state** | No |

---

## Layer 2: openclaw Platform Tools

These are registered by the openclaw server layer (`openclaw/src/agents/tools/`). They provide orchestration, messaging, web access, browser control, and infrastructure management.

---

### Sub-Agent Orchestration Tools

#### `sessions_spawn`

| Field | Value |
|-------|-------|
| **Name** | `sessions_spawn` |
| **Description** | Spawn a background sub-agent run in an isolated session and announce the result back to the requester chat. |
| **Parameters** | `task` (string, required), `label` (optional), `agentId` (optional), `model` (optional), `thinking` (optional), `runTimeoutSeconds` (optional), `cleanup` ("delete"\|"keep") |
| **Modifies state** | Yes — creates a new session, runs an agent |

This is the **production sub-agent spawn mechanism**. It calls `spawnSubagentDirect()` which creates a new isolated agent session via the gateway. The child session runs independently and announces its result back to the parent chat when complete.

Key parameters:
- `task` — the instruction the sub-agent receives
- `agentId` — target a specific agent configuration (different system prompt, model, tools)
- `model` — override the model for this sub-agent run
- `thinking` — override thinking level
- `runTimeoutSeconds` — kill the sub-agent after N seconds
- `cleanup: "delete"` — auto-delete the session after completion

---

#### `subagents`

| Field | Value |
|-------|-------|
| **Name** | `subagents` |
| **Description** | List, kill, or steer spawned sub-agents for this requester session. Use this for sub-agent orchestration. |
| **Parameters** | `action` ("list"\|"kill"\|"steer"), `target` (optional), `message` (optional), `recentMinutes` (optional) |
| **Modifies state** | Yes — can kill processes, steer running agents |

Three actions:

**`list`** — Shows all active and recent sub-agents with status, runtime, model, token usage. Targets can be addressed by: index number, "last", label (exact or prefix), run ID prefix, or full session key.

**`kill`** — Terminates a running sub-agent. Supports `target: "all"` or `target: "*"`. **Cascade kills** all descendants recursively (grandchildren, etc.). Aborts the embedded pi run, clears session queues, marks the run as terminated.

**`steer`** — The most sophisticated action. Aborts the sub-agent's current work, waits for it to settle, then injects a new message into the same session context. The sub-agent restarts with its full prior conversation preserved plus the new steering instruction. Rate-limited to prevent spam (2s cooldown). Self-steering is forbidden.

---

#### `agents_list`

| Field | Value |
|-------|-------|
| **Name** | `agents_list` |
| **Description** | List agent IDs you can target with sessions_spawn (based on allowlists). |
| **Parameters** | None |
| **Modifies state** | No |

---

### Session Management Tools

#### `sessions_list`

| Field | Value |
|-------|-------|
| **Name** | `sessions_list` |
| **Description** | List sessions with optional filters and last messages. |
| **Parameters** | `kinds` (array: main\|group\|cron\|hook\|node\|other), `limit`, `activeMinutes`, `messageLimit` |
| **Modifies state** | No |

---

#### `sessions_send`

| Field | Value |
|-------|-------|
| **Name** | `sessions_send` |
| **Description** | Send a message into another session. Use sessionKey or label to identify the target. |
| **Parameters** | `sessionKey` (optional), `label` (optional), `agentId` (optional), `message` (required), `timeoutSeconds` (optional) |
| **Modifies state** | Yes — sends messages to other sessions |

Supports agent-to-agent (A2A) messaging with configurable policies. Can wait synchronously for the target session's reply or fire-and-forget with `timeoutSeconds: 0`.

---

#### `sessions_history`

| Field | Value |
|-------|-------|
| **Name** | `sessions_history` |
| **Description** | Fetch message history for a session. |
| **Parameters** | `sessionKey` (required), `limit`, `includeTools` |
| **Modifies state** | No |

---

#### `session_status`

| Field | Value |
|-------|-------|
| **Name** | `session_status` |
| **Description** | Show session status card (usage + time + cost). Optional: set per-session model override. |
| **Parameters** | `sessionKey`, `model` |
| **Modifies state** | Only when setting model override |

---

### Communication Tools

#### `message`

| Field | Value |
|-------|-------|
| **Name** | `message` |
| **Description** | Send, delete, and manage messages via channel plugins. |
| **Parameters** | `action` (send\|reply\|delete\|react\|poll\|pin\|thread-reply\|broadcast\|searchMessages\|memberInfo\|roleInfo\|channelInfo\|...), plus routing and messaging fields |
| **Modifies state** | Yes |

This is the **universal messaging tool**. It dispatches to channel-specific handlers:
- Discord (30+ actions: messaging, guild, moderation, presence)
- Slack (11 actions: send, edit, delete, read, react, pin/unpin, memberInfo, emojiList)
- Telegram (7 actions: send, edit, delete, react, sticker search/send)
- WhatsApp (react only currently)

#### `tts`

| Field | Value |
|-------|-------|
| **Name** | `tts` |
| **Description** | Convert text to speech. Audio delivered automatically from tool result. |
| **Parameters** | `text` (required), `channel` |
| **Modifies state** | No (generates audio) |

---

### Web Tools

#### `web_search`

| Field | Value |
|-------|-------|
| **Name** | `web_search` |
| **Description** | Search the web. Supports Brave Search API, Perplexity Sonar, or xAI Grok backends. |
| **Parameters** | `query` (required), `count` (1-10), `country`, `search_lang`, `ui_lang`, `freshness` |
| **Modifies state** | No |

#### `web_fetch`

| Field | Value |
|-------|-------|
| **Name** | `web_fetch` |
| **Description** | Fetch and extract readable content from a URL (HTML to markdown/text). Supports Firecrawl backend. |
| **Parameters** | `url` (required), `extractMode` (markdown\|text), `maxChars` |
| **Modifies state** | No |

---

### Browser & Visual Tools

#### `browser`

| Field | Value |
|-------|-------|
| **Name** | `browser` |
| **Description** | Control the browser via OpenClaw's browser control server. Actions: status, start, stop, profiles, tabs, open, snapshot, screenshot, actions. Supports Chrome extension relay and isolated profiles. |
| **Parameters** | `action` (required), `profile`, `node`, `target`, `targetUrl`, `targetId`, `ref`, `element`, `type`, `selector`, `frame`, plus many more |
| **Modifies state** | Yes (browser automation) |

#### `canvas`

| Field | Value |
|-------|-------|
| **Name** | `canvas` |
| **Description** | Control node canvases (present/hide/navigate/eval/snapshot/A2UI). |
| **Parameters** | `action` (present\|hide\|navigate\|eval\|snapshot\|a2ui_push\|a2ui_reset), `node`, `url`, `javaScript`, `outputFormat`, etc. |
| **Modifies state** | Yes |

#### `image`

| Field | Value |
|-------|-------|
| **Name** | `image` |
| **Description** | Analyze one or more images with a vision model. Up to 20 images. |
| **Parameters** | `prompt`, `image` (single path/URL), `images` (array), `model`, `maxBytesMb`, `maxImages` |
| **Modifies state** | No |

---

### Infrastructure Tools

#### `cron`

| Field | Value |
|-------|-------|
| **Name** | `cron` |
| **Description** | Manage gateway cron jobs (status/list/add/update/remove/run/runs) and send wake events. |
| **Parameters** | `action` (status\|list\|add\|update\|remove\|run\|runs\|wake), `job`, `jobId`, `patch`, `text`, `mode`, etc. |
| **Modifies state** | Yes |

#### `gateway`

| Field | Value |
|-------|-------|
| **Name** | `gateway` |
| **Description** | Restart, apply config, or update the gateway. |
| **Parameters** | `action` (restart\|config.get\|config.schema\|config.apply\|config.patch\|update.run), `reason`, `note`, etc. |
| **Modifies state** | Yes (modifies server configuration) |

#### `nodes`

| Field | Value |
|-------|-------|
| **Name** | `nodes` |
| **Description** | Discover and control paired nodes (status/describe/pairing/notify/camera/screen/location/run/invoke). |
| **Parameters** | `action` (status\|describe\|pending\|approve\|reject\|notify\|camera_snap\|screen_record\|location_get\|run\|invoke), plus device-specific params |
| **Modifies state** | Yes (controls physical devices) |

---

### Memory Tools

#### `memory_search`

| Field | Value |
|-------|-------|
| **Name** | `memory_search` |
| **Description** | Mandatory recall step: semantically search MEMORY.md + memory/*.md before answering questions about prior work, decisions, dates, people, preferences, or todos. |
| **Parameters** | `query` (required), `maxResults`, `minScore` |
| **Modifies state** | No |

#### `memory_get`

| Field | Value |
|-------|-------|
| **Name** | `memory_get` |
| **Description** | Safe snippet read from MEMORY.md or memory/*.md with optional from/lines. Use after memory_search to pull needed lines. |
| **Parameters** | `path` (required), `from`, `lines` |
| **Modifies state** | No |

---

## Layer 3: Extension Tools

Registered at runtime via `api.registerTool()` in openclaw extensions (`openclaw/extensions/`).

### `llm-task`

| Field | Value |
|-------|-------|
| **Name** | `llm-task` |
| **Description** | Run a generic JSON-only LLM task and return schema-validated JSON. Designed for orchestration from Lobster workflows. |
| **Parameters** | `prompt`, `input`, `schema`, `provider`, `model`, `authProfileId`, `temperature`, `maxTokens`, `timeoutMs` |

Runs an embedded pi agent with all tools disabled. Validates output against an optional JSON Schema using AJV. Enables workflows to route specific extraction/classification tasks to specific models.

---

### `lobster`

| Field | Value |
|-------|-------|
| **Name** | `lobster` |
| **Description** | Run Lobster pipelines as a local-first workflow runtime (typed JSON envelope + resumable approvals). |
| **Parameters** | `action` (run\|resume), `pipeline`, `argsJson`, `token`, `approve`, `cwd`, `timeoutMs`, `maxStdoutBytes` |

---

### `voice_call`

| Field | Value |
|-------|-------|
| **Name** | `voice_call` |
| **Description** | Make phone calls and have voice conversations. |
| **Parameters** | `action` (initiate_call\|continue_call\|speak_to_user\|end_call\|get_status), or legacy mode/to/sid/message |

---

### `zalouser`

| Field | Value |
|-------|-------|
| **Name** | `zalouser` |
| **Description** | Send messages and access data via Zalo personal account. Actions: send, image, link, friends, groups, me, status. |
| **Parameters** | Action-specific |

---

### Feishu Suite (13 tools)

From `openclaw/extensions/feishu/`:

| Tool Name | Description |
|-----------|-------------|
| `feishu_bitable_get_meta` | Parse Bitable URL, get app_token/table_id |
| `feishu_bitable_list_fields` | List fields in a Bitable table |
| `feishu_bitable_list_records` | List records with pagination |
| `feishu_bitable_get_record` | Get single record by ID |
| `feishu_bitable_create_record` | Create new record |
| `feishu_bitable_update_record` | Update existing record |
| `feishu_bitable_create_app` | Create new Bitable application |
| `feishu_bitable_create_field` | Create new field/column |
| `feishu_doc` | Feishu document operations |
| `feishu_app_scopes` | Application scope management |
| `feishu_wiki` | Wiki operations |
| `feishu_drive` | Drive file operations |
| `feishu_perm` | Permission management |

---

### Memory (LanceDB) Extension (3 tools)

From `openclaw/extensions/memory-lancedb/`:

| Tool Name | Description |
|-----------|-------------|
| `memory_recall` | Vector search through long-term memories (LanceDB + OpenAI embeddings) |
| `memory_store` | Save information to long-term memory |
| `memory_forget` | Delete memories by query or ID |

Also registers lifecycle hooks: `before_agent_start` for auto-recall, `agent_end` for auto-capture.

---

## Layer 4: pi-mono Example Extensions

These are **reference implementations** in `pi-mono/packages/coding-agent/examples/extensions/`. User-installed, not active by default.

| Extension | Tool Name | Description |
|-----------|-----------|-------------|
| `subagent/` | `subagent` | Delegate tasks to specialized pi subprocess agents (single, parallel, chain) |
| `hello.ts` | `hello` | Simple greeting tool (example) |
| `question.ts` | `question` | Ask user a question with TUI options |
| `questionnaire.ts` | `questionnaire` | Multi-question form with tab-based TUI |
| `truncated-tool.ts` | `rg` | Ripgrep wrapper with truncation handling |
| `tool-override.ts` | `read` (override) | Audited read that logs access and blocks sensitive paths |
| `ssh.ts` | `read`, `write`, `edit`, `bash` (overrides) | Remote SSH variants of all four core tools |
| `antigravity-image-gen.ts` | `generate_image` | Google Antigravity image generation |
| `sandbox/` | `bash` (override) | OS-level sandboxed bash via `@anthropic-ai/sandbox-runtime` |
| `with-deps/` | `parse_duration` | Human-readable duration parsing (example with npm deps) |
| `tools.ts` | (slash command only) | `/tools` command to enable/disable tools at runtime |

---

## Layer 5: Web UI Tools

From `pi-mono/packages/web-ui/src/tools/` — browser-only context.

| Tool Name | Description |
|-----------|-------------|
| `artifacts` | Create and display rich artifacts (HTML, Markdown, SVG, PDF, DOCX, XLSX) as live previews |
| `javascript_repl` | Execute JavaScript in a browser sandbox |
| `extract_document` | Fetch and extract text from PDF/DOCX/XLSX/PPTX by URL |

---

## Sub-Agent Architecture Deep Dive

OpenClaw has **two completely different sub-agent systems** at different layers:

### System A: openclaw Production Sub-Agents (Layer 2)

**Tools:** `sessions_spawn` + `subagents` + `agents_list`
**Source:** `openclaw/src/agents/tools/sessions-spawn-tool.ts`, `subagents-tool.ts`

This is the **production system** used in deployed openclaw instances. Sub-agents are:

1. **Server-side sessions** — created via the gateway as new agent sessions
2. **Persistent** — have their own session key, conversation history, and token tracking
3. **Managed** — tracked in a `SubagentRunRecord` registry with status, timing, outcome
4. **Hierarchical** — parent-child relationships tracked via `spawnedBy` field; depth enforced by `maxSpawnDepth` config
5. **Announced** — results automatically delivered back to the parent chat when complete

**Lifecycle:**

```
Parent agent calls sessions_spawn(task, label, agentId, model)
    → spawnSubagentDirect() creates isolated session via gateway
    → Gateway creates new agent session with:
        - Unique session key (namespaced under parent)
        - Separate conversation context
        - Optional model/thinking overrides
    → Child runs to completion independently
    → Result announced back to parent's chat
    → SubagentRunRecord updated with outcome
```

**Depth control:**
```
Config: agents.defaults.subagents.maxSpawnDepth (default: 1)

Depth 0: root agent (can spawn children)
Depth 1: child agent (leaf by default, cannot spawn unless maxSpawnDepth > 1)
Depth N: limited by config
```

**Steer mechanism (unique to openclaw):**
```
Parent calls subagents(action: "steer", target: "scout-1", message: "also check the tests")
    → Rate limit check (2s cooldown per parent-child pair)
    → Suppress announce for interrupted run
    → Abort child's current embedded pi run
    → Clear session queues (pending messages, lane work)
    → Wait for interrupted run to settle (5s timeout)
    → Inject new message into child's EXISTING session context
    → Child restarts with full prior conversation + new steering instruction
    → SubagentRunRecord replaced with new run ID
```

**Cascade kill:**
```
Parent calls subagents(action: "kill", target: "orchestrator-1")
    → Kill the direct target
    → Recursively find all runs where parentChildSessionKey = target's session key
    → Kill each descendant
    → Continue recursion for grandchildren
    → Report total killed count
```

### System B: pi-mono Extension Sub-Agents (Layer 4)

**Tool:** `subagent` (from example extension)
**Source:** `pi-mono/packages/coding-agent/examples/extensions/subagent/index.ts`

This is the **reference implementation** for CLI-based sub-agent orchestration. Sub-agents are:

1. **OS processes** — spawned via `child_process.spawn("pi", [args])`
2. **Ephemeral** — run with `--no-session` (no persistent state)
3. **Isolated** — separate context window, separate model, separate tool set
4. **Three modes** — single, parallel (max 8, 4 concurrent), chain (sequential with `{previous}` placeholder)

**Agent definitions** are Markdown files with YAML frontmatter:

```markdown
---
name: scout
description: Fast codebase recon
tools: read, grep, find, ls, bash
model: claude-haiku-4-5
---
You are a scout. Quickly investigate a codebase...
```

**Pre-built agents:**

| Agent | Model | Tools | Purpose |
|-------|-------|-------|---------|
| `scout` | claude-haiku-4-5 | read, grep, find, ls, bash | Fast recon, returns compressed context |
| `planner` | claude-sonnet-4-5 | read, grep, find, ls | Creates implementation plans, NO writes |
| `reviewer` | claude-sonnet-4-5 | read, grep, find, ls, bash | Code review (bash for git diff only) |
| `worker` | claude-sonnet-4-5 | all defaults | General-purpose implementation |

**Pre-built workflows** (prompt templates):

| Workflow | Chain |
|----------|-------|
| `/implement <query>` | scout (Haiku) → planner (Sonnet) → worker (Sonnet) |
| `/scout-and-plan <query>` | scout → planner (no implementation) |
| `/implement-and-review <query>` | worker → reviewer → worker (apply feedback) |

---

## When and Why Sub-Agents Are Triggered

### The LLM Decides

In both systems, **the LLM autonomously decides when to spawn sub-agents**. There is no programmatic rule that triggers spawning. The LLM sees the tool descriptions and decides based on the task.

However, the LLM's decision is **guided** by:

### 1. System Prompt Instructions

The openclaw system prompt can include agent-specific instructions that mention when to use `sessions_spawn`. For example, an orchestrator agent's system prompt might say:

> "When the user asks for a complex implementation, spawn a scout sub-agent first to gather context, then a worker sub-agent to implement."

### 2. Workflow Prompt Templates (pi-mono)

The `/implement`, `/scout-and-plan`, and `/implement-and-review` commands expand into explicit instructions that tell the LLM to call the `subagent` tool in a specific chain pattern.

### 3. Tool Descriptions as Guidance

The tool description itself guides usage:
- `sessions_spawn`: "Spawn a **background** sub-agent run" — signals async, fire-and-forget use
- `subagent`: "Delegate tasks to **specialized** subagents with **isolated context**" — signals delegation and context separation

### Use Cases That Compel Sub-Agent Spawning

Based on the architecture and agent definitions, these are the real-world patterns:

#### Use Case 1: Context Window Management
**Problem:** A task requires reading 50 files, but that would overwhelm the parent's context window.
**Solution:** Spawn a `scout` sub-agent (cheap Haiku model) to read all files in its own context window and return a compressed summary.

```
LLM calls: subagent({ agent: "scout", task: "find all authentication code in this repo" })
→ Scout reads 50 files in its own 200K context window
→ Returns: structured findings (key files, interfaces, architecture)
→ Parent continues with compressed context, never reading those 50 files
```

#### Use Case 2: Cost Optimization
**Problem:** Scanning a large codebase with an expensive model wastes money on mechanical work.
**Solution:** Use Haiku for reconnaissance, Sonnet for planning, Sonnet for implementation.

```
Chain: scout (Haiku $0.80/M) → planner (Sonnet $3/M) → worker (Sonnet $3/M)
vs. one agent doing everything with Opus ($15/M)
```

#### Use Case 3: Parallel Independent Tasks
**Problem:** Need to investigate both the frontend and backend simultaneously.
**Solution:** Spawn two scouts in parallel.

```
LLM calls: subagent({
  tasks: [
    { agent: "scout", task: "find all React components related to auth" },
    { agent: "scout", task: "find all API endpoints related to auth" }
  ]
})
→ Both run concurrently (max 4 concurrent)
→ Results returned together
```

#### Use Case 4: Separation of Concerns (Tool Restriction)
**Problem:** A planning phase shouldn't accidentally modify files.
**Solution:** The `planner` agent only has read-only tools: `read, grep, find, ls`.

```
The planner agent literally cannot call edit, write, or bash
→ Architectural enforcement of read-only planning
→ Worker agent gets full tools for implementation
```

#### Use Case 5: Long-Running Background Tasks
**Problem:** A complex task takes 10+ minutes and the user wants to continue chatting.
**Solution:** `sessions_spawn` creates a background session that runs independently.

```
LLM calls: sessions_spawn({
  task: "Refactor all 200 test files to use the new testing framework",
  label: "test-migration",
  model: "claude-sonnet-4-5",
  runTimeoutSeconds: 600
})
→ Parent immediately returns: "Spawned 'test-migration' — I'll announce results when done"
→ User continues chatting with the parent
→ 8 minutes later: result announced in parent's chat
```

#### Use Case 6: Mid-Execution Course Correction
**Problem:** A sub-agent is working on the wrong approach.
**Solution:** Steer it without losing its accumulated context.

```
LLM calls: subagents({
  action: "steer",
  target: "test-migration",
  message: "Skip the integration tests, only migrate unit tests"
})
→ Sub-agent's current work is aborted
→ New message injected into its existing conversation (all prior context preserved)
→ Sub-agent restarts with corrected instructions
```

#### Use Case 7: Review-and-Fix Loop
**Problem:** Code needs to be implemented then reviewed then fixed.
**Solution:** Chain with feedback loop.

```
/implement-and-review add input validation to API endpoints
→ worker: implements validation
→ reviewer: finds 3 issues (SQL injection risk, missing rate limit, unclear error messages)
→ worker: receives review feedback via {previous}, fixes all 3 issues
```

#### Use Case 8: Agent-to-Agent Communication
**Problem:** One agent needs information that another agent has accumulated in its session.
**Solution:** `sessions_send` sends a message to another session and optionally waits for the reply.

```
LLM calls: sessions_send({
  label: "data-analyst",
  message: "What were the top 3 error patterns from yesterday's logs?",
  timeoutSeconds: 30
})
→ Message injected into the data-analyst's session
→ Data-analyst processes it with its full context
→ Reply returned to the requester within 30s
```

---

## Key Takeaways for loop_core

### What Makes This Sub-Agent System Powerful

1. **The LLM is the orchestrator** — No rules engine decides when to spawn. The LLM sees the full task, knows it has `sessions_spawn`/`subagent` available, and decides. This is simpler and more flexible than programmatic orchestration.

2. **Context isolation is the killer feature** — Each sub-agent gets a fresh context window. A scout can read 50 files without polluting the parent's context. This solves the #1 problem in agentic systems: context window exhaustion on large tasks.

3. **Cost tiering through model selection** — The scout uses Haiku ($0.80/M), the planner uses Sonnet ($3/M), the worker uses Sonnet ($3/M). The expensive model is only used where it matters. loop_core's Phase 1/Phase 2 split already does this within a single agent; sub-agents extend it across agents.

4. **Tool restriction enforces architectural boundaries** — The planner literally cannot write files. The reviewer can only read and run git commands. This prevents cognitive drift where a "planning" phase accidentally starts implementing.

5. **Steering preserves context** — When you steer a sub-agent, its entire prior conversation is preserved. It doesn't start from scratch. This is uniquely powerful for course-correcting long-running tasks.

6. **Two systems for two needs** — The openclaw layer (sessions-based, persistent, gateway-managed) handles production multi-agent systems. The pi-mono extension (process-based, ephemeral) handles developer workflows. loop_core should consider which pattern fits its autonomous agent model.

### What loop_core Should Build

For loop_core's autonomous heartbeat agents, the openclaw production model (`sessions_spawn` + `subagents`) is more relevant than the pi-mono CLI model. Key additions:

1. **`spawn_child(task, model, tools, timeout)`** — Creates a child `AgenticLoop` with its own `AtomicState`, tool subset, and model configuration. Child runs in the same process (ThreadPoolExecutor) with depth limits.

2. **`list_children()`** / **`kill_child(target)`** — Management tools that the parent agent's LLM can call to monitor and terminate child loops.

3. **`steer_child(target, message)`** — Injects a new message into a child loop's context without losing its accumulated state. This maps cleanly to loop_core's `AtomicState` — inject the steer message as a high-priority `pending_action`.

4. **Tool deny-lists by depth** — Leaf agents (deepest depth) get read-only tools. Only the root agent can send emails, make API calls, etc.

5. **Result announcement** — When a child completes, inject its output into the parent's `AtomicState.variables` or `pending_actions` so the parent's next Phase 1 sees it.

6. **Per-agent model routing** — Scout agents use the cheap Phase 2 model for everything. Planner agents use the expensive Phase 1 model. Worker agents use the standard two-phase split.
