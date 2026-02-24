# OpenClaw Binary Packaging Strategy Evaluation

## System Inventory

OpenClaw is a **multi-language, multi-component** system. Packaging it as a single
binary requires understanding what must be bundled:

| Component | Language | Files | Role |
|---|---|---|---|
| `openclaw/src/` | TypeScript (Node.js 22+) | ~3,129 | Multi-channel AI gateway, CLI, TUI, web UI |
| `openclaw/extensions/` | TypeScript | ~504 (31 packages) | Channel plugins (Slack, Telegram, WhatsApp, Discord, etc.) |
| `loop_core/` | Python | ~107 | Agentic loop framework (agents, tools, skills, memory) |
| `pi-mono/` | TypeScript | monorepo | Upstream TUI/AI/coding-agent packages |
| `apps/macos/` | Swift | native app | macOS desktop wrapper (IPC to gateway) |
| `apps/ios/`, `apps/android/` | Swift, Kotlin | native apps | Mobile clients |
| `Swabble/` | Swift | library | Swift linting/formatting tool |

### Native Addons That Complicate Bundling

These require platform-specific pre-compiled `.node` binaries:

- `sharp` (image processing, libvips)
- `@lydell/node-pty` (terminal PTY)
- `sqlite-vec` (vector search)
- `@whiskeysockets/baileys` (WhatsApp protocol, protobuf)
- `@napi-rs/canvas` (2D rendering)
- `@matrix-org/matrix-sdk-crypto-nodejs` (Matrix encryption)
- `node-llama-cpp` (local LLM inference)
- `playwright-core` (optional, browser automation — pulls Chromium)

### Runtime Requirements

- Node.js >= 22.12.0
- Python 3.x (for `loop_core/`)
- Optional: Chromium, Bun

---

## Strategy Comparison

### 1. Go Wrapper Binary (Recommended)

A thin Go binary that embeds and manages the Node.js + Python runtimes.

**How it works:**

```
openclaw (Go binary, ~150-300MB)
│
├─ Embedded: portable Node.js 22 runtime (node-build-standalone or official)
├─ Embedded: dist/ + pruned node_modules/ (pnpm deploy --prod)
├─ Embedded: portable Python 3.x (python-build-standalone)
├─ Embedded: loop_core/ + pip dependencies (pip install --target)
│
└─ On first run:
   1. Extract to ~/.openclaw/cache/<version>/
   2. Spawn: node dist/index.js gateway [args]
   3. Spawn: python -m loop_core.api (if agent mode enabled)
   4. Proxy signals, manage lifecycle, expose unified CLI
```

**Go project structure:**

```
cmd/openclaw/main.go           # CLI entry, flag parsing
internal/
  embed.go                     # //go:embed directives for all assets
  extract.go                   # Version-aware extraction to cache dir
  supervisor.go                # Process lifecycle (start/stop/restart)
  health.go                    # Health checks on child processes
  updater.go                   # Optional self-update from GitHub releases
```

**Build matrix (CI):**

| Target | Node runtime | Native addons |
|--------|-------------|---------------|
| `openclaw-linux-amd64` | node-v22-linux-x64 | prebuilt for linux-x64 |
| `openclaw-linux-arm64` | node-v22-linux-arm64 | prebuilt for linux-arm64 |
| `openclaw-darwin-amd64` | node-v22-darwin-x64 | prebuilt for darwin-x64 |
| `openclaw-darwin-arm64` | node-v22-darwin-arm64 | prebuilt for darwin-arm64 |
| `openclaw-windows-amd64.exe` | node-v22-win-x64 | prebuilt for win32-x64 |

**Pros:**
- Single download, `./openclaw gateway` just works
- Go cross-compiles trivially
- The macOS app (`scripts/package-mac-app.sh`) already follows this
  pattern — Swift wrapper embedding Node backend
- Native addon prebuilds are available for sharp, node-pty, etc.
- Supports self-update (like Sparkle for the macOS app)

**Cons:**
- Large binary (200-400MB uncompressed; ~100-150MB with zstd/upx)
- Each new native addon must be validated per platform
- First-run extraction adds a few seconds of latency

**Verdict: Best general-purpose option. Proven pattern (the macOS app does this already).**

---

### 2. Node.js SEA (Single Executable Application)

Node 22+ can bundle JavaScript into the Node binary itself.

**How it works:**

```bash
# Generate blob
node --experimental-sea-config sea-config.json
# Inject into Node binary copy
cp $(which node) openclaw
npx postject openclaw NODE_SEA_BLOB sea-prep.blob --sentinel-fuse NODE_SEA_FUSE_fce680ab2cc467b6ac44c7...
```

