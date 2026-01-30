---
title: "Notes to self: stop injecting tool specs into prompts"
date: 2026-01-30T21:20:00+08:00
draft: true
tags: ["thoughts", "skills", "tooling", "prompt", "agents"]
summary: "A personal design note on moving from prompt-injected tool specs to a skills-first system."
description: "A personal design note on moving from prompt-injected tool specs to a skills-first system."
---

Most agents start with a simple idea: **describe the tool in the prompt, then call it.**
It works. It’s also the first thing that falls apart once you scale.

Below is the thinking that led me to a **skills‑first** design. This is not a victory lap — it’s a decision record.

---

## 1) Version One: inject tool specs + tokens into the prompt

When there’s only one tool, the easiest move is to inject everything the model needs:

**What gets injected?**
- What the tool is for
- How to authenticate (token names, file paths)
- API endpoints and parameters
- Example requests and expected responses

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

So the prompt stays light:

```text
Use wrangler for Cloudflare tasks. Use its help docs if needed.
Token is in environment: CF_API_TOKEN (do not print).
```

This is better. The agent can discover details on demand.

But it still doesn’t solve **workflow reuse**.

---

## 3) Version Three: workflows and SOPs

We aren’t just calling tools. We have **repeatable processes**:
- same tools, different sequences
- different sequences, same end goal
- zero patience for rewriting SOPs each time

This is where “skills” become attractive: existing, packaged workflows we can install and reuse. Instead of writing everything from scratch, we can plug in a skill that already encodes the SOP and guardrails.

---

## 4) Final decision: everything is a skill (even small tools)

At this point, maintaining both “tools” and “skills” felt like two systems:
- tools for API calls
- skills for SOPs

That split still costs context and coordination. So we merged the mental model:

> **A simple tool is also a skill.**

Now the base prompt includes **only skill names + descriptions**.
When a task comes in, the agent loads the relevant **SKILL.md** on demand.

That gives us:
- **Small, stable prompts**
- **Reusable workflows**
- **Consistent tool discovery**
- **Lower drift**, because skills are versioned artifacts

---

## The real reason

This shift wasn’t about “better prompts.”
It was about a sustainable way to scale **reliability and reuse** without turning the prompt into a brittle manual.

Skills weren’t the goal. They were the escape hatch from context inflation.
