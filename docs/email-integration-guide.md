# Email Integration Architecture Guide

**Audience:** Platform developers integrating email capabilities into their own systems
**Source:** OpenClaw codebase reverse-engineering

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Mechanism 1: Gmail Pub/Sub Webhooks (Inbound Monitoring)](#2-mechanism-1-gmail-pubsub-webhooks-inbound-monitoring)
3. [Mechanism 2: Himalaya Skill (IMAP/SMTP Read/Write)](#3-mechanism-2-himalaya-skill-imapsmtp-readwrite)
4. [Mechanism 3: gogcli Skill (Gmail API Operations)](#4-mechanism-3-gogcli-skill-gmail-api-operations)
5. [How the Three Mechanisms Work Together](#5-how-the-three-mechanisms-work-together)
6. [Integration Checklist](#6-integration-checklist)

---

## 1. Architecture Overview

OpenClaw handles email through three complementary mechanisms, each serving a
different role in the email lifecycle:

| Mechanism | Direction | Protocol | Purpose |
|---|---|---|---|
| Gmail Pub/Sub Webhooks | **Inbound** (push) | Gmail API + GCP Pub/Sub | Real-time monitoring of inbox; triggers agent runs when new mail arrives |
| Himalaya Skill | **Bidirectional** | IMAP / SMTP | Read, search, reply, forward, compose, manage folders/flags via standard IMAP/SMTP |
| gogcli Skill | **Bidirectional** | Gmail REST API (OAuth2) | Send, search, draft, reply via Gmail API; also covers Calendar, Drive, Contacts, Sheets, Docs |

```
                         +-----------------------+
                         |   Gmail Inbox          |
                         +-----------+-----------+
                                     |
           Gmail API watch           | push notification
                                     v
                 +-------------------+-------------------+
                 |  GCP Pub/Sub Topic                     |
                 |  (projects/<id>/topics/gog-gmail-watch)|
                 +-------------------+-------------------+
                                     |
                       push subscription
                                     v
         +---------------------------+---------------------------+
         |  Push Endpoint (Tailscale Funnel / custom tunnel)      |
         |  https://<host>/gmail-pubsub?token=<shared>            |
         +---------------------------+---------------------------+
                                     |
                                     v
         +---------------------------+---------------------------+
         |  gog gmail watch serve                                 |
         |  (local process: 127.0.0.1:8788)                       |
         |  - Validates push token                                |
         |  - Fetches full email via Gmail API                    |
         |  - Forwards enriched payload to OpenClaw webhook       |
         +---------------------------+---------------------------+
                                     |
                      POST /hooks/gmail
                      Authorization: Bearer <hook-token>
                                     v
         +---------------------------+---------------------------+
         |  OpenClaw Gateway (HTTP Server)                        |
         |  - Auth check (hook token)                             |
         |  - Mapping resolution (preset "gmail")                 |
         |  - Template rendering (from/subject/snippet/body)      |
         |  - Spawns isolated agent run                           |
         +---------------------------+---------------------------+
                                     |
                 +-------------------+-------------------+
                 v                                       v
    +------------+------------+          +---------------+--------+
    |  Agent processes email  |          |  Delivery to chat       |
    |  (summarize, triage,    |          |  channel (WhatsApp,     |
    |   take action)          |          |  Telegram, Slack, etc.) |
    +-------------------------+          +------------------------+
```

---

## 2. Mechanism 1: Gmail Pub/Sub Webhooks (Inbound Monitoring)

### 2.1 What It Does

Provides real-time email arrival detection. When a new email lands in the
monitored Gmail inbox, a chain of events fires:

1. Gmail API pushes a notification to a GCP Pub/Sub topic
2. A Pub/Sub subscription pushes to a local HTTP endpoint
3. A local process (`gog gmail watch serve`) receives the push, fetches the
   full email content, and forwards it to the platform's webhook endpoint
4. The platform spawns an isolated agent to process the email

### 2.2 GCP Infrastructure Requirements

| Component | Purpose |
|---|---|
| **GCP Project** | Must be the same project that owns the OAuth2 client used by `gog` |
| **Gmail API** | `gmail.googleapis.com` — enabled on the project |
| **Pub/Sub API** | `pubsub.googleapis.com` — enabled on the project |
| **Pub/Sub Topic** | Named (e.g. `gog-gmail-watch`); Gmail push publishes here |
| **IAM Binding** | `gmail-api-push@system.gserviceaccount.com` gets `roles/pubsub.publisher` on the topic |
| **Pub/Sub Subscription** | Push subscription pointing to the public HTTPS endpoint |
| **Public HTTPS Endpoint** | Tailscale Funnel (supported), Cloudflare Tunnel, ngrok, or any reverse proxy |

### 2.3 Configuration Schema

The platform configuration lives in a JSON5 config file. The relevant section
is `hooks.gmail`:

```typescript
// Full type definition from src/config/types.hooks.ts

type HooksGmailConfig = {
  // Gmail account to watch (e.g. "myapp@gmail.com")
  account?: string;

  // Gmail label to watch (default: "INBOX")
  label?: string;

  // Full GCP Pub/Sub topic path: "projects/<project-id>/topics/<topic-name>"
  topic?: string;

  // Pub/Sub subscription name (default: "gog-gmail-watch-push")
  subscription?: string;

  // Shared token for authenticating Pub/Sub push → gog serve
  // (passed as ?token= in the push endpoint URL)
  pushToken?: string;

  // URL where gog serve forwards enriched payloads to your platform
  // (default: "http://127.0.0.1:<gateway-port>/hooks/gmail")
  hookUrl?: string;

  // Whether to include the email body in the forwarded payload (default: true)
  includeBody?: boolean;

  // Maximum bytes of body content to include (default: 20000)
  maxBytes?: number;

  // How often to renew the Gmail watch registration in minutes (default: 720 = 12h)
  // Gmail watches expire after 7 days, but renewal is cheap and idempotent
  renewEveryMinutes?: number;

  // DANGEROUS: Disable external content safety wrapping for Gmail hooks
  allowUnsafeExternalContent?: boolean;

  // Local HTTP server config for gog gmail watch serve
  serve?: {
    bind?: string;   // default: "127.0.0.1"
    port?: number;   // default: 8788
    path?: string;   // default: "/gmail-pubsub"
  };

  // Tailscale tunnel configuration
  tailscale?: {
    mode?: "off" | "serve" | "funnel";  // default in setup: "funnel"
    path?: string;    // public path (default: "/gmail-pubsub")
    // Optional tailscale serve/funnel target (port, host:port, or full URL)
    target?: string;
  };

  // Optional model override for Gmail hook processing (provider/model or alias)
  model?: string;

  // Optional thinking level override for Gmail hook processing
  thinking?: "off" | "minimal" | "low" | "medium" | "high";
};
```

**Parent hooks configuration:**

```typescript
type HooksConfig = {
  enabled?: boolean;       // Master switch — must be true
  path?: string;           // Webhook base path (default: "/hooks")
  token?: string;          // Shared secret for authenticating all hook requests

  // Session key policy for hook agent runs
  defaultSessionKey?: string;           // e.g. "hook:ingress"
  allowRequestSessionKey?: boolean;     // default: false
  allowedSessionKeyPrefixes?: string[]; // e.g. ["hook:"]

  // Agent routing restrictions
  allowedAgentIds?: string[];  // e.g. ["hooks", "main"] or ["*"] for unrestricted

  maxBodyBytes?: number;     // Max webhook payload size (default: 256KB)
  presets?: string[];         // Built-in mapping presets to enable (e.g. ["gmail"])
  transformsDir?: string;    // Custom JS/TS transform modules directory
  mappings?: HookMappingConfig[];  // Custom hook route mappings
  gmail?: HooksGmailConfig;        // Gmail-specific config (above)
};
```

### 2.4 Complete Example Configuration

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    path: "/hooks",
    presets: ["gmail"],
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],

    gmail: {
      account: "myapp@gmail.com",
      label: "INBOX",
      topic: "projects/my-gcp-project/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "randomly-generated-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: {
        bind: "127.0.0.1",
        port: 8788,
        path: "/",
      },
      tailscale: {
        mode: "funnel",
        path: "/gmail-pubsub",
      },
      model: "openai/gpt-4o-mini",
      thinking: "off",
    },

    // Override the default gmail preset to enable delivery to chat
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        deliver: true,
        channel: "last",
      },
    ],
  },
}
```

### 2.5 Built-in Gmail Preset Mapping

When you enable `presets: ["gmail"]`, the platform registers this built-in
mapping (from `src/gateway/hooks-mapping.ts`):

```typescript
const hookPresetMappings = {
  gmail: [
    {
      id: "gmail",
      match: { path: "gmail" },           // matches POST /hooks/gmail
      action: "agent",                      // spawns an isolated agent run
      wakeMode: "now",                      // triggers immediately
      name: "Gmail",                        // label in session summaries
      sessionKey: "hook:gmail:{{messages[0].id}}",  // unique per email
      messageTemplate:
        "New email from {{messages[0].from}}\n" +
        "Subject: {{messages[0].subject}}\n" +
        "{{messages[0].snippet}}\n" +
        "{{messages[0].body}}",
    },
  ],
};
```

**Template Variables** (resolved from the JSON payload forwarded by `gog`):

| Variable | Description |
|---|---|
| `{{messages[0].id}}` | Gmail message ID |
| `{{messages[0].from}}` | Sender address |
| `{{messages[0].subject}}` | Email subject |
| `{{messages[0].snippet}}` | Gmail snippet (preview text) |
| `{{messages[0].body}}` | Full email body (up to `maxBytes`) |
| `{{path}}` | The hook path (e.g., `"gmail"`) |
| `{{now}}` | ISO timestamp of when the hook fires |
| `{{headers.<name>}}` | Any request header |
| `{{query.<name>}}` | Any query string parameter |
| `{{payload.<path>}}` | Any nested path in the JSON payload |

### 2.6 The `gog gmail watch serve` Process

This is a standalone process that:

1. **Listens** on `bind:port/path` (default `127.0.0.1:8788/gmail-pubsub`)
2. **Receives** Pub/Sub push notifications (JSON with `historyId`)
3. **Validates** the push token (`--token`)
4. **Fetches** the actual email content via Gmail API (using `gog`'s OAuth credentials)
5. **Forwards** an enriched JSON payload to the hook URL (`--hook-url`) with
   the hook token (`--hook-token`)

Full command constructed by the platform:

```bash
gog gmail watch serve \
  --account myapp@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <push-shared-secret> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token <hook-shared-secret> \
  --include-body \
  --max-bytes 20000
