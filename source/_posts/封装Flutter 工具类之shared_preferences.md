---
created: 2026-05-31T14:25
updated: 2026-05-31T22:30
date: 2026/05/31
tags:
  - 工具
categories:
  - 工具
---

## 1. 简介

`shared_preferences` 是 Flutter 中常用的本地轻量级数据持久化方案，适合保存用户配置、登录状态、开关状态、缓存字段等简单数据。

本文封装了一个 `AppSharedPreferencesStorage` 工具类，用于统一管理 `shared_preferences` 的初始化、读取、保存、删除和清空操作。工具类采用单例模式，并支持 `String`、`int`、`double`、`bool`、`List<String>` 等基础类型存储。<span class="copilot-citation-ref">[1]</span>

---

## 2. 安装依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  shared_preferences: ^2.2.3
```

然后执行：

```bash
flutter pub get
```

---

## 3. 初始化

建议在 `main.dart` 中启动 App 前完成初始化。

```dart
import 'package:flutter/material.dart';
import 'app_shared_preferences_storage.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await AppSharedPreferencesStorage.init();

  runApp(const MyApp());
}
```

---

## 4. 完整工具类代码


```dart
import 'dart:convert';

import 'package:shared_preferences/shared_preferences.dart';

class AppSharedPreferencesStorage {
  AppSharedPreferencesStorage._internal();

  static final AppSharedPreferencesStorage instance =
      AppSharedPreferencesStorage._internal();

  static SharedPreferences? _prefs;

  /// 初始化
  static Future<void> init() async {
    _prefs ??= await SharedPreferences.getInstance();
  }

  /// 获取 SharedPreferences 实例
  Future<SharedPreferences> get _storage async {
    final prefs = _prefs;

    if (prefs != null) {
      return prefs;
    }

    await init();
    return _prefs!;
  }

  /// 保存数据
  Future<bool> set<T>(String key, T value) async {
    final prefs = await _storage;

    if (value is String) {
      return prefs.setString(key, value);
    }

    if (value is int) {
      return prefs.setInt(key, value);
    }

    if (value is double) {
      return prefs.setDouble(key, value);
    }

    if (value is bool) {
      return prefs.setBool(key, value);
    }

    if (value is List<String>) {
      return prefs.setStringList(key, value);
    }

    throw ArgumentError.value(
      value,
      'value',
      'Only String, int, double, bool and List<String> are supported.',
    );
  }

  /// 读取数据
  Future<T?> get<T>(String key, {T? defaultValue}) async {
    final prefs = await _storage;
    final value = prefs.get(key);

    if (value is T) {
      return value;
    }

    return defaultValue;
  }

  /// 读取字符串
  Future<String?> getString(String key, {String? defaultValue}) async {
    final prefs = await _storage;
    return prefs.getString(key) ?? defaultValue;
  }

  /// 读取整数
  Future<int?> getInt(String key, {int? defaultValue}) async {
    final prefs = await _storage;
    return prefs.getInt(key) ?? defaultValue;
  }

  /// 读取浮点数
  Future<double?> getDouble(String key, {double? defaultValue}) async {
    final prefs = await _storage;
    return prefs.getDouble(key) ?? defaultValue;
  }

  /// 读取布尔值
  Future<bool?> getBool(String key, {bool? defaultValue}) async {
    final prefs = await _storage;
    return prefs.getBool(key) ?? defaultValue;
  }

  /// 读取字符串列表
  Future<List<String>?> getStringList(
    String key, {
    List<String>? defaultValue,
  }) async {
    final prefs = await _storage;
    return prefs.getStringList(key) ?? defaultValue;
  }

  /// 保存对象
  Future<bool> setObject(String key, Map<String, dynamic> value) async {
    final jsonString = jsonEncode(value);
    return set<String>(key, jsonString);
  }

  /// 读取对象
  Future<Map<String, dynamic>?> getObject(String key) async {
    final jsonString = await getString(key);

    if (jsonString == null || jsonString.isEmpty) {
      return null;
    }

    try {
      final value = jsonDecode(jsonString);

      if (value is Map<String, dynamic>) {
        return value;
      }

      return null;
    } catch (_) {
      return null;
    }
  }

  /// 保存对象列表
  Future<bool> setObjectList(
    String key,
    List<Map<String, dynamic>> value,
  ) async {
    final jsonString = jsonEncode(value);
    return set<String>(key, jsonString);
  }

