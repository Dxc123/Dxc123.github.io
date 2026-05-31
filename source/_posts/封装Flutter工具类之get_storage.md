---
created: 2026-05-31T13:50
updated: 2026-05-31T22:29
date: 2026/05/31
tags:
  - 工具
---
# AppGetStorage 工具类

> get_storage: ^2.1.1

`AppGetStorage` 是项目里对 [`get_storage`](https://pub.dev/packages/get_storage) 的一层静态工具封装，用来统一处理本地轻量级键值存储。它隐藏了 `GetStorage` 实例创建细节，让业务代码可以直接通过静态方法完成初始化、读取、写入、删除、清空和监听。

## 核心设计

```dart
class AppGetStorage {
  AppGetStorage._();

  static const String defaultContainer = 'app_get_storage';
}
```

- `AppGetStorage._()`：私有构造方法，禁止外部实例化，说明这是一个纯工具类。
- `defaultContainer`：默认存储容器名为 `app_get_storage`。
- 所有方法都支持传入 `container`，可以按业务模块隔离不同的本地存储空间。

## 初始化

```dart
static Future<bool> init({String container = defaultContainer}) {
  return GetStorage.init(container);
}
```

应用启动时需要先初始化对应容器，通常放在 `main()` 中：

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await AppGetStorage.init();
  runApp(const MyApp());
}
```

如果使用自定义容器：

```dart
await AppGetStorage.init(container: 'user_cache');
```

## 内部 Box 创建

```dart
static GetStorage _box({
  String container = defaultContainer,
  String? path,
  Map<String, dynamic>? initialData,
}) {
  return GetStorage(container, path, initialData);
}
```

`_box()` 是内部方法，用来根据容器名创建 `GetStorage` 对象。外部业务代码不直接接触 `GetStorage`，而是通过 `AppGetStorage` 的静态方法访问。

当前公开方法只暴露了 `container` 参数，`path` 和 `initialData` 保留在内部方法里，后续如果需要支持自定义存储路径或初始数据，可以继续扩展。

## 读取数据

```dart
static T? read<T>(String key, {String container = defaultContainer}) {
  return _box(container: container).read<T>(key);
}
```

示例：

```dart
final token = AppGetStorage.read<String>('token');
final userId = AppGetStorage.read<int>('user_id');
```

`read<T>()` 使用泛型返回指定类型，如果 key 不存在会返回 `null`。

## 判断 Key 是否存在

```dart
static bool hasData(String key, {String container = defaultContainer}) {
  return _box(container: container).hasData(key);
}
```

示例：

```dart
final hasToken = AppGetStorage.hasData('token');
```

适合在读取前判断某个 key 是否已经被写入。

## 写入数据

```dart
static Future<void> write(
  String key,
  dynamic value, {
  String container = defaultContainer,
}) {
  return _box(container: container).write(key, value);
}
```

示例：

```dart
await AppGetStorage.write('token', token);
await AppGetStorage.write('is_first_open', false);
```

`value` 是 `dynamic`，可以写入字符串、数字、布尔值、Map、List 等 `get_storage` 支持的数据。

## 仅为空时写入

```dart
static Future<void> writeIfNull(
  String key,
  dynamic value, {
  String container = defaultContainer,
}) {
  return _box(container: container).writeIfNull(key, value);
}
```

示例：

```dart
await AppGetStorage.writeIfNull('theme_mode', 'system');
```

当 key 不存在时才写入，适合设置默认配置，避免覆盖用户已有设置。

## 删除单个 Key

```dart
static Future<void> remove(
  String key, {
  String container = defaultContainer,
}) {
  return _box(container: container).remove(key);
}
```

示例：

```dart
await AppGetStorage.remove('token');
```

常用于退出登录、清理临时状态等场景。

## 清空容器

```dart
static Future<void> clearAll({String container = defaultContainer}) {
  return _box(container: container).erase();
}
```

示例：

```dart
await AppGetStorage.clearAll();
```

会清空当前容器下的所有数据。使用前需要确认不会误删仍然需要保留的本地配置。

## 获取所有 Key

```dart
static Iterable<String> getKeys({String container = defaultContainer}) {
  return _box(container: container).getKeys<Iterable<String>>();
}
```

示例：

```dart
final keys = AppGetStorage.getKeys().toList();
```

适合调试、迁移本地缓存或批量清理时使用。

## 获取所有 Value

```dart
static Iterable<dynamic> getValues({String container = defaultContainer}) {
  return _box(container: container).getValues<Iterable<dynamic>>();
}
```

示例：

```dart
final values = AppGetStorage.getValues().toList();
```

返回当前容器里的所有 value。

## 监听存储变化

```dart
static VoidCallback listen(
  VoidCallback listener, {
  String container = defaultContainer,
}) {
  return _box(container: container).listen(listener);
}
```

示例：

```dart
final disposeStorageListener = AppGetStorage.listen(() {
  debugPrint('storage changed');
});

