#交互式折叠列表
**日期**: 2026年2月12日

**项目状态**: ✅ 已完成 (100%)

**核心技能**: `ExpansionPanelList`, `State Management`, `List.generate`, `map`, `Callback Logic`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码展示了如何创建一个动态列表，支持单独展开/收起每一项，并实现了点击图标删除特定数据的功能。

**1. 数据模型与程序入口 (main.dart)**


```Dart

import 'package:flutter/material.dart';

// 1. 数据模型：定义列表项的结构
class Item {
  String headerText;    // 标题
  String expandedText;  // 展开后的详情
  bool isExpanded;      // 核心状态：当前是开还是关

  Item({
    required this.headerText,
    required this.expandedText,
    this.isExpanded = false, // 默认收起
  });
}

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: HomePage(),
  ));
}
```

**2. 页面逻辑与构建 (home_page.dart / HomePage)**



```Dart
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // 2. Mock Data：使用 List.generate 自动生成 10 条测试数据
  final List<Item> _data = List.generate(10, (index) {
    return Item(
      headerText: 'Item $index',
      expandedText: '这是第 $index 号面板的详细内容...\n你可以点击这里查看更多信息。',
    );
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Expansion Panel List')),
      // 3. 滚动视图：防止展开后内容超出屏幕导致溢出错误
      body: SingleChildScrollView(
        child: Container(
          padding: const EdgeInsets.all(10.0),
          child: _buildPanel(),
        ),
      ),
    );
  }

  Widget _buildPanel() {
    return ExpansionPanelList(
      // 4. 回调函数：处理点击折叠按钮的逻辑
      expansionCallback: (int index, bool isExpanded) {
        setState(() {
          // 将当前点击的 Item 状态取反 (开变关，关变开)
          _data[index].isExpanded = !_data[index].isExpanded;
        });
      },
      // 5. 数据映射：将 Item 数据列表转换为 UI 组件列表
      children: _data.map<ExpansionPanel>((Item item) {
        return ExpansionPanel(
          // 关键属性：允许点击标题区域展开，提升用户体验
          canTapOnHeader: true,
          
          headerBuilder: (BuildContext context, bool isExpanded) {
            return ListTile(
              title: Text(item.headerText, style: const TextStyle(fontWeight: FontWeight.bold)),
            );
          },
          body: ListTile(
            title: Text(item.expandedText),
            subtitle: const Text('点击右侧垃圾桶删除我 ->'),
            trailing: const Icon(Icons.delete, color: Colors.red),
            // 6. 删除逻辑
            onTap: () {
              setState(() {
                // 从列表中移除当前数据，UI 会自动更新
                _data.removeWhere((currentItem) => item == currentItem);
              });
            },
          ),
          isExpanded: item.isExpanded, // 绑定每一项的状态
        );
      }).toList(),
    );
  }
}
```

---

## 📝 2. 源码报告 (Source Code Report)

- **数据驱动 UI (Data-Driven UI)**: 我们没有手动写 10 个 `ExpansionPanel`，而是维护了一个 `List<Item>` 数据源。UI 是数据的“投影”。
    
    - 当我们修改 `_data` 中的 `isExpanded` 或删除元素并调用 `setState` 时，Flutter 会自动重新构建 `ExpansionPanelList`，从而更新界面。
        
- **状态隔离 (State Isolation)**: 这里的关键是 `Item` 类中的 `isExpanded` 字段。
    
    - **痛点**: 如果只用一个全局变量 `bool isExpanded`，那么点开一个，所有的面板都会同时打开。
        
    - **解决**: 将状态封装在每个 `Item` 对象内部，确保每个面板的开关是独立的。
        
- **交互体验优化 (UX Improvement)**:
    
    - **`canTapOnHeader: true`**: 默认情况下，用户必须精准点击右侧的小箭头才能展开。加上这个属性后，点击整行标题都能触发回调，极大地提升了操作便利性。
        
    - **`SingleChildScrollView`**: `ExpansionPanelList` 高度是不固定的。展开所有面板后极易超出屏幕高度，必须包裹在可滚动的 Widget 中，否则会报 `RenderFlex overflowed` 错误。
        

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. `List.generate` 与 Mock Data

在开发初期，手动写死数据（Hardcode）效率很低。`List.generate(count, (index) => ...)` 是快速生成测试数据的神器，能模拟真实环境下的多条数据场景。

### 2. `map` 方法的威力

Flutter 开发中极少使用 `for` 循环来构建 Widget 列表。

- **模式**: `List<Data>` -> `.map()` -> `List<Widget>`。
    
- 我们将数据对象转换成界面组件，最后记得调用 `.toList()`，因为 `children` 属性需要的是 List 类型。
    

### 3. 删除逻辑 `removeWhere`

相比于通过 `index` 删除（容易因为列表长度变化导致越界错误），`removeWhere` 更加安全和语义化。它直接告诉列表：“把在这个条件下的那个元素删掉”，不需要关心它现在的索引是多少。

### 4. ExpansionCallback 的逻辑

点击面板时，`expansionCallback` 会被触发。在这个回调中，我们必须调用 `setState` 来更新数据模型。

- **逻辑公式**: `Model.isExpanded = !Model.isExpanded` (取反操作)。