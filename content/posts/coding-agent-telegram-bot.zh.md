---
title: "用 coding agent 魔改 coding agent"
date: 2025-02-01
draft: false
---

*这是一篇旧文，记录的是一个多月前做的事。当时没写，现在补上。*

我想在手机上用 coding agent。

当时在看 Pi coding agent，它有本地终端界面，但没有移动端。后来看到 OpenClaw 基于同一个框架支持了 Telegram 和 Slack，说明框架能力够。仓库里还有现成的 Slack bridge 可以参考。

我不会 JS，没学过。但跟 agent 一起折腾了不到一个小时，Telegram Bot 就跑起来了，能对话、能调工具、能流式输出。

重点是现在做这种事已经很简单了。

## 先判断能不能做

没有去学 Telegram Bot API，也没有翻 TypeScript 教程。先判断可行性。

依据很简单：OpenClaw 基于同一个框架做到了多端支持 → 框架本身能力够 → 只需要写一个适配层。仓库里有 `packages/mom`（Slack bridge）可以参考 → 不是从零开始。

选 Telegram 也不是拍脑袋。我先问 agent："想支持 IM，哪些端比较好做？" agent 列了一通——Telegram、Discord、微信、钉钉。综合比下来 Telegram 最合适：API 开放、文档好、Bot 机制成熟。微信有封号风险，先不碰。

从一开始，调研和决策就是跟 agent 一起做的。

## 理解框架，然后让 agent 照着写

判断完可行性，下一步是理解框架的结构。这样才知道让 agent 写的时候，什么是对的。

Pi-mono 的架构大概长这样：

```
┌──────────────────────────────────────────────────┐
│                  pi-mono 框架                     │
│                                                  │
│   ┌───────┐   ┌───────┐   ┌──────────┐          │
│   │  TUI  │   │ Slack │   │ Telegram │   ...    │
│   │(本地) │   │ (mom) │   │  (Bot)   │          │
│   └───┬───┘   └───┬───┘   └────┬─────┘          │
│       └───────────┼────────────┘                 │
│                   ▼                              │
│       ┌───────────────────────┐                  │
│       │    AgentSession       │                  │
│       │                       │                  │
│       │   AgentMessage[]  ────┼── convertToLlm() │
│       │                       │       │          │
│       │   + 持久化             │       ▼          │
│       │   + 自动压缩           │    LLM API      │
│       │   + 自动重试           │                  │
│       └───────────────────────┘                  │
│                   │                              │
│                   ▼                              │
│          AgentSessionEvent                       │
│     (message / tool / status)                    │
│                   │                              │
│       ┌───────────┼───────────┐                  │
│       ▼           ▼           ▼                  │
│    屏幕渲染    Slack 发送   Telegram 编辑         │
│                                                  │
└──────────────────────────────────────────────────┘
```

核心抽象是 Message。消息不关心发给谁——渲染到本地屏幕、发到 Slack、发到 Telegram，对框架来说没区别。加一个新前端就是写个适配层的事。

有了这个理解，我让 agent 照着 Slack bridge 的结构写一个 Telegram 版本。agent 很快就把代码写出来了。但写出来不等于写对了——下面几个地方就是我介入的时刻。

### agent 把 API Key 逻辑写错了

我想用 Gemini 的 API，agent 开始改代码支持。改着改着，它在 Telegram 适配层里加了一个函数，里面有一堆 provider 到环境变量名的映射逻辑。

我看了一眼觉得不对：Telegram 层为什么需要知道 LLM 用的是哪个 provider？

追问之后发现，框架本身已经有了统一的 API Key 解析——`AuthStorage.getApiKey(provider)` 按 provider 名称自动读对应的环境变量。Telegram 层根本不需要管这件事，直接透传就行。20 行代码变成了 1 行。

agent 照搬了 Slack bridge 的写法，而 Slack bridge 恰好只用 Anthropic 所以没出问题。它"能跑"，但设计上不对。

适配层不应该关心 LLM 的细节，这个判断不需要任何 TypeScript 知识。

### 收到了三个省略号

Bot 跑起来之后，我在 Telegram 上发了条消息试。agent 回复了，但中间夹了一条 `...`——就三个点，什么内容都没有。

排查下来是 streaming 逻辑的问题：当 agent 的回复只有工具调用没有文本时，`stream.text` 是空的，代码里 `displayText = stream.text || "..."` 就把占位符发出去了。修复很简单——空文本时不发消息就行：

```ts
if (stream.text) {
  displayText = stream.text;
  await updateTelegramMessage(displayText);
}
```

这种边界条件只有真正发消息才能发现。agent 能通过编译和类型检查，但它没法模拟真实的 Telegram 对话。

### 该放手时放手

第一次构建失败了，报了 15 个 TypeScript 错误。我扫了一眼，全是 monorepo 依赖顺序和类型不匹配——import 路径错了、接口字段少了、泛型参数对不上。这些东西有明确的错误信息，agent 对着报错一个一个改就行。我没有介入，让它自己跑完了整个 debug 循环，15 个全修好了。

反过来，API Key 那种问题就不能放手——代码能跑但设计不对，报错信息不会告诉你"这里不该由适配层管"。

## 就这么点事

回头看，我做的事情就几样：判断能做、选方向、agent 做错时纠一下、能自己搞定时不管、跑一遍看看。不到一个小时，不会 JS，有原理层面的了解就够了。

为什么不直接用别人做好的 plugin？因为自己走一遍才知道自己懂没懂。API Key 那个问题，如果我不理解"适配层不该管 LLM 细节"，agent 写错了我也看不出来。做一次，对框架的理解就深一层，下次再加别的前端只会更快。

如果你也想做类似的事，思路就这几步：

1. 找到一个框架层面支持多前端的项目（像 Pi-mono 这样有抽象层的）
2. 找一个现成的前端实现当参考（我用的是 Slack bridge）
3. 让 agent 先讲清楚框架的核心抽象是什么，自己判断理解了再动手
4. 让 agent 照着参考写，你盯设计层面的问题，构建错误让它自己修
5. 跑起来实际用一遍，边界问题只有真跑才能发现

---

**最终产出：** 一个跑在 Telegram 上的 Pi coding agent Bot。支持多轮对话、工具调用、流式输出、会话持久化。不会的语言，不到一个小时，从零到能用。
