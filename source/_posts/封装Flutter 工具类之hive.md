---
created: 2026-05-31T14:01
updated: 2026-05-31T22:35
date: 2015/05/31
tags:
  - 工具
categories:
  - 工具
title: 封装Flutter 工具类之hive
---

# Flutter 封装 Hive 工具类

## 1. 简介

本文档记录 Flutter 项目中对 `Hive` 本地存储的简单封装方式。

该工具类主要用于：

- 本地键值对缓存
- 字符串、数字、布尔值等基础类型存储
- 普通对象存储
- Model 列表 JSON 缓存
- 支持缓存过期时间
- 支持统一初始化、读取、删除、清空等操作

---

## 2. 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  hive: ^2.2.3
  path_provider: ^2.1.0
```

然后执行：

```bash
flutter pub get
```

---

## 3. 初始化 Hive

建议在 `main.dart` 中初始化：

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await AppHiveStorage.init();

  runApp(const MyApp());
}
```

---

## 4. 完整工具类代码

```dart
import 'dart:convert';

import 'package:hive/hive.dart';
import 'package:path_provider/path_provider.dart';

class AppHiveStorage {
  static const String _boxName = 'app_data';

  static Box? _box;

  /// 示例缓存 Key：充值列表
  static const String rechargeListKey = 'cache_recharge_list';

  AppHiveStorage._internal();

  static final AppHiveStorage instance = AppHiveStorage._internal();

  /// 初始化 Hive
  static Future<void> init() async {
    final appDir = await getApplicationDocumentsDirectory();

    Hive.init(appDir.path);

    /// 打开唯一 Box，所有数据统一放在该 Box 中
    _box = await Hive.openBox(_boxName);
  }

  /// 获取 Box 实例
  Box get box {
    assert(_box != null, 'AppHiveStorage.init() must be called first');
    return _box!;
  }

  /// 保存数据
  Future<void> set<T>(String key, T value) async {
    await box.put(key, value);
  }

  /// 读取数据
  T? get<T>(String key, {T? defaultValue}) {
    final value = box.get(key, defaultValue: defaultValue);

    if (value is T) {
      return value;
    }

    return defaultValue;
  }

  /// 删除指定 Key
  Future<void> remove(String key) async {
    await box.delete(key);
  }

  /// 判断是否包含指定 Key
  bool contains(String key) {
    return box.containsKey(key);
  }

  /// 清空整个 Box
  Future<void> clear() async {
    await box.clear();
  }

  /// 获取所有数据
  Map<String, dynamic> getAll() {
    return Map<String, dynamic>.from(box.toMap());
  }
}

/// JSON 缓存包装类
class JsonCacheWrapper {
  /// 缓存时间戳，单位：毫秒
  final int timestamp;

  /// JSON 字符串
  final String json;

  JsonCacheWrapper({
    required this.timestamp,
    required this.json,
  });

  Map<String, dynamic> toJson() {
    return {
      'timestamp': timestamp,
      'json': json,
    };
  }

  factory JsonCacheWrapper.fromJson(Map<String, dynamic> map) {
    return JsonCacheWrapper(
      timestamp: map['timestamp'] as int,
      json: map['json'] as String,
    );
  }

  /// 判断缓存是否过期
  bool isExpired(Duration maxAge) {
    return DateTime.now().millisecondsSinceEpoch - timestamp >
        maxAge.inMilliseconds;
  }
}

/// AppHiveStorage JSON 扩展方法
extension AppHiveStorageJson on AppHiveStorage {
  /// 保存 Model 列表
  ///
  /// 数据会被转换成 JSON String 后存入 Hive。
  Future<void> setJsonList<T>(
    String key,
    List<T> list,
    Map<String, dynamic> Function(T e) toJson,
  ) async {
    final jsonStr = jsonEncode(
      list.map(toJson).toList(),
    );

    final wrapper = JsonCacheWrapper(
      timestamp: DateTime.now().millisecondsSinceEpoch,
      json: jsonStr,
    );

    await set(key, jsonEncode(wrapper.toJson()));
  }

  /// 读取 Model 列表
  ///
  /// 如果 [maxAge] 不为空，则会判断缓存是否过期。
  /// 缓存过期或 JSON 解析异常时，会自动删除缓存并返回空数组。
  List<T> getJsonList<T>(
    String key,
    T Function(Map<String, dynamic> json) fromJson, {
    Duration? maxAge,
  }) {
    final raw = get<String>(key);

    if (raw == null) return [];

    try {
      final wrapperMap = jsonDecode(raw) as Map<String, dynamic>;
      final wrapper = JsonCacheWrapper.fromJson(wrapperMap);

      if (maxAge != null && wrapper.isExpired(maxAge)) {
        remove(key);
        return [];
      }

      final list = jsonDecode(wrapper.json) as List;

      return list
          .map(
            (e) => fromJson(
              Map<String, dynamic>.from(e as Map),
            ),
          )
          .toList();
    } catch (e) {
      /// JSON 异常时清除缓存，避免脏数据影响业务
      remove(key);
      return [];
    }
  }
}
```

