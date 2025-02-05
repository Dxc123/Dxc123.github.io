---
created: 2025-02-03T21:55
updated: 2025-02-05T11:20
title: 使用Hexo部署个人博客
date: 2025/02/03 21:56
tags:
  - Hexo博客部署
---
[Hexo官方文档](https://hexo.io/zh-cn/docs/)


新建一篇文章:使用布局 `draft` 来创建草稿
```
hexo new <title>
```
默认情况下，Hexo 会使用文章的标题来决定文章文件的路径。 对于独立页面来说，Hexo 会创建一个以标题为名字的目录，并在目录中放置一个 `index.md` 文件。


    hexo new page --path about/me
此时 Hexo 会创建 `source/_posts/about/me.md`

```
hexo new page tags
  会创建source/tags/index.md
hexo new page categories
  会创建source/categories/index.md
```

生成静态文件:
```
hexo generate
```

发表草稿
```
hexo publish <filename>
```

启动服务器。 默认情况下，访问网址为： `http://localhost:4000/`。
```
hexo server
```
清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。
```
hexo clean
```
列出所有路由
```
hexo list
```
显示版本信息
```
hexo version
```
列出网站的配置（`_config.yml`）
```
hexo config
```