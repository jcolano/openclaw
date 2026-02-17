# OpenClaw Complete Tool Inventory

> Exhaustive reference of every tool across all layers of openclaw/pi-mono.
> For each tool: name, source path, description, full parameters, key behaviors.

---

## Quick Reference: All Tools by Layer

| # | Tool Name | Layer | Source Path |
|---|-----------|-------|-------------|
| 1 | `read` | Core | `pi-mono/packages/coding-agent/src/core/tools/read.ts` |
| 2 | `bash` | Core | `pi-mono/packages/coding-agent/src/core/tools/bash.ts` |
| 3 | `edit` | Core | `pi-mono/packages/coding-agent/src/core/tools/edit.ts` |
| 4 | `write` | Core | `pi-mono/packages/coding-agent/src/core/tools/write.ts` |
| 5 | `grep` | Core | `pi-mono/packages/coding-agent/src/core/tools/grep.ts` |
| 6 | `find` | Core | `pi-mono/packages/coding-agent/src/core/tools/find.ts` |
| 7 | `ls` | Core | `pi-mono/packages/coding-agent/src/core/tools/ls.ts` |
| 8 | `sessions_spawn` | Platform | `openclaw/src/agents/tools/sessions-spawn-tool.ts` |
| 9 | `subagents` | Platform | `openclaw/src/agents/tools/subagents-tool.ts` |
| 10 | `agents_list` | Platform | `openclaw/src/agents/tools/agents-list-tool.ts` |
| 11 | `sessions_list` | Platform | `openclaw/src/agents/tools/sessions-list-tool.ts` |
| 12 | `sessions_send` | Platform | `openclaw/src/agents/tools/sessions-send-tool.ts` |
| 13 | `sessions_history` | Platform | `openclaw/src/agents/tools/sessions-history-tool.ts` |
| 14 | `session_status` | Platform | `openclaw/src/agents/tools/session-status-tool.ts` |
| 15 | `message` | Platform | `openclaw/src/agents/tools/message-tool.ts` |
| 16 | `tts` | Platform | `openclaw/src/agents/tools/tts-tool.ts` |
| 17 | `web_search` | Platform | `openclaw/src/agents/tools/web-search.ts` |
| 18 | `web_fetch` | Platform | `openclaw/src/agents/tools/web-fetch.ts` |
| 19 | `browser` | Platform | `openclaw/src/agents/tools/browser-tool.ts` |
| 20 | `canvas` | Platform | `openclaw/src/agents/tools/canvas-tool.ts` |
| 21 | `image` | Platform | `openclaw/src/agents/tools/image-tool.ts` |
| 22 | `cron` | Platform | `openclaw/src/agents/tools/cron-tool.ts` |
| 23 | `gateway` | Platform | `openclaw/src/agents/tools/gateway-tool.ts` |
| 24 | `nodes` | Platform | `openclaw/src/agents/tools/nodes-tool.ts` |
| 25 | `memory_search` | Platform | `openclaw/src/agents/tools/memory-tool.ts` |
| 26 | `memory_get` | Platform | `openclaw/src/agents/tools/memory-tool.ts` |
| 27 | `llm-task` | Extension | `openclaw/extensions/llm-task/src/llm-task-tool.ts` |
| 28 | `lobster` | Extension | `openclaw/extensions/lobster/src/lobster-tool.ts` |
| 29 | `voice_call` | Extension | `openclaw/extensions/voice-call/index.ts` |
| 30 | `zalouser` | Extension | `openclaw/extensions/zalouser/src/tool.ts` |
| 31 | `feishu_bitable_get_meta` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 32 | `feishu_bitable_list_fields` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 33 | `feishu_bitable_list_records` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 34 | `feishu_bitable_get_record` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 35 | `feishu_bitable_create_record` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 36 | `feishu_bitable_update_record` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 37 | `feishu_bitable_create_app` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 38 | `feishu_bitable_create_field` | Extension | `openclaw/extensions/feishu/src/bitable.ts` |
| 39 | `feishu_doc` | Extension | `openclaw/extensions/feishu/src/docx.ts` |
| 40 | `feishu_app_scopes` | Extension | `openclaw/extensions/feishu/src/docx.ts` |
| 41 | `feishu_wiki` | Extension | `openclaw/extensions/feishu/src/wiki.ts` |
| 42 | `feishu_drive` | Extension | `openclaw/extensions/feishu/src/drive.ts` |
| 43 | `feishu_perm` | Extension | `openclaw/extensions/feishu/src/perm.ts` |
| 44 | `memory_recall` | Extension | `openclaw/extensions/memory-lancedb/index.ts` |
| 45 | `memory_store` | Extension | `openclaw/extensions/memory-lancedb/index.ts` |
| 46 | `memory_forget` | Extension | `openclaw/extensions/memory-lancedb/index.ts` |
| 47 | `subagent` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/subagent/index.ts` |
| 48 | `hello` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/hello.ts` |
| 49 | `question` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/question.ts` |
| 50 | `questionnaire` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/questionnaire.ts` |
| 51 | `rg` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/truncated-tool.ts` |
| 52 | `read` (audited) | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/tool-override.ts` |
| 53 | `read/write/edit/bash` (SSH) | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/ssh.ts` |
| 54 | `generate_image` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/antigravity-image-gen.ts` |
| 55 | `bash` (sandboxed) | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/sandbox/index.ts` |
| 56 | `parse_duration` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/with-deps/index.ts` |
| 57 | `todo` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/todo.ts` |
| 58 | `finish_and_exit` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/shutdown-command.ts` |
| 59 | `deploy_and_exit` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/shutdown-command.ts` |
| 60 | `reload_runtime` | Example Ext | `pi-mono/packages/coding-agent/examples/extensions/reload-runtime.ts` |
| 61 | `artifacts` | Web UI | `pi-mono/packages/web-ui/src/tools/artifacts/artifacts.ts` |
| 62 | `javascript_repl` | Web UI | `pi-mono/packages/web-ui/src/tools/javascript-repl.ts` |
| 63 | `extract_document` | Web UI | `pi-mono/packages/web-ui/src/tools/extract-document.ts` |
| 64 | `attach` | Mom (sandbox) | `pi-mono/packages/mom/src/tools/attach.ts` |

---

## LAYER 1: pi-mono Core Tools (7 tools)

Source directory: `pi-mono/packages/coding-agent/src/core/tools/`

Tool groups (presets):
- **`codingTools`**: read, bash, edit, write
- **`readOnlyTools`**: read, grep, find, ls
- **`allTools`**: all 7

All core tools support pluggable `operations` for remote execution (SSH, sandbox).
All core tools are wrapped by the extension middleware pipeline (`wrapper.ts`) which fires `tool_call` (pre, can block) and `tool_result` (post, can modify) events.

---

### 1. `read`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/src/core/tools/read.ts` |
| **Name** | `read` |
| **Label** | `read` |
| **Group** | codingTools, readOnlyTools |

**Description:**
> Read the contents of a file. Supports text files and images (jpg, png, gif, webp). Images are sent as attachments. For text files, output is truncated to 2000 lines or 50KB (whichever is hit first). Use offset/limit for large files. When you need the full file, continue with offset until complete.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `path` | string | **Yes** | — | Path to the file to read (relative or absolute) |
| `offset` | number | No | 1 | Line number to start reading from (1-indexed) |
| `limit` | number | No | 2000 | Maximum number of lines to read |

**Key behaviors:**
- Text files: UTF-8 decoded, line-paginated. Head-truncated at 2000 lines or 50KB (whichever first).
- Images (jpg/png/gif/webp): returned as base64 with MIME type. Auto-resized to 2000x2000 max (configurable via `autoResizeImages` option).
- Truncation appends: `[Showing lines X-Y of Z. Use offset=N to continue.]`
- macOS path quirks handled: NFD normalization, curly quotes in screenshot names, `@`-prefix stripping, `~` expansion.
- Requires `R_OK` filesystem access.

---

### 2. `bash`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/src/core/tools/bash.ts` |
| **Name** | `bash` |
| **Label** | `bash` |
| **Group** | codingTools |