```

### 2.7 Watch Registration and Renewal

Gmail watches expire after **7 days**. The platform handles renewal
automatically:

```typescript
// From src/hooks/gmail-watcher.ts — automatic renewal
const renewMs = runtimeConfig.renewEveryMinutes * 60_000; // default: 12 hours
renewInterval = setInterval(() => {
  void startGmailWatch(runtimeConfig);
}, renewMs);
```

The watch start command:

```bash
gog gmail watch start \
  --account myapp@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

### 2.8 Gateway Auto-Start (Recommended Mode)

When `hooks.enabled=true` and `hooks.gmail.account` is set, the gateway
automatically starts the Gmail watcher on boot via `startGmailWatcher()`:

```typescript
// From src/hooks/gmail-watcher.ts
export async function startGmailWatcher(cfg: OpenClawConfig): Promise<GmailWatcherStartResult> {
  // 1. Checks: hooks enabled? gmail account set? gog binary available?
  // 2. Resolves full runtime config from OpenClawConfig
  // 3. Sets up Tailscale endpoint if configured
  // 4. Registers Gmail watch (gog gmail watch start)
  // 5. Spawns gog gmail watch serve as a child process
  // 6. Sets up periodic watch renewal timer
  // 7. Auto-restarts gog process on crash (with 5s delay)
  // 8. Detects "address already in use" and stops restart loop
}
```