**Pros:**
- No rewrite — uses existing `dist/` output directly
- Officially supported by Node.js

**Cons:**
- Native addons (`.node` files) cannot be embedded in the SEA blob — they
  must ship alongside the binary as separate files
- No solution for the Python `loop_core` component
- Results in a "binary" that is really Node.js + your JS, not truly standalone
- Still requires `node_modules/` for native deps

**Verdict: Partial solution. Good for the gateway-only case, but doesn't handle
native addons or Python cleanly.**

---

### 3. Deno/Bun Compile

Both offer `deno compile` / `bun build --compile` to produce standalone binaries.

**Pros:**
- Single command to produce a binary
- Smaller output than Go wrapper approach

**Cons:**
- OpenClaw uses Node-specific APIs heavily: `node-pty`, `node:` builtins,
  native N-API addons — Bun and Deno have incomplete support
- Would require significant porting effort (rewriting native addon usage)
- No Python `loop_core` solution
- Risk of subtle runtime incompatibilities

**Verdict: Not viable without major porting work.**

---

### 4. Docker-in-a-Binary (Podman/Lima)

Ship a Go binary that manages a rootless container.

**How it works:**

```
openclaw (Go binary, ~30MB)
│
└─ On first run:
   1. Check for podman/docker, offer to install if missing
   2. Pull ghcr.io/openclaw/openclaw:latest (~500MB image)
   3. Run container with volume mounts for config/workspace
   4. Expose gateway port to host
```

**Pros:**
- Zero porting — the existing `Dockerfile` and `docker-compose.yml` work as-is
- Handles both Node and Python perfectly
- Isolation and security from containerization

**Cons:**
- Requires container runtime on the host (Podman or Docker)
- Large download (Docker image ~500MB+)
- Not a "native" experience — networking, file access are mediated
- `setup-podman.sh` and `docker-setup.sh` already exist, so this just
  adds a wrapper

**Verdict: Good for server deployments. Poor for "download and double-click" laptop use.**

---

### 5. AppImage / Flatpak (Linux only)

Bundle Node.js + Python + all deps into a self-contained Linux package.

**Pros:**
- Well-understood packaging for Linux desktops
- Can include all runtimes and libraries

**Cons:**
- Linux only — no macOS or Windows
- AppImage can be 300-500MB
- Flatpak requires Flatpak runtime on the host

**Verdict: Good Linux-specific option, but not cross-platform.**

---

### 6. Rust Wrapper (Alternative to Go)

Same approach as Strategy 1, but using Rust instead of Go.

**Pros:**
- Smaller binaries than Go (no GC runtime)
- `rust-embed` / `include_bytes!` for asset embedding
- Better compression characteristics

**Cons:**
- More complex build tooling
- Slower to prototype than Go
- Cross-compilation requires more setup (cross-rs, zig cc)

**Verdict: Viable alternative to Go if binary size is a priority.**

---

## Recommendation

**Strategy 1 (Go wrapper)** is the strongest approach for "run on any laptop":

1. **Precedent exists in this codebase** — the macOS app is already a native
   wrapper (Swift) that embeds and manages the Node.js backend via IPC.
   A Go binary does the same thing cross-platform.

2. **Native addon problem is solved** — `sharp`, `node-pty`, `sqlite-vec` all
   ship platform-specific prebuilds. Use `pnpm deploy --prod` to get a minimal
   `node_modules/`, then embed per-platform.

3. **Python embedding is solved** — The
   [python-build-standalone](https://github.com/indygreg/python-build-standalone)
   project provides relocatable Python builds for every major platform.

4. **User experience**: Download one file, run `./openclaw gateway`, done.

### Sizing Estimates

| Component | Approximate Size |
|---|---|
| Portable Node.js 22 | ~45MB |
| `dist/` (built JS) | ~5MB |
| `node_modules/` (pruned, prod) | ~80-120MB |
| Native addons (per platform) | ~20-30MB |
| Portable Python 3.x | ~30MB |
| `loop_core/` + pip deps | ~10-20MB |
| Go wrapper binary | ~8MB |
| **Total (uncompressed)** | **~200-250MB** |
| **Total (zstd compressed)** | **~80-120MB** |

### Key Risks to Mitigate

- **Native addon breakage**: CI must build and test on all target platforms
- **Update mechanism**: Implement self-update (check GitHub releases, download delta)
- **Optional components**: Make Python/loop_core optional — many users only need the gateway
- **First-run latency**: Cache extraction so it only happens once per version
- **Platform testing**: Especially important for ARM Linux (Raspberry Pi, Synology NAS — mentioned in existing Dockerfile comments)