**Description:**
> Execute a bash command in the current working directory. Returns stdout and stderr. Output is truncated to last 2000 lines or 50KB (whichever is hit first). If truncated, full output is saved to a temp file. Optionally provide a timeout in seconds.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `command` | string | **Yes** | — | Bash command to execute |
| `timeout` | number | No | none | Timeout in seconds |

**Key behaviors:**
- Spawns a local shell process (`sh` or system shell via `getShellConfig()`).
- **Tail-truncated** (keeps last N lines — shows end of output, appropriate for seeing errors).
- Overflow saved to `/tmp/pi-bash-<hex>.log`.
- Non-zero exit codes returned as errors.
- Process tree killed on abort: SIGTERM then SIGKILL.
- Streaming progress via `onUpdate` callback.
- Configurable via `commandPrefix` (prepended to every command) and `BashSpawnHook` (mutates command/cwd/env before execution).

---

### 3. `edit`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/src/core/tools/edit.ts` |
| **Name** | `edit` |
| **Label** | `edit` |
| **Group** | codingTools |

**Description:**
> Edit a file by replacing exact text. The oldText must match exactly (including whitespace). Use this for precise, surgical edits.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `path` | string | **Yes** | — | Path to the file to edit |
| `oldText` | string | **Yes** | — | Exact text to find and replace (must match exactly) |
| `newText` | string | **Yes** | — | New text to replace the old text with |

**Key behaviors:**
- First tries exact match, then fuzzy match (normalizes trailing whitespace per line, Unicode smart quotes to ASCII, Unicode dashes to `-`, Unicode spaces to space).
- Rejects if `oldText` matches 0 times (not found) or >1 times (ambiguous).
- Rejects if replacement is identical to original.
- Preserves BOM and original line endings (CRLF vs LF).
- Returns unified diff on success.
- Requires `R_OK | W_OK` filesystem access.

---

### 4. `write`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/src/core/tools/write.ts` |
| **Name** | `write` |
| **Label** | `write` |
| **Group** | codingTools |

**Description:**
> Write content to a file. Creates the file if it doesn't exist, overwrites if it does. Automatically creates parent directories.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `path` | string | **Yes** | — | Path to the file to write |
| `content` | string | **Yes** | — | Content to write to the file |

**Key behaviors:**
- Creates parent directories recursively (`mkdir -p`).
- Completely overwrites existing files.
- Returns byte count written.

---

### 5. `grep`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/src/core/tools/grep.ts` |
| **Name** | `grep` |
| **Label** | `grep` |
| **Group** | readOnlyTools |

**Description:**
> Search file contents for a pattern. Returns matching lines with file paths and line numbers. Respects .gitignore. Output is truncated to 100 matches or 50KB (whichever is hit first). Long lines are truncated to 500 chars.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `pattern` | string | **Yes** | — | Search pattern (regex or literal) |
| `path` | string | No | cwd | Directory or file to search |
| `glob` | string | No | — | Filter files by glob pattern (e.g. `*.ts`) |
| `ignoreCase` | boolean | No | false | Case-insensitive search |
| `literal` | boolean | No | false | Treat pattern as literal string |
| `context` | number | No | 0 | Lines before and after each match |
| `limit` | number | No | 100 | Maximum number of matches |

**Key behaviors:**
- Backed by ripgrep (`rg`) with `--json` and `--hidden` flags. Respects `.gitignore`.
- Auto-downloads `rg` via `ensureTool` if not available.
- Results formatted as `path:lineNumber: text`.
- Process killed once match limit reached.
- Lines longer than 500 chars truncated with `... [truncated]`.

---

### 6. `find`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/src/core/tools/find.ts` |
| **Name** | `find` |
| **Label** | `find` |
| **Group** | readOnlyTools |

**Description:**
> Search for files by glob pattern. Returns matching file paths relative to the search directory. Respects .gitignore. Output is truncated to 1000 results or 50KB (whichever is hit first).

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `pattern` | string | **Yes** | — | Glob pattern (e.g. `*.ts`, `**/*.json`) |
| `path` | string | No | cwd | Directory to search in |
| `limit` | number | No | 1000 | Maximum results |

**Key behaviors:**
- Backed by `fd` binary with `--glob`, `--hidden`, `--color=never`.
- Auto-downloads `fd` via `ensureTool` if not available.
- Auto-discovers nested `.gitignore` files and passes them as `--ignore-file`.
- Returns relative paths.

---

### 7. `ls`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/src/core/tools/ls.ts` |
| **Name** | `ls` |
| **Label** | `ls` |
| **Group** | readOnlyTools |

**Description:**
> List directory contents. Returns entries sorted alphabetically, with '/' suffix for directories. Includes dotfiles. Output is truncated to 500 entries or 50KB (whichever is hit first).

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `path` | string | No | cwd | Directory to list |
| `limit` | number | No | 500 | Maximum entries |

**Key behaviors:**
- `readdirSync`, sorted case-insensitively.
- Appends `/` to directory entries.
- Includes dotfiles.

---

## LAYER 2: openclaw Platform Tools (19 tools)

Source directory: `openclaw/src/agents/tools/`

These tools are created by factory functions (`createXxxTool(opts)`) and injected into the agent based on configuration and permissions. Many accept optional `gatewayUrl`, `gatewayToken`, `timeoutMs` parameters for gateway communication overrides.

---

### 8. `sessions_spawn`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/sessions-spawn-tool.ts` |
| **Name** | `sessions_spawn` |
| **Label** | `Sessions` |
| **Category** | Sub-agent orchestration |

**Description:**
> Spawn a background sub-agent run in an isolated session and announce the result back to the requester chat.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `task` | string | **Yes** | — | The instruction the sub-agent receives |
| `label` | string | No | — | Human-readable label for the spawned session |
| `agentId` | string | No | — | Target a specific agent config (different system prompt/model/tools) |
| `model` | string | No | — | Override the model for this sub-agent run |
| `thinking` | string | No | — | Override thinking level |
| `runTimeoutSeconds` | number | No | — | Kill the sub-agent after N seconds (min: 0) |
| `timeoutSeconds` | number | No | — | Back-compat alias for `runTimeoutSeconds` |
| `cleanup` | `"delete"` \| `"keep"` | No | `"keep"` | Auto-delete session after completion |

**Key behaviors:**
- Calls `spawnSubagentDirect()` which creates an isolated session via the gateway.
- The child session gets its own session key, conversation history, and token tracking.
- Child runs independently; result announced back to parent's chat when complete.
- Parent-child relationship tracked via `spawnedBy` field.
- Depth enforced by `agents.defaults.subagents.maxSpawnDepth` config (default: 1).
- Constructor receives routing context: `agentSessionKey`, `agentChannel`, `agentAccountId`, `agentTo`, `agentThreadId`, `agentGroupId`, `agentGroupChannel`, `agentGroupSpace`, `sandboxed`, `requesterAgentIdOverride`.

---

### 9. `subagents`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/subagents-tool.ts` |
| **Name** | `subagents` |
| **Label** | `Subagents` |
| **Category** | Sub-agent orchestration |

**Description:**
> List, kill, or steer spawned sub-agents for this requester session. Use this for sub-agent orchestration.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `action` | `"list"` \| `"kill"` \| `"steer"` | **Yes** | — | Management action |
| `target` | string | No | — | Target sub-agent (index, "last", label prefix, runId prefix, "all", "*") |
| `message` | string | No | — | New instruction for `steer` action |
| `recentMinutes` | number | No | — | Time window filter for `list` |

**Key behaviors:**

**`list`**: Shows all active and recent sub-agents with status, runtime, model, token usage.

**`kill`**: Terminates a running sub-agent. Supports `target: "all"` / `"*"`. **Cascade kills** all descendants recursively (grandchildren, etc.). Aborts the embedded pi run, clears session queues, marks run as terminated.

**`steer`**: Aborts the sub-agent's current work, waits for it to settle (5s timeout), then injects a new message into the same session context. Sub-agent restarts with full prior conversation preserved plus new steering instruction. Rate-limited: 2s cooldown per parent-child pair. Self-steering forbidden.