**Crash recovery:** If the `gog serve` process exits unexpectedly, it
auto-restarts after 5 seconds — unless the error is `EADDRINUSE`, which
indicates another watcher is already running.

Set `OPENCLAW_SKIP_GMAIL_WATCHER=1` to disable auto-start (if running the
daemon manually).

### 2.9 Webhook Endpoint: Authentication and Routing

The gateway HTTP server processes hook requests in this order:

1. **Path matching**: Request path must start with `hooks.path` (default `/hooks`)
2. **Authentication**: Token extracted from `Authorization: Bearer <token>` or
   `x-openclaw-token: <token>` header. Query-string tokens are rejected.
3. **Rate limiting**: Repeated auth failures from the same client trigger `429`
   with `Retry-After`
4. **Sub-path routing**:
   - `/hooks/wake` → system event (wake the main session)
   - `/hooks/agent` → isolated agent run (direct payload)
   - `/hooks/<name>` → mapping resolution (presets + custom mappings)
5. **Payload normalization**: JSON body parsed, validated against schema
6. **Agent run**: Isolated session spawned with the processed message

**Response codes:**

| Code | Meaning |
|---|---|
| `200` | Wake endpoint success |
| `202` | Agent endpoint success (async run started) |
| `400` | Invalid payload |
| `401` | Authentication failure |
| `413` | Payload too large |
| `429` | Rate limited (check `Retry-After`) |

