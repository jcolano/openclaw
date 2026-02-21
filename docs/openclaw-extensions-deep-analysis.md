# OpenClaw Extensions: Complete Architecture & Deep Analysis

> Exhaustive reference covering all 37 extensions across 5 categories: messaging channels,
> utility plugins, auth providers, memory systems, and infrastructure services.

---

## Table of Contents

1. [Plugin Architecture Overview](#1-plugin-architecture-overview)
2. [Messaging Channel Extensions (22)](#2-messaging-channel-extensions)
   - [iMessage](#imessage)
   - [BlueBubbles](#bluebubbles)
   - [Discord](#discord)
   - [Telegram](#telegram)
   - [Slack](#slack)
   - [Signal](#signal)
   - [WhatsApp](#whatsapp)
   - [IRC](#irc)
   - [Matrix](#matrix)
   - [Microsoft Teams](#microsoft-teams)
   - [Google Chat](#google-chat)
   - [Feishu/Lark](#feishulark)
   - [Mattermost](#mattermost)
   - [Nextcloud Talk](#nextcloud-talk)
   - [Nostr](#nostr)
   - [Twitch](#twitch)
   - [LINE](#line)
   - [Tlon (Urbit)](#tlon-urbit)
   - [Zalo Bot](#zalo-bot)
   - [Zalo Personal](#zalo-personal)
3. [Utility & Infrastructure Extensions (12)](#3-utility--infrastructure-extensions)
   - [LLM Task](#llm-task)
   - [Lobster](#lobster)
   - [Voice Call](#voice-call)
   - [Talk Voice](#talk-voice)
   - [Phone Control](#phone-control)
   - [Device Pair](#device-pair)
   - [Open Prose](#open-prose)
   - [Thread Ownership](#thread-ownership)
   - [Copilot Proxy](#copilot-proxy)
   - [Diagnostics OTEL](#diagnostics-otel)
4. [Memory Extensions (2)](#4-memory-extensions)
   - [Memory Core](#memory-core)
   - [Memory LanceDB](#memory-lancedb)
5. [Auth Provider Extensions (4)](#5-auth-provider-extensions)
   - [Google Antigravity Auth](#google-antigravity-auth)
   - [Google Gemini CLI Auth](#google-gemini-cli-auth)
   - [MiniMax Portal Auth](#minimax-portal-auth)
   - [Qwen Portal Auth](#qwen-portal-auth)
6. [Shared Utilities (1)](#6-shared-utilities)
7. [Cross-Extension Comparison Tables](#7-cross-extension-comparison-tables)
8. [Plugin SDK Reference](#8-plugin-sdk-reference)

---

## 1. Plugin Architecture Overview

Every OpenClaw extension follows a consistent plugin architecture:

### Plugin Module Structure

```
extensions/<name>/
  ├── index.ts                  # Entry point: exports register() or plugin definition
  ├── openclaw.plugin.json      # Manifest: id, configSchema, channels, providers
  ├── package.json              # NPM package with "openclaw.extensions" field
  └── src/
      ├── channel.ts            # ChannelPlugin implementation (for messaging extensions)
      ├── config-schema.ts      # Zod schema for config validation
      ├── accounts.ts           # Account resolution logic
      ├── monitor.ts            # Inbound message monitoring
      ├── send.ts               # Outbound message delivery
      ├── targets.ts            # Target ID parsing and normalization
      └── ...
```

### Plugin Registration API

The `register(api: OpenClawPluginApi)` function receives an API object with:

| Method | Purpose |
|--------|---------|
| `api.registerChannel()` | Register a messaging channel plugin |
| `api.registerProvider()` | Register a model provider |
| `api.registerTool()` | Register agent tools |
| `api.registerHttpHandler()` | Register raw HTTP request handler |
| `api.registerHttpRoute()` | Register HTTP route handler |
| `api.registerGatewayMethod()` | Register gateway RPC method |
| `api.registerCommand()` | Register CLI/chat commands |
| `api.registerCli()` | Register CLI subcommands |
| `api.registerService()` | Register lifecycle-managed services |
| `api.registerHook()` / `api.on()` | Register event hooks |

### ChannelPlugin Interface

Channel plugins implement `ChannelPlugin<ResolvedAccount>` with these adapters:

| Adapter | Purpose | Required |
|---------|---------|----------|
| `config` | Account resolution, config schema | Yes |
| `capabilities` | Feature declarations (polls, reactions, edit, etc.) | Yes |
| `meta` | Channel metadata (label, docs, blurb) | Yes |
| `security` | Security context building | Optional |
| `messaging` | Inbound message handling | Optional |
| `outbound` | Outbound message delivery | Optional |
| `status` | Account/connection status | Optional |
| `gateway` | Gateway lifecycle (startAccount/stopAccount) | Optional |
| `setup` | Account setup wizard | Optional |
| `pairing` | Device pairing flow | Optional |
| `groups` | Group handling | Optional |
| `mentions` | Mention stripping/formatting | Optional |
| `actions` | Message actions (react, edit, unsend, etc.) | Optional |
| `directory` | User/group directory | Optional |
| `streaming` | Block streaming config | Optional |
| `threading` | Reply/thread handling | Optional |
| `agentTools` | Channel-specific agent tools | Optional |
| `heartbeat` | Connection monitoring | Optional |

### Plugin Manifest (`openclaw.plugin.json`)

```json
{
  "id": "unique-plugin-id",
  "configSchema": { "type": "object", "properties": {} },
  "channels": ["channel-id"],
  "providers": ["provider-id"],
  "skills": ["skill-id"],
  "kind": "memory"
}
```

### Plugin Loading Lifecycle

1. **Discovery** — scans bundled, global, workspace, and config paths
2. **Manifest Validation** — validates `openclaw.plugin.json` (requires `id` + `configSchema`)
3. **Enable Check** — respects allowlist/denylist and dependency checks
4. **Module Loading** — Jiti-based TypeScript/JS loading
5. **Config Validation** — validates against plugin's JSON schema
6. **Registration** — calls `register(api)` with the full plugin API
7. **Tracking** — maintains `PluginRecord` with metadata

---

## 2. Messaging Channel Extensions

### iMessage

| | |
|---|---|
| **Package** | `@openclaw/imessage` |
| **Path** | `extensions/imessage/` + `src/imessage/` (core library) |
| **Transport** | JSON-RPC 2.0 over stdio subprocess (`imsg rpc`) |
| **Platform** | macOS only (or SSH tunnel to macOS) |

#### Entry Point

```
index.ts → setIMessageRuntime(api.runtime)
         → api.registerChannel({ plugin: imessagePlugin })
```

#### Connection Lifecycle

1. `gateway.startAccount()` → `monitorIMessageProvider()`
2. `waitForTransportReady()` — polls `probeIMessage` every 500ms for up to 30s
3. `detectBinary(cliPath)` → `probeRpcSupport(cliPath)` → `createIMessageRpcClient()`
4. `client.request("chats.list")` — liveness check
5. `client.request("watch.subscribe", { attachments })` → `{ subscription: N }`
6. `client.waitForClose()` — blocks until subprocess exits
7. On abort: `watch.unsubscribe` → `client.stop()`

#### Inbound Message Flow

1. `imsg rpc` emits `{"method": "message", "params": {...}}` notification
2. `parseIMessageNotification(raw)` validates payload (strict field-by-field type guards)
3. `inboundDebouncer.enqueue()` — coalesces rapid messages from same sender/chat
4. `resolveIMessageInboundDecision()` — full allow/block/pairing decision tree
5. If dispatch: `buildIMessageInboundContext()` → `dispatchInboundMessage()`
6. Reply via `deliverReplies()` → `sendMessageIMessage()` per chunk

#### Outbound Targeting

| Target Kind | Parameter Sent |
|------------|----------------|
| `chat_id` | `chat_id: N` |
| `chat_guid` | `chat_guid: "..."` |
| `chat_identifier` | `chat_identifier: "..."` |
| `handle` | `to: "..."` (with `service`, `region`) |

#### Config Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `cliPath` | string | `"imsg"` | Path to imsg binary or SSH wrapper |
| `dbPath` | string | — | Custom iMessage SQLite database path |
| `service` | string | `"auto"` | Default service: `imessage`, `sms`, `auto` |
| `region` | string | `"US"` | Phone number region |
| `dmPolicy` | enum | `"pairing"` | `pairing`, `allowlist`, `open`, `disabled` |
| `groupPolicy` | enum | `"open"` | `open`, `disabled`, `allowlist` |
| `allowFrom` | string[] | `[]` | DM allow-list |
| `groupAllowFrom` | string[] | `[]` | Group allow-list |
| `includeAttachments` | boolean | `false` | Pass inbound attachments to agent |
| `mediaMaxMb` | number | 16 | Max outbound media size |
| `textChunkLimit` | number | 4000 | Max chars per outbound chunk |
| `probeTimeoutMs` | number | 10000 | RPC timeout |
| `remoteHost` | string | auto-detect | SSH remote host label |
| `blockStreaming` | boolean | — | Block streaming mode |

#### Limitations

- No actions interface (no reactions, edits, unsends, effects, group management)
- No outbound reply threading
- `includeAttachments` defaults to false
- Single `imsg rpc` subprocess per account, no connection pooling
- No built-in reconnect — gateway must restart account
- Echo detection relies on exact text match (5s SentMessageCache)
- SSH wrapper detection uses regex heuristics

---

### BlueBubbles

| | |
|---|---|
| **Package** | `@openclaw/bluebubbles` |
| **Path** | `extensions/bluebubbles/` |
| **Transport** | HTTP REST API + Webhook |
| **Platform** | BlueBubbles app on macOS; OpenClaw gateway on any platform |

#### Entry Point

```
index.ts → setBlueBubblesRuntime(api.runtime)
         → api.registerChannel({ plugin: bluebubblesPlugin })
         → api.registerHttpHandler(handleBlueBubblesWebhookRequest)
```

#### Connection Lifecycle

1. `gateway.startAccount()` → `fetchBlueBubblesServerInfo()` (probes `/api/v1/server/info`)
2. `registerBlueBubblesWebhookTarget()` — registers in webhook routing table
3. Waits for `AbortSignal`
4. On abort: unregister + debouncer cleanup

#### Inbound Message Flow

1. BlueBubbles POSTs JSON to `/bluebubbles-webhook`
2. Validates HTTP method, body (max 1MB, 30s timeout)
3. Filters events: `new-message`, `updated-message`, `message-reaction`, `reaction`
4. Authenticates: timing-safe password comparison (passwordless from loopback only)
5. Debouncer coalesces rapid events by `messageId`
6. `processMessage()` → full auth/policy/dispatch pipeline

#### Rich Actions (11 total, most require Private API)

| Action | Private API | API Endpoint |
|--------|-------------|-------------|
| `sendMessage` | No | POST `/api/v1/message/text` |
| `react` | Yes | POST `/api/v1/message/react` |
| `edit` | Yes | POST `/api/v1/message/:guid/edit` |
| `unsend` | Yes | POST `/api/v1/message/:guid/unsend` |
| `reply` | Yes | POST `/api/v1/message/text` + `selectedMessageGuid` |
| `sendWithEffect` | Yes | POST `/api/v1/message/text` + `effectId` |
| `renameGroup` | Yes | PUT `/api/v1/chat/:guid` |
| `setGroupIcon` | Yes | POST `/api/v1/chat/:guid/icon` |
| `addParticipant` | Yes | POST `/api/v1/chat/:guid/participant` |
| `removeParticipant` | Yes | DELETE `/api/v1/chat/:guid/participant` |
| `leaveGroup` | Yes | POST `/api/v1/chat/:guid/leave` |
| `sendAttachment` | No | POST `/api/v1/message/attachment` |

#### Message Effects

12 named effects: slam, loud, gentle, invisible-ink, echo, spotlight, balloons, confetti, love, lasers, fireworks, celebration.

#### Short Message IDs

Incremental counter (`1`, `2`, `3`...) mapped to full UUID GUIDs via in-memory LRU cache (max 2000 entries, 6-hour TTL). Saves tokens when agents reference messages for reactions/replies/edits.

#### Config Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `serverUrl` | string | **required** | BlueBubbles server base URL |
| `password` | string | **required** | BlueBubbles API password |
| `webhookPath` | string | `/bluebubbles-webhook` | HTTP path for webhooks |
| `dmPolicy` | enum | `"pairing"` | DM access policy |
| `groupPolicy` | enum | `"allowlist"` | Group access policy |
| `sendReadReceipts` | boolean | true | Mark chats as read on inbound |
| `mediaMaxMb` | int | 8 | Max media size |
| `mediaLocalRoots` | string[] | `[]` | Allowed local media directories |
| `textChunkLimit` | int | 4000 | Max chars per chunk |
| `chunkMode` | enum | `"length"` | `length` or `newline` |
| `actions.*` | boolean | true | Enable/disable individual actions |

#### Limitations

- Private API required for most advanced features (special macOS setup)
- `edit` blocked on macOS 26+
- Only 6 canonical Apple tapback types
- Chat GUID resolution paginates up to 5000 chats (no caching)
- Reply cache is in-memory, lost on restart
- Webhook auth is static password (no HMAC rotation)
- No WebSocket support (webhook-only)

---

### iMessage vs BlueBubbles Comparison

| Feature | iMessage (imsg CLI) | BlueBubbles (REST API) |
|---------|--------------------|-----------------------|
| Transport | JSON-RPC 2.0 via stdio | HTTP REST + Webhook |
| Inbound | RPC push notifications | Webhook POST |
| Platform | macOS (direct or SSH) | BB app on macOS; OpenClaw anywhere |
| Reactions/Tapbacks | Not supported | Full support (Private API) |
| Edit/Unsend | Not supported | Supported (Private API, macOS <26) |
| Reply threading | Context only (no outbound) | Full thread replies (Private API) |
| Message effects | Not supported | 12 named effects (Private API) |
| Group management | Not supported | rename/icon/add/remove/leave |
| Typing indicators | Not supported | Full (start/stop/restart) |
| Read receipts | Not supported | Configurable |
| Media inbound | Local file path | Download via REST API |
| Media outbound | Local temp file to binary | Multipart upload to API |
| Short message IDs | No | Yes (incremental counter) |
| Config complexity | Low | High (actions, groups, markdown) |

---

### Discord

| | |
|---|---|
| **Package** | `@openclaw/discord` |
| **Path** | `extensions/discord/` |
| **Transport** | Discord Bot API with WebSocket gateway + polling fallback |
| **Platform** | Any |

#### Key Features
- Direct/channel/thread messaging with polls, reactions, threads
- Media attachments with configurable size limits
- Native Discord commands and components/modals
- Streaming with block coalescing
- Reply-to mode support
- Guild/channel allowlisting with mention-gating
- Multi-account support

#### Config Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `token` | string | **required** | Discord bot token (env: `DISCORD_BOT_TOKEN`) |
| `guilds` | object | — | Guild/channel allowlists |
| `groupPolicy` | enum | — | `open`, `allowlist` |
| `dm.allowFrom` | string[] | — | DM allow list |
| `mediaMaxMb` | number | — | Max media size |
| `historyLimit` | number | — | Max history messages |
| `replyToMode` | string | — | Reply-to mode behavior |

---

### Telegram

| | |
|---|---|
| **Package** | `@openclaw/telegram` |
| **Path** | `extensions/telegram/` |
| **Transport** | Telegram Bot API (HTTPS) with polling (default) or webhook |
| **Platform** | Any |

#### Key Features
- Direct/group/channel/thread messaging
- Polls (anonymous supported), reactions, threads
- Media attachment support
- Native bot commands
- Group membership auditing
- E.164 phone number normalization
- Bot mention detection with `requireMention` config
- Proxy support for restricted networks

#### Config Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `botToken` | string | **required** | Bot token (env: `TELEGRAM_BOT_TOKEN`) |
| `tokenFile` | string | — | Alternative: path to token file |
| `webhookUrl` | string | — | Webhook URL (else polling) |
| `webhookSecret` | string | — | Webhook HMAC secret |
| `groupPolicy` | enum | — | Group access policy |
| `groups` | object | — | Per-group config (requireMention) |
| `dmPolicy` | enum | — | DM access policy |
| `allowFrom` | string[] | — | DM allow-list |
| `proxy` | string | — | HTTP proxy URL |

---

### Slack

| | |
|---|---|
| **Package** | `@openclaw/slack` |
| **Path** | `extensions/slack/` |
| **Transport** | Slack Web API (HTTP) with dual-token auth (bot + app tokens) |
| **Platform** | Any |

#### Key Features
- Direct/channel/thread messaging with reply-to threading
- Reactions, threads, media, native commands
- User token read-only override
- Channel allowlisting with mention-gating
- Live directory queries for user/channel discovery
- Channel archival detection
- Slash command integration

#### Config Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `botToken` | string | **required** | Bot token (env: `SLACK_BOT_TOKEN`) |
| `appToken` | string | **required** | App token (env: `SLACK_APP_TOKEN`) |
| `userToken` | string | — | Optional user token |
| `userTokenReadOnly` | boolean | — | User token read-only mode |
| `groupPolicy` | enum | — | Group access policy |
| `channels` | object | — | Channel allowlist config |
| `dm.allowFrom` | string[] | — | DM allow-list |
| `dm.policy` | enum | — | DM access policy |
| `mediaMaxMb` | number | — | Max media size |
| `slashCommand` | string | — | Slash command name |

---

### Signal

| | |
|---|---|
| **Package** | `@openclaw/signal` |
| **Path** | `extensions/signal/` |
| **Transport** | Local signal-cli daemon via HTTP or native CLI subprocess |
| **Platform** | Any (requires signal-cli) |

#### Key Features
- Direct/group messaging with media and reactions
- E.164 phone number support
- Group/UUID/JID targeting
- Local HTTP proxy to signal-cli
- Account auto-probing

#### Config Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `account` | string | **required** | E.164 phone number |
| `httpUrl` | string | — | signal-cli HTTP daemon URL |
| `httpHost` | string | — | HTTP host |
| `httpPort` | number | — | HTTP port |
| `cliPath` | string | — | Path to signal-cli binary |
| `dmPolicy` | enum | — | DM access policy |
| `groupPolicy` | enum | — | Group access policy |
| `mediaMaxMb` | number | — | Max media size |

---

### WhatsApp

| | |
|---|---|
| **Package** | `@openclaw/whatsapp` |
| **Path** | `extensions/whatsapp/` |
| **Transport** | WhatsApp Web client emulation (Puppeteer/Chrome) |
| **Platform** | Any (requires Chrome/Chromium) |

#### Key Features
- Direct/group messaging with polls and reactions
- E.164 + JID targeting
- React actions with emoji removal
- GIF playback control
- Web auth via QR code login (session persisted in `authDir`)
- Self-identity resolution (E.164/JID)
- Group intro hints
- Owner-only command enforcement

#### Config Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `accounts[id].authDir` | string | — | Web session storage path |
| `accounts[id].enabled` | boolean | true | Per-account enable |
| `dmPolicy` | enum | — | DM access policy |
| `groupPolicy` | enum | — | Group access policy |
| `groups` | object | — | Group allowlist config |
| `actions.reactions` | boolean | — | Enable reactions |
| `actions.polls` | boolean | — | Enable polls |
| `web.enabled` | boolean | — | Master switch |

---

### IRC

| | |
|---|---|
| **Package** | `@openclaw/irc` |
| **Path** | `extensions/irc/` |
| **Transport** | TCP/WebSocket to IRC servers (TLS supported) |
| **Platform** | Any |

#### Key Features
- Direct and group chat
- Multi-account with allowlist/pairing DM policies
- Markdown text chunking (350 char limit)
- Media attachment support
- NickServ registration
- Group-level tool policies
- Event-driven message handling

---

### Matrix

| | |
|---|---|
| **Package** | `@openclaw/matrix` |
| **Path** | `extensions/matrix/` |
| **Transport** | Matrix homeserver via HTTP/REST API (access token or password auth) |
| **Platform** | Any |

#### Key Features
- Full message threading, reactions, polls
- Rich media handling
- Lazy-loaded provider with serialized startup (avoids import race conditions)
- Direct messages and room interactions
- Live directory updates for discovering peers/rooms
- Reply modes configurable
- Multi-homeserver support

---

### Microsoft Teams

| | |
|---|---|
| **Package** | `@openclaw/msteams` |
| **Path** | `extensions/msteams/` |
| **Transport** | Microsoft Bot Framework with webhook inbound + Graph API outbound |
| **Platform** | Any |

#### Key Features
- Adaptive Card support for rich messages
- Polling/threading capabilities
- Team/channel organization
- Webhook server for receiving messages
- User/conversation ID resolution via Microsoft Graph
- Tool hints for card-based messages
- Multi-team support with conversation-level access control

---

### Google Chat

| | |
|---|---|
| **Package** | `@openclaw/googlechat` |
| **Path** | `extensions/googlechat/` |
| **Transport** | Google Chat REST API with service account auth or webhook |
| **Platform** | Any |

#### Key Features
- Webhook-based inbound with path configuration
- Media upload with configurable max size
- Text chunking (4000 chars)
- Reaction/thread support
- Service account JSON or file-based auth
- Audience validation (app-url or project-number)
- Live directory for space/user discovery

---

### Feishu/Lark

| | |
|---|---|
| **Package** | `@openclaw/feishu` |
| **Path** | `extensions/feishu/` |
| **Transport** | Feishu/Lark Bot API with webhook or WebSocket |
| **Platform** | Any |

#### Key Features
- Interactive card support for rich messages
- Multi-app environment (feishu.cn or lark.com)
- Edit/reply capabilities
- Reaction/poll support
- Webhook and WebSocket connection modes
- App auth via appId/appSecret
- **Bitable integration tools**: CRUD operations on Feishu spreadsheets
- **Doc/Wiki/Drive integration**: Document creation and management
- Message history limits per chat type
- Entity extraction for mentions

---

### Mattermost

| | |
|---|---|
| **Package** | `@openclaw/mattermost` |
| **Path** | `extensions/mattermost/` |
| **Transport** | Mattermost WebSocket for real-time + REST API for outbound |
| **Platform** | Any |

#### Key Features
- Reaction emoji support (configurable per account)
- Direct/channel/group/thread support
- Markdown text chunking
- Bot token authentication
- WebSocket-based event stream
- Channel/user directory listing
- Thread reply tracking
- Media attachment handling (inline media URLs)

---

### Nextcloud Talk

| | |
|---|---|
| **Package** | `@openclaw/nextcloud-talk` |
| **Path** | `extensions/nextcloud-talk/` |
| **Transport** | Webhook bot integration with HMAC-SHA256 shared secret signing |
| **Platform** | Any |

#### Key Features
- Direct/group chat with reaction capabilities
- Webhook-based inbound with HMAC-SHA256 signing
- Rate limiting and security warnings for open group policies
- Room allowlist with wildcard support for mention requirements
- Text chunking (4000 chars)

---

### Nostr

| | |
|---|---|
| **Package** | `@openclaw/nostr` |
| **Path** | `extensions/nostr/` |
| **Transport** | Nostr relays with NIP-04 encryption |
| **Platform** | Any |

#### Key Features
- Decentralized DM-only (no group/thread support)
- Multi-relay support with circuit breaker patterns
- Direct metric tracking for relay health
- Profile publishing (kind:0 events)
- Pubkey normalization (npub1 or hex format)
- Active bus handle management per account

---

### Twitch

| | |
|---|---|
| **Package** | `@openclaw/twitch` |
| **Path** | `extensions/twitch/` |
| **Transport** | Twitch Chat API (WebSocket) via client manager registry |
| **Platform** | Any |

#### Key Features
- Group chat only (no DMs)
- Username/user ID resolution
- Onboarding adapter for OAuth setup
- Client connection pooling per account
- Status monitoring with probe capability

---

### LINE

| | |
|---|---|
| **Package** | `@openclaw/line` |
| **Path** | `extensions/line/` |
| **Transport** | LINE Messaging API with webhook callbacks + REST outbound |
| **Platform** | Any |

#### Key Features
- Rich message formats: Flex Messages, Templates, Location, Quick Replies
- Text chunking (5000 chars — highest of any channel)
- Direct and group support
- QR-login via webhook
- Channel access token + secret validation
- Card directives for structured responses (buttons, confirm dialogs, media players)

---

### Tlon (Urbit)

| | |
|---|---|
| **Package** | `@openclaw/tlon` |
| **Path** | `extensions/tlon/` |
| **Transport** | Urbit HTTP API with cookies and SSRF policy support |
| **Platform** | Any |

#### Key Features
- Decentralized P2P messaging
- Direct DMs and group messages
- Threading with reply tracking
- Ship normalization with `~` prefix handling
- SSRF protection (configurable private network access)
- Auth via ship URL + access code

---

### Zalo Bot

| | |
|---|---|
| **Package** | `@openclaw/zalo` |
| **Path** | `extensions/zalo/` |
| **Transport** | Zalo Bot API with webhook or polling |
| **Platform** | Any |

#### Key Features
- DM-only (no groups/threads)
- Webhook or polling configurable at runtime
- Bot token validation with probe
- Text chunking (2000 chars)
- Proxy support (HTTP/HTTPS)
- Vietnam-focused platform

---

### Zalo Personal

| | |
|---|---|
| **Package** | `@openclaw/zalouser` |
| **Path** | `extensions/zalouser/` |
| **Transport** | ZCA CLI wrapper (Node.js subprocess) with JSON output parsing |
| **Platform** | Any (requires ZCA CLI) |

#### Key Features
- QR code auth via personal Zalo account (stored as ZCA profile)
- Direct and group messaging
- Friends/groups directory discovery
- Agent tool for programmatic actions (friends, groups, me, status)
- Streaming message reception via ZCA monitoring
- Group tool policies with wildcard support

---

## 3. Utility & Infrastructure Extensions

### LLM Task

| | |
|---|---|
| **Path** | `extensions/llm-task/` |
| **Purpose** | LLM sub-task execution tool |

Registers a single tool via `createLlmTaskTool()`. Loads the embedded PI agent runner dynamically (src/dist fallback). Allows agents to invoke sub-tasks through `runEmbeddedPiAgent()`. Validates JSON schema via AJV. Strips code fences from responses. Marked as optional tool.

---

### Lobster

| | |
|---|---|
| **Path** | `extensions/lobster/` |
| **Purpose** | External binary execution tool (non-sandboxed only) |

Spawns external `lobster` binary as subprocess. Requires absolute path (platform-specific validation). JSON envelope response format with `ok`, `status`, `output`, and optional `requiresApproval` fields. Supports approval workflow for sensitive operations. Skipped if sandboxed.

---

### Voice Call

| | |
|---|---|
| **Path** | `extensions/voice-call/` |
| **Purpose** | Voice calling via Telnyx/Twilio/Plivo providers |

Registers gateway methods, tool, CLI, and service. Capabilities:
- Initiate/continue/end calls
- TTS runtime integration
- Tailscale tunneling and ngrok for webhooks
- Streaming mode with OpenAI Realtime API
- Inbound policy with allowlist
- Gateway methods: `voicecall.initiate/continue/speak/end/status`
- Call log storage
- Configurable response model and timeout

---

### Talk Voice

| | |
|---|---|
| **Path** | `extensions/talk-voice/` |
| **Purpose** | List/set ElevenLabs voice for iOS Talk playback |

Commands: `/voice status`, `/voice list [limit]`, `/voice set <voiceId|name>`. Queries ElevenLabs API for available voices. Supports partial name matching. Stores voice ID in config. Sensitive API key masking in output.

---

### Phone Control

| | |
|---|---|
| **Path** | `extensions/phone-control/` |
| **Purpose** | Arm/disarm high-risk phone node commands |

Manages allow/deny command lists in gateway config. Temporal arm state stored in JSON file. Four command groups: camera, screen, writes, all. Configurable duration (30s, 10m, 2h, 1d). Auto-disarm on expiry. Commands: `/phone status`, `/phone arm <group> [duration]`, `/phone disarm`.

---

### Device Pair

| | |
|---|---|
| **Path** | `extensions/device-pair/` |
| **Purpose** | Generate setup codes and approve device pairing |

Generates base64-encoded setup payloads with gateway URL and token/password. QR code generation for iOS app scanning. Resolves gateway URL from config (Tailscale, remote, LAN, loopback). IP detection (private IPv4, Tailnet). Telegram integration with split message sending. Commands: `/pair`, `/pair qr`, `/pair status`, `/pair approve <requestId|latest>`.

---

### Open Prose

| | |
|---|---|
| **Path** | `extensions/open-prose/` |
| **Purpose** | Skill delivery mechanism |

No-op `register()` function — actual functionality delivered via skills directory. Skills auto-loaded from the `skills/` subdirectory.

---

### Thread Ownership

| | |
|---|---|
| **Path** | `extensions/thread-ownership/` |
| **Purpose** | Prevent multiple agents from responding in same Slack thread |

Listens to `message_received` and `message_sending` events. Tracks @-mentions in in-memory map (5-minute TTL). HTTP API call to forwarder service to claim/check thread ownership. Supports A/B testing on specific channels. Fails open on network errors. Cancels message send on HTTP 409. Env vars: `SLACK_FORWARDER_URL`, `SLACK_BOT_USER_ID`, `THREAD_OWNERSHIP_CHANNELS`.

---

### Copilot Proxy

| | |
|---|---|
| **Path** | `extensions/copilot-proxy/` |
| **Purpose** | Local VS Code Copilot Proxy provider |

Registers a provider via `api.registerProvider()`. Custom auth flow with base URL and model ID configuration. Supports multiple models (gpt-5.x, claude-opus/sonnet/haiku, gemini, grok). Uses OpenAI completions API format. Auto-normalizes base URL (`/v1` suffix). Requires Copilot Proxy VS Code extension running locally.

---

### Diagnostics OTEL

| | |
|---|---|
| **Path** | `extensions/diagnostics-otel/` |
| **Purpose** | Export diagnostics to OpenTelemetry |

Sets up OpenTelemetry Node SDK with OTLP exporters (trace, metric, logs). Configurable sampling rates and OTLP endpoints. Exports metrics and structured logs via HTTP/protobuf. Respects `diagnostics.otel.enabled` config. Lazy initialization.

---

## 4. Memory Extensions

### Memory Core

| | |
|---|---|
| **Path** | `extensions/memory-core/` |
| **Purpose** | File-backed memory search and CLI |

Registers tools `memory_search` and `memory_get` via runtime tools factory. Also registers CLI command `memory`. Basic memory search and retrieval interface backed by file storage.

---

### Memory LanceDB

| | |
|---|---|
| **Path** | `extensions/memory-lancedb/` |
| **Purpose** | Long-term vector memory with semantic search |
| **Kind** | `memory` |

#### Tools Registered

| Tool | Purpose |
|------|---------|
| `memory_recall` | Vector similarity search |
| `memory_store` | Save information |
| `memory_forget` | Delete memories |

#### Key Features
- LanceDB-backed persistent storage
- OpenAI API embeddings for semantic search
- **Auto-recall**: inject relevant memories before agent starts (lifecycle hook)
- **Auto-capture**: analyze & store important info after agent ends (lifecycle hook)
- Prompt injection detection
- Rule-based memory capture filters
- Memory categories: preference, decision, entity, fact, other
- CLI commands: `ltm list`, `ltm search`, `ltm stats`
- Dual safety measures for prompt integrity
- Max 2000 entries per operation

---

## 5. Auth Provider Extensions

### Google Antigravity Auth

| | |
|---|---|
| **Path** | `extensions/google-antigravity-auth/` |
| **Purpose** | OAuth for Google Cloud Code Assist |
| **Auth Mechanism** | PKCE-based OAuth 2.0 with GCP |

Default model: `google-antigravity/claude-opus-4-6-thinking`. Supports automated localhost callback (port 51121) and manual URL-based auth for remote/WSL2. Exchanges auth code for access/refresh tokens. Fetches user email and GCP project ID. Scopes: cloud-platform, userinfo.email, userinfo.profile, cclog, experimentsandconfigs.

---

### Google Gemini CLI Auth

| | |
|---|---|
| **Path** | `extensions/google-gemini-cli-auth/` |
| **Purpose** | OAuth for Gemini CLI / Google Code Assist |
| **Auth Mechanism** | PKCE-based OAuth 2.0 with credential resolution |

Default model: `google-gemini-cli/gemini-3-pro-preview`. Credential resolution chain: (1) env vars, (2) installed Gemini CLI's `oauth2.js`, (3) custom config. Localhost callback (port 8085) or manual URL auth. Project discovery via loadCodeAssist endpoint with tier detection (free/legacy/standard).

---

### MiniMax Portal Auth

| | |
|---|---|
| **Path** | `extensions/minimax-portal-auth/` |
| **Purpose** | OAuth for MiniMax AI models |
| **Auth Mechanism** | Device code OAuth flow |

Fixed CLIENT_ID. User opens verification URI and enters code, client polls token endpoint. Scopes: group_id, profile, model.completion. Auto-refreshes tokens. Supported models: MiniMax-M2.1 (text), MiniMax-M2.5 (text + reasoning). Region-specific endpoints (Global + CN).

---

### Qwen Portal Auth

| | |
|---|---|
| **Path** | `extensions/qwen-portal-auth/` |
| **Purpose** | OAuth for Alibaba Qwen free-tier models |
| **Auth Mechanism** | Device code OAuth flow |

Fixed CLIENT_ID. Device code flow with backoff/slow-down support. Scopes: openid, profile, email, model.completion. Supported models: Qwen Coder (text), Qwen Vision (text + image). Default base URL: `https://portal.qwen.ai/v1`.

---

## 6. Shared Utilities

### Shared

| | |
|---|---|
| **Path** | `extensions/shared/` |
| **Purpose** | Test helpers shared across extensions |

Single file `resolve-target-test-helpers.ts` exporting `installCommonResolveTargetErrorCases()` for Vitest-based testing of target resolution logic. Used by other extensions' test suites for common validation scenarios.

---

## 7. Cross-Extension Comparison Tables

### Transport Mechanisms

| Channel | Transport | Inbound | Outbound |
|---------|-----------|---------|----------|
| iMessage | JSON-RPC stdio subprocess | RPC push notifications | RPC `send` request |
| BlueBubbles | HTTP REST + Webhook | Webhook POST | REST API calls |
| Discord | WebSocket gateway | WebSocket events | HTTP REST API |
| Telegram | HTTPS polling/webhook | Poll or webhook | Bot API HTTP |
| Slack | WebSocket (Socket Mode) | WebSocket events | Web API HTTP |
| Signal | signal-cli HTTP/CLI | HTTP daemon events | HTTP/CLI calls |
| WhatsApp | Puppeteer/Chrome | Web client events | Web client actions |
| IRC | TCP/WebSocket | Socket events | Socket write |
| Matrix | HTTP REST | Sync API polling | HTTP REST |
| MS Teams | Bot Framework webhook | Webhook POST | Graph API HTTP |
| Google Chat | Webhook + REST | Webhook POST | REST API |
| Feishu | Webhook/WebSocket | Webhook or WS | Bot API HTTP |
| Mattermost | WebSocket + REST | WebSocket events | REST API |
| Nextcloud Talk | HMAC webhook | Webhook POST | REST API |
| Nostr | Relay WebSocket | NIP-04 DMs | Relay publish |
| Twitch | Chat WebSocket | WebSocket events | Chat write |
| LINE | Webhook + REST | Webhook POST | Messaging API |
| Tlon | Urbit HTTP API | HTTP polling | HTTP API |
| Zalo | Webhook/polling | Webhook or poll | Bot API |
| Zalo Personal | ZCA CLI subprocess | ZCA monitor | ZCA send |

### Feature Matrix

| Channel | DM | Group | Thread | React | Edit | Unsend | Polls | Media | Effects |
|---------|-----|-------|--------|-------|------|--------|-------|-------|---------|
| iMessage | Y | Y | context | - | - | - | - | Y | - |
| BlueBubbles | Y | Y | Y | Y | Y* | Y | - | Y | Y |
| Discord | Y | Y | Y | Y | - | - | Y | Y | - |
| Telegram | Y | Y | Y | Y | - | - | Y | Y | - |
| Slack | Y | Y | Y | Y | - | - | - | Y | - |
| Signal | Y | Y | - | Y | - | - | - | Y | - |
| WhatsApp | Y | Y | - | Y | - | - | Y | Y | - |
| IRC | Y | Y | - | - | - | - | - | Y | - |
| Matrix | Y | Y | Y | Y | - | - | Y | Y | - |
| MS Teams | Y | Y | Y | - | - | - | - | Y | - |
| Google Chat | Y | Y | Y | Y | - | - | - | Y | - |
| Feishu | Y | Y | Y | Y | - | - | Y | Y | - |
| Mattermost | Y | Y | Y | Y | - | - | - | Y | - |
| Nextcloud Talk | Y | Y | - | Y | - | - | - | - | - |
| Nostr | Y | - | - | - | - | - | - | - | - |
| Twitch | - | Y | - | - | - | - | - | - | - |
| LINE | Y | Y | - | - | - | - | - | Y | - |
| Tlon | Y | Y | Y | - | - | - | - | Y | - |
| Zalo | Y | - | - | - | - | - | - | Y | - |
| Zalo Personal | Y | Y | - | - | - | - | - | - | - |

*BlueBubbles edit is blocked on macOS 26+

### Text Chunk Limits

| Channel | Limit | Notes |
|---------|-------|-------|
| LINE | 5000 | Highest limit |
| Discord | 4000 | — |
| Telegram | 4000 | — |
| Slack | 4000 | — |
| Google Chat | 4000 | — |
| Nextcloud Talk | 4000 | — |
| iMessage | 4000 | — |
| BlueBubbles | 4000 | — |
| Zalo | 2000 | — |
| IRC | 350 | IRC protocol constraint |

### Auth Mechanisms

| Channel | Auth Method |
|---------|------------|
| iMessage | Local binary (no auth) |
| BlueBubbles | Shared password |
| Discord | Bot token |
| Telegram | Bot token |
| Slack | Bot token + App token |
| Signal | Phone number + signal-cli |
| WhatsApp | QR code web login |
| IRC | NickServ password |
| Matrix | Access token or password |
| MS Teams | Bot Framework |
| Google Chat | Service account |
| Feishu | App ID + secret |
| Mattermost | Bot token |
| Nextcloud Talk | HMAC shared secret |
| Nostr | Private key (NIP-04) |
| Twitch | OAuth token |
| LINE | Channel access token + secret |
| Tlon | Ship URL + access code |
| Zalo | Bot token |
| Zalo Personal | QR code (ZCA profile) |

---

## 8. Plugin SDK Reference

### 18 Hook Events

| Hook | Trigger |
|------|---------|
| `before_model_resolve` | Before model selection |
| `before_prompt_build` | Before system prompt assembly |
| `before_agent_start` | Before agent loop begins |
| `llm_input` | Before LLM API call |
| `llm_output` | After LLM API response |
| `agent_end` | After agent loop completes |
| `before_compaction` | Before context compaction |
| `after_compaction` | After context compaction |
| `before_reset` | Before session reset |
| `message_received` | Inbound message received |
| `message_sending` | Before outbound message sent |
| `message_sent` | After outbound message sent |
| `before_tool_call` | Before tool execution |
| `after_tool_call` | After tool execution |
| `tool_result_persist` | When tool result is persisted |
| `before_message_write` | Before message written to store |
| `session_start` | Session started |
| `session_end` | Session ended |

### Common Config Patterns

All messaging channels share a consistent config structure:

```yaml
channels:
  <channel-id>:
    enabled: true
    name: "Display Name"
    dmPolicy: pairing | allowlist | open | disabled
    allowFrom: []
    groupPolicy: open | allowlist | disabled
    groupAllowFrom: []
    groups:
      <group-id>:
        requireMention: true
        tools: [...]
    historyLimit: 50
    mediaMaxMb: 16
    textChunkLimit: 4000
    blockStreaming: false
    accounts:
      <account-id>:
        enabled: true
        # ... per-account overrides
```

### Extension Count Summary

| Category | Count | Extensions |
|----------|-------|-----------|
| Messaging Channels | 20 | imessage, bluebubbles, discord, telegram, slack, signal, whatsapp, irc, matrix, msteams, googlechat, feishu, mattermost, nextcloud-talk, nostr, twitch, line, tlon, zalo, zalouser |
| Utility/Infrastructure | 10 | llm-task, lobster, voice-call, talk-voice, phone-control, device-pair, open-prose, thread-ownership, copilot-proxy, diagnostics-otel |
| Memory | 2 | memory-core, memory-lancedb |
| Auth Providers | 4 | google-antigravity-auth, google-gemini-cli-auth, minimax-portal-auth, qwen-portal-auth |
| Shared Utilities | 1 | shared |
| **Total** | **37** | — |