---

### 10. `agents_list`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/agents-list-tool.ts` |
| **Name** | `agents_list` |
| **Label** | `Agents` |
| **Category** | Sub-agent orchestration |

**Description:**
> List agent ids you can target with sessions_spawn (based on allowlists).

**Parameters:** None (empty schema).

**Key behaviors:**
- Reads `agents.list` from config for configured agent entries.
- Reads `subagents.allowAgents` from the requester's agent config. `"*"` allows all.
- Returns `{ requester, allowAny, agents: [{ id, name?, configured }] }`.
- Always includes the requester's own agent ID.

---

### 11. `sessions_list`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/sessions-list-tool.ts` |
| **Name** | `sessions_list` |
| **Label** | `Sessions` |
| **Category** | Session management |

**Description:**
> List sessions with optional filters and last messages.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `kinds` | string[] | No | all | Filter: `main`, `group`, `cron`, `hook`, `node`, `other` |
| `limit` | number | No | — | Maximum sessions to return (min: 1) |
| `activeMinutes` | number | No | — | Only sessions active in last N minutes (min: 1) |
| `messageLimit` | number | No | 0 | Last N messages per session (0-20) |

**Key behaviors:**
- Calls gateway `sessions.list`.
- Applies session visibility guards based on sandbox policy and A2A policy.
- Classifies each session into kinds (main, group, cron, hook, node, other).
- Optionally fetches last N messages per session (max 4 concurrent, tool messages stripped).
- Resolves transcript paths for each session.
- Returns `{ count, sessions: SessionListRow[] }`.

---

### 12. `sessions_send`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/sessions-send-tool.ts` |
| **Name** | `sessions_send` |
| **Label** | `Session Send` |
| **Category** | Session management |

**Description:**
> Send a message into another session. Use sessionKey or label to identify the target.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `sessionKey` | string | No | — | Target session key (mutually exclusive with `label`) |
| `label` | string | No | — | Target session label (max length enforced) |
| `agentId` | string | No | — | Agent ID for label-based lookup (max 64 chars) |
| `message` | string | **Yes** | — | Message to send |
| `timeoutSeconds` | number | No | 30 | Wait timeout; 0 = fire-and-forget (min: 0) |

**Key behaviors:**
- `sessionKey` and `label` are mutually exclusive.
- Agent-to-agent (A2A) policy checked: `tools.agentToAgent.enabled` must be true for cross-agent sends.
- With `timeoutSeconds: 0`: fire-and-forget (returns immediately with `status: "accepted"`).
- With `timeoutSeconds > 0`: waits for the target session's reply via `agent.wait`.
- After completion, starts A2A announcement flow for result delivery.
- Sandboxed sessions can only send to sessions they spawned.
- Visibility guards applied based on config.

---

### 13. `sessions_history`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/sessions-history-tool.ts` |
| **Name** | `sessions_history` |
| **Label** | `Session History` |
| **Category** | Session management |

**Description:**
> Fetch message history for a session.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `sessionKey` | string | **Yes** | — | Session to fetch history for |
| `limit` | number | No | — | Maximum messages (min: 1) |
| `includeTools` | boolean | No | false | Include tool call/result messages |

**Key behaviors:**
- Calls gateway `chat.history`.
- Tool messages stripped unless `includeTools=true`.
- History sanitized: text truncated to 4000 chars per block, `thinkingSignature` removed, image `data` replaced with `{ omitted: true, bytes }`, `details`/`usage`/`cost` fields removed.
- Hard cap: 80KB total JSON size. If exceeded, only last message kept. If still too large, replaced with placeholder.
- Returns `{ sessionKey, messages, truncated, droppedMessages, contentTruncated, bytes }`.

---

### 14. `session_status`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/session-status-tool.ts` |
| **Name** | `session_status` |
| **Label** | `Session Status` |
| **Category** | Session management |

**Description:**
> Show a /status-equivalent session status card (usage + time + cost when available). Optional: set per-session model override (model=default resets overrides).

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `sessionKey` | string | No | — | Session to check |
| `model` | string | No | — | Set model override (`"default"` resets) |

---

### 15. `message`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/message-tool.ts` |
| **Name** | `message` |
| **Label** | `Message` |
| **Category** | Communication |

**Description (dynamic):**
> Send, delete, and manage messages via channel plugins. Current channel supports: [action list based on active channel].

**Parameters (comprehensive — dynamically filtered by channel):**

**Routing:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | string enum | **Yes** | `send`, `reply`, `delete`, `react`, `reactions`, `poll`, `pin`, `unpin`, `listPins`, `thread-reply`, `threadCreate`, `threadList`, `broadcast`, `sendWithEffect`, `sendAttachment`, `searchMessages`, `memberInfo`, `roleInfo`, `emojiList`, `emojiUpload`, `stickerUpload`, `roleAdd`, `roleRemove`, `channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`, `channelCreate`, `channelEdit`, `channelDelete`, `channelMove`, `categoryCreate`, `categoryEdit`, `categoryDelete`, `channelPermissionSet`, `channelPermissionRemove`, `timeout`, `kick`, `ban`, `setPresence` |
| `channel` | string | No | Channel name/id |
| `target` | string | No | Target channel/user id or name |
| `targets` | string[] | No | Multiple targets |
| `accountId` | string | No | Account id override |
| `dryRun` | boolean | No | Dry-run mode |

**Messaging:**

| Name | Type | Description |
|------|------|-------------|
| `message` | string | Message text |
| `media` | string | Media URL or local path |
| `filename` | string | Attachment filename |
| `buffer` | string | Base64 payload for attachments |
| `contentType` | string | MIME content type |
| `caption` | string | Media caption |
| `replyTo` | string | Message id to reply to |
| `threadId` | string | Thread id |
| `asVoice` | boolean | Send as voice message |
| `silent` | boolean | Silent send |
| `buttons` | array | Telegram inline keyboard rows |
| `card` | object | Adaptive Card JSON |
| `components` | object | Discord Components v2 payload |
| `effectId` | string | iMessage effect name |

**Reactions:** `messageId`, `emoji`, `remove`, `targetAuthor`
**Fetch:** `limit`, `before`, `after`, `around`, `fromMe`
**Poll:** `pollQuestion`, `pollOption`, `pollDurationHours`, `pollMulti`
**Thread:** `threadName`, `autoArchiveMin`
**Events:** `query`, `eventName`, `eventType`, `startTime`, `endTime`, `location`, `durationMin`
**Moderation:** `reason`, `deleteDays`
**Channel management:** `name`, `type`, `parentId`, `topic`, `position`, `nsfw`, `categoryId`
**Presence:** `activityType`, `activityName`, `activityUrl`, `activityState`, `status`

**Key behaviors:**
- Dispatches to channel-specific handlers: Discord (30+ actions), Slack (11), Telegram (7), WhatsApp (1).
- `<think>...</think>` reasoning tags stripped from message text before sending.
- Abort check on entry: throws `AbortError` if signal already aborted.
- `requireExplicitTarget` mode enforces explicit target for send-type actions.
- Schema is dynamically built based on configured channel plugins and current channel.

---

### 16. `tts`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/tts-tool.ts` |
| **Name** | `tts` |
| **Label** | `TTS` |
| **Category** | Communication |

**Description:**
> Convert text to speech. Audio is delivered automatically from the tool result — reply with [SILENT_REPLY_TOKEN] after a successful call to avoid duplicate messages.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `text` | string | **Yes** | — | Text to convert to speech |
| `channel` | string | No | — | Channel id for output format selection |

**Key behaviors:**
- Returns `MEDIA:<audioPath>` on success.
- If `result.voiceCompatible` is true, prepends `[[audio_as_voice]]` (Telegram voice bubble).
- On failure: returns error text (does NOT throw).

---

### 17. `web_search`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/web-search.ts` |
| **Name** | `web_search` |
| **Label** | `Web Search` |
| **Category** | Web |

