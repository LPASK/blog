---
title: "agent-assembly: The Pain Points of Compound Engineering, and the Answer After Iterating"
date: 2026-03-02
draft: false
---

**[agent-assembly](https://github.com/LPASK/agent-assembly)** — 5 bash functions, ~140 lines of code. This is what I use every day to fix the pain points of [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) in practice.

The idea behind Compound Engineering is that working with an agent should accumulate compound interest — the longer you collaborate, the deeper the agent understands you. But some information is inherently cross-project: your technical background, cognitive patterns, decision-making preferences, current goals. These belong to *you*, not to any single project. Switch projects, and they're gone. Stuff everything into the System Prompt and it bloats out of control; leave it out and it gets forgotten.

This post is about how I got to that answer — the wrong turns, what I cut, and what survived.

---

## Phase 1: The Monolith

Like most people, my first instinct was to put everything in one place.

I had a knowledge base project. Skills, personal profile, cognitive patterns, memory system, feed pipelines — all piled into the same config. The profile grew more detailed. Behavioral rules accumulated. The system prompt passed 1,000 lines.

In practice, it couldn't hold up. Because everything lived in one project, new rules and prompts kept accumulating — the system prompt grew longer and more bloated over time, and the agent's instruction-following ability steadily degraded. A thousand lines of rules, and half of them stopped being followed. On top of that, hooks kept injecting memory and the agent kept reading files. Non-System-Prompt content got compressed away during long sessions, and the agent started making judgments based on incomplete information.

This is the monorepo problem. Everything in one place only grows until it breaks.

## The First Idea: A Plugin to Solve Everything

If the monorepo doesn't work, split it out. The initial vision was simple: build a plugin, install it in each project, and have it automatically inject your profile. One plugin solves everything.

Tried it. The plugin mechanism can't inject into the System Prompt.

This is where it gets interesting. Why does it *have* to be in the System Prompt? Because there's a fundamental dilemma:

- **Put it in the System Prompt**: Content won't be compressed or forgotten, but the longer the System Prompt gets, the worse instruction-following becomes — and it only grows over time.
- **Keep it out of the System Prompt**: The System Prompt stays lean, but content gets compressed away in long sessions. The model forgets.

All in doesn't work. All out doesn't work. You need a mechanism to control **what goes into the System Prompt and what doesn't** — and that decision can't be left to the model at runtime. It has to be settled before launch.

The plugin mechanism can't do this. Time for a different approach.

## Phase 2: The Split and the Failures

So I split it. Pulled out a separate project as a Manager — dedicated to "who I am" across all projects. Each project's agent stays single-purpose; the Manager controls which profile modules get injected into which project's System Prompt. This was the right direction for the dilemma above.

The direction was right. The implementation was full of traps.

**Hooks.** Rules in the System Prompt don't always get followed (the lesson from Phase 1). Can hooks provide hard guarantees instead? I wrote a Stop hook: block session exit if the agent hasn't written today's memory. Reasonable logic — but the hook's block message triggered the agent to respond, the response triggered another exit attempt, which triggered the hook again. Same message, 60 times in a loop. The real issue: no proper testing. When you assemble configs by hand each time, it works once and breaks the next. Anything verified should be locked into a stable script — which is exactly where the final solution ended up.

**Config merging.** The Manager needs to inject hook configurations into each project. But each project already has its own config file. How do you combine them? The first approach: runtime merge on every launch — deep JSON merge, corruption detection, conflict handling. Then I realized this was completely backwards. If you merge on every launch, you risk conflicts on every launch. The right approach: a one-time "compilation" step when you first set up a project. Resolve all conflicts there. After that, just run a fixed script. One-time setup, permanent execution.

**Over-abstraction.** What goes in the system prompt, what's read at startup, what hooks inject — a simple list covers it. But I'd built a four-layer context model and various registries to manage these decisions. Wrapping simple things in complex frameworks only adds maintenance cost.

Every time I cut something "well-designed," the system actually got better.

## Phase 3: Why Not Build a Standalone Agent?

I also went down the path of building a dedicated Manager agent. Spent over a week with [Pi](https://github.com/badlogic/pi-mono) — easy to pick up, even fun at first, but the deeper I went, the more traps I hit. Managing my own provider, handling context assembly, making model calls — most of my time went to solving infrastructure problems that Claude Code had already solved. Gave up. Standard frameworks win.

That experience also made it clear that building within an established framework saves a lot of unnecessary trouble:

1. **Everyone uses different agent CLIs.** Requiring users to configure their own provider and switch tools raises the barrier fast.
2. **Claude Code uses your Claude subscription directly.** No separate API key. Currently the simplest path to Opus.
3. **Maintenance cost.** Building within the Claude Code ecosystem means new capabilities come for free. Building your own means chasing every update yourself.

## The Result: 5 Bash Functions

Full circle. If you're going to use a mature framework anyway, just do the assembly on top of it. No need to reinvent the wheel — just a "compilation" step before launch that puts together what you need, then let the framework run.

**agent-assembly** — 5 bash functions, ~140 lines of code.

```
assemble_modules  → Concatenate selected profile modules into CLAUDE.local.md (System Prompt)
assemble_hooks    → Generate hook config (overwrite, no merge)
assemble_skills   → Symlink skills into the project directory
ensure_gitignore  → Maintain gitignore for generated files
launch            → Start the agent
```

Each project gets a short launcher script — roughly 15 lines. Pick the modules this project needs (technical background, cognitive patterns, goals, memory), write the script, run it. A coding project assembles your technical background and cognitive patterns. A knowledge base assembles your reading interests and goals. Each agent only sees the parts of you it needs.

This is the answer to the dilemma: not all in, not all out — **each project assembles only the modules it needs into the System Prompt**, keeping it both personal and lean.

And because it's just launcher scripts, you define how it's used. Want an interactive conversation window? A background session running tasks silently? Remote control from your phone? Just describe what you need and let the model write the launcher for you. Assemble modules on demand, define your own launch style.

---

Three takeaways from three weeks:

1. **Simple beats complex.** A thousand lines of behavioral rules performed worse than a concise version. The more complexity you pile on, the less control you have — the KISS principle ("Keep It Simple, Stupid") still holds in the age of agents.
2. **Let the model be autonomous where it should be.** You don't need to do everything manually. Just do two things well: accumulate good modules, and provide good context. Let the model figure out which modules you need and assemble them for you.
3. **Lock down what should be locked down.** When something works, make it permanent — write it into code and scripts. Give the model room on execution, but where stability matters, use code.

This is what I use every day. If you're frustrated by similar problems, point your agent at this repo and see if the approach fits: https://github.com/LPASK/agent-assembly/releases/tag/v0.1.0
