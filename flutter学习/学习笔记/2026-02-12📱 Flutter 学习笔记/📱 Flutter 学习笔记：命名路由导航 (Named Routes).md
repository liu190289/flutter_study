# 命名路由导航

**日期**: 2026年2月12日

**项目状态**: ✅ 已完成 (100%)

**核心技能**: `Navigator`, `Named Routes`, `MaterialApp Theme`, `StatelessWidget`

---

## 📂 1. 原始源码记录 (Source Code)

为了结构清晰，我将你的代码分成了三个逻辑文件部分。

**1. 入口与路由配置 (main.dart)**



```Dart
import 'package:flutter/material.dart';
import 'package:navigation_routes/pages/home_page.dart';
import 'package:navigation_routes/pages/intro_page.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      // 1. 全局主题配置：强制统一 AppBar 颜色
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
          primary: Colors.deepPurple,
        ),
        appBarTheme: const AppBarTheme(
          backgroundColor: Colors.deepPurple,
          foregroundColor: Colors.white, // 标题和返回箭头变白
        ),
      ),
      // 2. 初始页面
      home: const IntroPage(),
      // 3. 路由表：定义 路径字符串 -> 页面 的映射关系
      routes: {
        '/homepage': (context) => const HomePage(),
        '/intropage': (context) => const IntroPage(),
      },
    );
  }
}
```

**2. 介绍页 (pages/intro_page.dart)**



```Dart
import 'package:flutter/material.dart';

class IntroPage extends StatelessWidget {
  const IntroPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('I N T R O P A G E')),
      body: Center(
        child: ElevatedButton(
          child: const Text('Go to Home Page'),
          onPressed: () {
            // 4. 执行导航：使用名字跳转
            Navigator.pushNamed(context, '/homepage');
          },
        ),
      ),
    );
  }
}
```

**3. 主页 (pages/home_page.dart)**



```Dart
import 'package:flutter/material.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('H O M E P A G E'),
      ),
      // 这里没有内容，因为我们主要测试能不能跳过来
    );
  }
}
```

---

## 📝 2. 源码报告 (Source Code Report)

- **路由表 (Routes Map)**: 在 `MaterialApp` 中定义的 `routes` 属性就像是一张“地图”。
    
    - 键 (Key) 是字符串（如 `'/homepage'`）。
        
    - 值 (Value) 是构建器函数 `(context) => const HomePage()`。
        
    - **作用**: 这样我们在代码深处只需要喊一声“去 `/homepage`”，Flutter 就会查表并把我们带过去，而不需要每次都写 `MaterialPageRoute(builder: ...)`。
        
- **全局主题定制 (Global Theming)**: 代码中不仅实现了导航，还通过 `ThemeData` 统一了 UI 风格。
    
    - **`appBarTheme`**: 这是一个非常高效的写法。它一次性规定了 App 里**所有**页面的 AppBar 都是深紫色背景、白色文字。这样就不需要在每个页面的 AppBar 里重复写 `backgroundColor: ...` 了。
        
- **导航动作 (Push Named)**: `Navigator.pushNamed(context, '/homepage')` 是核心动作。
    
    - **Push**: 意思是“推入”。想象一个栈（Stack），主页被“推”到了介绍页的上面。此时，AppBar 左上角会自动出现一个返回箭头（Back Button），点击它会自动执行 `pop`（弹出），回到介绍页。
        

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. 命名路由 vs 直接路由

- **直接路由 (`Navigator.push`)**: 需要引入目标页面的类文件，耦合度高。
    
- **命名路由 (`Navigator.pushNamed`)**: 只需要知道字符串名字。适合大型项目，方便统一管理所有页面路径。
    

### 2. 栈 (Stack) 的概念

Flutter 的导航是基于“栈”的结构。

- **Push**: 进栈。新页面盖在旧页面上。
    
- **Pop**: 出栈。把当前页面扔掉，露出下面的页面。 这也是为什么跳转后，Flutter 自动帮你加了返回按钮，因为它知道底下还有个 `IntroPage`。
    

### 3. `context` 的重要性

在 `Navigator.pushNamed(context, ...)` 中，`context` 起了定位作用。它告诉 Navigator：“我是当前这个组件，请帮我在这个组件所在的 App 树里找到导航器，并进行跳转。”如果没有 context，Flutter 就不知道该去哪里找导航器。