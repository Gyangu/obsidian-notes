---
title: Hexo博客与Obsidian笔记自动部署完整指南
date: 2025-10-24 10:28:48
tags: [Hexo, Obsidian, GitHub Actions, 自动化部署, 博客]
categories: [技术教程]
---

# Hexo博客与Obsidian笔记自动部署完整指南

## 📖 概述

本指南将详细介绍如何搭建一个基于 Hexo 的静态博客，并将其与 Obsidian 笔记软件集成，实现**只需在 Obsidian 中编辑笔记，自动部署到博客**的完整自动化流程。

## 🎯 目标

- ✅ 使用 Hexo 搭建静态博客
- ✅ 集成 Obsidian 笔记作为内容源
- ✅ 实现自动化部署流程
- ✅ 只需管理一个 Git 仓库（obsidian-notes）

## 🏗️ 架构设计

### 整体架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Obsidian      │    │ obsidian-notes  │    │   blog 仓库     │
│   笔记编辑      │───▶│    Git 仓库     │───▶│   (子模块)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                                                       ▼
                                               ┌─────────────────┐
                                               │ GitHub Actions  │
                                               │   自动部署      │
                                               └─────────────────┘
                                                       │
                                                       ▼
                                               ┌─────────────────┐
                                               │ GitHub Pages    │
                                               │   静态网站      │
                                               └─────────────────┘
```

### 核心组件

1. **obsidian-notes 仓库**：存储所有笔记内容
2. **blog 仓库**：Hexo 博客项目，包含 obsidian-notes 作为子模块
3. **GitHub Actions**：自动化部署工作流
4. **GitHub Pages**：静态网站托管

## 🚀 详细实施步骤

### 第一步：创建基础 Hexo 博客

#### 1.1 初始化项目

```bash
# 创建博客目录
mkdir blog && cd blog

# 初始化 Hexo
npx hexo init .

# 安装依赖
npm install
```

#### 1.2 配置 Hexo

编辑 `_config.yml`：

```yaml
# Site
title: Gyangu's Blog
subtitle: 'A Hexo Blog'
description: 'Personal blog powered by Hexo'
keywords: blog, hexo, static site
author: Gyangu
language: zh-CN
timezone: 'Asia/Shanghai'

# URL
url: https://gyangu.github.io/blog

# Deployment
deploy:
  type: git
  repo: https://github.com/Gyangu/blog.git
  branch: gh-pages
  message: "Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"
```

#### 1.3 安装部署插件

```bash
npm install hexo-deployer-git --save
```

#### 1.4 配置 Yarn 命令

编辑 `package.json`：

```json
{
  "scripts": {
    "build": "hexo generate",
    "clean": "hexo clean",
    "deploy": "hexo deploy",
    "server": "hexo server",
    "dev": "hexo server",
    "start": "hexo server",
    "publish": "hexo clean && hexo generate && hexo deploy",
    "new:post": "hexo new post",
    "new:page": "hexo new page",
    "new:draft": "hexo new draft"
  }
}
```

### 第二步：创建 Obsidian 笔记仓库

#### 2.1 创建远程仓库

```bash
# 使用 GitHub CLI 创建仓库
gh repo create obsidian-notes --public --description "My Obsidian notes repository"
```

#### 2.2 初始化本地仓库

```bash
# 创建临时目录
mkdir -p /tmp/obsidian-notes && cd /tmp/obsidian-notes

# 初始化 Git
git init

# 创建 README
echo "# Obsidian Notes

This repository contains my Obsidian notes.

## Notes

- This is a submodule for the Hexo blog
- Markdown files in this repo will be used as blog posts" > README.md

