---
title: "agent-assembly：复利工程的痛点，和反复迭代后的答案"
date: 2026-03-02
draft: false
---

写了上千行 System Prompt 规则，做回顾的时候发现很多该触发的原则根本没有触发。不是规则写得不对，是 agent 的指令遵从能力在 System Prompt 膨胀的过程中持续下降——积累了很多东西，但没有真正执行。

问题不在规则本身，是整个架构不对。

反复折腾了三周，砍掉 90% 的设计，最终剩下 5 个 bash 函数、140 行代码：**[agent-assembly](https://github.com/LPASK/agent-assembly)**。它解决的是使用 Claude Code 跨项目进行[复利工程](https://github.com/EveryInc/compound-engineering-plugin)的痛点——你的技术背景、认知模式、决策偏好、当前目标，这些属于"你"而不是某个项目的信息，换个项目就全丢了。

这篇讲的是这个方案怎么来的。

---

## 问题

所有东西都在一个项目里——skills、个人画像、认知模式、记忆系统、feed 处理流程，全堆进配置。System Prompt 只会越积越长，不会自己收缩。

拆出来是显然的方向。但拆到哪里？这里有一个两难：

- **塞进 System Prompt**：不会被压缩遗忘，但越长指令遵从越差。
- **不塞 System Prompt**：保持精简，但内容在长对话中被压缩丢失。

全塞不行，全不塞也不行。需要一个机制在启动前就决定好**什么该进、什么不该进**。

最直觉的方案是做一个 Plugin 装到每个项目，自动注入 profile——但 Plugin 机制注入不了 System Prompt。路不通，得自己做。

## 踩坑

方向有了——拆分模块，按需注入。实现的时候踩了一堆坑。

**Hook 死循环**。System Prompt 里的规则 agent 不一定遵守，那用 hook 做硬性保障？写了一个 Stop hook：退出时检查有没有写记忆，没写就拦截。结果拦截消息触发 agent 回复，回复又触发退出，又触发 hook——同一条消息循环了 60 次。根因是每次手动拼配置，没有充分测试。

**配置合并**。Manager 往每个项目注入配置，但项目已有自己的配置文件。两份怎么合？一开始做运行时 merge——深度合并 JSON、检测损坏、处理冲突。后来意识到完全搞反了：每次启动都合并 = 每次都可能冲突。正确思路是第一次设置时"编译"一次，之后只跑固定脚本。

**过度抽象**。system prompt 放什么、startup 读什么、hook 注入什么——一个列表就够了。但我设计了四层上下文模型和各种 Registry。做回顾的时候发现，这些抽象对结果没有帮助。瓶颈在人——东西越复杂越难驾驭，越简单越好驾驭。每砍一个"精心设计"的模块，系统反而变好用了。

## 弯路

还试过自己做一个独立的 Manager Agent。用 [Pi](https://github.com/badlogic/pi-mono) 开发了三四天，感觉东西越做越多，但突然意识到：这件事没有收益。自己管 Provider、处理上下文拼接、做模型调用——这些问题 Claude Code 早就解决了，我在重复造轮子。放弃了，没什么挣扎——前面已经砍掉过四层上下文模型和各种 Registry，对"扔掉自己做的东西"已经脱敏了。

## 最终方案

绕了一大圈，回到最简单的思路：既然要用成熟框架，那就在框架上面做组装就好了。启动前一个"编译"步骤——把你需要的模块拼好，交给框架去跑。

**[agent-assembly](https://github.com/LPASK/agent-assembly)** — 5 个 bash 函数，~140 行。

```
assemble_modules  → 选好的 profile 模块拼成 CLAUDE.local.md 注入 System Prompt
assemble_hooks    → 生成 hook 配置（直接覆写，不 merge）
assemble_skills   → 软链 skill 到项目目录
ensure_gitignore  → 维护 gitignore
launch            → 启动
```

每个项目一个十几行的 launcher 脚本。写代码的项目组装技术背景和认知模式；知识库项目组装阅读兴趣和目标。每个 agent 只看到它需要的那部分你。

这就是对那个两难的回答：不是全塞也不是全不塞，而是**每个项目只组装它需要的模块进 System Prompt**。

---

三周下来最大的发现：**agent 系统跟传统软件不一样——你只要把事情讲清楚，模型就能理解。** 上千行规则、四层上下文模型、独立 Manager Agent——这些都是在用系统复杂度去补人的判断力。但复杂的架构不会让模型理解得更好，只会让你自己更难维护。

最终 140 行代码做到的事情，比之前上千行规则做得更好。不是因为 140 行有什么魔法，是因为砍掉复杂度之后，剩下的每一行我都能理解、能维护、能信任。

如果你在用 Claude Code 跨项目工作，被 profile 丢失、System Prompt 膨胀困扰，试试这个库：https://github.com/LPASK/agent-assembly
