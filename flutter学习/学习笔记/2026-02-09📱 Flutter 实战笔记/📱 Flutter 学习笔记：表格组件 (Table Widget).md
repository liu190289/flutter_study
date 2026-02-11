**日期**: 2026年2月9日

**项目状态**: ✅ 已完成 (100%)

**核心技能**: `Table`, `TableRow`, `FlexColumnWidth`, `Conditional Styling`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码展示了如何使用 `Table` 组件构建一个比例精确、且支持表头加粗样式的响应式数据表格。



```Dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(title: const Text('My Flutter App')),
        body: Center(
          child: Table(
            border: TableBorder.all(), // 绘制表格边框
            columnWidths: const {
              0: FlexColumnWidth(0.5), // 第一列占 50%
              1: FlexColumnWidth(0.2), // 第二列占 20%
              2: FlexColumnWidth(0.3), // 第三列占 30%
            },
            children: [
              buildRow(['Name', 'City', 'Age'], isHeader: true),
              buildRow(['Emma', 'Sydney', '24']),
              buildRow(['John', 'London', '22']),
            ],
          ),
        ),
      ),
    );
  }

  // 自定义行生成函数
  TableRow buildRow(List<String> cells, {bool isHeader = false}) => TableRow(
    children: cells.map((cell) {
      final style = TextStyle(
        fontSize: 18,
        fontWeight: isHeader ? FontWeight.bold : FontWeight.normal,
      );
      return Padding(
        padding: const EdgeInsets.all(8.0),
        child: Center(child: Text(cell, style: style)),
      );
    }).toList(),
  );
}
```

---

## 📝 2. 源码报告 (Source Code Report)

- **比例化布局 (Proportional Layout)**: 通过 `FlexColumnWidth` 属性精确定义了每列的权重（0.5, 0.2, 0.3）。这种响应式设计确保了表格在不同分辨率的屏幕上都能保持稳定的列宽比例。
    
- **工厂化构建 (Method Factorization)**: 核心在于 `buildRow` 辅助方法。它将原始字符串列表转化为 `TableRow` 对象，极大地减少了 `build` 方法中的重复代码，提高了代码的可维护性。
    
- **条件渲染逻辑 (Conditional Rendering)**: 源码巧妙运用了可选参数 `{bool isHeader = false}` 和三元运算符。这使得同一套构建逻辑能够根据传入标识自动切换“表头加粗”或“普通内容”样式。
    
- **层级嵌套规范 (Component Nesting)**: 单元格通过 `Padding` 控制呼吸感，配合 `Center` 实现内容的双向居中，构建了符合专业 UI 标准的数据展示格。
    

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. 为什么选择 Table 而不是 Column/Row?

虽然 `Column` 套 `Row` 也能做出表格的样子，但 `Table` 组件能确保：

- **列对齐**: 每一行的第一列宽度严格一致，不会因为文字长短不一而对不齐。
    
- **性能优化**: `Table` 在渲染大量网格数据时比嵌套 Row 更加高效。
    

### 2. FlexColumnWidth 的计算方式

它是一种相对比例单位。总宽度等于所有 Flex 值之和。例如 $0.5 + 0.2 + 0.3 = 1.0$，则第一列占据总宽度的 $\frac{0.5}{1.0} = 50\%$。

### 3. 数据到 UI 的映射 (.map)

使用 `cells.map(...).toList()` 是 Flutter 开发中的高阶技巧。它建立了一条“流水线”，将数据源（String）批量加工为 UI 组件（Widget），是处理动态列表的核心思想。