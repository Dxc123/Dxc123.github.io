# CocoaPods 常用命令

## 1. 安装 CocoaPods

```bash
sudo gem install cocoapods
```

如果使用 Homebrew Ruby：

```bash
gem install cocoapods
```

---

## 2. 查看版本

```bash
pod --version
```

---

## 3. 初始化 Podfile

进入 iOS 工程目录：

```bash
cd ios
pod init
```

会生成：

```bash
Podfile
```

---

## 4. 安装依赖

```bash
pod install
```

Flutter 常用：

```bash
cd ios && pod install
```

---

## 5. 更新 Pods

更新 Podfile 中依赖：

```bash
pod update
```

更新指定库：

```bash
pod update AFNetworking
```

---

## 6. 更新 CocoaPods 仓库

```bash
pod repo update
```

或者：

```bash
pod install --repo-update
```

Flutter 推荐：

```bash
pod install --repo-update
```

---

## 7. 删除 Pods 重新安装

常用于 Flutter/iOS 编译异常：

```bash
rm -rf Pods
rm -rf Podfile.lock
pod install
```

Flutter：

```bash
flutter clean
rm -rf ios/Pods
rm -rf ios/Podfile.lock
cd ios && pod install
```

---

## 8. 清理 CocoaPods 缓存

查看缓存：

```bash
pod cache list
```

清理全部缓存：

```bash
pod cache clean --all
```

清理指定库：

```bash
pod cache clean Alamofire --all
```

---

## 9. 查看 Pod 仓库

```bash
pod repo list
```

---

## 10. 删除 Specs 仓库

```bash
pod repo remove trunk
```

重新安装：

```bash
pod setup
```

---

## 11. 搜索库

```bash
pod search afnetworking
```

---

## 12. 查看 Pod 环境

```bash
pod env
```

---

## 13. 常见 Flutter CocoaPods 修复命令

### arm64 / ffi / Ruby 问题

```bash
sudo gem pristine ffi --version 1.17.2
```

---

### 重新安装 CocoaPods

```bash
sudo gem uninstall cocoapods
sudo gem install cocoapods
```

---

### Flutter iOS 完整修复

```bash
flutter clean

rm -rf ios/Pods
rm -rf ios/Podfile.lock
rm -rf ~/Library/Developer/Xcode/DerivedData

flutter pub get

cd ios
pod deintegrate
pod install --repo-update
```

---

## 14. 国内加速（推荐）

### 查看当前源

```bash
gem sources -l
```

---

### 替换 RubyGems 国内源

```bash
gem sources --remove https://rubygems.org/

gem sources --add https://gems.ruby-china.com/

gem sources -l
```

---

## 15. M系列 Mac 推荐环境

推荐：

- Homebrew Ruby
- CocoaPods 最新版
- 不使用系统 Ruby

安装：

```bash
brew install ruby
gem install cocoapods
```

配置 `.zshrc`：

```bash
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
```
