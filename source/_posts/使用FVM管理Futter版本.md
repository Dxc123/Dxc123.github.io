---
created: 2024-02-01T17:55
updated: 2026-05-31T22:39
title: 使用FVM管理Futter版本
tags:
  - FVM
categories:
  - FVM
---
## Flutter多版本管理工具fvm使用

[参考链接](https://juejin.cn/post/7119468563630555166)

1. 使用`brew`命令安装`Fvm`

```
brew tap leoafarias/fvm
brew install fvm
```

卸载命令：

```
brew uninstall fvm brew untap leoafarias/fvm
```

2.配置 `FVM_HOME`

```
export FVM_HOME="$HOME/.fvm"
```

如果不使用上面的配置，则默认使用`$HOME/fvm`

3. 安装`flutter sdk`
- 安装指定的sdk版本
  
  ```
  fvm install 3.0.4
  ```

- 查看所有已经安装的sdk
  
  ```
  fvm list
  ```
4. 使用`fvm`配置管理全局sdk版本

//将3.22.0版本设置为全局默认版本
```
fvm global 3.22.0
```

//一个常用的使用场景，就是单独给某个项目设置使用特定flutter版本，使用如下命令：
```
fvm use 3.22.0
```

//移除某个版本的 Flutter
```
fvm remove 3.22.0
```

在执行完上面的命令之后，会在.fvm下生成一个default文件夹。可以将这个目录下配置为flutter的环境变量，后续可以通过fvm use命令结合fvm global 命令直接更改flutter使用版本了

```
//将 fvm 路径添加到 shell 配置文件(这里是.zshrc文件)
export PATH=”$PATH:`pwd`/flutter/bin”$ fvm install
export PATH=”$PATH:`pwd`/bin/cache/dart-sdk/bin”
export PATH=”$PATH:`pwd`/.pub-cache/bin”

export PATH="$HOME/fvm/default/bin:$PATH"
```

5. **设置 IDE**
- Android Studio配置本地flutter路径
  
  在 Android Studio 的菜单中打开 Languages and Frameworks->  flutter SDK path 选择本地 fvm文件下versions 下具体某个版本

- VS Code配置本地flutter路径
  
    //a执行 fvm list 命令,拷贝获得路径：/Users/daixingchuang/fvm/versions

b.快捷键cmd + shift + p 打开命令面板，输入 setting,选择打开用户设置

在最后添加上：

```
 // 配置本地 flutter 路径
    "dart.flutterSdkPaths": [
        "/Users/daixingchuang/fvm/versions",
    ],
```

c.快捷键cmd + shift + p 打开命令面板,然后输入 change sdk，选择你本地 fvm文件下的版本即可


fvm install 3.24.5  报错：
RPC 失败。curl 18 transfer closed with outstanding read data remaining

解决：
git config --global http.postBuffer 524288000

在尝试一下 fvm install 3.24.5