### 2.10 Security Model

```
Token Flow:

  Pub/Sub Push  ──pushToken──>  gog serve  ──hookToken──>  Gateway
  (GCP managed)                 (local)                    (local)
```

- **pushToken**: Authenticates Pub/Sub push notifications to the `gog serve`
  process. Included as `?token=` in the Pub/Sub subscription endpoint URL.
- **hookToken**: Authenticates `gog serve` → Gateway requests. Sent as
  `Authorization: Bearer` or `x-openclaw-token` header.
- Both tokens are randomly generated (24 bytes → hex) during setup.
- External content from email payloads is wrapped with safety boundaries by
  default to prevent prompt injection.

### 2.11 One-Time Setup Sequence (Manual)

```bash
# 1. Authenticate with GCP
gcloud auth login
gcloud config set project <project-id>

# 2. Enable required APIs
gcloud services enable gmail.googleapis.com pubsub.googleapis.com

# 3. Create Pub/Sub topic
gcloud pubsub topics create gog-gmail-watch

# 4. Allow Gmail push to publish
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher

# 5. Set up Tailscale Funnel (or other tunnel)
tailscale funnel --bg --set-path /gmail-pubsub --yes 8788

# 6. Create push subscription
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<tailscale-hostname>/gmail-pubsub?token=<pushToken>"

# 7. Register Gmail watch
gog gmail watch start \
  --account myapp@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch

# 8. Start the serve process
gog gmail watch serve \
  --account myapp@gmail.com \
  --bind 127.0.0.1 --port 8788 --path /gmail-pubsub \
  --token <pushToken> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token <hookToken> \
  --include-body --max-bytes 20000
```

### 2.12 One-Time Setup Sequence (Automated Wizard)

```bash
openclaw webhooks gmail setup --account myapp@gmail.com
```

This single command performs all of the above steps:
- Installs dependencies (`gcloud`, `gog`, `tailscale`) via Homebrew on macOS
- Authenticates with GCP
- Creates topic, IAM binding, subscription
- Sets up Tailscale Funnel
- Registers Gmail watch
- Writes all configuration to the config file
- Generates random tokens

### 2.13 Key Implementation Files

| File | Purpose |
|---|---|
| `src/hooks/gmail.ts` | Configuration types, defaults, runtime config resolution, CLI argument builders |
| `src/hooks/gmail-ops.ts` | CLI operations: `setup` (full wizard) and `run` (daemon mode) |
| `src/hooks/gmail-setup-utils.ts` | GCP/gcloud/Tailscale setup utilities: dependency checking, auth, topic/subscription management |
| `src/hooks/gmail-watcher.ts` | Auto-start daemon: process lifecycle, crash recovery, watch renewal |
| `src/gateway/hooks-mapping.ts` | Mapping engine: preset definitions, template rendering, transform loading |
| `src/gateway/hooks.ts` | Webhook handler: auth, payload normalization, session/agent policies |
| `src/config/types.hooks.ts` | TypeScript type definitions for all hook configuration |

---

## 3. Mechanism 2: Himalaya Skill (IMAP/SMTP Read/Write)

### 3.1 What It Does

Himalaya is a Rust CLI email client that gives agents direct IMAP/SMTP access.
Unlike the Gmail Pub/Sub webhook (which is push-based and inbound-only),
Himalaya provides full bidirectional email operations: reading, searching,
composing, replying, forwarding, managing folders, handling attachments.

### 3.2 Architecture

