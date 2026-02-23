
---

**日期**: 2026年2月23日  
**项目状态**: ✅ 已完成 (100%)  
**核心技能**: `Hive` (本地存储), `Dataset Heatmap` (热力图), `Slidable` (侧滑交互), `CRUD Operations`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码展示了如何使用 **Hive** 进行本地数据持久化，并结合 **Heatmap** 组件可视化展示用户的习惯打卡记录。核心逻辑在于 `HabitDatabase` 类对数据的增删改查管理，以及主页面的状态同步。

### 1.1 数据管理层 (HabitDatabase)
封装了 Hive 的操作，处理当天的习惯列表加载与热力图数据更新。

```dart
import 'package:hive_flutter/hive_flutter.dart';
import 'package:habit_tracker_app/datetime/date_time.dart';

// 引用全局 Box
final _myBox = Hive.box('Habit_Database');

class HabitDatabase {
  List todayHabitList = [];
  Map<DateTime, int> heatMapDataSet = {};

  // 1.首次启动初始化数据
  void createInitialData() {
    todayHabitList = [
      ['Run', false],
      ['Read Book', false],
    ];
    _myBox.put("START_DATE", todaysDateFormatted());
  }

  // 2.加载数据（处理跨天逻辑）
  void loadData() {
    // 如果今天是新的一天（数据库里没今天的记录）
    if (_myBox.get(todaysDateFormatted()) == null) {
      // 获取"通用"的习惯列表
      todayHabitList = _myBox.get("CURRENT_HABIT_LIST");
      // 重置所有状态为未完成
      for (int i = 0; i < todayHabitList.length; i++) {
        todayHabitList[i][1] = false;
      }
    } else {
      // 否则加载今天的特定状态
      todayHabitList = _myBox.get(todaysDateFormatted());
    }
  }

  // 3.更新数据库
  void updateDatabase() {
    // 更新今天的记录
    _myBox.put(todaysDateFormatted(), todayHabitList);
    // 更新通用的习惯模板列表
    _myBox.put("CURRENT_HABIT_LIST", todayHabitList);
    
    // 计算并更新热力图数据
    calculateHabitPercentages();
    loadHeatMap();
  }
}
```

### 1.2 UI 交互层 (HomePage)
使用 `Slidable` 实现侧滑删除/编辑，并展示 `HeatMap`。

```dart
// ... imports omitted ...

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  HabitDatabase db = HabitDatabase();
  final _myBox = Hive.box('Habit_Database');

  @override
  void initState() {
    // 检查是否第一次打开 App
    if (_myBox.get("CURRENT_HABIT_LIST") == null) {
      db.createInitialData();
    } else {
      db.loadData();
    }
    db.updateDatabase();
    super.initState();
  }

  // 勾选习惯
  void checkBoxTapped(bool? value, int index) {
    setState(() {
      db.todayHabitList[index][1] = value;
    });
    db.updateDatabase();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey[300],
      floatingActionButton: MyFloatingActionButton(onPressed: createNewHabit),
      body: ListView(
        children: [
          // 1. 月度热力图
          MonthlySummary(
            datasets: db.heatMapDataSet,
            startDate: _myBox.get("START_DATE"),
          ),

          // 2. 习惯列表
          ListView.builder(
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            itemCount: db.todayHabitList.length,
            itemBuilder: (context, index) {
              return HabitTile(
                habitName: db.todayHabitList[index][0],
                habitCompleted: db.todayHabitList[index][1],
                onChanged: (value) => checkBoxTapped(value, index),
                settingsTapped: (context) => openHabitSettings(index),
                deleteTapped: (context) => deleteHabit(index),
              );
            },
          )
        ],
      ),
    );
  }
}
```

---

## 📝 2. 源码报告 (Source Code Report)

- **本地持久化 (Local Persistence)**: 采用了轻量级的 NoSQL 数据库 `Hive`。它不需要复杂的 SQL 语句，直接以 `Key-Value` 的形式存储 List 和 Map，非常适合这种记录用户每日状态的小型应用。
    
- **跨日逻辑 (Date Logic)**: `HabitDatabase` 中的 `loadData` 方法巧妙地解决了“新的一天”问题。它会检查当天日期是否已在数据库中成为 Key。如果没有，说明跨天了，它会加载“习惯模板”并将所有完成状态重置为 `false`，从而实现每日任务刷新。

- **可视化反馈 (Visual Feedback)**: 引入了 `flutter_heatmap_calendar`。通过计算每日完成度的百分比（0.0 - 1.0），生成不同深浅的绿色方块，直观地展示用户习惯养成的连续性。

- **手势交互 (Gesture Interaction)**: 使用 `flutter_slidable` 包裹每个 `HabitTile`。这允许用户向左滑动列表项来显示隐藏的“设置”和“删除”按钮，保持了 UI 的整洁性，避免了界面上堆积过多的操作图标。

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. Hive 的初始化与 Box 管理
在 main.dart 中，必须先执行 `await Hive.initFlutter();` 初始化环境，然后打开通过 `openBox` 打开一个盒子。这个盒子就像一个文件，随处可以引用。
*   **注意**: 所有的存储操作（put/get）都是同步的（在内存中），但 Hive 会异步写入磁盘。对于 UI 渲染来说速度极快。

### 2. `ListView` 嵌套 `ListView`
在 `HomePage` 中，外部是一个垂直滚动的 `ListView`，内部包含了热力图和另一个 `ListView.builder` (用于展示习惯列表)。
*   **关键设置**: 内部的 `ListView.builder` 必须设置 `shrinkWrap: true` 和 `physics: const NeverScrollableScrollPhysics()`。
    *   `shrinkWrap: true`: 让列表高度由内容决定，而不是无限延伸。
    *   `NeverScrollableScrollPhysics`: 禁用内部滚动，让所有手势事件由最外层的 `ListView` 统一接管，防止滑动冲突。

### 3. 数据集结构 (Datasets)
热力图组件 `HeatMap` 需要的数据格式是 `Map<DateTime, int>`。
*   **DateTime**: 代表具体的日期。
*   **int**: 代表当天的“强度”或“完成度”。
*   在本项目中，我们需要编写算法遍历每一天的习惯完成情况，计算出完成百分比（例如 10 个习惯完成了 5 个，强度就是 5），然后存入这个 Map 中供热力图渲染。