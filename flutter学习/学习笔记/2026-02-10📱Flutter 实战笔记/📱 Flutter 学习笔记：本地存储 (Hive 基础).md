**日期**: 2026年2月10日 
**项目状态**: ✅ 已完成 (100%) 
**核心技能**: `Hive`, `Box`, `put`, `get`, `delete`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码演示了 Hive 数据库最基础的 CRUD（增删改查）操作。假定 `myBox` 已经在 `main()` 函数中被打开。

Dart  镖

```
import 'package:flutter/material.dart';
import 'package:hive/hive.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // 1. 获取已打开的 Box 引用
  final _myBox = Hive.box('myBox');

  // 2. 写入数据 (Create / Update)
  void writeData() {
    // 写入一个键值对：Key 是 1，Value 是一个列表
    _myBox.put(1, ['Apple', 'Banana', 'Mango']);
  }

  // 3. 读取数据 (Read)
  void readData() {
    // 通过 Key 获取数据
    var data = _myBox.get(1);
    print(data); 
  }

  // 4. 删除数据 (Delete)
  void deleteData() {
    // 通过 Key 删除数据
    _myBox.delete(1);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            MaterialButton(
              onPressed: writeData,
              color: Colors.blue[200],
              child: const Text("Write"),
            ),
            MaterialButton(
              onPressed: readData,
              color: Colors.blue[200],
              child: const Text("Read"),
            ),
            MaterialButton(
              onPressed: deleteData,
              color: Colors.blue[200],
              child: const Text("Delete"),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 📝 2. 源码报告 (Source Code Report)  
📝 2.源码报告（Source Code Report）

- **Box 引用 (Box Reference)**: 代码使用 `Hive.box('myBox')` 获取数据库操作句柄。在 Hive 中，"Box" 类似于 SQL 中的"表" (Table)，但它没有严格的列结构，更像是一个文件夹。
    
- **写入机制 (Put Mechanism)**: `_myBox.put(key, value)` 方法既用于新增也用于更新。如果 Key `1` 不存在，则创建；如果已存在，则覆盖。Hive 支持直接存储 `List`、`Map` 等复杂类型，无需像 SharedPreferences 那样手动转换成 JSON 字符串。
    
- **读取机制 (Get Mechanism)**: `_myBox.get(key)` 是同步操作（在内存中读取），速度非常快。如果 Key 不存在，它默认返回 `null`，或者可以设置 `defaultValue` 参数来避免空指针。
    

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. 前置条件 (Prerequisite)

这段代码能运行的前提是 **Box 必须先被打开**。通常在 `main.dart` 中执行：

Dart  镖

```
await Hive.initFlutter();
await Hive.openBox('myBox');
```

如果在 `HomePage` 加载时 Box 还没打开，`Hive.box('myBox')` 会抛出异常。

### 2. 键值对存储 (Key-Value Store)  
2. 键值存储（Key-Value Store）

Hive 是 NoSQL 数据库。

- **Key**: 唯一标识符（通常是 `int` 或 `String`）。
    
- **Value**: 存储的内容。 在 `deleteData` 和 `readData` 中，必须准确知道数据的 `key` 才能操作。
    

### 3. 轻量级 (Lightweight)  3.轻量级（Lightweight）

相比 SQLite，Hive 是纯 Dart 编写的，不需要原生桥接，因此读写速度极快，非常适合存储用户偏好设置、简单的列表数据或缓存。