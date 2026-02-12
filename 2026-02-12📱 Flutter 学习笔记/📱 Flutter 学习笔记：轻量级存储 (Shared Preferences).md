**日期**: 2026年2月12日 
**项目状态**: ✅ 已完成 (100%) **核心技能**: `WidgetsFlutterBinding`, `SharedPreferences`, `Async Main`, `Singleton`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码展示了完整的 Shared Preferences 实现流程：从入口函数的初始化，到工具类的封装，再到页面的读写操作。

**1. 入口配置 (main.dart) - 新增**



```Dart
import 'package:flutter/material.dart';
import 'package:shared_preferences_demo/user_page.dart';
import 'package:shared_preferences_demo/user_simple_preferences.dart';

// 1. 将 main 函数改为 async
void main() async {
  // 2. 确保 Flutter 绑定初始化 (必须在 runApp 之前调用原生代码时使用)
  WidgetsFlutterBinding.ensureInitialized();

  // 3. 在 App 启动前预加载 SharedPreferences
  await UserSimplePreferences.init();
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: MyuserPage(),
    );
  }
}
```

**2. 工具类 (user_simple_preferences.dart)**



```Dart
import 'package:shared_preferences/shared_preferences.dart';

class UserSimplePreferences {
  static const String keyUsername = 'username';
  static SharedPreferences? _preferences;

  static Future init() async {
    _preferences = await SharedPreferences.getInstance();
  }

  static Future setUsername(String username) async {
    await _preferences?.setString(keyUsername, username);
  }

  static String? getUsername() {
    return _preferences?.getString(keyUsername);
  }
}
```

**3. 界面层 (user_page.dart)**



```Dart
import 'package:flutter/material.dart';
import 'package:shared_preferences_demo/user_simple_preferences.dart';

class MyuserPage extends StatefulWidget {
  const MyuserPage({super.key});
  @override
  State<MyuserPage> createState() => _MyuserPageState();
}

class _MyuserPageState extends State<MyuserPage> {
  final controller = TextEditingController();
  String username = '';

  @override
  void initState() {
    super.initState();
    // 此时 Preferences 已在 main 中初始化，可直接同步读取
    username = UserSimplePreferences.getUsername() ?? '';
    controller.text = username;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('User Page')),
      body: Center(
        child: Column(
          children: [
             // ... 输入框和按钮 UI 代码同上 ...
             ElevatedButton(
                onPressed: () async {
                  await UserSimplePreferences.setUsername(username);
                  // ... SnackBar 反馈 ...
                },
                child: const Text('Save'),
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

- **绑定初始化 (Widgets Binding)**: `WidgetsFlutterBinding.ensureInitialized()` 是至关重要的一行代码。因为 `main` 函数是异步的，且我们在 `runApp` 之前就调用了 `SharedPreferences`（通过 Platform Channels 与原生系统通信）。Flutter 必须先建立好与底层的通信桥梁，否则会抛出 "BinaryMessenger is null" 错误。
    
- **预加载策略 (Pre-loading Strategy)**: 在 `main` 函数中等待 `init()` 完成 (`await`)，意味着 App 在启动画面（Splash Screen）之后、渲染第一帧 UI 之前，数据就已经准备好了。
    
    - **优点**: 进入首页 (`MyuserPage`) 时，`init` 已经完成，可以直接同步读取数据，避免了在首页还要显示 "Loading..." 转圈的情况。
        
- **单例模式 (Singleton Access)**: 通过 `UserSimplePreferences` 静态类，我们在整个 App 生命周期中只持有一个 `_preferences` 实例。这比每次读写都调用 `getInstance()` 效率更高。
    

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. 为什么 main 是 async？

Dart 的入口函数 `main` 默认是同步的。但为了在 App 启动前执行耗时操作（如读取磁盘配置、初始化数据库），我们需要将其标记为 `async`。这会稍微延迟 App 的首屏渲染时间，但换来了数据的一致性。

### 2. 确保初始化的时机

如果你在 `runApp` 之前使用了任何 Flutter 插件（Firebase, Hive, SharedPrefs 等），务必调用 `WidgetsFlutterBinding.ensureInitialized()`。这是 Flutter 引擎启动流程的标准规范。

### 3. 数据持久化生命周期

- **Init (main)**: 打开连接。
    
- **Read (initState)**: 填充 UI。
    
- **Write (onPressed)**: 保存修改。 这个闭环保证了用户关闭 App 后，下次打开依然能看到之前的数据。
    

---