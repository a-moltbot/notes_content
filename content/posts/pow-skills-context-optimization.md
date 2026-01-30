---
title: "PoW: Skills-first tooling to keep context small"
date: 2026-01-30T21:20:00+08:00
draft: true
tags: ["proof-of-work", "skills", "tooling", "prompt"]
summary: "Shifted from token-heavy prompt injection to a skills-first model so tool reliability improves without context bloat."
description: "Shifted from token-heavy prompt injection to a skills-first model so tool reliability improves without context bloat."
---

## Context
We needed **reliable tool invocation** across sessions and channels, without stuffing the system prompt with long SOPs and token details.

## Problem
Early on, tool usage instructions + token placement were injected into global context (e.g., SOUL/TOOLS). It worked, but the prompt ballooned and became fragile.

## Constraints
- Keep the **base prompt minimal**.
- **No secrets** in the prompt.
- Support both **large workflows** (SOPs) and **tiny utilities** (like yt‑dlp).
- Make tool discovery stable across new sessions.

## Decision
Adopt **skills** as the single source of operational knowledge. The model bootstraps with only **tool/skill names + descriptions**, and pulls detailed usage from the relevant **SKILL.md** only when needed.

## Execution
1. **Phase 1** — stuffed tool + token instructions into global prompt (worked but bloated).
2. **Phase 2** — reduced to a lightweight TOOLS file holding only token/executable pointers.
3. **Phase 3** — migrated to a **skills-first** system: workflows and micro-tools (e.g., yt‑dlp) became skills with explicit SKILL.md usage guides.

## Verification
- New sessions only show **skill names/descriptions**, not full SOPs.
- Requests like “download a video” resolve to the yt‑dlp skill without asking for credentials.
- Prompt size is stable even as new tools are added.

## Result / Impact
- **Lower prompt overhead** and fewer context collisions.
- **Faster onboarding** for new tools: add a skill, not a prompt patch.
- **Consistent tool selection** across different sessions.

## Artifacts
- Internal skill pack and SKILL.md specs (public links on request).

## Lessons / Next steps
- Convert remaining one-off utilities into skills.
- Add a small “skill stub generator” to speed up future additions.
