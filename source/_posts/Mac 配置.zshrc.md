---
created: 2026-05-28T00:34
updated: 2026-05-31T22:36
date: 2026/05/31
tags:
  - Mac配置
categories:
  - Mac配置
title: Mac 配置.zshrc
---

## Mac 中 `.zshrc` 配置 Ruby 路径以及 Homebrew 路径

### 打开 `.zshrc`

```bash
open ~/.zshrc
```

或者：

```bash
nano ~/.zshrc
```

---

## 一、配置 Homebrew 路径

### Apple Silicon（M1/M2/M3）

Homebrew 默认路径：

```bash
/opt/homebrew
```

在 `.zshrc` 添加：

```bash
# Homebrew
export HOMEBREW_HOME=/opt/homebrew
export PATH=$HOMEBREW_HOME/bin:$HOMEBREW_HOME/sbin:$PATH
```

---

### Intel Mac

Homebrew 默认路径：

```bash
/usr/local
```

配置：

```bash
# Homebrew
export HOMEBREW_HOME=/usr/local
export PATH=$HOMEBREW_HOME/bin:$HOMEBREW_HOME/sbin:$PATH
```

---

## 二、配置 Ruby 路径

先查看 Ruby 安装位置：

```bash
which ruby
```

例如：

```bash
/opt/homebrew/opt/ruby/bin/ruby
```

则 `.zshrc` 配置：

```bash
# Ruby
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"
```

---

## 三、推荐完整配置（M系列 Mac）

适合 Flutter / CocoaPods 开发：

```bash
# Homebrew
export HOMEBREW_HOME=/opt/homebrew
export PATH=$HOMEBREW_HOME/bin:$HOMEBREW_HOME/sbin:$PATH

# Ruby
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"

# CocoaPods
export PATH="$HOME/.gem/bin:$PATH"

# Flutter
export PATH="$PATH:$HOME/flutter/bin"

# Android
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
```

---

## 四、配置生效

保存后执行：

```bash
source ~/.zshrc
```

---

## 五、验证是否成功

### 查看 brew

```bash
brew -v
```

### 查看 ruby

```bash
ruby -v
which ruby
```

### 查看 pod

```bash
pod --version
```

---

## 六、推荐安装新版 Ruby（避免系统 Ruby 问题）

Mac 自带 Ruby 很老，容易导致：

- CocoaPods 安装失败
- ffi 报错
- gem install 权限问题

推荐：

```bash
brew install ruby
```

然后使用上面的 `.zshrc` 配置。