---

## 5. 基础使用示例

### 5.1 保存数据

```dart
await AppHiveStorage.instance.set('token', '123456');
await AppHiveStorage.instance.set('theme', 'dark');
await AppHiveStorage.instance.set('age', 18);
```

### 5.2 读取数据

```dart
final token = AppHiveStorage.instance.get<String>('token');

final age = AppHiveStorage.instance.get<int>(
  'age',
  defaultValue: 0,
);
```

### 5.3 删除数据

```dart
await AppHiveStorage.instance.remove('age');
```

### 5.4 判断 Key 是否存在

```dart
final hasToken = AppHiveStorage.instance.contains('token');

print(hasToken);
```

### 5.5 获取所有缓存数据

```dart
final allData = AppHiveStorage.instance.getAll();

print(allData);
```

### 5.6 清空所有缓存

```dart
await AppHiveStorage.instance.clear();
```

---

## 6. 对象存储示例

如果对象已经注册了 Hive Adapter，可以直接存储对象。

```dart
await AppHiveStorage.instance.set(
  'user',
  User('Tom', 20),
);

final user = AppHiveStorage.instance.get<User>('user');
```

> 如果没有注册 Hive Adapter，建议使用 JSON 字符串方式进行存储。

---

## 7. Model 列表缓存示例

### 7.1 示例 Model

```dart
class User {
  final String name;
  final int age;

  User({
    required this.name,
    required this.age,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      name: json['name'] as String,
      age: json['age'] as int,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'name': name,
      'age': age,
    };
  }
}
```

### 7.2 保存 Model 列表

```dart
await AppHiveStorage.instance.setJsonList<User>(
  'user_list',
  users,
  (e) => e.toJson(),
);
```

说明：

- `users`：需要缓存的 User 列表
- `(e) => e.toJson()`：Model 转 JSON 方法

### 7.3 读取 Model 列表

```dart
final users = AppHiveStorage.instance.getJsonList<User>(
  'user_list',
  (json) => User.fromJson(json),
  maxAge: const Duration(minutes: 10),
);
```

说明：

- `maxAge`：缓存有效期，可不传
- 不传 `maxAge` 表示读取时不进行过期校验
- 如果缓存不存在，返回空数组
- 如果缓存过期，自动删除缓存并返回空数组
- 如果 JSON 解析失败，自动删除缓存并返回空数组

---

## 8. 常见缓存 Key 管理方式

建议统一管理缓存 Key，避免字符串散落在业务代码中。

```dart
class AppCacheKeys {
  static const String token = 'token';
  static const String theme = 'theme';
  static const String userInfo = 'user_info';
  static const String rechargeList = 'cache_recharge_list';
}
```

使用示例：

```dart
await AppHiveStorage.instance.set(
  AppCacheKeys.token,
  '123456',
);

final token = AppHiveStorage.instance.get<String>(
  AppCacheKeys.token,
);
```

---

## 9. 使用场景

| 场景 | 说明 |
| --- | --- |
| Token 缓存 | 保存登录 Token |
| 用户信息缓存 | 保存用户基础信息 |
| 主题配置 | 保存深色模式、语言等配置 |
| 接口缓存 | 缓存接口返回的 Model 列表 |
| 离线数据 | 保存简单的离线业务数据 |
| 表单草稿 | 保存用户未提交的表单内容 |

---

## 10. 注意事项

- 使用前必须先调用 `AppHiveStorage.init()`。
- 当前封装使用单 Box 存储所有数据。
- 基础类型可以直接存储。
- 自定义对象如果不使用 JSON，需要注册 Hive Adapter。
- Model 列表推荐使用 `setJsonList` 和 `getJsonList`。
- JSON 缓存读取失败时会自动清理，避免脏数据影响业务。
- 如果缓存数据较大，建议按业务拆分多个 Box。
- 不建议将高度敏感信息直接存储在 Hive 中，如密码、支付信息等。

---

## 11. 推荐目录结构

```text
lib/
├── common/
│   └── storage/
│       ├── app_hive_storage.dart
│       └── app_cache_keys.dart
├── models/
│   └── user.dart
└── main.dart
```

---

## 12. 总结

该封装适合 Flutter 项目中轻量级本地缓存场景。

主要优点：

- 使用简单
- 统一管理
- 支持泛型读取
- 支持 JSON Model 列表缓存
- 支持缓存过期时间
- 代码侵入性低

如果项目中需要缓存大量结构化数据，可以进一步结合 Hive Adapter、分 Box 管理或数据库方案进行扩展。
```