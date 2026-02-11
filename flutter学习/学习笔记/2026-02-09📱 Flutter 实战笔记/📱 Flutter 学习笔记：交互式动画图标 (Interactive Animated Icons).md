**日期**: 2026年2月9日

**核心概念**: `AnimationController`, `AnimatedIcon`, `状态切换`

---

## 📂 1. 原始源码 (Source Code)



```Dart
import 'package:flutter/material.dart';

class Homepage extends StatefulWidget {
  const Homepage({super.key});

  @override
  State<Homepage> createState() => _HomepageState();
}

class _HomepageState extends State<Homepage> with SingleTickerProviderStateMixin {
  // 1. 声明控制器
  late AnimationController _animatedController;
  bool videoPlaying = false;

  @override
  void initState() {
    super.initState();
    // 2. 初始化动画时长（1秒）
    _animatedController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
  }

  // 3. 编写点击切换逻辑
  void _iconTapped() {
    if (videoPlaying == false) {
      _animatedController.forward(); // 播放动画
      videoPlaying = true;
    } else {
      _animatedController.reverse(); // 倒放动画
      videoPlaying = false;
    }
  }



  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: GestureDetector(
          onTap: _iconTapped,
          child: AnimatedIcon(
            icon: AnimatedIcons.search_ellipsis, 
            progress: _animatedController,
            size: 120,
          ),
        ),
      ),
    );
  }
}
```

---

## 📝 2. 核心总结 (Quick Report)

### **学到了什么？**

- **动画时长控制**：通过 `Duration` 设置 `seconds` 或 `milliseconds` 来精确控制图标变形的快慢。
    
- **双向播放**：学会了使用 `.forward()` 启动动画以及 `.reverse()` 让动画倒退回初始状态。
    
- **生命周期管理**：理解了 `initState` 用于创建资源，`dispose` 用于在页面销毁时释放资源的重要性。
    
- **Ticker 机制**：掌握了 `SingleTickerProviderStateMixin` 的用法，它能让动画与屏幕刷新率保持同步，防止掉帧。
    

### **解决了什么？**

- **速度失控问题**：通过调整 `Duration` 解决了之前动画一闪而过（太快）导致看不清变形过程的问题。
    
- **交互逻辑断层**：通过布尔变量 `videoPlaying` 解决了点击图标后无法在两个状态之间自由切换的逻辑漏洞。
