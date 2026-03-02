# Blog — LPASK/blog

Hugo + PaperMod + GitHub Pages 博客。Push to main 自动部署。

- **站点**: https://lpask.github.io/blog/
- **Repo**: https://github.com/LPASK/blog
- **身份**: LazyBeaver / LPASK@users.noreply.github.com
- **双语**: 中文（默认，`/zh/`）、English（`/en/`）

## 发布新文章

### 1. 写文章

在 Store 中完成文章撰写（通常在 `ideas/` 中讨论和迭代）。

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
- 中英文是独立文件，不是翻译关系

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

GitHub Actions 自动构建部署。推送后约 30 秒上线。

### 5. 验证

确认文章在站点上可访问：
- 中文: `https://lpask.github.io/blog/zh/posts/<slug>/`
- English: `https://lpask.github.io/blog/en/posts/<slug>/`

## 注意事项

- Git 身份必须是 LazyBeaver（repo 已配好 local config）
- `public/` 目录在 `.gitignore` 中，不要提交
- PaperMod 主题是 git submodule，clone 后需要 `git submodule update --init`
- 菜单链接使用相对路径（因为部署在 `/blog/` 子目录下）
