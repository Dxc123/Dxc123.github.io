---
created: 2025-02-03T15:11
updated: 2025-02-17T11:35
title: 通过 GitHub action 同步 obsidian 文章到 Hexo
date: 2025/02/03 15:11
tags:
  - Hexo博客
categories:
  - Hexo博客
---
目的：通过 GitHub action 同步 obsidian 文章到 Hexo博客

1. 需要两个GitHub 仓库：obsidian_library和 Dxc123.github.io （注：Hexo博客托管到 GitHub，只能叫 yourname.github.io ）
2.将需要发布的文章放到blog文件夹并push到obsidian_library仓库，
3.obsidian_library仓库里的 action会自动把blog目录下面的文章发布到你的hexo 博Dxc123.github.io仓库 
4.Dxc123.github.io仓库里的  action会自动发布到hexo线上博客

obsidian_library仓库里的 action：
.github/workflows/push-to-hexo.yml文件
```
name: Sync Markdown to Another Repo

on:
  push:
    paths:
      - "blog/**"

jobs:
  sync_markdown_files:
    runs-on: ubuntu-latest
    steps:
      - name: Check out source repository
        uses: actions/checkout@v4

      - name: Push changes to the target repository
        run: |
          # 设置git用户信息
          git config --global user.email "daixingchuang@163.com"
          git config --global user.name "dxc_123"

          # 克隆目标仓库
          git clone https://${{ secrets.AUTH_TOKEN }}@github.com/Dxc123/Dxc123.github.io.git

          # 创建一个临时目录并复制博客目录中的更改
          mkdir temp_sync
          cp -r blog/*.md temp_sync/

          # 移动到目标仓库directory
          cd Dxc123.github.io

          # 检出 main 分支
          git checkout main

          # 删除 hexo 已有的 Markdown 文件
          rm -rf source/_posts/*.md
          # 复制 Markdown 文件到 hexo 目录
          cp -r ../temp_sync/*.md source/_posts/
          rm -rf ../temp_sync
          
          # Commit files
          git add .
          git commit -m "Sync Markdown files from source repository"

          # 使用 Access Token 推送更改
          git push https://${{ secrets.AUTH_TOKEN }}@github.com/Dxc123/Dxc123.github.io.git main
```


Dxc123.github.io仓库里的  action 如下：
[在 GitHub Pages 上部署 Hexo](https://hexo.io/zh-cn/docs/github-pages.html)