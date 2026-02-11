
**日期**: 2026年2月9日

**核心概念**: `PageView`, `scrollDirection`, `Multi-file Structure`

---

## 📂 1. 原始源码 (Source Code)

此代码展示了如何利用 `PageView` 实现类似 TikTok 的垂直翻页效果，并采用了多文件导入的工程化写法。


```Dart
import 'package:flutter/material.dart';
import 'package:pageviwe/posts/post_2.dart';
import 'package:pageviwe/posts/post_1.dart';
import 'package:pageviwe/posts/post_3.dart';

class Homepage extends StatelessWidget {
  final _controller = PageController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageView(
        controller: _controller,
        scrollDirection: Axis.vertical, // 设置为垂直滚动
        children: [
          MyPost1(),
          MyPost2(),
          MyPost3(),
        ],
      ),
    );
  }
}
```

---

## 📝 2. 核心总结 (Quick Report)

### **学到了什么？**

- **滚动轴控制**：掌握了通过 `scrollDirection: Axis.vertical` 将默认的水平滑动更改为垂直滑动的方法。
    
- **项目解耦**：学会了将不同页面的 UI 分散到 `posts/` 文件夹下的独立文件中，通过 `import` 进行整合，使主文件逻辑保持简洁。
    
- **控制器应用**：了解了 `PageController` 的基本挂载方式，为后续实现自动跳转或监听滑动进度打下基础。
    

### **解决了什么？**

- **代码臃肿问题**：通过引入 `MyPost1()` 等自定义组件，解决了所有页面代码堆叠在一个文件中的混乱状态。
    
- **全屏交互实现**：利用 `PageView` 的特性，实现了页面之间“一滑一停”的沉浸式切换体验。
