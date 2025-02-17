---
created: 2025-02-04T16:16
updated: 2025-02-15T19:23
title: 关于Git常用操作
tags:
  - Git
categories:
  - Git
date: 2025/02/04 16:16
---

[菜鸟git教程](https://www.runoob.com/git/git-tutorial.html)


![Git操作命令.png](https://raw.githubusercontent.com/Dxc123/picture_store/main/images/20250215114426592.png?token=AF7QJWGVOYTAIDZL46TFZJ3HWAG5U)

##### 日志提交规范：
- feat: 加入新特性
- fix: 修复bug
- improvement: 在现有特性上的改进
- docs: 更改文档
- style: 修改了代码的格式
- refactor: 代码重构,不包含 bug 的修复以及新增特性
- perf: 提升性能的改动
- test: 测试用例的改动
- build: 改变了构建系统或者增加了新的依赖
- ci: 修改了自动化流程的配置文件或者脚本
- chore: 除了源码目录以及测试用例的其他修改
- revert: 回退到之前的一个 commit

##### 使用GitHub Access Token 克隆一个 Git 仓库:
```
git clone https://your_token_here@github.com/username/repository.git
```

##### 查看全局 Git 用户信息（适用于所有仓库）:
```
git config --global user.name 
git config --global user.email
```

##### 查看当前仓库的 Git 用户信息:
```
git config user.name 
git config user.email
```


```
dxc_123
daixingchuang@163.com
```

##### 查看所有的 Git 配置信息（包括全局和本地设置）:
```
git config --list
```

##### 配置`git`账号:
```
git config --global user.name "张三"
git config --global user.email "zhangsan@marsview.cc"

```

##### 本地已经存在一个项目，想要推送到远程已有的仓库:
```
git init
git remote add origin your_git_repository_url
git add .
git commit -m "Initial commit"
git push -u origin master
```


##### 第一次克隆`github`项目需要在`github`配置公钥
**1.本地生产密钥**
```
ssh-keygen -t rsa -C "youremail@email.com"
```
**2.复制本地公钥**

找到 mac根目录`id_rsa.pub`文件下的内容

**3.打开Github**
点击头像=>setting=> SSH and GPG keys=> SSH keys目录下
 点击右上角点击新建【New SSH Key】，把公钥复制进去即可。

以后克隆项目和推送提交，就不需要账号密码了。

#####  代码写一半，想要临时保存工作进,去干某事比如切换分支？
使用`git`暂存：`git stash`
```
# 缓存本地开发文件：
git stash

# 查看所有缓存列表：
git stash list

# 取出最后一条缓存记录并删除：
git stash pop

#应用最近一次存储的进度：
git stash apply

# 应用某一条缓存：
git stash apply stash@{index}

#删除特定存储：
git stash drop stash@{n}

#清空所有存储：
git stash clear
```
`git stash`就是临时保存，等你忙完手里事情以后，在回来，把临时的代码还原，这样可以非常完美的看到刚刚改的代码记录。


##### 想撤回提交的 commit,扎办？
1.使用**软回滚**：

`git reset --soft`：回退到某个版本 。将撤回的代码，存放到暂存区。同时会保留本地未提交的内容。
```
#查看提交的commit历史（按wq，退出编辑模式）
git log 
git reset --soft <commit version>
或者 git revert <commit version> (会多提交一次记录上去)
```

``` 
 git  reset fbf377c048eb9ad29fe6f79e66  # 回退到指定版本
```

撤回以后，代码会暂存在本地，你做二次编辑以后，再次commit提交上去，但是此时提交的时候，必须使用`git push --force`强力推上去，因为本地跟远程已经有冲突了


2.使用**硬回滚**:
`git reset --hard`：彻底回退到某个版本，丢弃将撤回的代码，本地没有commit的修改会被全部擦掉。(`慎用`)

````
git log 
git reset --hard <commit version>
```
[菜鸟git教程](https://www.runoob.com/git/git-tutorial.html)


![Git操作命令.png](https://raw.githubusercontent.com/Dxc123/picture_store/main/images/20250215114426592.png?token=AF7QJWGVOYTAIDZL46TFZJ3HWAG5U)

##### 日志提交规范：
- feat: 加入新特性
- fix: 修复bug
- improvement: 在现有特性上的改进
- docs: 更改文档
- style: 修改了代码的格式
- refactor: 代码重构,不包含 bug 的修复以及新增特性
- perf: 提升性能的改动
- test: 测试用例的改动
- build: 改变了构建系统或者增加了新的依赖
- ci: 修改了自动化流程的配置文件或者脚本
- chore: 除了源码目录以及测试用例的其他修改
- revert: 回退到之前的一个 commit

##### 使用GitHub Access Token 克隆一个 Git 仓库:
```
git clone https://your_token_here@github.com/username/repository.git
```

##### 查看全局 Git 用户信息（适用于所有仓库）:
```
git config --global user.name 
git config --global user.email
```

##### 查看当前仓库的 Git 用户信息:
```
git config user.name 
git config user.email
```


```
dxc_123
daixingchuang@163.com
```

##### 查看所有的 Git 配置信息（包括全局和本地设置）:
```
git config --list
```

##### 配置`git`账号:
```
git config --global user.name "张三"
git config --global user.email "zhangsan@marsview.cc"

```

##### 本地已经存在一个项目，想要推送到远程已有的仓库:
```
git init
git remote add origin your_git_repository_url
git add .
git commit -m "Initial commit"
git push -u origin master
```


##### 第一次克隆`github`项目需要在`github`配置公钥
**1.本地生产密钥**
```
ssh-keygen -t rsa -C "youremail@email.com"
```
**2.复制本地公钥**

找到 mac根目录`id_rsa.pub`文件下的内容

**3.打开Github**
点击头像=>setting=> SSH and GPG keys=> SSH keys目录下
 点击右上角点击新建【New SSH Key】，把公钥复制进去即可。

以后克隆项目和推送提交，就不需要账号密码了。

#####  代码写一半，想要临时保存工作进,去干某事比如切换分支？
使用`git`暂存：`git stash`
```
# 缓存本地开发文件：
git stash

# 查看所有缓存列表：
git stash list

# 取出最后一条缓存记录并删除：
git stash pop

#应用最近一次存储的进度：
git stash apply

# 应用某一条缓存：
git stash apply stash@{index}

#删除特定存储：
git stash drop stash@{n}

#清空所有存储：
git stash clear
```
`git stash`就是临时保存，等你忙完手里事情以后，在回来，把临时的代码还原，这样可以非常完美的看到刚刚改的代码记录。


##### 想撤回提交的 commit,扎办？
1.使用**软回滚**：

`git reset --soft`：回退到某个版本 。将撤回的代码，存放到暂存区。同时会保留本地未提交的内容。
```
#查看提交的commit历史（按wq，退出编辑模式）
git log 
git reset --soft <commit version>
或者 git revert <commit version> (会多提交一次记录上去)
```

``` 
 git  reset fbf377c048eb9ad29fe6f79e66  # 回退到指定版本
```

撤回以后，代码会暂存在本地，你做二次编辑以后，再次commit提交上去，但是此时提交的时候，必须使用`git push --force`强力推上去，因为本地跟远程已经有冲突了


2.使用**硬回滚**:
`git reset --hard`：彻底回退到某个版本，丢弃将撤回的代码，本地没有commit的修改会被全部擦掉。(`慎用`)

````
git log 
git reset --hard <commit version>
```
