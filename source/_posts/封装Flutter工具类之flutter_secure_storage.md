---
created: 2026-05-31T13:59
updated: 2026-05-31T22:35
date: 2026/05/31
tags:
  - 工具
title: 封装Flutter工具类之flutter_secure_storage
---


本文记录如何在 Flutter 项目中封装 `flutter_secure_storage`，用于安全存储敏感数据，例如 Token、用户凭证、加密配置等。

## 安装依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter_secure_storage: ^10.3.1
```

## 工具类封装

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class AppSecureStorage {
  AppSecureStorage._();

  static final AppSecureStorage instance = AppSecureStorage._();

  final FlutterSecureStorage _storage = const FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
    ),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock,
    ),
  );

  /// 写入敏感数据
  Future<void> write({
    required String key,
    required String value,
  }) async {
    await _storage.write(key: key, value: value);
  }

  /// 读取敏感数据
  Future<String?> read({
    required String key,
  }) async {
    return _storage.read(key: key);
  }

  /// 删除指定 key
  Future<void> delete({
    required String key,
  }) async {
    await _storage.delete(key: key);
  }

  /// 清空所有数据
  Future<void> deleteAll() async {
    await _storage.deleteAll();
  }

  /// 读取全部数据
  Future<Map<String, String>> readAll() async {
    return _storage.readAll();
  }

  /// 判断 key 是否存在
  Future<bool> contains({
    required String key,
  }) async {
    return _storage.containsKey(key: key);
  }
}
```

## 使用示例

### 写入数据

```dart
await AppSecureStorage.instance.write(
  key: 'token',
  value: 'your_token_value',
);
```

### 读取数据

```dart
final token = await AppSecureStorage.instance.read(
  key: 'token',
);
```

### 删除指定数据

```dart
await AppSecureStorage.instance.delete(
  key: 'token',
);
```

### 清空所有数据

```dart
await AppSecureStorage.instance.deleteAll();
```

### 判断 key 是否存在

```dart
final hasToken = await AppSecureStorage.instance.contains(
  key: 'token',
);
```

### 读取全部数据

```dart
final allData = await AppSecureStorage.instance.readAll();
```

## 说明

- Android 端使用 `encryptedSharedPreferences: true` 开启加密 SharedPreferences。
- iOS 端使用 `KeychainAccessibility.first_unlock`，表示设备首次解锁后即可访问 Keychain 数据。
- 建议只存储敏感且必要的数据，例如 `accessToken`、`refreshToken`、用户身份标识等。
- 不建议存储大量业务数据，普通缓存数据可以使用 `shared_preferences` 或本地数据库。

## 常见使用场景

- 存储登录 Token
- 存储 Refresh Token
- 存储用户加密凭证
- 存储设备唯一标识
- 存储需要安全保护的配置项
````