**Description (varies by backend):**
- **Brave:** "Search the web using Brave Search API. Supports region-specific and localized search via country and language parameters. Returns titles, URLs, and snippets for fast research."
- **Perplexity:** "Search the web using Perplexity Sonar. Returns AI-synthesized answers with citations from real-time web search."
- **Grok:** "Search the web using xAI Grok. Returns AI-synthesized answers with citations from real-time web search."

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `query` | string | **Yes** | — | Search query |
| `count` | number | No | — | Number of results (1-10) |
| `country` | string | No | — | Country code for region-specific results |
| `search_lang` | string | No | — | Search language |
| `ui_lang` | string | No | — | UI language |
| `freshness` | string | No | — | Time filter for results |

**Key behaviors:**
- Returns `null` (tool not created) if no search provider is configured.
- Supports three backends: Brave Search API, Perplexity Sonar (direct or via OpenRouter), xAI Grok.
- Results wrapped with `wrapWebContent` (marks as untrusted external content).
- Caching with configurable TTL.

---

### 18. `web_fetch`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/web-fetch.ts` |
| **Name** | `web_fetch` |
| **Label** | `Web Fetch` |
| **Category** | Web |

**Description:**
> Fetch and extract readable content from a URL (HTML → markdown/text). Use for lightweight page access without browser automation. When exploring a new domain, also check for /llms.txt or /.well-known/llms.txt — these files describe how AI agents should interact with the site.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `url` | string | **Yes** | — | HTTP or HTTPS URL to fetch |
| `extractMode` | `"markdown"` \| `"text"` | No | `"markdown"` | Extraction mode |
| `maxChars` | number | No | 50,000 | Maximum characters to return (min: 100) |

**Key behaviors:**
- Returns `null` if `tools.web.fetch.enabled` is false.
- SSRF protection via `fetchWithSsrFGuard`.
- Extraction chain: Cloudflare Markdown for Agents (pre-rendered) → Readability → Firecrawl fallback.
- HTML: `extractReadableContent` (Readability library) → if fails, tries Firecrawl API → if fails, throws.
- JSON: pretty-printed.
- Firecrawl backend: optional, enabled if `tools.web.fetch.firecrawl.apiKey` or `FIRECRAWL_API_KEY` env var set.
- Caching with configurable TTL (`tools.web.fetch.cacheTtlMinutes`).
- All content wrapped with `wrapWebContent` / `wrapExternalContent` (untrusted marker).
- Configurable: `maxCharsCap`, `maxResponseBytes` (32KB–10MB, default 2MB), `maxRedirects` (default 3), `timeoutSeconds`, `userAgent`.
- Constants: `DEFAULT_FETCH_MAX_CHARS = 50,000`, `DEFAULT_FETCH_MAX_RESPONSE_BYTES = 2,000,000`, `DEFAULT_FIRECRAWL_MAX_AGE_MS = 172,800,000` (48 hours).

---

### 19. `browser`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/browser-tool.ts` |
| **Schema** | `openclaw/src/agents/tools/browser-tool.schema.ts` |
| **Name** | `browser` |
| **Label** | `Browser` |
| **Category** | Browser & Visual |

**Description:**
> Control the browser via OpenClaw's browser control server (status/start/stop/profiles/tabs/open/snapshot/screenshot/actions). Profiles: use profile="chrome" for Chrome extension relay takeover (your existing Chrome tabs). Use profile="openclaw" for the isolated openclaw-managed browser. [...]

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | string enum | **Yes** | `status`, `start`, `stop`, `profiles`, `tabs`, `open`, `focus`, `close`, `snapshot`, `screenshot`, `navigate`, `console`, `pdf`, `upload`, `dialog`, `act` |
| `target` | `"sandbox"` \| `"host"` \| `"node"` | No | Browser location |
| `node` | string | No | Node id/name (only with `target="node"`) |
| `profile` | string | No | `"chrome"` or `"openclaw"` |
| `targetUrl` | string | No | URL for `open`/`navigate` |
| `targetId` | string | No | Browser tab/target id |
| `limit` | number | No | Snapshot result limit |
| `maxChars` | number | No | Max chars in snapshot output |
| `mode` | `"efficient"` | No | Efficient snapshot mode |
| `snapshotFormat` | `"aria"` \| `"ai"` | No | Snapshot format |
| `refs` | `"role"` \| `"aria"` | No | Element reference style |
| `interactive` | boolean | No | Snapshot interactive filter |
| `compact` | boolean | No | Compact snapshot output |
| `depth` | number | No | Snapshot DOM depth |
| `selector` | string | No | CSS selector for snapshot scope |
| `frame` | string | No | Frame name/selector |
| `labels` | boolean | No | Visual labels in snapshot |
| `fullPage` | boolean | No | Screenshot full page |
| `ref` | string | No | Element ref for screenshot/upload/act |
| `element` | string | No | Element selector |
| `type` | `"png"` \| `"jpeg"` | No | Screenshot image type |
| `level` | string | No | Console log level filter |
| `paths` | string[] | No | File paths for `upload` |
| `inputRef` | string | No | File input ref for `upload` |
| `timeoutMs` | number | No | Timeout for upload/dialog |
| `accept` | boolean | No | Accept/dismiss dialog |
| `promptText` | string | No | Text for prompt dialog |
| `request` | BrowserActSchema | No | Act request object |

**BrowserActSchema (nested `request` parameter):**

| Field | Type | Description |
|-------|------|-------------|
| `kind` | string enum | **Required.** `click`, `type`, `press`, `hover`, `drag`, `select`, `fill`, `resize`, `wait`, `evaluate`, `close` |
| `ref` | string | Element reference |
| `doubleClick` | boolean | Double-click mode |
| `button` | string | Mouse button |
| `modifiers` | string[] | Key modifiers |
| `text` | string | Text to type |
| `submit` | boolean | Submit after typing |
| `slowly` | boolean | Type slowly |
| `key` | string | Key name for `press` |
| `startRef` / `endRef` | string | Drag start/end refs |
| `values` | string[] | Select option values |
| `fields` | object[] | Form fields for `fill` |
| `width` / `height` | number | Viewport size for `resize` |
| `timeMs` | number | Wait duration |
| `textGone` | string | Wait until text disappears |
| `fn` | string | JS function for `evaluate` |

**Key behaviors:**
- Default proxy timeout: 20,000ms.
- Target routing: `profile="chrome"` defaults to `target="host"`. Node auto-routing if `gateway.nodes.browser.mode="auto"` and exactly one browser-capable node.
- `upload`: paths must be within `DEFAULT_UPLOAD_DIR` (path traversal protection).
- All snapshot/console/tabs results wrapped with `wrapExternalContent` (untrusted).
- `snapshot` with `labels=true`: returns both image and text output.
- Config: `browser.enabled`, `gateway.nodes.browser.mode` (`auto`|`manual`|`off`), `agents.defaults.sandbox.browser.enabled`.

---

### 20. `canvas`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/canvas-tool.ts` |
| **Name** | `canvas` |
| **Label** | `Canvas` |
| **Category** | Browser & Visual |

**Description:**
> Control node canvases (present/hide/navigate/eval/snapshot/A2UI). Use snapshot to capture the rendered UI.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | string enum | **Yes** | `present`, `hide`, `navigate`, `eval`, `snapshot`, `a2ui_push`, `a2ui_reset` |
| `node` | string | No | Node id/name (**required at runtime**) |
| `target` / `url` | string | No | URL for `present`/`navigate` |
| `x`, `y`, `width`, `height` | number | No | Placement for `present` |
| `javaScript` | string | No | JS code for `eval` (**required for eval**) |
| `outputFormat` | `"png"` \| `"jpg"` \| `"jpeg"` | No | Snapshot image format |
| `maxWidth` | number | No | Snapshot max width |
| `quality` | number | No | JPEG quality |
| `delayMs` | number | No | Delay before snapshot capture |
| `jsonl` / `jsonlPath` | string | No | JSONL data for `a2ui_push` |
| `gatewayUrl`, `gatewayToken`, `timeoutMs` | — | No | Gateway overrides |

**Key behaviors:**
- `node` always required at runtime (`resolveNodeId` with `required=true`).
- `snapshot` returns base64 image content written to temp file.
- `a2ui_push` requires either `jsonl` (inline) or `jsonlPath` (file read).
- All actions use idempotency keys (`crypto.randomUUID()`).

