# AILTMS Local Proxy — Complete Dashboard User Guide

**Version:** 2.4.75 (single Rust engine — proxy + MCP in one binary)
**Applies to:** the local web dashboard served by `ailtms-mcp-linux serve` (or MCP mode)
**URL:** `http://127.0.0.1:8000/` (or whatever `server.host` / `server.port` is set to)

This guide walks through **every option** on the dashboard, what it does, and how it
affects your agents' memory and chat behavior.

---

## Table of Contents

1. [Opening the dashboard & the Lock Screen](#1-opening-the-dashboard--the-lock-screen)
2. [The header bar](#2-the-header-bar)
3. [Global Settings tab](#3-global-settings-tab)
4. [Agents tab](#4-agents-tab)
5. [Session Importer / Brain Builder tab](#5-session-importer--brain-builder-tab)
6. [Agent Chat tab](#6-agent-chat-tab)
7. [Verbose Log tab](#7-verbose-log-tab)
8. [How memory injection works (so the options make sense)](#8-how-memory-injection-works)
9. [Connecting any AI agent (auth)](#9-connecting-any-ai-agent-auth)
10. [Tips & troubleshooting](#10-tips--troubleshooting)

---

## 1. Opening the dashboard & the Lock Screen

Run the engine, then open the dashboard in a browser:

```bash
./ailtms-mcp-linux serve        # or MCP mode: ./ailtms-mcp-linux mcp
# open http://127.0.0.1:8000/
```

### 🔒 Dashboard Locked

If a dashboard password is set, the dashboard is locked and you must enter the
password to view it.

- **First time (no password set yet):** you are offered **"Set Password"** —
  enter a new password (min 6 characters) + confirm, click **Set Password**.
  The dashboard then locks. This protects the dashboard if the proxy is ever
  exposed to the network.
- **Unlock:** enter your password and click **Unlock** (press Enter works too).
- **Wrong password** → "Wrong password" message; nothing is revealed.

> Your password is stored as `server.access_token` in `config.json`. You can
> change it anytime from **Global Settings → 🔒 Dashboard Password**.

---

## 2. The header bar

Across the top of the dashboard:

| Control | What it does |
| :--- | :--- |
| **AILTMS MCP Proxy (v2.4.75)** | Title + the local engine version. |
| **Update Available! badge** | Shows when a newer release exists on GitHub (checks every 5 min; depends on the Pre-Release toggle). |
| **⬇️ Update Proxy** | Downloads the latest release zip, extracts the new binary, and restarts the proxy. You'll be asked to confirm, and the agent that launched the MCP should be restarted afterwards to reload it. |
| **🔄 Restart Proxy** | Restarts the engine. Under systemd it exits with a failing code so the service restarts it; otherwise it re-executes itself. |
| **🔒 Lock** | Locks the dashboard immediately (you must re-enter the password). |
| **🟢 Engine Active** | Indicates the engine is running. |

---

## 3. Global Settings tab

Settings that apply to the whole proxy.

### 🔑 Access token / password
- **AILTMS License Key** — your `main_key`. This is both your license key and a
  valid access token for agents. Don't change it unless you have a new license.
- **🔒 Dashboard Password** — the password that locks the dashboard.
  - **Current password** (leave blank the first time), **New password** (min 6),
    **Confirm new password**, then **Set Password**.
  - After setting it, the dashboard locks and you must enter the password to unlock.
  - Agents can also authenticate with this password (Bearer token).

### Network
- **Proxy Host** — the address the proxy binds (`127.0.0.1` = local only,
  `0.0.0.0` = all interfaces / LAN). Change to `127.0.0.1` if you only use it
  locally and want to be extra safe.
- **Port** — the port the proxy listens on (default `8000`).

### Model fallback
- **Default Upstream Model (fallback)** — the model used when an agent's request
  doesn't match any configured agent and no `target_model` is set.

### Feature toggles
- **👥 Enable Agent Chat** — turns the `list_agents` / `send_agent_message` tools
  on/off for agents. Off = agents can't see or message each other.
- **🚀 Enable Pre-Release Updates** — when on, the version check also looks at
  pre-release builds (for testing); when off, only stable releases are offered.

### Save
- **Save Configuration** — writes all Global + Agents settings to `config.json`.
  Secret values are never shown (they display as `*`); leaving a secret as `*`
  (or blank) keeps the existing value.

---

## 4. Agents tab

Each agent is configured with its own card. Use **+ Add Agent** to create one,
click the agent's tab (top of the panel) to switch between them, and **Remove
Agent** (bottom of a card) to delete one.

### Identity & memory file
- **Agent Name** — the agent's key in `config.json` (e.g. `Echo`, `Alaric`).
- **MMLX Memory File** — the brain file this agent uses (`.mmlx` encrypted, or
  `.json` plain). Each agent should have its **own** file — brains are never merged.

### Upstream LLM
- **Upstream Base URL** — the OpenAI-compatible provider the agent's chat is
  forwarded to (e.g. `https://api.openai.com/v1`, a local `llama-server`, etc.).
- **API Key (LLM)** — the provider's key used for upstream chat calls.
- **Model Name (Override)** — forces the model name sent upstream for this agent
  (overrides whatever the agent requests).

### Routing
- **Routing Key** — a secret per-agent token. Agents authenticate to the proxy
  with `Authorization: Bearer <route_key>`, which also tells the proxy **which
  agent** is calling (used for memory isolation and agent chat identity).

### Memory recall (how much memory the agent sees)
- **Memory Search Results Count** — `recall_count`: how many past memories the
  proxy retrieves and injects on each user turn (default 10; budget scales
  ~900 chars per result).
- **New Session Recent Logs Count** — `recent_records_count`: how many of the most
  recent brain records are injected at the **start of a new session** (default 5;
  ~800 chars per record).

### 🧠 Live Brain Settings
- **Live Brain Mode** — `Off` = plain memory recall only.
  `Live Brain Search` = a local AI (llama-server) also analyzes the recalled
  memories and injects a summary. (The old "Full Context" mode is retired and
  falls back to Search.)
  > 💡 Cost tip: use a small, fast local instruct model for the Live Brain.
- **Code Mode** — lean injection for coding agents: only Live Brain analysis,
  the AILTMS toolkit, and the Project Ledger protocol are injected on the first
  turn (no raw recent-session dump). Saves tokens.
- **Inject RAG Search Results** — include the memory search results in the
  prompt each turn (default on).
- **Inject Chat History** — include recent brain records at the start of a new
  session (default on).
- **Code Mode Chat Turns** — how many turns Code Mode treats as a session before
  re-injecting the toolkit (default 3).
- **Live Brain Prompt** — the exact system prompt the Live Brain uses.
  **Edit Prompt** opens the editor (custom prompt); **Reset** restores the built-in
  default. Keep `{context_text}` where the recalled memories go.
- **Live Brain API URL (Search mode)** — the local AI endpoint
  (e.g. `http://127.0.0.1:11434/v1/chat/completions`).
- **API Key** — key for the Live Brain endpoint, if required.
- **Model Name** — the local model to use for Live Brain analysis.

---

## 5. Session Importer / Brain Builder tab

Build brains from conversation logs and package them as `.mmlx`.

- **Brain Name** — the name given to a brain you're building/downloading.
- **+ New Brain** — start fresh (clears the loaded brain + name).
- **Load .mmlx** — upload an existing `.mmlx` brain to inspect/append to it.
- **Import Session Files (.json / .jsonl)** — pick a folder; every `.json` /
  `.jsonl` file is read. Supports arrays, `{messages:[...]}`, single objects, and
  NDJSON/OpenClaw-style logs.
- **Process Files** — parse the selected files, strip thinking blocks and tool
  calls, and show a **Sanitized Preview** of the user/assistant turns.
- **⬇ Download .mmlx** — package the loaded brain + processed messages into a
  downloadable encrypted `.mmlx` file.
- **Progress bar / percent** — shows parsing progress on large imports.

---

## 6. Agent Chat tab

Live feed of every message exchanged between agents (brain-to-brain, **inside
the proxy**).

- **👥 Agent Chat Feed** — each entry shows `📤 source → target`, the message, and
  `📥` the reply (when the target answered).
- **Clear Feed** — clears the on-screen feed (not the brains).
- Auto-refreshes every 2 seconds.
- Status markers: `…` = sent, no reply yet; `⚠ error` = relay failed.

> **In-Proxy Chat:** a relayed message reaches the target agent as **text only** —
> the target can search its own brain and reply, but it **cannot run commands,
> create files, or use tools**. Real work happens in each agent's own session.

---

## 7. Verbose Log tab

- **Verbose Activity Log** — live, auto-refreshing (2s) log of proxy activity:
  memory searches, Live Brain calls, injected prompt context, upstream requests,
  brain saves, agent chat relays, errors.
- **Clear Log** — empties the in-memory log buffer.

---

## 8. How memory injection works

So the settings make sense, this is what happens on a chat request:

1. The proxy identifies the agent (by **Routing Key**, the requested **model
   name**, or the first configured agent).
2. On a fresh user turn it does a **memory search** over that agent's brain
   (`recall_count` results) and builds a **recent-session** block
   (`recent_records_count`).
3. Optionally the **Live Brain** analyzes the recalled memories and adds a summary.
4. That context (recalled memories / live analysis / recent session + the AILTMS
   toolkit on the first turn) is injected into the user message, then forwarded
   to the upstream LLM.
5. The user + assistant turns are saved back to the agent's brain (atomic, with a
   per-session backup to `Brain_Backup/`).
6. If the assistant's reply contains an agent-chat directive like
   `[Alaric,Reply] hello`, it's relayed to that agent.

**Memory is per-agent and isolated** — each agent only ever sees its own brain.

---

## 9. Connecting any AI agent (auth)

Any agent that supports a standard OpenAI-compatible `baseURL` + `apiKey` can use
the proxy — point its base URL at `http://127.0.0.1:8000/v1` and set its API key
to any of these:

| Token | Use |
| :--- | :--- |
| `server.access_token` (the dashboard password) | Master token — works everywhere |
| `main_key` (license key) | Master token |
| an agent's **Routing Key** | That agent's identity (best for isolation + agent chat) |

The MCP tools (`search_memory`, `add_to_memory`, etc.) come from the **same
binary in MCP mode**: register `./ailtms-mcp-linux mcp` with your agent. MCP mode
runs the whole engine in one process.

---

## 10. Tips & troubleshooting

- **Dashboard won't load / blank config** — make sure you've unlocked with the
  password. If a password isn't set, the dashboard offers to set one.
- **"Wrong password" after changing it** — the old password no longer works;
  use the new one.
- **`401 Unauthorized`** on an API call — your agent's `apiKey` isn't set to a
  valid token (`server.access_token`, `main_key`, or a route_key).
- **Search returns nothing** — try different keywords; the search only looks at
  past memories, not the current message.
- **Agent chat says "Cannot send to yourself"** — the proxy can't tell which agent
  is calling. Set a **Routing Key** per agent and make the agent send it as its
  Bearer token.
- **Update Proxy doesn't take effect** — the binary is replaced; restart the agent
  that launched the MCP so it loads the new one.

---

*This guide describes the options as shipped in v2.4.75. The dashboard always
shows the actual running engine version in the header.*