```
  +------------------+     CLI commands      +------------------+
  |  Agent (LLM)     | ------------------->  |  himalaya binary |
  |  via skill        |                       |  (Rust CLI)      |
  +------------------+     stdout/stderr     +--------+---------+
                       <-------------------           |
                                               IMAP/SMTP
                                                      |
                                                      v
                                            +------------------+
                                            |  Mail Server     |
                                            |  (Gmail, iCloud, |
                                            |   Exchange, etc) |
                                            +------------------+
```

The agent invokes `himalaya` CLI commands as shell operations. No long-running
process is needed — each command connects, performs the operation, and exits.

### 3.3 Skill Manifest

```yaml
name: himalaya
description: >
  CLI to manage emails via IMAP/SMTP. Use `himalaya` to list, read, write,
  reply, forward, search, and organize emails from the terminal. Supports
  multiple accounts and message composition with MML (MIME Meta Language).
homepage: https://github.com/pimalaya/himalaya
requires:
  bins: ["himalaya"]
install:
  - id: brew
    kind: brew
    formula: himalaya
    bins: ["himalaya"]
```

### 3.4 Configuration File

Location: `~/.config/himalaya/config.toml`

**Gmail configuration:**

```toml
[accounts.gmail]
email = "you@gmail.com"
display-name = "Your Name"
default = true

# IMAP backend for reading
backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "you@gmail.com"
backend.auth.type = "password"
backend.auth.cmd = "pass show google/app-password"

# SMTP backend for sending
message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "you@gmail.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "pass show google/app-password"
```

**iCloud configuration:**

```toml
[accounts.icloud]
email = "you@icloud.com"
display-name = "Your Name"

backend.type = "imap"
backend.host = "imap.mail.me.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "you@icloud.com"
backend.auth.type = "password"
backend.auth.cmd = "pass show icloud/app-password"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.mail.me.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "you@icloud.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "pass show icloud/app-password"
```

### 3.5 Authentication Options

| Method | Config Key | Example | Security |
|---|---|---|---|
| Raw password | `backend.auth.raw` | `"my-password"` | Testing only, **not recommended** |
| Command | `backend.auth.cmd` | `"pass show email/imap"` | **Recommended** — fetches from password manager |
| System keyring | `backend.auth.keyring` | `"imap-example"` | Good — uses OS keychain |
| OAuth2 | `backend.auth.type = "oauth2"` | (see below) | Best — for providers that support it |

**OAuth2 configuration:**

```toml
backend.auth.type = "oauth2"
backend.auth.client-id = "your-client-id"
backend.auth.client-secret.cmd = "pass show oauth/client-secret"
backend.auth.access-token.cmd = "pass show oauth/access-token"
backend.auth.refresh-token.cmd = "pass show oauth/refresh-token"
backend.auth.auth-url = "https://provider.com/oauth/authorize"
backend.auth.token-url = "https://provider.com/oauth/token"
```

### 3.6 Complete Command Reference

#### Folder Management

```bash
himalaya folder list                          # List all folders/labels
```

#### Listing Emails

```bash
himalaya envelope list                         # List INBOX (default)
himalaya envelope list --folder "Sent"         # List specific folder
himalaya envelope list --page 1 --page-size 20 # Paginated listing
himalaya envelope list --output json           # JSON output
```

#### Searching

```bash
himalaya envelope list from john@example.com subject meeting
```

#### Reading Emails

```bash
himalaya message read 42                       # Plain text view
himalaya message export 42 --full              # Raw MIME export
```

#### Composing and Sending

```bash
# Interactive (opens $EDITOR):
himalaya message write

# Direct send from stdin:
cat << 'EOF' | himalaya template send
From: you@example.com
To: recipient@example.com
Subject: Test Message

Hello from the platform!
EOF

# With pre-filled headers:
himalaya message write \
  -H "To:recipient@example.com" \
  -H "Subject:Quick Message" \
  "Message body here"
```

#### Reply and Forward

```bash
himalaya message reply 42                      # Reply (opens editor)
himalaya message reply 42 --all                # Reply-all
himalaya message forward 42                    # Forward
```

#### Organization

```bash
himalaya message move 42 "Archive"             # Move to folder
himalaya message copy 42 "Important"           # Copy to folder
himalaya message delete 42                     # Delete
himalaya flag add 42 --flag seen               # Mark as read
himalaya flag remove 42 --flag seen            # Mark as unread
```