---

### 21. `image`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/image-tool.ts` |
| **Helpers** | `openclaw/src/agents/tools/image-tool.helpers.ts` |
| **Name** | `image` |
| **Label** | `Image` |
| **Category** | Browser & Visual |

**Description (vision-capable model):**
> Analyze one or more images with a vision model. Use image for a single path/URL, or images for multiple (up to 20). Only use this tool when images were NOT already provided in the user's message.

**Description (non-vision model):**
> Analyze one or more images with the configured image model (agents.defaults.imageModel). Use image for a single path/URL, or images for multiple (up to 20). Provide a prompt describing what to analyze.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `prompt` | string | No | "Describe the image." | Analysis prompt |
| `image` | string | No | — | Single image path or URL |
| `images` | string[] | No | — | Multiple image paths/URLs |
| `model` | string | No | — | Model override (e.g. `"openai/gpt-5-mini"`) |
| `maxBytesMb` | number | No | — | Max image size in MB |
| `maxImages` | number | No | 20 | Max images to process |

**Key behaviors:**
- Returns `null` (tool not created) if no auth available for any image model provider.
- Images deduplicated, `@`-prefix stripped from paths.
- Model fallback chain: explicit config → same-provider vision model → MiniMax → ZAI → OpenAI → Anthropic.
- MiniMax special-cased: single-image only, uses `minimaxUnderstandImage`.
- Sandboxed mode: remote URLs disallowed, paths resolved via `SandboxFsBridge`.
- Model fallback: tries primary then each fallback; all attempt errors recorded in `details.attempts`.

---

### 22. `cron`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/cron-tool.ts` |
| **Name** | `cron` |
| **Label** | `Cron` |
| **Category** | Infrastructure |

**Description:**
> Manage Gateway cron jobs (status/list/add/update/remove/run/runs) and send wake events. [Includes full inline documentation for job schema, schedule types, payload types, delivery modes, and constraints.]

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `action` | string enum | **Yes** | — | `status`, `list`, `add`, `update`, `remove`, `run`, `runs`, `wake` |
| `includeDisabled` | boolean | No | — | Include disabled jobs in `list` |
| `job` | object | No | — | Job creation object for `add` |
| `jobId` | string | No | — | Job identifier for `update`/`remove`/`run`/`runs` |
| `id` | string | No | — | Backward-compat alias for `jobId` |
| `patch` | object | No | — | Patch object for `update` |
| `text` | string | No | — | Wake event text (**required for `wake`**) |
| `mode` | `"now"` \| `"next-heartbeat"` | No | `"next-heartbeat"` | Wake mode |
| `runMode` | `"due"` \| `"force"` | No | `"force"` | Run trigger mode |
| `contextMessages` | number | No | — | Recent messages as context (0-10) |
| `gatewayUrl`, `gatewayToken`, `timeoutMs` | — | No | — | Gateway overrides (default timeout: 60s) |

**Job schema (for `add`):**
- `schedule.kind`: `"at"` (one-shot ISO timestamp), `"every"` (interval ms), `"cron"` (cron expression + tz)
- `payload.kind`: `"systemEvent"` (inject text into session), `"agentTurn"` (run agent with message)
- `delivery.mode`: `"none"`, `"announce"`, `"webhook"`
- `sessionTarget`: `"main"` (requires `systemEvent`) or `"isolated"` (requires `agentTurn`)

**Key behaviors:**
- Flat-params recovery: if `job` is empty, reconstructs from top-level params.
- `agentId` and `sessionKey` auto-injected from session context.
- Delivery target inferred from session key structure for `agentTurn` jobs.
- Context messages: up to 10 recent user/assistant messages appended (per-message max 220 chars, total max 700 chars).
- `webhook` delivery: `delivery.to` must be valid `http(s)` URL.

---

### 23. `gateway`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/gateway-tool.ts` |
| **Name** | `gateway` |
| **Label** | `Gateway` |
| **Category** | Infrastructure |

**Description:**
> Restart, apply config, or update the gateway in-place (SIGUSR1). Use config.patch for safe partial config updates (merges with existing). Use config.apply only when replacing entire config. Both trigger restart after writing. Always pass a human-readable completion message via the `note` parameter so the system can deliver it to the user after restart.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `action` | string enum | **Yes** | — | `restart`, `config.get`, `config.schema`, `config.apply`, `config.patch`, `update.run` |
| `delayMs` | number | No | — | Restart delay |
| `reason` | string | No | — | Restart reason (max 200 chars) |
| `raw` | string | No | — | Config JSON/YAML (**required for apply/patch**) |
| `baseHash` | string | No | auto-fetched | Config hash for CAS (compare-and-swap) |
| `sessionKey` | string | No | current | Session for post-restart delivery |
| `note` | string | No | — | Human-readable message delivered after restart |
| `restartDelayMs` | number | No | — | Delay before restart after config write |
| `gatewayUrl`, `gatewayToken`, `timeoutMs` | — | No | — | Gateway overrides |

**Key behaviors:**
- `restart` requires `commands.restart=true` in config.
- Writes restart sentinel file with delivery context for post-restart message.
- `update.run` timeout: 20 minutes default.
- `config.patch` merges with existing; `config.apply` replaces entirely.

---

### 24. `nodes`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/nodes-tool.ts` |
| **Name** | `nodes` |
| **Label** | `Nodes` |
| **Category** | Infrastructure |

**Description:**
> Discover and control paired nodes (status/describe/pairing/notify/camera/screen/location/run/invoke).

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `action` | string enum | **Yes** | — | `status`, `describe`, `pending`, `approve`, `reject`, `notify`, `camera_snap`, `camera_list`, `camera_clip`, `screen_record`, `location_get`, `run`, `invoke` |
| `node` | string | No | — | Node id/name (required for most actions) |
| `requestId` | string | No | — | Pairing request id for `approve`/`reject` |
| `title` / `body` | string | No | — | Notification content |
| `sound` | string | No | — | Notification sound |
| `priority` | `"passive"` \| `"active"` \| `"timeSensitive"` | No | — | Notification priority |
| `delivery` | `"system"` \| `"overlay"` \| `"auto"` | No | — | Notification delivery method |
| `facing` | `"front"` \| `"back"` \| `"both"` | No | `"both"` | Camera facing for `camera_snap` |
| `maxWidth` / `quality` / `delayMs` | number | No | — | Image capture options |
| `deviceId` | string | No | — | Specific camera device |
| `duration` | string | No | — | Human-readable duration (e.g. `"3s"`) |
| `durationMs` | number | No | 3000/10000 | Duration in ms for clip/screen |
| `includeAudio` | boolean | No | true | Include audio in recording |
| `fps` | number | No | 10 | Screen recording fps |
| `screenIndex` | number | No | 0 | Screen index for recording |
| `command` | string[] | No | — | Command argv for `run` |
| `cwd` / `env` | string/string[] | No | — | Working directory / env vars for `run` |
| `commandTimeoutMs` / `invokeTimeoutMs` | number | No | — | Execution timeouts |
| `needsScreenRecording` | boolean | No | — | Screen recording during `run` |
| `invokeCommand` | string | No | — | Command for `invoke` (**required**) |
| `invokeParamsJson` | string | No | — | JSON params for `invoke` |

**Key behaviors:**
- `camera_snap` with `facing="both"`: captures front + back sequentially.
- `run` action: approval flow — if denied, creates approval request with 120s timeout, waits for allow-once/allow-always/deny.
- `invoke`: raw gateway command invocation with JSON params.
- All write operations use `crypto.randomUUID()` as idempotency key.

---

### 25. `memory_search`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/memory-tool.ts` |
| **Name** | `memory_search` |
| **Label** | `Memory Search` |
| **Category** | Memory |