disposeStorageListener();
```

`listen()` 会监听当前容器中任意数据变化，并返回一个 `VoidCallback`，用于取消监听。

## 监听指定 Key

```dart
static VoidCallback listenKey<T>(
  String key,
  ValueSetter<T?> listener, {
  String container = defaultContainer,
}) {
  return _box(container: container).listenKey(key, (dynamic value) {
    listener(value as T?);
  });
}
```

示例：

```dart
final disposeTokenListener = AppGetStorage.listenKey<String>(
  'token',
  (token) {
    debugPrint('token changed: $token');
  },
);

disposeTokenListener();
```

`listenKey<T>()` 会监听指定 key 的变化，并把回调值转换成 `T?`。调用时建议显式指定泛型，避免类型转换错误。

## 常见使用场景

- 保存登录态：`token`、`user_id`、用户基础信息。
- 保存应用配置：主题模式、语言、首次启动状态。
- 保存轻量缓存：接口筛选条件、页面临时状态。
- 监听状态变化：例如 token 改变后刷新登录状态。

## 使用建议

- 在应用启动阶段先调用 `AppGetStorage.init()`。
- 对重要业务字段建议统一维护 key 常量，避免字符串拼写错误。
- 不要把大文件、图片、复杂数据库型数据放进 `get_storage`。
- `clearAll()` 会清空整个容器，退出登录时要确认是否需要保留主题、语言等全局配置。
- `listenKey<T>()` 内部有强制类型转换，读取和写入同一个 key 时要保持类型一致。

## API 速查

| 方法 | 作用 |
| --- | --- |
| `init()` | 初始化存储容器 |
| `read<T>(key)` | 读取指定 key |
| `hasData(key)` | 判断 key 是否存在 |
| `write(key, value)` | 写入或覆盖数据 |
| `writeIfNull(key, value)` | key 不存在时写入 |
| `remove(key)` | 删除指定 key |
| `clearAll()` | 清空当前容器 |
| `getKeys()` | 获取所有 key |
| `getValues()` | 获取所有 value |
| `listen(listener)` | 监听容器变化 |
| `listenKey<T>(key, listener)` | 监听指定 key 变化 |

## 完整代码

```dart
import 'package:flutter/widgets.dart';
import 'package:get_storage/get_storage.dart';

class AppGetStorage {
  AppGetStorage._();

  static const String defaultContainer = 'app_get_storage';

  // 初始化
  static Future<bool> init({String container = defaultContainer}) {
    return GetStorage.init(container);
  }

  static GetStorage _box({
    String container = defaultContainer,
    String? path,
    Map<String, dynamic>? initialData,
  }) {
    return GetStorage(container, path, initialData);
  }

  // 读取
  static T? read<T>(String key, {String container = defaultContainer}) {
    return _box(container: container).read<T>(key);
  }

  // 判断是否存在
  static bool hasData(String key, {String container = defaultContainer}) {
    return _box(container: container).hasData(key);
  }

  // 写入
  static Future<void> write(
    String key,
    dynamic value, {
    String container = defaultContainer,
  }) {
    return _box(container: container).write(key, value);
  }

  // 写入
  static Future<void> writeIfNull(
    String key,
    dynamic value, {
    String container = defaultContainer,
  }) {
    return _box(container: container).writeIfNull(key, value);
  }

  // 删除某个 key
  static Future<void> remove(
    String key, {
    String container = defaultContainer,
  }) {
    return _box(container: container).remove(key);
  }

  // 清空all
  static Future<void> clearAll({String container = defaultContainer}) {
    return _box(container: container).erase();
  }

  // 获取所有 key
  static Iterable<String> getKeys({String container = defaultContainer}) {
    return _box(container: container).getKeys<Iterable<String>>();
  }

  // 获取所有 value
  static Iterable<dynamic> getValues({String container = defaultContainer}) {
    return _box(container: container).getValues<Iterable<dynamic>>();
  }

  // 监听
  static VoidCallback listen(
    VoidCallback listener, {
    String container = defaultContainer,
  }) {
    return _box(container: container).listen(listener);
  }

  // 监听 key
  static VoidCallback listenKey<T>(
    String key,
    ValueSetter<T?> listener, {
    String container = defaultContainer,
  }) {
    return _box(container: container).listenKey(key, (dynamic value) {
      listener(value as T?);
    });
  }
}
```