#### Attachments

```bash
himalaya attachment download 42                # Save attachments
himalaya attachment download 42 --dir ~/Downloads  # To specific directory
```

#### Multiple Accounts

```bash
himalaya account list                          # List all accounts
himalaya --account work envelope list          # Use specific account
```

### 3.7 Message Composition with MML (MIME Meta Language)

Himalaya uses MML for composing rich emails. MML is a simple XML-based syntax
that compiles to proper MIME messages.

**Plain text:**

```
From: alice@example.com
To: bob@example.com
Subject: Hello

This is a plain text email.
```

**With attachments:**

```
From: alice@example.com
To: bob@example.com
Subject: Report

<#multipart type=mixed>
<#part type=text/plain>
Please find the attached report.

Best,
Alice
<#part filename=/path/to/report.pdf><#/part>
<#/multipart>
```

**HTML with inline images:**

```
From: alice@example.com
To: bob@example.com
Subject: With Image

<#multipart type=related>
<#part type=text/html>
<html><body>
<p>Check out this image:</p>
<img src="cid:image1">
</body></html>
<#part disposition=inline id=image1 filename=/path/to/image.png><#/part>
<#/multipart>
```

**MML Tag Reference:**

| Tag | Attributes | Purpose |
|---|---|---|
| `<#multipart>` | `type=alternative\|mixed\|related` | Groups message parts |
| `<#part>` | `type`, `filename`, `name`, `disposition`, `id` | Defines a message part |

### 3.8 Integration Considerations

- **No long-running process**: Each command is a one-shot operation.
  Good for agent tools, but not for real-time monitoring. Use Gmail Pub/Sub
  for that.
- **Provider-agnostic**: Works with any IMAP/SMTP server, not just Gmail.
- **Secure credential storage**: Password is fetched at runtime via
  `backend.auth.cmd` — the config file never stores plaintext passwords.
- **Structured output**: Use `--output json` for machine-readable output
  that your agent can parse.
- **Gmail requires App Passwords** when 2FA is enabled. Generate at
  `myaccount.google.com → Security → App passwords`.

### 3.9 Key Files

| File | Purpose |
|---|---|
| `skills/himalaya/SKILL.md` | Skill manifest and command reference |
| `skills/himalaya/references/configuration.md` | Config file format, provider examples, auth options |
| `skills/himalaya/references/message-composition.md` | MML syntax reference for rich email composition |

---

## 4. Mechanism 3: gogcli Skill (Gmail API Operations)

### 4.1 What It Does

`gog` (gogcli) is a CLI for Google Workspace: Gmail, Calendar, Drive,
Contacts, Sheets, and Docs. For email specifically, it provides Gmail API
access via OAuth2 — sending, searching, drafting, replying, and watch
management (used by Mechanism 1).

### 4.2 Architecture

```
  +------------------+     CLI commands      +------------------+
  |  Agent (LLM)     | ------------------->  |  gog binary      |
  |  via skill        |                       |  (Go CLI)        |
  +------------------+     stdout/stderr     +--------+---------+
                       <-------------------           |
                                              Gmail REST API
                                              (OAuth2)
                                                      |
                                                      v
                                            +------------------+
                                            |  Gmail API       |
                                            |  (Google Cloud)  |
                                            +------------------+
```

### 4.3 Skill Manifest

```yaml
name: gog
description: Google Workspace CLI for Gmail, Calendar, Drive, Contacts, Sheets, and Docs.
homepage: https://gogcli.sh
requires:
  bins: ["gog"]
install:
  - id: brew
    kind: brew
    formula: steipete/tap/gogcli
    bins: ["gog"]
```

### 4.4 OAuth Setup (One-Time)

```bash
# 1. Import OAuth client credentials (from GCP console)
gog auth credentials /path/to/client_secret.json

# 2. Authorize a Gmail account (interactive browser flow)
gog auth add you@gmail.com --services gmail,calendar,drive,contacts,docs,sheets

# 3. Verify authorization
gog auth list
```

Credentials are stored at:
- `~/.config/gogcli/credentials.json` (Linux)
- `~/Library/Application Support/gogcli/credentials.json` (macOS)

### 4.5 Gmail Command Reference

#### Searching

