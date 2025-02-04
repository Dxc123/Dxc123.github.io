---
created: 2025-02-03T15:11
updated: 2025-02-04T15:55
title: 通过 GitHub action 同步 obsidian 文章到 Hexo
date: 2025/02/03 15:11
---
将需要发布的文章放到blog文件夹并同步到git，action会自动把blog目录下面的文章发布到hexo github repo，触发vercel等服务就回自动发布到hexo线上博客了。


### 安装 Hexo
npm install -g hexo-cli

使用[Hexo CLI](https://hexo.io/docs/index.html#Installation)初始化项目：
hexo init obsidian-hexo-blog


# 在 GitHub Pages 上部署 Hexo