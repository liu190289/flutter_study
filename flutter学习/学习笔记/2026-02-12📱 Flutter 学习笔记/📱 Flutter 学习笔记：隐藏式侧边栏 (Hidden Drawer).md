 **日期**: 2026年2月12日 
 **项目状态**: ✅ 已完成 (100%) 
 **核心技能**: `hidden_drawer_menu`, `ScreenHiddenDrawer`, `ItemHiddenMenu`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码使用了第三方包 `hidden_drawer_menu` 来实现一种非常炫酷的导航效果。不同于普通的 Drawer 覆盖在页面上方，它是将主页面缩小并向右滑动，暴露出底部的菜单层。

**1. 侧边栏控制器 (hidden_drawer.dart)**



```Dart
import 'package:flutter/material.dart';
import 'package:hidden_drawer_menu/hidden_drawer_menu.dart';
import 'package:hidden_page/pages/home_page.dart';
import 'package:hidden_page/pages/settings_page.dart';

class HiddenDrawer extends StatefulWidget {
  const HiddenDrawer({super.key});

  @override
  State<HiddenDrawer> createState() => _HiddenDrawerState();
}

class _HiddenDrawerState extends State<HiddenDrawer> {
  // 1. 统一定义菜单文本的样式
  final myTextStyle = const TextStyle(
    color: Colors.white,
    fontSize: 18.0,
    fontWeight: FontWeight.bold
  );

  // 2. 声明页面和菜单配置的列表
  List<ScreenHiddenDrawer> _pages = [];

  @override
  void initState() {
    super.initState();
    // 3. 初始化列表：绑定菜单项 (ItemHiddenMenu) 和具体的页面 (Widget)
    _pages = [
      ScreenHiddenDrawer(
        ItemHiddenMenu(
          name: 'Homepage',
          baseStyle: myTextStyle,
          selectedStyle: myTextStyle, 
          colorLineSelected: Colors.deepPurple // 选中左侧的指示条颜色
        ), 
        const HomePage()
      ),
      ScreenHiddenDrawer(
        ItemHiddenMenu(
          name: 'Settings',
          baseStyle: myTextStyle,
          selectedStyle: myTextStyle,
          colorLineSelected: Colors.deepPurple
        ), 
        const SettingsPage()
      ),
    ];
  }
  
  @override
  Widget build(BuildContext context) {
    // 4. 构建 HiddenDrawerMenu 核心组件
    return HiddenDrawerMenu(
      screens: _pages,
      backgroundColorMenu: Colors.deepPurple.shade200, // 底部菜单层的背景色
      initPositionSelected: 0, // 默认选中第一个页面 (HomePage)
      slidePercent: 40.0, // 主页面滑开的比例 (40% 留在屏幕外)
      contentCornerRadius: 10, // 主页面滑开后的圆角效果
    );
  }
}
```

**2. 入口文件 (main.dart)**



```Dart
import 'package:flutter/material.dart';
import 'package:hidden_page/hidden_drawer.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.deepPurple,
        useMaterial3: false, // 禁用 Material 3 以保持某些旧版 UI 库的经典外观
      ),
      home: const HiddenDrawer(), // 将 HiddenDrawer 设为根页面
    );
  }
}
```

---

## 📝 2. 源码报告 (Source Code Report)

- **第三方库驱动 (Package Driven)**: 普通的 Flutter `Scaffold.drawer` 无法直接实现这种 3D/缩放的滑动效果。引入 `hidden_drawer_menu` 库极大减少了手写复杂动画 (`AnimationController`, `Transform.scale`, `Transform.translate`) 的工作量。
    
- **生命周期挂载 (Lifecycle Mounting)**: 在 `initState` 中初始化 `_pages` 列表是标准的做法。这保证了菜单数据在 Widget 树第一次构建之前就已经准备好，避免了在 `build` 方法中重复创建列表导致的性能浪费。
    
- **视觉错觉 (Visual Illusion)**: `HiddenDrawerMenu` 的本质是在一个 `Stack` 组件中，把"菜单页"放在最底层（深紫色背景），把"当前内容页"放在上层。当点击左上角按钮时，上层的内容页会执行缩放并向右侧平移的动画 (`slidePercent` 和 `contentCornerRadius` 参数控制了变形的程度)。
    

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. 结构配对映射

`ScreenHiddenDrawer` 是一个包装器，它强制你将两个东西绑定在一起：一个是左侧长什么样的菜单按钮 (`ItemHiddenMenu`)，另一个是点击这个按钮后右侧要显示什么页面 (比如 `HomePage`)。

### 2. UI 细节微调

- `slidePercent`: 决定主屏幕滑动后留给菜单的宽度。如果数值太大，主屏幕可能被挤出屏幕外；数值太小，菜单可能显示不全。
    
- `contentCornerRadius`: 给滑动后的主屏幕加上圆角，使其看起来像一张悬浮的卡片，这是增强 UI 质感的关键点。
    

### 3. Material 3 兼容性提醒

在 `main.dart` 中，作者刻意使用了 `useMaterial3: false`。这是因为许多早期的 Flutter UI 包是基于 Material 2 设计的。如果开启 M3，可能会导致 AppBar 颜色变浅、阴影消失或文本样式不符合预期。

---