---
title: "代码块与图片渲染测试"
date: 2026-09-01T09:30:00+08:00
draft: false
description: "第二篇示例文章，包含代码块和图片，用来验证 Hugo 与 PaperMod 主题的渲染效果。"
summary: "这篇文章包含一段 Python 代码和一张流程图，用来验证代码高亮、封面图和文中插图是否都正常工作。"
tags: ["Hugo", "示例"]
categories: ["技术"]
cover:
  image: "cover.svg"
  alt: "发布流程示意图"
  caption: "从写文章到自动上线的整个流程"
  relative: true
ShowToc: true
TocOpen: false
---

这篇文章用来验证两件事：**代码块高亮**和**图片渲染**（包括封面图和正文插图）是否都能在 GitHub Pages 上正常显示。

## 代码块

下面是一段带语法高亮的 Python 代码：

```python
def greet(name: str) -> str:
    """返回一句问候语"""
    return f"你好，{name}！欢迎来到我的博客。"


if __name__ == "__main__":
    print(greet("世界"))
```

再来一段 Bash，演示这个博客的发布方式：

```bash
hugo new content/posts/my-new-post.md
hugo server -D          # 本地预览
git add content/posts/my-new-post.md
git commit -m "新增文章"
git push                # 推送后 GitHub Actions 自动构建部署
```

## 图片

文章的封面图（`cover.svg`，见 frontmatter）之外，正文里也可以插入图片，比如下面这张示意图：

![发布流程示意图](cover.svg)

这张图和本文使用的是 Hugo 的 **page bundle**（页面束）方式组织：`index.md` 和 `cover.svg` 放在同一个目录 `content/posts/hello-code-and-image/` 下，图片用相对路径引用，Hugo 构建时会自动把图片一起处理、复制到输出目录，不需要手动放到 `static/` 目录。

如果只是想快速插一张图、不想建目录，也可以把图片放进项目的 `static/` 目录，然后在正文里用 `/xxx.png` 这种以斜杠开头的绝对路径引用，两种方式都可以，看个人习惯。