```bash
# Search threads (one row per thread)
gog gmail search 'newer_than:7d' --max 10

# Search individual messages (one row per email, ignores threading)
gog gmail messages search "in:inbox from:ryanair.com" --max 20 --account you@gmail.com
```

Uses Gmail's native search syntax (same as the Gmail web search bar):
`from:`, `to:`, `subject:`, `newer_than:`, `older_than:`, `in:`, `is:`,
`has:`, `label:`, etc.

#### Sending (Plain Text)

```bash
# Simple send
gog gmail send --to recipient@example.com --subject "Hello" --body "Hi there"

# Multi-line body from file
gog gmail send --to recipient@example.com --subject "Report" --body-file ./message.txt

# Multi-line body from stdin (heredoc)
gog gmail send --to recipient@example.com \
  --subject "Meeting Follow-up" \
  --body-file - <<'EOF'
Hi Name,

Thanks for meeting today. Next steps:
- Item one
- Item two

Best regards,
Your Name
EOF
```

#### Sending (HTML)

```bash
gog gmail send --to recipient@example.com \
  --subject "Formatted Message" \
  --body-html "<p>Hi,</p><p>This is <strong>important</strong>.</p><ul><li>Item 1</li><li>Item 2</li></ul>"
```

#### Drafts

```bash
# Create draft
gog gmail drafts create --to recipient@example.com \
  --subject "Draft Subject" --body-file ./message.txt

# Send existing draft
gog gmail drafts send <draftId>
```

#### Replying

```bash
gog gmail send --to recipient@example.com \
  --subject "Re: Original Subject" \
  --body "Reply content" \
  --reply-to-message-id <messageId>
```

#### Watch Management (used by Mechanism 1)

```bash
# Start watching for new emails
gog gmail watch start \
  --account you@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch

# Check watch status
gog gmail watch status --account you@gmail.com

# View email history since a specific historyId
gog gmail history --account you@gmail.com --since <historyId>

# Stop watching
gog gmail watch stop --account you@gmail.com

# Run the push handler server (Mechanism 1 daemon)
gog gmail watch serve \
  --account you@gmail.com \
  --bind 127.0.0.1 --port 8788 --path /gmail-pubsub \
  --token <pushToken> --hook-url <hookUrl> --hook-token <hookToken> \
  --include-body --max-bytes 20000
```

### 4.6 Email Formatting Best Practices

- **Prefer plain text** with `--body-file` for multi-paragraph messages
- `--body` does **not** unescape `\n`. For inline newlines use a heredoc or
  `$'Line 1\n\nLine 2'`
- Use `--body-file -` to pipe content from stdin
- Use `--body-html` only when rich formatting is needed
- HTML tags: `<p>`, `<br>`, `<strong>`, `<em>`, `<a href>`, `<ul>/<li>`

### 4.7 Environment Variables

| Variable | Purpose |
|---|---|
| `GOG_ACCOUNT` | Default Gmail account (avoids `--account` on every command) |

### 4.8 Scripting Tips

- Use `--json` for machine-readable output
- Use `--no-input` to suppress interactive prompts
- `gog gmail search` returns one row per **thread**; use
  `gog gmail messages search` for individual emails
- Always confirm with the user before sending mail or creating events

### 4.9 Key Differences: gogcli vs. Himalaya

| Aspect | gogcli (gog) | Himalaya |
|---|---|---|
| Protocol | Gmail REST API (OAuth2) | IMAP/SMTP |
| Providers | Google only | Any IMAP/SMTP server |
| Auth | OAuth2 (browser flow) | Password, keyring, OAuth2 |
| Watch/Push | Yes (Gmail Pub/Sub) | No |
| Calendar/Drive/etc | Yes | No (email only) |
| Search syntax | Gmail native query syntax | IMAP search |
| Folder management | Limited (Gmail labels) | Full IMAP folder ops |
| Attachment handling | Via Gmail API | Full MML-based composition |
| Message composition | `--body`, `--body-file`, `--body-html` | MML (XML-based MIME) |

### 4.10 Key Files

| File | Purpose |
|---|---|
| `skills/gog/SKILL.md` | Skill manifest and full command reference |

---

## 5. How the Three Mechanisms Work Together

### 5.1 Typical Flow

