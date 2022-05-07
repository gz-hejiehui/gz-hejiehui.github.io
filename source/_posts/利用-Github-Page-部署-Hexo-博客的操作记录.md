---
title: 利用 Github Page 部署 Hexo 博客的操作记录
categories: 操作记录
tags:
  - Hexo
  - Github Page
  - Github Actions
abbrlink: 3931055798
date: 2022-05-07 16:36:09
---

{% note tip Tips %}

这又是一个教你部署 Hexo 的教程，尽管网上已经有大量类似的文章.....

{% endnote %}

## 前提

1. 本地安装 hexo-cli 程序；
2. 创建一个用于部署站点的 GitHub 仓库。

### 安装 hexo-cli 程序

参考[官方文档](https://hexo.io/docs/index.html#Install-Hexo)，先安装 `nodejs` 环境，再安装 `hexo-cli` 程序：

```bash
brew install nodejs
npm install -g hexo-cli
```

>  Windows 用户可以看 nodejs 官网的安装教程。

### 创建 GitHub 仓库

GitHub 上创建一个 Public 仓库，仓库名为 `用户名.github.io`。

## 构建站点

### 初始化

使用 `hexo` 工具创建站点：

```bash
hexo init Blog
cd Blog
npm install
```

修改 `_config.yml`，补充基本信息，例如：

```plaintext
language: zh-CN
timezone: 'Asia/Shanghai'
```

### 自动部署

[官方文档](https://hexo.io/docs/github-pages)已经给了 Github Action 的配置（简中文档还是万年不更新！）。

创建 `.github/workflows/pages.yml` 文件：

```yml
name: Pages

on:
  push:
    branches:
      - main  # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2
        with:
          submodules: 'true'
      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

> GITHUB_TOKEN 这个是默认就存在，不需要自己创建。

由于上述的脚本会把生成的站点静态文件推送到 `gh-pages` 分支，因此我们还需要修改 Action 读写仓库的权限。

在仓库页面点击 Settings -> Actions -> General，将 `Workflow permissions` 权限改为 `Read and write permissions`。

然后 commit 一下，GitHub Actions 就可以自动构建、部署站点了。最后我们还需要将 Settings -> Pages 中将 source 改为 `gh-pages`。
 
## 主题推荐

推荐使用 submodule 的方式安装主题，可以获取最新的主题更新。

1. [Cards](https://github.com/ChrAlpha/hexo-theme-cards)：本博客目前使用的主题

## 插件推荐

1. [hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink)：优化文章链接

## 结尾

写博客还是靠坚持，样式这些差不多就好了！