**Description:**
> Mandatory recall step: semantically search MEMORY.md + memory/*.md (and optional session transcripts) before answering questions about prior work, decisions, dates, people, preferences, or todos; returns top snippets with path + lines.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `query` | string | **Yes** | — | Semantic search query |
| `maxResults` | number | No | — | Maximum results |
| `minScore` | number | No | — | Minimum similarity score |

**Key behaviors:**
- Returns `null` if no memory config.
- On failure: returns `{ results: [], disabled: true, error }` (no throw).
- Citations mode: `on`/`off`/`auto` (config: `memory.citations`). Auto = shown in direct chats, suppressed in groups.
- Citation format: `<path>#L<startLine>-L<endLine>`.

---

### 26. `memory_get`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/src/agents/tools/memory-tool.ts` |
| **Name** | `memory_get` |
| **Label** | `Memory Get` |
| **Category** | Memory |

**Description:**
> Safe snippet read from MEMORY.md or memory/*.md with optional from/lines; use after memory_search to pull only the needed lines and keep context small.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `path` | string | **Yes** | — | Relative path (e.g. `MEMORY.md`, `memory/notes.md`) |
| `from` | number | No | — | Starting line number |
| `lines` | number | No | — | Number of lines to read |

---

## LAYER 3: openclaw Extension Tools (20 tools)

Source directory: `openclaw/extensions/`

Registered at runtime via `api.registerTool()`. Available per deployment based on extension configuration.

---

### 27. `llm-task`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/llm-task/src/llm-task-tool.ts` |
| **Name** | `llm-task` |
| **Label** | `LLM Task` |

**Description:**
> Run a generic JSON-only LLM task and return schema-validated JSON. Designed for orchestration from Lobster workflows via openclaw.invoke.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `prompt` | string | **Yes** | — | Task instruction for the LLM |
| `input` | any | No | — | Optional input payload |
| `schema` | object | No | — | JSON Schema to validate output |
| `provider` | string | No | — | Provider override (e.g. `"openai-codex"`) |
| `model` | string | No | — | Model id override |
| `authProfileId` | string | No | — | Auth profile override |
| `temperature` | number | No | — | Temperature override |
| `maxTokens` | number | No | — | Max tokens override |
| `timeoutMs` | number | No | 30,000 | Timeout |

**Key behaviors:**
- Spawns embedded pi agent with `disableTools: true`.
- Constructs JSON-only system prompt.
- Strips code fences from output.
- Validates against optional JSON schema using AJV.
- Returns `{ json: parsed, provider, model }`.

---

### 28. `lobster`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/lobster/src/lobster-tool.ts` |
| **Name** | `lobster` |
| **Label** | `Lobster Workflow` |

**Description:**
> Run Lobster pipelines as a local-first workflow runtime (typed JSON envelope + resumable approvals).

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `action` | `"run"` \| `"resume"` | **Yes** | — | Run or resume pipeline |
| `pipeline` | string | No | — | Pipeline name/path (required for `run`) |
| `argsJson` | string | No | — | JSON args |
| `token` | string | No | — | Resume token (required for `resume`) |
| `approve` | boolean | No | — | Approval decision (required for `resume`) |
| `cwd` | string | No | — | Relative working directory |
| `timeoutMs` | number | No | 20,000 | Timeout |
| `maxStdoutBytes` | number | No | 512,000 | Max stdout size |

**Key behaviors:**
- Returns `null` if sandboxed.
- Validates `lobsterPath` is absolute and points to binary named `lobster`.
- `cwd` must not escape gateway working directory.
- Sets `LOBSTER_MODE=tool` env var.
- Parses typed JSON envelope from stdout.

---

### 29. `voice_call`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/voice-call/index.ts` |
| **Name** | `voice_call` |
| **Label** | `Voice Call` |

**Description:**
> Make phone calls and have voice conversations via the voice-call plugin.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `action` | string enum | **Yes** | `initiate_call`, `continue_call`, `speak_to_user`, `end_call`, `get_status` |
| `to` | string | No | Phone number for `initiate_call` |
| `callId` | string | No | Call id (required for non-initiate actions) |
| `message` | string | No | Message to speak |
| `mode` | `"notify"` \| `"conversation"` | No | Call mode |

**Key behaviors:**
- Supports Telnyx, Twilio, and mock providers.
- Lazily initializes call runtime on first use.
- Also registers gateway methods for external invocation.

---

### 30. `zalouser`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/zalouser/src/tool.ts` |
| **Name** | `zalouser` |
| **Label** | `Zalo Personal` |

**Description:**
> Send messages and access data via Zalo personal account. Actions: send, image, link, friends, groups, me, status.

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `action` | string enum | **Yes** | — | `send`, `image`, `link`, `friends`, `groups`, `me`, `status` |
| `threadId` | string | No | — | Thread ID for messaging |
| `message` | string | No | — | Message text |
| `isGroup` | boolean | No | — | Is group chat |
| `profile` | string | No | — | Profile name |
| `query` | string | No | — | Search query for friends |
| `url` | string | No | — | URL for media/link |

**Key behaviors:**
- Delegates to `zca-cli` subprocess.
- `send` requires `threadId` + `message`.
- `friends` searches if `query` provided, else lists all.

---

### 31–38. Feishu Bitable Tools (8 tools)

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/feishu/src/bitable.ts` |

All tools require Feishu `appId`/`appSecret` in config. Registered via `registerFeishuBitableTools(api)`.

#### 31. `feishu_bitable_get_meta`
- **Description:** Parse a Bitable URL and get app_token, table_id, and table list.
- **Params:** `url` (string, required) — supports `/wiki/` and `/base/` URL formats.

#### 32. `feishu_bitable_list_fields`
- **Description:** List all fields (columns) in a Bitable table with types and properties.
- **Params:** `app_token` (required), `table_id` (required).

#### 33. `feishu_bitable_list_records`
- **Description:** List records (rows) with pagination support.
- **Params:** `app_token` (required), `table_id` (required), `page_size` (1-500, default 100), `page_token`.

#### 34. `feishu_bitable_get_record`
- **Description:** Get a single record by ID.
- **Params:** `app_token` (required), `table_id` (required), `record_id` (required).

#### 35. `feishu_bitable_create_record`
- **Description:** Create a new record (row).
- **Params:** `app_token` (required), `table_id` (required), `fields` (Record<string, any>, required — keyed by field name).

#### 36. `feishu_bitable_update_record`
- **Description:** Update an existing record.
- **Params:** `app_token` (required), `table_id` (required), `record_id` (required), `fields` (required).

#### 37. `feishu_bitable_create_app`
- **Description:** Create a new Bitable application.
- **Params:** `name` (required), `folder_token` (optional).
- **Behavior:** On creation, runs `cleanupNewBitable` (removes default placeholder rows/fields, renames primary field to table name).

#### 38. `feishu_bitable_create_field`
- **Description:** Create a new field (column).
- **Params:** `app_token` (required), `table_id` (required), `field_name` (required), `field_type` (number, required — 1=Text, 2=Number, 3=SingleSelect, 4=MultiSelect, 5=DateTime, 7=Checkbox, 11=User, 13=Phone, 15=URL, 17=Attachment, 18=SingleLink, 19=Lookup, 20=Formula, 21=DuplexLink, 22=Location, 23=GroupChat, 1001=CreatedTime, 1002=ModifiedTime, 1003=CreatedUser, 1004=ModifiedUser, 1005=AutoNumber), `property` (optional).

---

### 39. `feishu_doc`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/feishu/src/docx.ts` |
| **Name** | `feishu_doc` |
| **Label** | `Feishu Doc` |

**Description:**
> Feishu document operations. Actions: read, write, append, create, list_blocks, get_block, update_block, delete_block.

**Parameters (by action):**
- `read`: `doc_token` (required)
- `write`: `doc_token` + `content` (markdown, replaces entire doc)
- `append`: `doc_token` + `content` (markdown, appended)
- `create`: `title` (required) + `folder_token` (optional)
- `list_blocks`: `doc_token`
- `get_block`: `doc_token` + `block_id`
- `update_block`: `doc_token` + `block_id` + `content`
- `delete_block`: `doc_token` + `block_id`

**Key behaviors:**
- `write` clears document before inserting.
- Supports image upload within markdown (downloads remote images, re-uploads to Feishu).
- Max media upload: `mediaMaxMb` config (default 30 MB).

---

### 40. `feishu_app_scopes`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/feishu/src/docx.ts` |
| **Name** | `feishu_app_scopes` |
| **Label** | `Feishu App Scopes` |

**Description:**
> List current app permissions (scopes). Use to debug permission issues.

**Parameters:** None.

---

### 41. `feishu_wiki`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/feishu/src/wiki.ts` |
| **Name** | `feishu_wiki` |
| **Label** | `Feishu Wiki` |

**Description:**
> Feishu knowledge base operations. Actions: spaces, nodes, get, create, move, rename.

**Parameters (by action):**
- `spaces`: no params
- `nodes`: `space_id` (required) + `parent_node_token` (optional)
- `get`: `token` (required — wiki node token from URL)
- `search`: `query` (required) + `space_id` (optional) — **Note: stubbed with error, directs to browse instead**
- `create`: `space_id` + `title` (required) + `obj_type` (`"docx"` | `"sheet"` | `"bitable"`, default `"docx"`) + `parent_node_token` (optional)
- `move`: `space_id` + `node_token` (required) + `target_space_id` + `target_parent_token`
- `rename`: `space_id` + `node_token` + `title` (required)

---

### 42. `feishu_drive`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/feishu/src/drive.ts` |
| **Name** | `feishu_drive` |
| **Label** | `Feishu Drive` |

**Description:**
> Feishu cloud storage operations. Actions: list, info, create_folder, move, delete.

**Parameters (by action):**
- `list`: `folder_token` (optional — omit for root)
- `info`: `file_token` (required) + `type` (required)
- `create_folder`: `name` (required) + `folder_token` (optional)
- `move`: `file_token` + `type` + `folder_token` (target, required)
- `delete`: `file_token` + `type`
- `type` enum: `"doc"` | `"docx"` | `"sheet"` | `"bitable"` | `"folder"` | `"file"` | `"mindnote"` | `"shortcut"`

---

### 43. `feishu_perm`

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/feishu/src/perm.ts` |
| **Name** | `feishu_perm` |
| **Label** | `Feishu Perm` |

**Description:**
> Feishu permission management. Actions: list, add, remove.

**Parameters (by action):**
- `list`: `token` + `type` (TokenType)
- `add`: `token` + `type` + `member_type` + `member_id` + `perm` (`"view"` | `"edit"` | `"full_access"`)
- `remove`: `token` + `type` + `member_type` + `member_id`
- `TokenType` enum: `"doc"` | `"docx"` | `"sheet"` | `"bitable"` | `"folder"` | `"file"` | `"wiki"` | `"mindnote"`
- `MemberType` enum: `"email"` | `"openid"` | `"userid"` | `"unionid"` | `"openchat"` | `"opendepartmentid"`

**Key behaviors:**
- Disabled by default (`toolsCfg.perm` must be explicitly enabled).

---

### 44–46. Memory LanceDB Tools (3 tools)

| Field | Value |
|-------|-------|
| **Source** | `openclaw/extensions/memory-lancedb/index.ts` |

#### 44. `memory_recall`
- **Label:** `Memory Recall`
- **Description:** Search through long-term memories. Use when you need context about user preferences, past decisions, or previously discussed topics.
- **Params:** `query` (string, required), `limit` (number, default 5).
- **Behavior:** Embeds query via OpenAI, searches LanceDB with L2 distance → similarity score (`1/(1+d)`). Default `minScore` 0.1.

#### 45. `memory_store`
- **Label:** `Memory Store`
- **Description:** Save important information in long-term memory. Use for preferences, facts, decisions.
- **Params:** `text` (required), `importance` (0-1, default 0.7), `category` (`preference` | `decision` | `entity` | `fact` | `other`).
- **Behavior:** Checks near-duplicate (score >= 0.95) before storing. Returns `"created"` or `"duplicate"`.

#### 46. `memory_forget`
- **Label:** `Memory Forget`
- **Description:** Delete specific memories. GDPR-compliant.
- **Params:** `query` (string, optional), `memoryId` (string, optional — UUID).
- **Behavior:** If `memoryId`: validates UUID, deletes directly. If `query`: searches (limit 5, minScore 0.7); auto-deletes if single result with score > 0.9, else returns candidate list.

**Lifecycle hooks:**
- `before_agent_start`: auto-recall (injects relevant memories as context).
- `agent_end`: auto-capture (saves user message content, max 3 per conversation, dedup at 0.95).

---

## LAYER 4: pi-mono Example Extensions (14 tools)

Source directory: `pi-mono/packages/coding-agent/examples/extensions/`

User-installed, not active by default. Reference implementations demonstrating the extension API.

---

### 47. `subagent`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/subagent/index.ts` |
| **Agents** | `pi-mono/packages/coding-agent/examples/extensions/subagent/agents/*.md` |
| **Prompts** | `pi-mono/packages/coding-agent/examples/extensions/subagent/prompts/*.md` |
| **Discovery** | `pi-mono/packages/coding-agent/examples/extensions/subagent/agents.ts` |
| **Name** | `subagent` |
| **Label** | `Subagent` |

**Description:**
> Delegate tasks to specialized subagents with isolated context. Modes: single (agent + task), parallel (tasks array), chain (sequential with {previous} placeholder). Default agent scope is "user" (from ~/.pi/agent/agents). To enable project-local agents in .pi/agents, set agentScope: "both" (or "project").

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `agent` | string | No | — | Agent name (for single mode) |
| `task` | string | No | — | Task (for single mode) |
| `tasks` | array of `{agent, task, cwd?}` | No | — | Parallel execution (max 8) |
| `chain` | array of `{agent, task, cwd?}` | No | — | Sequential with `{previous}` placeholder |
| `agentScope` | `"user"` \| `"project"` \| `"both"` | No | `"user"` | Agent directory scope |
| `confirmProjectAgents` | boolean | No | true | Prompt before project-local agents |
| `cwd` | string | No | — | Working directory (single mode) |

**Key behaviors:**
- Exactly one of `agent+task`, `tasks`, or `chain` must be provided.
- Max 8 parallel tasks, max 4 concurrent.
- Chain uses `{previous}` placeholder to pass output between steps.
- Spawns `pi --mode json -p --no-session` subprocess per agent.
- Agent definitions: `.md` files with YAML frontmatter (`name`, `description`, `tools`, `model`) + system prompt body.
- Discovery: user agents from `~/.pi/agent/agents/*.md`, project agents from `.pi/agents/*.md`.
- Project agents: UI confirmation dialog for untrusted repos.

**Pre-built agents:**

| Agent | Model | Tools | Purpose |
|-------|-------|-------|---------|
| `scout` | claude-haiku-4-5 | read, grep, find, ls, bash | Fast codebase recon |
| `planner` | claude-sonnet-4-5 | read, grep, find, ls | Implementation plans (NO writes) |
| `reviewer` | claude-sonnet-4-5 | read, grep, find, ls, bash | Code review (bash = git only) |
| `worker` | claude-sonnet-4-5 | all defaults | General-purpose implementation |

**Pre-built workflows:**

| Command | Chain |
|---------|-------|
| `/implement <query>` | scout → planner → worker |
| `/scout-and-plan <query>` | scout → planner |
| `/implement-and-review <query>` | worker → reviewer → worker |

---

### 48. `hello`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/hello.ts` |
| **Name** | `hello` |
| **Label** | `Hello` |

**Description:** A simple greeting tool.
**Params:** `name` (string, required).
**Returns:** `"Hello, ${name}!"`.

---

### 49. `question`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/question.ts` |
| **Name** | `question` |
| **Label** | `Question` |

**Description:** Ask the user a question and let them pick from options.
**Params:** `question` (string, required), `options` (array of `{label, description?}`, required).
**Behavior:** Requires UI. Interactive TUI with free-text "Type something." entry. Escape cancels.

---

### 50. `questionnaire`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/questionnaire.ts` |
| **Name** | `questionnaire` |
| **Label** | `Questionnaire` |

**Description:** Ask one or more questions. Single question = option list. Multiple = tab-based interface.
**Params:** `questions` (array of `{id, label?, prompt, options: [{value, label, description?}], allowOther?}`).
**Behavior:** Requires UI. Tab bar with Submit tab. Free-text via inline editor. Submit only when all answered.

---

### 51. `rg`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/truncated-tool.ts` |
| **Name** | `rg` |
| **Label** | `ripgrep` |

**Description:** Search file contents using ripgrep with truncation handling.
**Params:** `pattern` (string, required), `path` (optional), `glob` (optional).
**Behavior:** Runs `rg --line-number --color=never`. Truncation saves full output to `/tmp/pi-rg-XXXX/output.txt`.

---

### 52. `read` (audited override)

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/tool-override.ts` |
| **Name** | `read` (overrides core) |
| **Label** | `read (audited)` |

**Description:** Read file with access logging. Sensitive paths blocked.
**Params:** Same as core `read`.
**Behavior:** Blocks `.env`, `secrets.json`, `.ssh/`, `.aws/`, `.gnupg/`. Logs to `~/.pi/agent/read-access.log`.

---

### 53. `read/write/edit/bash` (SSH overrides)

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/ssh.ts` |
| **Names** | `read`, `write`, `edit`, `bash` (all overrides) |

**Behavior:** When `--ssh user@host` flag set, all file operations and commands run over SSH. Paths transformed by replacing `localCwd` with `remoteCwd`.

---

### 54. `generate_image`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/antigravity-image-gen.ts` |
| **Name** | `generate_image` |
| **Label** | `Generate image` |

**Description:** Generate an image via Google Antigravity image models.
**Params:** `prompt` (required), `model` (default `"gemini-3-pro-image"`), `aspectRatio` (enum of ratios), `save` (`"none"` | `"project"` | `"global"` | `"custom"`), `saveDir`.
**Behavior:** OAuth credentials from model registry. SSE request to Google API. Returns image as `{type: "image", data, mimeType}`.

---

### 55. `bash` (sandboxed override)

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/sandbox/index.ts` |
| **Name** | `bash` (overrides core) |
| **Label** | `bash (sandboxed)` |

**Behavior:** Uses `@anthropic-ai/sandbox-runtime`. Allows common npm/PyPI/GitHub domains. Denies reads of `~/.ssh`, `~/.aws`, `~/.gnupg`. Denies writes to `.env`, `*.pem`, `*.key`. Config from `~/.pi/agent/sandbox.json` and `.pi/sandbox.json`.

---

### 56. `parse_duration`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/with-deps/index.ts` |
| **Name** | `parse_duration` |
| **Label** | `Parse Duration` |

**Description:** Parse human-readable duration string to milliseconds.
**Params:** `duration` (string, required — e.g. `"2 days"`, `"1h"`, `"5m"`).
**Behavior:** Uses `ms` npm package.

---

### 57. `todo`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/todo.ts` |
| **Name** | `todo` |
| **Label** | `Todo` |

**Description:** Manage a todo list. Actions: list, add, toggle, clear.
**Params:** `action` (enum, required), `text` (for add), `id` (for toggle).
**Behavior:** In-memory state reconstructed from session entries. Branch-safe replay via `details`.

---

### 58–59. `finish_and_exit` / `deploy_and_exit`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/shutdown-command.ts` |

**`finish_and_exit`:** No params. Calls `ctx.shutdown()`.
**`deploy_and_exit`:** `environment` (string, required). Calls `ctx.shutdown()` after deployment.

---

### 60. `reload_runtime`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/coding-agent/examples/extensions/reload-runtime.ts` |
| **Name** | `reload_runtime` |
| **Label** | `Reload Runtime` |

**Description:** Reload extensions, skills, prompts, and themes.
**Params:** None.
**Behavior:** Queues `/reload-runtime` as a follow-up user message.

---

## LAYER 5: Web UI & Sandbox Tools (4 tools)

---

### 61. `artifacts`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/web-ui/src/tools/artifacts/artifacts.ts` |
| **Name** | `artifacts` |
| **Label** | (dynamic) |

**Description:** Create and display rich artifacts (HTML, Markdown, SVG, PDF, DOCX, XLSX, images) as live previews in the browser UI.
**Params:** `filename` (string, required — including extension), `content` (inferred).
**Behavior:** Renders based on file extension. Registered to web UI renderer registry.

---

### 62. `javascript_repl`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/web-ui/src/tools/javascript-repl.ts` |
| **Name** | `javascript_repl` |
| **Label** | `JavaScript REPL` |

**Description:** Execute JavaScript in a sandboxed browser environment with full Web APIs.
**Params:** `title` (string, required — brief description), `code` (string, required — JS to execute).
**Behavior:** Creates hidden `SandboxIframe`. Captures console output, return value, file results (base64). Iframe cleaned up after execution. Runtime providers configurable.

---

### 63. `extract_document`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/web-ui/src/tools/extract-document.ts` |
| **Name** | `extract_document` |
| **Label** | `Extract Document` |

**Description:** Extract plain text from documents on the web (PDF, DOCX, XLSX, PPTX).
**Params:** `url` (string, required).
**Behavior:** 50MB size limit. Tries direct fetch, then CORS proxy if configured. Special-cases arxiv.org PDFs. Uses `loadAttachment` for parsing.

---

### 64. `attach`

| Field | Value |
|-------|-------|
| **Source** | `pi-mono/packages/mom/src/tools/attach.ts` |
| **Name** | `attach` |
| **Label** | `attach` |

**Description:** Attach a file to your response. Use this to share files, images, or documents with the user. Only files from /workspace/ can be attached.
**Params:** `label` (string, required), `path` (string, required), `title` (string, optional).
**Behavior:** Uploads via pluggable `uploadFn`. Only operational when `uploadFn` is configured.

---

## Tool Middleware System

**Source:** `pi-mono/packages/coding-agent/src/core/extensions/wrapper.ts`

ALL tools (core + extension-registered) are wrapped by `wrapToolsWithExtensions()`:

**Phase 1: `tool_call` event (pre-execution, CAN BLOCK)**
- Fired before any tool executes.
- If any extension handler returns `{ block: true, reason? }`, tool is blocked with error.

**Phase 2: `tool_result` event (post-execution, CAN MODIFY)**
- Fired after successful execution.
- Extensions can replace `content`, `details`, and/or `isError`.

**Additional lifecycle events:**
- `tool_execution_start` — tool starts
- `tool_execution_update` — streaming updates
- `tool_execution_end` — tool finishes (with `isError` flag)

---

## Configuration That Controls Tool Availability

| Config Path | Controls |
|-------------|----------|
| `agents.defaults.subagents.maxSpawnDepth` | Sub-agent nesting depth (default: 1) |
| `agents.defaults.subagents.allowAgents` | Which agents can be spawned (`["*"]` = all) |
| `tools.web.fetch.enabled` | `web_fetch` availability |
| `tools.web.fetch.firecrawl.apiKey` | Firecrawl backend for `web_fetch` |
| `tools.web.search.provider` | `web_search` backend (brave/perplexity/grok) |
| `tools.agentToAgent.enabled` | Cross-agent `sessions_send` |
| `browser.enabled` | Host `browser` control |
| `agents.defaults.sandbox.browser.enabled` | Sandbox `browser` |
| `gateway.nodes.browser.mode` | Node `browser` proxy |
| `commands.restart` | `gateway` restart action |
| `memory.citations` | `memory_search` citation display |
| `agents.defaults.imageModel` | `image` tool model selection |
| `agents.defaults.mediaMaxMb` | Max image/media upload size |

---

## Summary Statistics

| Layer | Count | Modifies State | Read-Only |
|-------|-------|----------------|-----------|
| Core (pi-mono) | 7 | 3 (bash, edit, write) | 4 (read, grep, find, ls) |
| Platform (openclaw) | 19 | 12 | 7 |
| Extensions (openclaw) | 20 | 12 | 8 |
| Example Extensions (pi-mono) | 14 | 8 | 6 |
| Web UI / Sandbox | 4 | 1 | 3 |
| **Total** | **64** | **36** | **28** |
