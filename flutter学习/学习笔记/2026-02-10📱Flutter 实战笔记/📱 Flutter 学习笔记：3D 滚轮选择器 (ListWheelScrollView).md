**日期**: 2026年2月10日 
**项目状态**: ✅ 已完成 (100%) 
**核心技能**: `ListWheelScrollView`, `FixedExtentScrollPhysics`, `childDelegate`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码实现了一个具有 3D 透视效果的时间选择器，模仿了 iOS 的滚轮风格。它包含三列：小时、分钟和 AM/PM。



```Dart
import 'package:flutter/material.dart';
import 'package:timescrolling/am_pm.dart';
import 'package:timescrolling/hours.dart';
import 'package:timescrolling/minutes.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  late FixedExtentScrollController hourController;

  @override
  void initState() {
    super.initState();
    hourController = FixedExtentScrollController();
  }

  @override
  void dispose() {
    hourController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          // 1. 小时滚轮 (Hours Wheel)
          SizedBox(
            width: 70,
            child: ListWheelScrollView.useDelegate(
              controller: hourController,
              itemExtent: 50, // 每个子组件的高度
              perspective: 0.005, // 3D 透视效果，0 为平面
              diameterRatio: 1.2, // 圆柱体直径比例，越小弯曲越明显
              physics: const FixedExtentScrollPhysics(), // 磁性吸附效果
              childDelegate: ListWheelChildBuilderDelegate(
                childCount: 13,
                builder: (context, index) {
                  return MyHours(hours: index);
                },
              ),
            ),
          ),

          const SizedBox(width: 10),

          // 2. 分钟滚轮 (Minutes Wheel)
          SizedBox(
            width: 70,
            child: ListWheelScrollView.useDelegate(
              itemExtent: 50,
              perspective: 0.005,
              diameterRatio: 1.2,
              physics: const FixedExtentScrollPhysics(),
              childDelegate: ListWheelChildBuilderDelegate(
                childCount: 60, // 通常分钟是60个
                builder: (context, index) {
                  return MyMinutes(mins: index);
                },
              ),
            ),
          ),

          const SizedBox(width: 10),

          // 3. 上午/下午滚轮 (AM/PM Wheel)
          SizedBox(
            width: 70,
            child: ListWheelScrollView.useDelegate(
              itemExtent: 50,
              perspective: 0.005,
              diameterRatio: 1.2,
              physics: const FixedExtentScrollPhysics(),
              childDelegate: ListWheelChildBuilderDelegate(
                childCount: 2,
                builder: (context, index) {
                  if (index == 0) {
                    return const AmPm(isItAm: true);
                  } else {
                    return const AmPm(isItAm: false);
                  }
                },
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 📝 2. 源码报告 (Source Code Report)

- **3D 视觉效果 (Visual Physics)**: `ListWheelScrollView` 是核心组件。通过 `perspective` (透视度) 和 `diameterRatio` (直径比) 参数，可以将普通的列表渲染成圆柱体表面的旋转效果。
    
- **磁性滚动 (Snapping Physics)**: `FixedExtentScrollPhysics` 是实现"选择器"手感的关键。普通的滚动视图可以停在任何位置（比如两个数字中间），但加上这个属性后，滚动停止时会自动"吸附"并对齐到最近的一个选项上，保证用户选中的是一个确定的值。
    
- **性能优化 (Performance)**: 使用了 `.useDelegate` 构造函数配合 `ListWheelChildBuilderDelegate`。这相当于 `ListView.builder`，是懒加载模式。它只渲染当前屏幕可见的几个数字，而不是一次性渲染所有选项，这对内存非常友好。
    

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. itemExtent 的重要性

在 `ListWheelScrollView` 中，`itemExtent` 是必须提供的参数。它强制所有子组件（例如 `MyHours`）具有固定的高度（这里是 50）。如果子组件实际高度不匹配，可能会导致布局重叠或间距异常。

### 2. Controller 的作用

`FixedExtentScrollController` 不仅能控制滚动的初始位置（例如打开时默认显示当前时间），还可以通过 `controller.selectedItem` 获取当前选中的索引值。如果不绑定 controller，很难获取用户到底选了哪个数字。

### 3. 视觉参数微调

- **perspective**: 范围通常在 0.001 到 0.01 之间。太大会导致变形严重。
    
- **diameterRatio**: 默认是 2.0。值越小，圆柱体越细，滚动的弧度看起来越弯曲；值越大，看起来越像平面列表。