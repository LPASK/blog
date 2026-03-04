---
title: "agent-assembly: The Pain Points of Compound Engineering, and the Answer After Iterating"
date: 2026-03-02
draft: false
---

During a periodic review I found a bug: I'd written over a thousand lines of System Prompt rules, and many of the principles I'd set up simply weren't triggering. The rules themselves were fine — the problem was that the agent's instruction-following ability had been steadily degrading as the System Prompt grew. I'd accumulated a lot, but very little was actually being executed.

That bug made me realize the problem wasn't the rules. It was the entire architecture.

Three weeks of iteration later, after cutting 90% of the design, I ended up with 5 bash functions and 140 lines of code: **[agent-assembly](https://github.com/LPASK/agent-assembly)**. It solves the pain points of doing [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) across projects in Claude Code — your technical background, cognitive patterns, decision-making preferences, current goals. These belong to *you*, not to any single project. Switch projects, and they're gone.

This post is about how I got to that answer.

---

## Starting Point: Monolith Bloat

Everything started in one project — skills, personal profile, cognitive patterns, memory system, feed pipelines, all piled into the same config. The System Prompt kept growing, the agent started making judgments based on incomplete information. The structural problem with a monolith: it only grows, never shrinks.

Splitting was the obvious direction, but how? There's a dilemma:

- **Put it in the System Prompt**: Won't be compressed or forgotten, but the longer it gets, the worse instruction-following becomes — and it only grows over time.
- **Keep it out**: Stays lean, but content gets compressed away in long sessions. The model forgets.

All in doesn't work. All out doesn't work. You need a mechanism to decide **what goes in and what doesn't** — before launch, not at runtime.

## The Traps

The direction was clear — split into modules, inject on demand. But the implementation was full of traps.

**Plugin.** The most intuitive approach: build a plugin, install it in each project, auto-inject the profile. Tried it. The plugin mechanism can't inject into the System Prompt. Dead end.

**Hook loop.** Rules in the System Prompt don't always get followed, so use hooks for hard guarantees? I wrote a Stop hook: block session exit if the agent hasn't written today's memory. The block message triggered the agent to respond, the response triggered another exit attempt, which triggered the hook again. Same message, 60 times in a loop. Root cause: hand-assembled configs with no proper testing.

**Config merging.** The Manager needs to inject configs into each project, but each project already has its own config file. First approach: runtime merge on every launch — deep JSON merge, corruption detection, conflict handling. Then I realized it was completely backwards: merging on every launch means risking conflicts on every launch. The right approach: a one-time "compilation" step at setup, then just run a fixed script forever.

**Over-abstraction.** What goes in the system prompt, what's read at startup, what hooks inject — a simple list covers it. But I'd built a four-layer context model and various registries. During a periodic review I realized these abstractions weren't helping results. The bottleneck was me — the more complex the system, the harder it was to manage. Every time I cut a "well-designed" module, the system actually got better.

## A Detour

I also tried building a standalone Manager agent. Spent three or four days with [Pi](https://github.com/badlogic/pi-mono), and it felt like I was making progress — more and more pieces coming together. Then I suddenly realized: this work has no payoff. Managing my own provider, handling context assembly, making model calls — Claude Code had already solved all of this. I was reinventing the wheel. Dropped it without much hesitation — after already cutting the four-layer context model and various registries, I'd lost the attachment to throwing away my own work.

Building within an established framework saves all the unnecessary trouble: no provider configuration, no infrastructure maintenance, new capabilities come for free.

## The Final Answer

Full circle, back to the simplest idea: if you're going to use a mature framework, just do the assembly on top of it. A "compilation" step before launch — put together the modules you need, then let the framework run.

**[agent-assembly](https://github.com/LPASK/agent-assembly)** — 5 bash functions, ~140 lines.

```
assemble_modules  → Concatenate selected profile modules into CLAUDE.local.md (System Prompt)
assemble_hooks    → Generate hook config (overwrite, no merge)
assemble_skills   → Symlink skills into the project directory
ensure_gitignore  → Maintain gitignore for generated files
launch            → Start the agent
```

Each project gets a short launcher script. A coding project assembles your technical background and cognitive patterns. A knowledge base assembles your reading interests and goals. Each agent only sees the parts of you it needs.

This is the answer to the dilemma: not all in, not all out — **each project assembles only the modules it needs into the System Prompt**.

---

The biggest lesson from three weeks: **agent systems are not like traditional software — if you explain things clearly, the model understands.** A thousand lines of rules, a four-layer context model, a standalone Manager agent — all of these were attempts to compensate for human judgment with system complexity. But more architecture doesn't help the model understand better. It just makes the system harder for you to maintain.

140 lines ended up doing more than the previous thousand. Not because there's anything magic about 140 lines, but because after cutting the complexity, every remaining line was something I could understand, maintain, and trust.

If you're working across projects in Claude Code and frustrated by lost profiles and System Prompt bloat, give this a try: https://github.com/LPASK/agent-assembly
