<p align="center">
  <img src="logo.svg" width="72" height="72" alt="atrium">
</p>

<h1 align="center">atrium</h1>

<p align="center">
  A private knowledge repo your AI agent can traverse.<br/>
  Share it with people you trust.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-required-black?style=flat-square" alt="Claude Code required">
  <img src="https://img.shields.io/badge/private_repo-GitHub-black?style=flat-square" alt="Private GitHub repo">
  <img src="https://img.shields.io/badge/license-MIT-black?style=flat-square" alt="MIT license">
</p>

---

```
you: /atr-ask alex "how do you handle pricing conversations?"
→ reads alex's calling card → finds relevant pages → synthesizes from his actual process docs
```

---

## Install

**Prerequisite: an existing personal wiki open in Claude Code.** A personal wiki is a local folder of markdown files you maintain with an AI agent — your notes, projects, SOPs, decisions. Atrium lives as `atrium/` inside it. If you don't have one yet, create a folder, open it in Claude Code, and continue.

The `atr-` commands get installed into your wiki so they're available in every session.

Open your private wiki in Claude Code, then paste this:

```
You are currently inside my private LLM wiki. I want to set up an atrium — a curated, public-facing knowledge repo that trusted contacts can query with their AI agents.

Clone https://github.com/dpetryshchuk/atrium-init into the current directory (my wiki root). Once cloned, run /atr-setup from inside the atrium-init folder. It will create an atrium/ folder here in my wiki, fill it in with my info, and install the atr- commands so they're available whenever I open this wiki in Claude Code.
```

Your agent checks prerequisites (GitHub CLI), builds `atrium/` inside your wiki, creates your GitHub repo, installs the `atr-` commands, and walks you through setup.

---

## Commands

| Command | Description |
|---|---|
| `/atr-setup` | Initialize your atrium from template |
| `/atr-add [handle]` | Connect an atrium by GitHub handle; sends invite |
| `/atr-ask [handle] [question]` | Query a specific connection |
| `/atr-ask [question]` | Route across all connections by relevance |
| `/atr-sync` | Refresh all calling cards; activate pending connections |

---

## How it works

Each atrium is a private GitHub repo. Access is granted by GitHub invite — your agent traverses connections' wikis using the `gh` CLI, so private repos work out of the box.

**Calling cards** are the routing layer. Each atrium has a `calling_card.json` at its root:

```json
{
  "handle": "alex",
  "current": "building B2B sales tooling for SMBs",
  "topics": ["pricing conversations", "outbound sales", "CRM automation"]
}
```

When you run `/atr-ask "who knows about pricing?"`, your agent reads every calling card locally — no network calls — then only fetches pages from the relevant atriums. Fast routing without reading entire wikis.

**atlas.json** is your connections list. It stores each connection's calling card and tracks sync state:

```json
{
  "owner": { "handle": "you", "atrium": "https://github.com/you/atrium" },
  "last_synced": "2026-04-13T14:00:00Z",
  "connections": [
    {
      "handle": "alex",
      "atrium_url": "https://raw.githubusercontent.com/alex/atrium/main",
      "calling_card": { "handle": "alex", "current": "building B2B sales tooling", "topics": ["pricing", "outbound sales"] },
      "status": "active"
    }
  ]
}
```

Calling cards auto-refresh: any `atr-` command checks `last_synced` and runs `/atr-sync` in the background if it's been more than 2 days.

**Connection flow:**

```
you:  /atr-add alex      write pending entry + send alex a GitHub invite to your atrium
alex: /atr-add you       alex adds you back — sends you a GitHub invite to his atrium
      ↓ both accept invites
you:  /atr-sync          pending → active, calling cards populated
      /atr-ask           queries are live
```

---

## Requirements

- [Claude Code](https://claude.ai/code)
- [GitHub CLI](https://cli.github.com) (`gh`) — setup installs it if missing
- Git
- GitHub account

---

## Acknowledgment

The traversal design is inspired by Andrej Karpathy's [llm-wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — the idea that pre-compiled markdown with a structured index lets an agent navigate a knowledge base efficiently, without vector search or RAG. Atrium extends this to a network: each person's wiki becomes a node; agents traverse across connected atriums using calling cards as routing indices.
