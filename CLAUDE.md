# Blog — LPASK/blog

Hugo + PaperMod + GitHub Pages 博客。Push to main 自动部署。

- **站点**: https://lpask.github.io/blog/
- **Repo**: https://github.com/LPASK/blog
- **身份**: LazyBeaver / LPASK@users.noreply.github.com
- **双语**: 中文（默认，`/zh/`）、English（`/en/`）

## 博客定位

这个博客记录的是**构建过程中的真实经历**——踩了什么坑、做了什么判断、砍了什么设计、留下了什么。不是教程，不是布道，不是产品宣传。

每篇文章的核心是一个**具体项目或功能的设计复盘**，而不是一个抽象话题的讨论。

目标读者：Agent CLI 重度用户（知道 System Prompt / Hook / context window 是什么），兼顾对 AI agent 感兴趣的探索者。

## 写作流程

### Phase 1: 定位（写之前必须想清楚）

**这一步最重要。** 定位错了，标题和开头都会跟着错，后面改多少轮都是在补救。

回答三个问题：

1. **这个东西解决的是什么层级的问题？** 区分清楚：是解决一个大方向的问题，还是解决大方向下某个具体的体验痛点？定位层级决定了整篇文章的说法。（例：agent-assembly 不是解决"AI 认识你"的问题，那是复利工程的事；agent-assembly 解决的是"使用 Claude Code 跨项目进行复利工程时的痛点"。）
2. **这篇文章的性质是什么？** 项目介绍？设计复盘？使用经验？技术教程？性质决定叙事结构。
3. **读完之后读者应该做什么？** 去看 repo？试用？思考一个问题？这决定了 CTA 怎么写。

### Phase 2: 标题

标题要言简意赅，包含以下要素（按场景取舍，不是每个都必须）：

- **项目名** — 开宗明义，告诉读者这篇在讲什么
- **文章性质** — 复盘、介绍、经验
- **解决的问题** — 用一个短语说清楚痛点
- **做法/结果** — 怎么做的或得到了什么

示例：`agent-assembly：复利工程的痛点，和反复迭代后的答案`（项目名 + 问题 + 结果）

**不要做的事**：
- 不要用大词（"让 AI Agent 认识你"——这不是你解决的问题层级）
- 不要堆砌过程描述（"三周弯路踩坑"——不言简意赅）
- 不要做标题党

### Phase 3: 开头

**开宗明义。** 对于项目相关的文章，第一段直接亮出：

1. 项目链接 + 一句话说它是什么
2. 它解决什么问题（精确的定位语，Phase 1 确定的）
3. 这篇文章讲什么（复盘过程？使用经验？）

不要从"我遇到了一个问题"开始铺垫。读者想先知道"这是什么"，不是"你为什么做这个"。

### Phase 4: 正文

- 叙事结构跟随文章性质（Phase 1 确定的）
- 设计复盘：按时间线讲迭代过程，每个阶段交代动机→做法→结果→为什么放弃/保留
- 项目介绍：先结论后过程
- 使用经验：先结果后细节
- 中英文是独立写作，不是翻译

### Phase 5: 结尾

- 总结核心 takeaways（如有）
- CTA：基于 Phase 1 确定的"读者应该做什么"来写
- 不要做的事：不要用"如果你也有类似问题"这种泛化表达，要具体（"如果你在用 Claude Code 跨项目工作，被 profile 丢失困扰"）

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

构建成功且无 error 即可。

### 4. 提交并推送

```bash
git add content/posts/<slug>.zh.md content/posts/<slug>.en.md
git commit -m "post: <简要描述>"
git push
```

GitHub Actions 自动构建部署，推送后约 30 秒上线。

### 5. 验证

确认文章在站点上可访问：
- 中文: `https://lpask.github.io/blog/zh/posts/<slug>/`
- English: `https://lpask.github.io/blog/en/posts/<slug>/`

## 注意事项

- Git 身份必须是 LazyBeaver（repo 已配好 local config）
- `public/` 目录在 `.gitignore` 中，不要提交
- PaperMod 主题是 git submodule，clone 后需要 `git submodule update --init`
- 菜单链接使用相对路径（因为部署在 `/blog/` 子目录下）
