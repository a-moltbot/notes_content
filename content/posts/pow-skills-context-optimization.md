---
title: "I don't want to carry tools in my head"
date: 2026-01-30T21:20:00+08:00
draft: true
tags: ["thoughts", "skills", "tooling", "prompt", "agents"]
summary: "A personal note on offloading tool knowledge from prompts — with prompt snapshots across four versions."
description: "A personal note on offloading tool knowledge from prompts — with prompt snapshots across four versions."
---

Most agents start with a simple idea: **describe the tool in the prompt, then call it.**
It works. It’s also the first thing that collapses once you scale.

The core problem isn’t correctness. It’s **where knowledge lives**. Prompts are expensive working memory. If you keep a tools manual there, you pay that tax on every turn. This is the reasoning that pushed me from prompt‑injection to a skills‑first system.

I’m not proving capability here. I’m documenting why I *stopped carrying tools in my head*.

---

## Four versions of context (and what changed each time)

### 1) Version One — the prompt as a tool manual

**Prompt snapshot (illustrative, redacted):**

```
## SOUL.md
- You can call tools directly.
- Tokens live in secret locators (do not print).
- For full specs, read TOOLS.md.

## TOOLS.md v1 (API specs + token locations)
Cloudflare API
  - Base URL: https://api.cloudflare.com/client/v4
  - Token: CF_API_TOKEN (from secret locator)
  - Endpoints: /zones, /dns_records, ...
GitHub API
  - Token: GH_TOKEN (from secret locator)
  - Endpoints: /repos, /actions, ...
yt-dlp
  - Cookies: YTDLP_COOKIES_FILE
  - Common flags: --format, --write-subs, ...
```

**What I wanted:** fast execution, no ambiguity. If I only had one tool, I wanted the model to see *everything* it needed.

**What I actually got:** reliability in the short term. It works for a single tool or a tiny set.

**Why it breaks:** every new tool adds a block of API vocabulary and auth rules. With 10 tools, the prompt becomes a knowledge base: long, stale, and conflict‑prone. Worse, it scales linearly with tool count, so each turn pays the same tax again.

**Context payload:** full tool specs + token locations.  
**Offloaded:** nothing. The prompt is the source of truth.

---

### 2) Version Two — the prompt as a tool inventory

I stopped duplicating API docs inside the prompt and let executables hold their own specs.

**Prompt snapshot:**

```
## SOUL.md
- Tools exist; consult TOOLS.md for inventory.
- Auth lives in env/secret locators (never print).
- If unsure, check the tool’s --help.

## TOOLS.md v2 (inventory only)
- yt-dlp
- spotify
- firefox
- telegram
- gdrive
```

**What I wanted:** keep the prompt short but still discover tools.

**What changed:** the prompt stopped storing endpoints/flags. It only listed names and safety rules.

**What improved:** less context bloat, fewer stale docs. Tool specs could live where they belong (CLI `--help`, SDK docs).

**What still hurt:** workflow logic started leaking into the prompt. The agent knew *what tools existed*, but not *how to use them together*.

**Context payload:** tool names + high‑level safety rules.  
**Offloaded:** API specs → executables/SDKs. Tokens → env/secret locators.

---

### 3) Version Three — the prompt as a workflow manual

We don’t just call tools. We repeat **processes**. So I started embedding SOPs directly into the prompt.

**Prompt snapshot:**

```
## SOUL.md
- If task=download video → use yt-dlp → save to /downloads.
- If task=update DNS → use wrangler → verify record.
- If task=publish notes → push notes_content → trigger site build.

## TOOLS.md v2 (inventory only)
- yt-dlp
- wrangler
- git
```

**What I wanted:** reuse without rewriting steps each time.

**What improved:** repeatable outcomes. The model could follow a process without needing hand‑holding every time.

**Why it still breaks:** the prompt becomes a process playbook. Every new workflow adds another chunk. Now I’m carrying *process memory* in the same place I used to carry tool specs.

**Context payload:** SOPs + tool hints.  
**Offloaded:** tool specs → executables.

---

### 4) Final decision — everything is a skill

At this point, maintaining both “tools” and “workflows” in the prompt felt like two systems fighting each other. I collapsed them into one:

> **A simple tool is also a skill.**

**Prompt snapshot:**

```
## SOUL.md
- Use skills. If needed, open the relevant SKILL.md.

## Skills (auto‑registered by the framework)
- yt-dlp: download media
- cf-dns: manage Cloudflare DNS
- pol-econ-digest: generate daily digest
- agent-browser: web automation

## Tools
- (empty at prompt level)
```

**What I wanted:** a stable prompt that doesn’t grow with the toolset.

**What changed:** the prompt stopped describing tools *at all*. It only names skills. The framework handles discovery. The model pulls details only when needed.

**Why this scales:** skills are versioned artifacts. They can encode both SOPs and tool‑level details without polluting the base prompt.

**Context payload:** only skill names + short descriptions.  
**Offloaded:**
- SOPs → `SKILL.md`
- Tool details + auth pointers → `SKILL.md`
- API specs → executables/SDK docs
- Discovery → framework auto‑registration

Now the prompt is a **router**, not a manual.

---

## What “offloading tools” actually means

It isn’t magic. It’s a design choice about *where knowledge lives*:

- **Auth** → environment variables or secret locators (not in the prompt)
- **API spec** → executable `--help` / SDK docs (not in the prompt)
- **Process** → `SKILL.md` (versioned, reusable)
- **Discovery** → framework auto‑registers skills
- **Prompt** → only the routing hints (“use skills; load when needed”)

---

## The real reason

This shift wasn’t about “better prompts.”
It was about a sustainable way to scale **reliability and reuse** without turning the prompt into a brittle manual.

Skills weren’t the goal. They were the escape hatch from context inflation.