# 提交并推送
git add README.md
git commit -m "Initial commit: Obsidian notes repository"
git remote add origin https://github.com/Gyangu/obsidian-notes.git
git branch -M main
git push -u origin main
```

### 第三步：配置子模块集成

#### 3.1 备份现有内容

```bash
cd /path/to/blog
mv source source_backup
```

#### 3.2 克隆 obsidian-notes 作为 source

```bash
git clone https://github.com/Gyangu/obsidian-notes.git source
```

#### 3.3 合并备份内容

```bash
cp -r source_backup/* source/
cd source
git add .
git commit -m "Initialize with blog content"
git push origin main
```

#### 3.4 添加为子模块

```bash
cd /path/to/blog
rm -rf source/.git
git add source
git commit -m "Use obsidian-notes as source directory"

# 移除并重新添加为子模块
git rm -rf source
git submodule add https://github.com/Gyangu/obsidian-notes.git source
git commit -m "Add obsidian-notes as submodule"
git push origin master
```

### 第四步：配置 GitHub Actions 自动部署

#### 4.1 博客仓库部署工作流

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy Hexo Blog

on:
  push:
    branches:
      - master
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Blog Repo
        uses: actions/checkout@v4
        with:
          submodules: true  # 自动拉取子模块

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Dependencies
        run: npm ci

      - name: Generate Site
        run: |
          npx hexo clean
          npx hexo g

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

#### 4.2 Obsidian 仓库更新工作流

在 `source/.github/workflows/update-blog.yml`：

```yaml
name: Update Blog

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: write

jobs:
  update-blog:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Blog Repo
        uses: actions/checkout@v4
        with:
          repository: Gyangu/blog
          token: ${{ secrets.BLOG_TOKEN }}
          path: blog
          
      - name: Initialize Submodule
        run: |
          cd blog
          git submodule update --init --recursive
        
      - name: Update Submodule
        run: |
          cd blog
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git submodule update --remote source
          git add source
          git diff --staged --quiet || git commit -m "Update obsidian notes"
          git push origin master
```

### 第五步：配置权限和密钥

#### 5.1 创建 Personal Access Token

1. 访问 https://github.com/settings/tokens
2. 点击 "Generate new token" -> "Generate new token (classic)"
3. 设置权限：
   - `repo`（完整仓库权限）
4. 生成并复制 token

#### 5.2 在 obsidian-notes 仓库添加 Secret

1. 访问 https://github.com/Gyangu/obsidian-notes/settings/secrets/actions
2. 点击 "New repository secret"
3. Name: `BLOG_TOKEN`
4. Value: 粘贴刚复制的 token
5. 点击 "Add secret"

#### 5.3 启用 GitHub Pages

1. 访问 https://github.com/Gyangu/blog/settings/pages
2. 在 "Source" 选择 "GitHub Actions"
3. 保存设置

## 🔧 故障排除

### 常见问题及解决方案

#### 问题1：GitHub Actions 权限不足

**错误信息：**
```
Permission to Gyangu/blog.git denied to github-actions[bot]
```

**解决方案：**
- 确保已创建 Personal Access Token
- 在 obsidian-notes 仓库正确配置 `BLOG_TOKEN` Secret

#### 问题2：子模块未初始化

**错误信息：**
```
Submodule path 'source' not initialized
```

**解决方案：**
- 在工作流中添加 `git submodule update --init --recursive`

#### 问题3：Node.js 版本不兼容

**错误信息：**
```
Unsupported engine: required: { node: '>=20.19.0' }
```

**解决方案：**
- 在 GitHub Actions 中使用 Node.js 20

#### 问题4：GitHub Pages 部署失败

**错误信息：**
```
Branch "master" is not allowed to deploy to github-pages
```

**解决方案：**
- 在仓库设置中启用 GitHub Actions 作为 Pages 源

## 📝 使用方法

### 日常写作流程

#### 方法1：在 Obsidian 中编辑（推荐）

1. **在 Obsidian 中打开 `source` 目录**
2. **编辑笔记内容**
3. **提交并推送：**
   ```bash
   cd /path/to/blog/source
   git add .
   git commit -m "Update notes"
   git push origin main
   ```
4. **自动完成：**
   - ✅ GitHub Actions 自动更新 blog 子模块
   - ✅ 触发 blog 部署
   - ✅ 更新网站内容

#### 方法2：使用 Hexo 命令

```bash
# 创建新文章
yarn new:post "文章标题"

# 发布
yarn publish
```

### 内容类型说明

#### 文章 (Post)
- **位置**：`source/_posts/文章标题.md`
- **特点**：包含日期，支持标签，显示在首页
- **URL**：`/年/月/日/文章标题/`

#### 页面 (Page)
- **位置**：`source/页面标题/index.md`
- **特点**：独立页面，不显示在首页
- **URL**：`/页面标题/`

#### 草稿 (Draft)
- **位置**：`source/_drafts/草稿标题.md`
- **特点**：无日期，不会在网站显示
- **用途**：未完成的文章

## 🎉 最终效果

### 自动化流程

```
Obsidian 编辑 → Git 推送 → GitHub Actions → 自动部署 → 网站更新
     ↓              ↓            ↓            ↓          ↓
   写笔记        提交代码      更新子模块    构建部署    访问网站
```

### 优势

1. **简化工作流**：只需管理一个 Git 仓库
2. **自动化部署**：无需手动操作
3. **版本控制**：所有内容都有 Git 历史
4. **跨平台同步**：Obsidian 支持多设备同步
5. **快速发布**：推送即发布

## 📚 相关资源

### 官方文档
- [Hexo 官方文档](https://hexo.io/zh-cn/docs/)
- [GitHub Actions 文档](https://docs.github.com/cn/actions)
- [GitHub Pages 文档](https://docs.github.com/cn/pages)
- [Obsidian 官方文档](https://help.obsidian.md/)

### 相关链接
- **博客网站**：https://gyangu.github.io/blog
- **博客仓库**：https://github.com/Gyangu/blog
- **笔记仓库**：https://github.com/Gyangu/obsidian-notes

## 🔮 扩展功能

### 可能的改进

1. **多主题支持**：集成多个 Hexo 主题
2. **评论系统**：添加 Gitalk 或 Utterances
3. **搜索功能**：集成 Algolia 搜索
4. **SEO 优化**：添加 sitemap 和 meta 标签
5. **RSS 订阅**：自动生成 RSS 源
6. **多语言支持**：国际化配置

### 高级配置

#### 自定义域名
```yaml
# _config.yml
url: https://yourdomain.com
```

#### CDN 加速
```yaml
# _config.yml
cdn:
  js: https://cdn.jsdelivr.net/npm/
  css: https://cdn.jsdelivr.net/npm/
```

## 📊 性能优化

### 构建优化
- 使用 `npm ci` 而不是 `npm install`
- 缓存 node_modules
- 并行构建任务

### 部署优化
- 使用 GitHub Pages 官方 Actions
- 启用 Gzip 压缩
- 优化图片资源

## 🎯 总结

通过本指南，我们成功搭建了一个完整的自动化博客系统：

1. ✅ **Hexo 静态博客**：功能完整，主题美观
2. ✅ **Obsidian 集成**：笔记即博客内容
3. ✅ **自动化部署**：推送即发布
4. ✅ **GitHub Pages**：免费托管
5. ✅ **版本控制**：完整的历史记录

现在你可以专注于内容创作，让技术细节自动处理！

---

*本文档详细记录了从零开始搭建 Hexo + Obsidian 自动化博客的完整过程，希望对有类似需求的读者有所帮助。*
