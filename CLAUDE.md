# Blog — LPASK/blog

Hugo + PaperMod + GitHub Pages 博客。Push to main 自动部署。

- **站点**: https://lpask.github.io/blog/
- **Repo**: https://github.com/LPASK/blog
- **身份**: LazyBeaver / LPASK@users.noreply.github.com
- **双语**: 中文（默认，`/zh/`）、English（`/en/`）

## 发布

```bash
hugo --gc --minify                    # 本地验证
git add content/posts/<slug>.zh.md    # 加文件
git commit -m "post: <描述>"          # commit
git push                              # 自动部署，~30s 上线
```

## Hugo 博文格式

```yaml
---
title: "文章标题"
date: YYYY-MM-DD
draft: false
---
```

## 注意事项

- Git 身份必须是 LazyBeaver（repo 已配好 local config）
- `public/` 在 `.gitignore` 中，不要提交
- PaperMod 是 git submodule，clone 后 `git submodule update --init`
- 菜单链接用相对路径（部署在 `/blog/` 子目录下）
- 写作流程和方法论见 hub skill `blog-publish`
