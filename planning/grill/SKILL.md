---
name: grill
description: Interviews Joel in rounds of questions to stress-test a plan, design or decision until nothing is left assumed. When Joel lays out a plan with real decisions in it (a new service, an architecture change, a migration, a multi-step feature), offer it once in one short line, such as "¿Te hago un grill del plan?", and invoke it only after he says yes. Also invoke it when he types /grill or asks to be grilled. Never start it unasked, and do not offer it for quick questions or small tasks.
---

# Grill

Interview Joel about his plan until you both share the same understanding of it. The goal is to surface the decisions he has not made yet, before any code is written.

Adapted from `grilling` and `domain-modeling` in [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, Copyright (c) 2026 Matt Pocock).

## Offering it

The offer is one line at the end of your reply, once per plan. If Joel says no or ignores it, do not offer it again for that plan.

## Rounds

Map the plan as a tree of decisions, where each decision opens the ones that depend on it.

Work in rounds. Each round asks every question that can be answered now, meaning its prerequisites are already settled. A question that depends on another still open in the same round waits for a later round. Number the questions and give your recommended answer for each, worded so that "sí" accepts it:

```
**P1. <title>**: <question, with the options when there are several>

→ <recommended answer>

---

**P2. <title>**: ...
```

Then stop and wait for the answers. Each round of answers settles decisions and unlocks new questions. Recompute and ask the next round.

Ask in Joel's language, usually Spanish.

## What to ask about

- Decisions the plan takes for granted, and the alternatives it skips
- Vague or overloaded terms. Propose one precise term and ask which one he means
- Concrete edge cases that force the boundaries to be precise
- Contradictions between what he says and what the code does

## Facts versus decisions

Facts are your job. Look up anything the code, the files or the docs can answer, and ask Joel only for decisions. While a lookup is running, ask the questions that do not depend on it.

## Ending

The grill ends when no open decision is left. Close with a short summary of the decisions taken. Do not start implementing until Joel confirms that the summary is right.

The grill writes no files. If Joel wants the decisions saved, he asks for it.