  /// 读取对象列表
  Future<List<Map<String, dynamic>>> getObjectList(String key) async {
    final jsonString = await getString(key);

    if (jsonString == null || jsonString.isEmpty) {
      return [];
    }

    try {
      final value = jsonDecode(jsonString);

      if (value is List) {
        return value
            .whereType<Map>()
            .map((item) => Map<String, dynamic>.from(item))
            .toList();
      }

      return [];
    } catch (_) {
      return [];
    }
  }

  /// 判断 key 是否存在
  Future<bool> containsKey(String key) async {
    final prefs = await _storage;
    return prefs.containsKey(key);
  }

  /// 删除指定 key
  Future<bool> remove(String key) async {
    final prefs = await _storage;
    return prefs.remove(key);
  }

  /// 清空所有数据
  Future<bool> clear() async {
    final prefs = await _storage;
    return prefs.clear();
  }

  /// 获取所有 key
  Future<Set<String>> getKeys() async {
    final prefs = await _storage;
    return prefs.getKeys();
  }
}
```

---

## 5. 使用示例

### 5.1 保存数据

```dart
final storage = AppSharedPreferencesStorage.instance;

await storage.set<String>('token', 'abc123');
await storage.set<int>('user_id', 1001);
await storage.set<bool>('is_login', true);
await storage.set<double>('font_size', 16.0);
await storage.set<List<String>>('roles', ['admin', 'user']);
```

---

### 5.2 读取数据

```dart
final token = await storage.getString('token');
final userId = await storage.getInt('user_id');
final isLogin = await storage.getBool('is_login');
final fontSize = await storage.getDouble('font_size');
final roles = await storage.getStringList('roles');
```

---

### 5.3 设置默认值

```dart
final isLogin = await storage.getBool(
  'is_login',
  defaultValue: false,
);

final username = await storage.getString(
  'username',
  defaultValue: '游客',
);
```

---

### 5.4 保存对象

```dart
await storage.setObject('user_info', {
  'id': 1,
  'name': '张三',
  'age': 20,
});
```

---

### 5.5 读取对象

```dart
final userInfo = await storage.getObject('user_info');

print(userInfo?['name']);
```

---

### 5.6 保存对象列表

```dart
await storage.setObjectList('user_list', [
  {
    'id': 1,
    'name': '张三',
  },
  {
    'id': 2,
    'name': '李四',
  },
]);
```

---

### 5.7 读取对象列表

```dart
final userList = await storage.getObjectList('user_list');

for (final user in userList) {
  print(user['name']);
}
```

---

### 5.8 判断 key 是否存在

```dart
final hasToken = await storage.containsKey('token');

if (hasToken) {
  print('token 存在');
}
```

---

### 5.9 删除指定数据

```dart
await storage.remove('token');
```

---

### 5.10 清空所有数据

```dart
await storage.clear();
```

---

## 6. Key 统一管理

建议将所有缓存 key 统一维护，避免字符串散落在业务代码中。

```dart
class StorageKeys {
  static const String token = 'token';
  static const String userId = 'user_id';
  static const String isLogin = 'is_login';
  static const String userInfo = 'user_info';
  static const String userList = 'user_list';
}
```

使用方式：

```dart
await storage.set(StorageKeys.token, 'abc123');

final token = await storage.getString(StorageKeys.token);
```

---

## 7. 推荐目录结构

```text
lib
├── main.dart
├── utils
│   └── app_shared_preferences_storage.dart
└── constants
    └── storage_keys.dart
```

---

## 8. 常见使用场景

- 保存用户登录状态
- 保存用户 token
- 保存用户基础信息
- 保存 App 主题配置
- 保存语言配置
- 保存本地开关状态
- 保存简单缓存数据
- 保存搜索历史
- 保存引导页是否展示过

---

## 9. 注意事项

- `shared_preferences` 适合保存轻量级数据，不适合保存大量数据。
- 不建议使用 `shared_preferences` 保存敏感信息，例如密码、支付信息等。
- 如果需要保存敏感数据，建议使用 `flutter_secure_storage`。
- 如果需要保存大量结构化数据，建议使用 `Hive`、`Isar` 或 `SQLite`。
- 对象和对象列表需要通过 `jsonEncode` 转为字符串后再保存。
- 读取对象时需要通过 `jsonDecode` 还原为 Dart 对象。
- 建议在 App 启动时提前调用 `AppSharedPreferencesStorage.init()`，避免首次读取时再初始化。

---

## 10. 总结

通过封装 `AppSharedPreferencesStorage`，可以让本地轻量级缓存的使用更加统一、简洁和安全。

该工具类适合用于保存 App 配置、登录状态、用户信息、简单缓存等数据。如果项目中同时存在安全存储或数据库存储，也建议根据数据类型和安全级别选择合适的存储方案。