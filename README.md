# CL 的 CUDA 学习笔记

使用 Hexo + Icarus，发布到 https://clu1207.github.io/CL.github.io/。

## 第一次发布

1. 将本项目提交并推送到仓库的 `main` 分支。
2. 在 GitHub 仓库 **Settings → Pages → Build and deployment → Source** 中选择 **GitHub Actions**。
3. 在 **Actions** 中查看 `Deploy blog to GitHub Pages`。如果先前部署失败，修改 Pages 设置后重新运行工作流。

不要继续使用 Jekyll 的 Deploy from a branch 模式。

## 本地预览

安装 Node.js 22 后，在项目目录运行：

```bash
npm ci
npm run server
```

访问终端显示的本地地址，加上 `/CL.github.io/` 路径（通常是 http://localhost:4000/CL.github.io/）。

## 写文章

```bash
npm run new -- "cuda-thread-block-grid"
```

编辑生成的 `source/_posts/cuda-thread-block-grid.md`，把 `title` 改为中文标题。正文使用 Markdown，`<!-- more -->` 之前的内容作为首页摘要。代码块使用 `cpp` 标注语言。已开启 MathJax，可使用 `$$ ... $$` 编写独立公式。

也可以在 GitHub 网页直接新增 `source/_posts/文章英文名.md`，复制 `scaffolds/post.md`，把其中的标题和日期占位符替换为实际值。

图片放在 `source/images/`，文章内这样引用：

```markdown
![图片说明](/CL.github.io/images/example.png)
```

写完后提交并推送，GitHub Actions 自动构建发布。

```bash
npm run clean
npm run build
git add .
git commit -m "Add CUDA learning notes"
git push origin main
```

## 修改个人信息

- `_config.yml`：网站名称、简介、作者和部署地址。
- `_config.icarus.yml`：导航、侧栏、头像和主题功能。
- `source/about/index.md`：关于页面。
- `scaffolds/post.md`：新文章模板。

原来的第一篇文章已保留在 `source/_posts/hello-world.md`。
