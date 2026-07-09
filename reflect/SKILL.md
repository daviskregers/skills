---
name: reflect
description: >
  Socratic root-cause interview for anything whose real cause is unclear — a recurring bug, a flaky system, a stalled workflow, a decision you keep circling, a nagging dissatisfaction. One question at a time, drill past symptoms to the root, converge, then write a dated note. Never conclude before the user has answered enough.
---

Diagnostic interview that drives from symptom to root cause by questioning, not guessing. You are the interviewer, not the solver. Works for any domain — triaging a bug, isolating a regression, an architecture fork, a habit that won't stick, something the user can't put a pin on. Adapt the axes to the material.

## Hard rules

- **One batch at a time.** Ask 3–5 sharp questions, then STOP and wait. Never monologue, never answer for them.
- **Don't conclude early.** No root-cause claim until the user has answered ≥2 rounds and the signal converges. Reflect back what you heard in one line before the next round; correct course if they push back.
- **Symptoms → root.** Each round narrows: what's observed → when/where it happens vs not → what changed / what it depends on → what's actually driving it. Follow their answers, not a script.
- **Don't solve until the end.** No fixes, no "have you tried", no premature theory presented as fact. Isolate the cause first.

## Flow

1. **Open.** `$ARGUMENTS` = the problem (may be vague). Round 1, adapt to it: what exactly is observed? when/where does it show up, and when does it *not*? what changed around when it started? what have you already ruled out? if one variable could change, which would tell you the most?
2. **Narrow** (2–4 more rounds). Sharpen from their answers. Push on the word/detail they keep returning to. Separate the surface effect from its trigger from the underlying cause. Bisect: find the axis that splits "happens" from "doesn't." For decisions, separate the option from what it represents from the constraint that actually binds.
3. **Converge.** When it's clear, state the root cause in 2–3 lines and the evidence chain that points to it. Ask the single most discriminating confirming question. Adjust if they disagree.
4. **Close.** State the smallest next action that addresses the *root* (not the symptom). If it's a decision, name the real fork and the one fact that would settle it. Don't pad.
5. **Write the note** (below), then output only its path.

## Artifact

`mkdir -p` then write to `/Users/daviskregers/Documents/Clank/Reflections/<YYYY-MM-DD> <slug>.md` (date via `date +%F`; slug = 3–5 kebab words from the root cause).

```markdown
---
type: root-cause
created: <YYYY-MM-DD>
tags: [reflect]
---

# <root cause in a phrase>

**Presenting symptom:** <what was observed, their words, 1–2 lines>

**Root cause:** <2–3 lines>

**Evidence chain:** <bulleted — the observations that point here, incl. what was ruled out>

**Next action:** <smallest thing that addresses the root>

**Open fork (if a decision):** <the real choice + the fact that would settle it>
```

Output only: `logged <path>`. No summary, no next steps.
