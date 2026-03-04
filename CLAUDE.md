# Blog — LPASK/blog

Hugo + PaperMod + GitHub Pages 博客。Push to main 自动部署。

- **站点**: https://lpask.github.io/blog/
- **Repo**: https://github.com/LPASK/blog
- **身份**: LazyBeaver / LPASK@users.noreply.github.com
- **双语**: 中文（默认，`/zh/`）、English（`/en/`）

## 博客定位

记录**构建过程中的真实经历**——踩了什么坑、做了什么判断、砍了什么设计、留下了什么。不是教程，不是布道，不是产品宣传。

每篇文章的核心是一个**具体项目或功能的设计复盘**。

目标读者：Agent CLI 重度用户（知道 System Prompt / Hook / context window），兼顾对 AI agent 感兴趣的探索者。

## 写作流程

写作的核心原则：**从读者出发，不是从自己出发。** 写给自己=按时间线复盘我做了什么（日记）；写给别人=从读者的问题出发，用我的经历回答他的问题。同样的素材，组织方式完全不同。

### Phase 1: 定位（写之前必须想清楚）

定位错了，标题和开头都会跟着错，后面改多少轮都是在补救。

回答三个问题：

1. **这个东西解决的是什么层级的问题？** 区分清楚：是解决一个大方向的问题，还是大方向下某个具体的体验痛点？（例：agent-assembly 不是解决"AI 认识你"的问题，那是复利工程的事；agent-assembly 解决的是"使用 Claude Code 跨项目进行复利工程时的痛点"。）
2. **这篇文章的性质是什么？** 项目介绍？设计复盘？使用经验？性质决定叙事结构。
3. **读完之后读者应该做什么？** 去看 repo？试用？思考一个问题？

### Phase 2: 标题

言简意赅，可选要素：

- **项目名** — 开宗明义
- **解决的问题** — 一个短语说清楚痛点
- **做法/结果** — 怎么做的或得到了什么

不要用大词（"让 AI Agent 认识你"——不是你解决的问题层级），不要堆砌过程描述，不要标题党。

### Phase 3: 开头

**从读者能感知的痛点开始**，不是从项目介绍开始，不是从"我遇到了一个问题"的铺垫开始。

读者给你 10 秒。他需要在开头就判断"这跟我有关吗"。用一个他可能亲身经历过的具体场景切入。

### Phase 4: 正文

- 主线选**解法**，把框架/洞察作为论证嵌入——而不是反过来
- 每段问自己：这对读者有用吗？他已经知道这个了吗？他会继续往下读吗？
- 删掉读者能自己推出的总结性段落
- 不同章节节奏要有变化——不要每段都是"问题→尝试→失败→教训"的相同结构
- 中英文是独立写作，不是翻译

### Phase 5: 结尾

- Takeaway 必须是**只有你经历过才能说出来的话**，不是通用工程格言
- CTA 要具体直接（"试试这个库"），不要泛化（"如果你也有类似问题"）

## 迭代流程

写完初稿后不直接发，走一轮迭代：

1. **目标读者视角通读** — 假设自己是目标读者，从头读一遍。在每个读者会卡住或走神的地方标记：这里对他有用吗？他已经读过这个了吗？他会继续往下读吗？
2. **改** — 基于通读发现的问题改
3. **可选：sub-agent 评审** — 用干净上下文的 sub-agent 做独立评审，避免写作过程中的上下文污染。给明确的评分维度和目标读者描述
4. **确认后发布**

关键：迭代的视角是"读者在哪里会走"，不是"评审说什么改什么"。评审能指出问题，但修改的判断标准是读者体验。

## 发布流程

### 1. 在 Store 中完成写作

文章通常在 Store 的对话中讨论和迭代，成稿存在 `ideas/` 里。

### 2. 转为 Hugo 博文

创建 `content/posts/<slug>.zh.md`（和可选的 `.en.md`）：

```yaml
---
title: "文章标题"
date: YYYY-MM-DD
draft: false
---
```

- 去掉 Store 的 YAML frontmatter（summary/tags/ref），换成 Hugo 格式
- Hugo frontmatter 只需要 `title`、`date`、`draft`

### 3. 本地验证

```bash
cd ~/blog && hugo --gc --minify
```

### 4. 提交并推送

```bash
git add content/posts/<slug>.zh.md content/posts/<slug>.en.md
git commit -m "post: <简要描述>"
git push
```

GitHub Actions 自动构建部署，推送后约 30 秒上线。

### 5. 验证

- 中文: `https://lpask.github.io/blog/zh/posts/<slug>/`
- English: `https://lpask.github.io/blog/en/posts/<slug>/`

## 注意事项

- Git 身份必须是 LazyBeaver（repo 已配好 local config）
- `public/` 目录在 `.gitignore` 中，不要提交
- PaperMod 主题是 git submodule，clone 后需要 `git submodule update --init`
- 菜单链接使用相对路径（部署在 `/blog/` 子目录下）
