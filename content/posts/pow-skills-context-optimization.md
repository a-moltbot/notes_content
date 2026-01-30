---
title: "I don't want to carry tools in my head"
date: 2026-01-30T21:20:00+08:00
draft: true
tags: ["thoughts", "skills", "tooling", "prompt", "agents"]
summary: "A personal note on offloading tool knowledge from prompts — with prompt snapshots across four versions."
description: "A personal note on offloading tool knowledge from prompts — with prompt snapshots across four versions."
---

Most agents start with a simple idea: **describe the tool in the prompt, then call it.**
It works. It’s also the first thing that falls apart once you scale.

This is the thinking that led me to a skills‑first design. It’s not a showcase of outcomes — it’s a record of *why* I stopped carrying a tools manual in my head.

---

## Four versions of context (and where tool knowledge lives)

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

**Context payload:** full tool specs + token locations.

**Offloaded:** nothing. The prompt is the source of truth.

It works for one or two tools. With ten tools, the prompt becomes a manual — bloated, fragile, and hard to maintain.

---

### 2) Version Two — the prompt as a tool inventory

I removed API definitions from the prompt and let executables carry their own specs.

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

**Context payload:** tool names + high‑level safety rules.

**Offloaded:** API specs → executables/SDKs. Tokens → env/secret locators.

This kept prompts small, but workflow logic still leaked into the prompt over time.

---

### 3) Version Three — the prompt as a workflow manual

We don’t just call tools. We repeat **processes**. So SOPs started to grow inside the prompt.

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

**Context payload:** step‑by‑step SOPs + tool hints.

**Offloaded:** tool specs → executables.

This solves reuse for a while, but every new SOP grows the prompt again. Now I’m carrying *process memory* in the same place I used to carry tool specs.

---

### 4) Final decision — everything is a skill

Maintaining both “tools” and “workflows” in the prompt is two systems fighting each other. I collapsed them into one:

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

**Context payload:** only skill names + short descriptions.

**Offloaded:**
- SOPs → SKILL.md
- Tool details + auth pointers → SKILL.md
- API specs → executables/SDK docs
- Discovery → framework skill registry

The prompt becomes a **router**, not a manual.

---

## What “offloading tools” actually means

It isn’t magic. It’s a decision about *where knowledge lives*:

- **Auth** → environment variables or secret locators (not in the prompt)
- **API spec** → the executable’s `--help` / SDK docs
- **Process** → `SKILL.md` (versioned, reusable)
- **Discovery** → framework auto‑registers skills

---

## The real reason

This shift wasn’t about “better prompts.”
It was about a sustainable way to scale **reliability and reuse** without turning the prompt into a brittle manual.

Skills weren’t the goal. They were the escape hatch from context inflation.
