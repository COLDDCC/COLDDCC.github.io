# COLDDCC.github.io

个人技术博客。静态生成器用的是 [Hugo Extended](https://gohugo.io/)，主题是
[PaperMod](https://github.com/adityatelange/hugo-PaperMod)，通过 GitHub Actions
自动构建并部署到 GitHub Pages。

线上地址：`https://COLDDCC.github.io`

> 这份 README 是写给"以后忘光了怎么用"的自己看的操作手册，照着抄命令就行。

---

## 0. 上线前必须确认一次的事

GitHub 仓库需要手动打开一个开关，Actions 构建成功才会真正上线：

1. 打开仓库页面 → **Settings**
2. 左侧菜单 → **Pages**
3. **Build and deployment → Source** 选择 **GitHub Actions**（不是 "Deploy from a branch"）

这一步不做，即使 workflow 跑得全绿，也不会有网页能打开。只需要设置一次。

---

## 1. 怎么新建一篇文章

在仓库根目录执行：

```bash
hugo new content/posts/我的新文章标题.md
```

会根据 `archetypes/default.md` 模板生成一篇新文章，文件里已经带好了
`title` / `date` / `tags` 等 frontmatter 字段，照着填就行，例如：

```yaml
---
title: "我的新文章标题"
date: 2026-09-02T20:00:00+08:00
draft: false
description: "一句话简介，会出现在列表页和搜索引擎摘要里"
summary: ""
tags: ["Hugo", "笔记"]
categories: ["技术"]
ShowToc: true
TocOpen: false
---
```

**注意：`draft: false` 才会被构建出来**，新建文章默认就是 `false`，如果手动
改成了 `true`，push 上去也不会显示，忘记改回来是最容易踩的坑之一。

**日期不能是未来时间。** Hugo 对日期晚于"现在"的文章会静默跳过，不报错也不
提示，页面上就是"文章没了"，很难排查。新建文章时archetype会自动填当前时间，
不要手动往后改。

想插图片，推荐用"页面束"（page bundle）的方式，把文章单独放一个目录：

```bash
mkdir -p content/posts/我的新文章标题
# 把 index.md（内容）和图片文件放进这个目录
```

正文里用相对路径引用同目录下的图片即可，例如 `![说明文字](my-image.png)`。

---

## 2. 怎么本地预览

第一次用需要先装 Hugo Extended（版本号要跟 `.github/workflows/deploy.yml`
里 `HUGO_VERSION` 那一行保持一致，目前是 `0.165.0`），去
[Hugo Releases](https://github.com/gohugoio/hugo/releases) 下载对应系统的
`hugo_extended_<版本号>_<系统>.tar.gz` / `.deb` / `.zip`，或者用包管理器装：

```bash
# macOS
brew install hugo

# 装完确认一下是不是 extended 版本
hugo version   # 输出里要带 "+extended" 字样
```

启动本地预览（包含草稿）：

```bash
hugo server -D
```

然后打开终端里提示的地址（通常是 `http://localhost:1313/`），改文件保存后
浏览器会自动刷新。

只想看不带草稿的"发布效果"，去掉 `-D`：

```bash
hugo server
```

---

## 3. 怎么发布

不需要手动构建，也不需要手动上传任何东西。流程就是：

```bash
git add content/posts/我的新文章标题.md
git commit -m "新增文章：我的新文章标题"
git push
```

push 到 `main` 分支之后，GitHub Actions 会自动：

1. 用写死版本号的 Hugo Extended 构建站点
2. 把构建产物部署到 GitHub Pages

大约 30 秒到 1 分钟左右，`https://COLDDCC.github.io` 上就能看到新文章。可以在
仓库的 **Actions** 标签页看构建进度，绿色对勾就是成功。

---

## 4. 构建失败了先看哪里

按下面顺序排查：

1. **先看 Actions 日志**：仓库 → **Actions** → 点进失败的那次运行 → 展开
   报红的步骤，日志里一般会直接说是哪个文件、第几行出的问题。
2. **本地复现**：在本地跑一遍 `hugo --minify`，本地能复现的问题改起来更快，
   不用每次都等 Actions。
3. **常见原因排查清单**：
   - 新文章的 `date` 是不是未来时间（会被静默跳过，不算报错，但如果是别的
     报错，先排除这个可能性）
   - frontmatter 的 YAML 缩进 / 引号是不是写错了（比如中文标题忘了加引号、
     冒号后面忘了加空格）
   - `tags` / `categories` 是不是漏了方括号，写成了 `tags: 技术` 而不是
     `tags: ["技术"]`
   - 引用了本地不存在的图片文件名（page bundle 里图片文件名和正文里写的
     对不上，大小写也算不一样）
   - 是不是手滑改动了 `hugo.toml` 或 `themes/PaperMod/` 里的内容
4. 如果是 Hugo 本身报错（不是我们内容的问题），把 Actions 里的**完整日志**
   贴出来看，不要先自己瞎改，尤其不要为了让构建通过就把 Hugo 版本降级到很
   老的版本——优先看看是不是主题版本要跟着升级。

---

## 5. 以后想换主题，需要动哪几个文件

1. 把新主题的源码整个复制进 `themes/<新主题名>/`（**不要用 git
   submodule**，直接复制文件、正常 git add/commit，Actions 里最常见的构建
   失败原因就是 submodule 没拉下来）
2. 改 `hugo.toml` 里的 `theme = "PaperMod"` 为新主题名
3. `hugo.toml` 里 `[params]`、`[menu]` 等配置项一般每个主题字段都不一样，
   照抄新主题官方文档/exampleSite 里的示例配置改一遍（社交链接、首页文案
   这些占位内容记得重新填）
4. 本地跑 `hugo server -D` 确认没问题，再 `git push`

不需要动 `.github/workflows/deploy.yml`（除非新主题要求更高的 Hugo 版本，
那就把里面的 `HUGO_VERSION` 也升级一下）。

---

## 5.5 只是想调配色/字体，不换主题

现在这套无障碍优先的极简视觉（黑白配色、系统字体、链接始终带下划线、
键盘焦点粗描边、跳到正文的 skip link）不是改主题本身，而是叠在 PaperMod
默认样式上面的一层覆盖，在这几个文件里：

- `assets/css/extended/custom.css` —— 配色变量、圆角、链接/焦点样式，改这
  个文件里最上面的 `:root` 变量最省事（`:root[data-theme="dark"]` 那段是
  深色模式单独的链接颜色，别漏改）
- `layouts/_partials/extend_head.html` —— 如果想加自定义网络字体（比如
  Google Fonts），在这里加 `<link>`；现在是空的，用的是系统自带字体
- `layouts/baseof.html` —— 页面整体骨架，目前只多了一行 skip link。这个
  文件是从主题复制出来改的，**不是**新建的空文件，以后主题升级要留意这里
  会不会有值得同步的新变化（概率很低，这个文件基本不怎么变）

这几个文件都在仓库根目录，不在 `themes/PaperMod/` 里面，所以主题以后升级
（重新复制一遍新版本的主题文件）也不会把这层样式覆盖掉，唯独 `baseof.html`
因为是复制出来改的，主题升级后如果它有更新，需要手动比对一下要不要同步。
改完照常 `hugo server -D` 本地看一眼、`git push` 就行。

---

## 目录结构速查

```
.
├── .github/workflows/deploy.yml   # 构建 + 部署，一般不用动
├── hugo.toml                      # 站点配置：站名/简介/导航/社交链接
├── archetypes/default.md          # `hugo new` 用的新文章模板
├── content/posts/                 # 所有文章放这里
├── content/archives.md            # 归档页
├── content/search.md              # 搜索页
├── static/                        # 直接原样发布的静态文件（favicon 等）
├── assets/css/extended/custom.css # 样式覆盖层，改配色/圆角/链接样式在这
├── layouts/_partials/extend_head.html  # 自定义网络字体入口，现在是空的
├── layouts/baseof.html            # 页面骨架，从主题复制出来改的（加了 skip link）
└── themes/PaperMod/                # 主题文件，直接提交进仓库，不是 submodule
```

## 待填清单（我自己后面填）

- [ ] `hugo.toml` 里的 `title`、`params.description`、`params.author`、
      `params.keywords`
  等标了 `# TODO` 的字段
- [ ] `hugo.toml` 里 `[[params.socialIcons]]` 换成自己真实的社交链接
- [ ] `static/favicon.ico` 目前是占位纯色图标，换成自己喜欢的图标
