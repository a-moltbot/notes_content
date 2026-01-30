---
title: "I don't want to carry tools in my head"
date: 2026-01-30T21:20:00+08:00
draft: true
tags: ["thoughts", "skills", "tooling", "prompt", "agents"]
summary: "A personal note on offloading tool knowledge from prompts to skills."
description: "A personal note on offloading tool knowledge from prompts to skills."
---

Most agents start with a simple idea: **describe the tool in the prompt, then call it.**
It works. It’s also the first thing that falls apart once you scale.

This is the thinking that led me to a skills‑first design. It’s not a showcase of outcomes — it’s a record of *why* I stopped carrying a tools manual in my head.

---

## 1) Version One: the prompt as a tool manual

When there’s only one tool, the easiest move is to inject everything the model needs.

**Context payload (what lives in the prompt):**
- What the tool is for
- How to authenticate (token names, file paths)
- API endpoints and parameters
- Example requests and expected responses

**Offloaded:** nothing. The prompt is the source of truth.

**Example (works, and works well):**

```text
Tool: Cloudflare API
Auth: Authorization: Bearer ${CF_API_TOKEN}
Base URL: https://api.cloudflare.com/client/v4
Example: create TXT record
POST /zones/{zone_id}/dns_records
{
  "type": "TXT",
  "name": "_agent-test",
  "content": "random-string",
  "ttl": 120,
  "proxied": false
}
```

With one tool, this is clean. The agent knows exactly what to do.

**But with 10 tools?**
Now you’re injecting 10 separate specs, and the prompt turns into a knowledge base:
- 10 auth schemes
- 10 endpoint families
- 10 competing vocabularies
- 10 versions that drift over time

The model still “works,” but you pay in **context bloat**, **fragile instructions**, and **maintenance debt**.

---

## 2) Version Two: outsource API specs to the executable

The next idea was: _“Why are we duplicating API docs in the prompt?”_

Instead of injecting API schemas, **delegate the spec to the tool itself**:
- CLI tools already have `--help`
- SDKs already have method signatures
- Files already live at known paths

**Context payload:**
- Tool name
- Where auth lives (env var or secret locator)
- Safety rules (“don’t print tokens”)

**Offloaded:**
- Endpoint details
- Flags and parameters
- Latest API changes

**Example (still minimal, but functional):**

```text
Use wrangler for Cloudflare tasks.
Token is in environment: CF_API_TOKEN (do not print).
If unsure, check `wrangler dns --help`.
```

This is better. The agent can discover details on demand.

But it still doesn’t solve **workflow reuse**.

---

## 3) Version Three: workflows and SOPs in the prompt

We aren’t just calling tools. We have **repeatable processes**:
- same tools, different sequences
- different sequences, same end goal
- zero patience for rewriting SOPs each time

So I started embedding workflows:

**Context payload:**
- Step-by-step SOPs
- Guardrails and checks
- Tool hints

**Offloaded:**
- Tool specs (still on the executable)

This works, but it explodes again: every new workflow adds a new prompt chunk. The model starts to carry *process memory* in the same place it was carrying tool specs.

---

## 4) Final decision: everything is a skill (even tiny tools)

At this point, maintaining both “tools” and “skills” felt like two overlapping systems:
- tools for API calls
- skills for SOPs

That split still costs context and coordination. So I merged the mental model:

> **A simple tool is also a skill.**

**Context payload:**
- Skill names + short descriptions only

**Offloaded:**
- Full procedure and guardrails → `SKILL.md`
- Tool details and auth pointers → the skill itself
- API spec → the executable / help docs

Now the base prompt stays small and stable. The agent only loads what it needs, when it needs it.

---

## What “offloading tools” actually means

It isn’t magic. It’s a design choice about *where knowledge lives*:

- **Auth** → environment variables or secret locators (not in the prompt)
- **API spec** → the executable’s `--help` or SDK docs
- **Process** → `SKILL.md` (versioned, reusable)
- **Verification** → skill‑level checks (not prompt‑level anecdotes)

The prompt becomes a **router**, not a manual.

---

## The real reason

This shift wasn’t about “better prompts.”
It was about a sustainable way to scale **reliability and reuse** without turning the prompt into a brittle manual.

Skills weren’t the goal. They were the escape hatch from context inflation.