```
                    INBOUND (monitoring)
                    =====================
                    Gmail Pub/Sub (Mechanism 1)
                    - Pushes "new email" event to platform
                    - Agent spawned to triage/summarize
                    - Summary delivered to chat channel
                           |
                           v
                    PROCESSING (read/analyze)
                    =========================
                    gogcli or Himalaya (Mechanism 2 or 3)
                    - Agent reads full email content
                    - Searches for related threads
                    - Downloads attachments if needed
                           |
                           v
                    OUTBOUND (respond)
                    ==================
                    gogcli or Himalaya (Mechanism 2 or 3)
                    - Agent composes reply/forward
                    - Sends via Gmail API or SMTP
```

### 5.2 Which Mechanism to Use When

| Scenario | Recommended Mechanism |
|---|---|
| "Alert me when I get an email" | Gmail Pub/Sub (#1) |
| "Read and summarize my last 10 emails" | gogcli (#3) for Gmail, Himalaya (#2) for other providers |
| "Reply to this email thread" | gogcli (#3) for Gmail, Himalaya (#2) for other providers |
| "Send a new email with attachment" | Himalaya (#2) for MML attachment support |
| "Search emails matching criteria" | gogcli (#3) for Gmail-native search syntax |
| "Forward email to someone" | Himalaya (#2) — has dedicated `forward` command |
| "Manage IMAP folders and flags" | Himalaya (#2) — full IMAP folder operations |
| "Update a Google Sheet and send email" | gogcli (#3) — unified Google Workspace CLI |

### 5.3 Integration Patterns for Your Platform

**Pattern A: Gmail-Only (simplest)**
- Use Mechanism 1 (Pub/Sub) for inbound monitoring
- Use Mechanism 3 (gogcli) for all read/send operations
- Single auth flow (OAuth2 via `gog auth`)

**Pattern B: Multi-Provider**
- Use Mechanism 1 (Pub/Sub) for Gmail inbound monitoring
- Use Mechanism 2 (Himalaya) for all read/send across providers
- Separate auth per provider (IMAP/SMTP credentials)

**Pattern C: Full Feature Set**
- Use all three: Pub/Sub for inbound, gogcli for Gmail-specific API
  operations, Himalaya for non-Gmail providers and attachment composition

---

## 6. Integration Checklist

### GCP / Gmail Pub/Sub Setup

- [ ] Create a GCP project (or use the one that owns the OAuth client)
- [ ] Enable `gmail.googleapis.com` and `pubsub.googleapis.com`
- [ ] Create a Pub/Sub topic
- [ ] Add IAM binding: `gmail-api-push@system.gserviceaccount.com` → `roles/pubsub.publisher`
- [ ] Set up a public HTTPS endpoint (Tailscale Funnel, Cloudflare Tunnel, etc.)
- [ ] Create a push subscription pointing to the public endpoint
- [ ] Install and authorize `gog` CLI
- [ ] Generate and store push token + hook token
- [ ] Configure `hooks.gmail` in platform config
- [ ] Start the `gog gmail watch serve` process (or enable gateway auto-start)

### Himalaya Setup

- [ ] Install `himalaya` binary (`brew install himalaya`)
- [ ] Create `~/.config/himalaya/config.toml` with IMAP/SMTP settings
- [ ] Set up secure credential storage (password manager command or keyring)
- [ ] Test with `himalaya envelope list`
- [ ] For Gmail: generate an App Password if 2FA is enabled

### gogcli Setup

- [ ] Install `gog` binary (`brew install steipete/tap/gogcli`)
- [ ] Download OAuth client credentials JSON from GCP console
- [ ] Import credentials: `gog auth credentials /path/to/client_secret.json`
- [ ] Authorize account: `gog auth add you@gmail.com --services gmail,...`
- [ ] Test with `gog gmail search 'newer_than:1d' --max 5`

### Webhook Handler Implementation

- [ ] Implement `POST /hooks/agent` endpoint with token authentication
- [ ] Implement `POST /hooks/wake` endpoint for system events
- [ ] Implement `POST /hooks/<name>` with mapping/routing logic
- [ ] Add rate limiting on auth failures
- [ ] Wrap external content with safety boundaries
- [ ] Implement session key management and policy enforcement
- [ ] Implement agent routing and isolation
- [ ] Add model override support for cost-optimized processing
