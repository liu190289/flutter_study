**日期**: 2026年2月6日 **项目状态**: ✅ 已完成 (100%) **核心技能**: `ListView.builder`, `GridDelegate`, `Custom Widgets`, `Expanded`

---

## 📂 1. 项目结构 (Project Structure)

我们将代码拆分为三个文件，以保持代码整洁和模块化：

1. **`lib/main.dart`**: 程序入口，负责整体布局（竖向排列的故事栏和帖子流）。
    
2. **`lib/circle.dart`**: 封装好的圆形组件（用于 Stories）。
    
3. **`lib/square.dart`**: 封装好的方形组件（用于 Posts）。
    

---

## 📝 2. 代码文件详解

### A. 主入口文件 (`lib/main.dart`)

这是 App 的骨架。我们使用了 `Column` 来垂直排列“Stories 区域”和“Posts 区域”。


```Dart
import 'package:flutter/material.dart';
import 'circle.dart'; // 导入自定义圆形组件
import 'square.dart'; // 导入自定义方形组件

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  // 模拟数据源
  static const List posts = ["post1", "post2", "post3", "post4", "post5", "post6"];
  static const List storys = ["story1", "story2", "story3", "story4", "story5", "story6"];

  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Column(
          children: [
            // 1. Instagram Stories (横向滚动区域)
            Container(
              height: 150, // 限制高度，否则 ListView 会报错
              child: ListView.builder(
                scrollDirection: Axis.horizontal, // 关键：设置为水平滚动
                itemCount: storys.length,
                itemBuilder: (context, index) {
                  return Mycircle(
                    child: storys[index], // 传递数据给子组件
                  );
                },
              ),
            ),

            // 2. Instagram Posts (纵向滚动列表)
            Expanded(
              // Expanded 占据剩余的所有空间
              child: ListView.builder(
                itemCount: posts.length,
                itemBuilder: (context, index) {
                  return Mysquare(
                    child: posts[index], // 传递数据给子组件
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```
### B. 圆形组件 (`lib/circle.dart`)

用于显示类似 Instagram 快拍 (Stories) 的圆形头像。


```Dart
import 'package:flutter/material.dart';

class Mycircle extends StatelessWidget {
  final String child; // 接收传进来的文字

  Mycircle({required this.child});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Container(
        height: 100,
        width: 100,
        decoration: BoxDecoration(
          shape: BoxShape.circle, // 形状：圆形
          color: Colors.pink, // 背景色
        ),
        child: Center(
          child: Text(
            child,
            style: TextStyle(fontSize: 20, color: Colors.white),
          ),
        ),
      ),
    );
  }
}
```
### C. 方形组件 (`lib/square.dart`)

用于显示 Feed 流中的帖子内容。


```Dart
import 'package:flutter/material.dart';

class Mysquare extends StatelessWidget {
  final String child; // 接收传进来的文字

  Mysquare({required this.child});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8.0), // 垂直间距
      child: Container(
        height: 200,
        width: 200, // 注意：在 ListView 中通常宽度会自适应充满屏幕
        color: Colors.deepPurple[200], // 浅紫色背景
        child: Center(
          child: Text(
            child,
            style: TextStyle(fontSize: 40),
          ),
        ),
      ),
    );
  }
}
```
## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. 为什么需要 `Expanded`?

在 `Column` 中放 `ListView` 是一个经典的坑。

- `Column` 希望高度无限延伸。
    
- `ListView` 也希望高度无限延伸。
    
- **结果**: 冲突报错。
    
- **解决**: 使用 `Expanded` 包裹 `ListView`，告诉它：“别抢了，剩下的空间都给你，但不能超过屏幕。”
    

### 2. 横向滚动 (Horizontal Scrolling)

- 默认 `ListView` 是垂直滚动的。
    
- 设置 `scrollDirection: Axis.horizontal` 即可变为横向。
    
- **注意**: 横向 ListView 通常需要包裹在一个有固定高度 (`height`) 的 `Container` 里。
    

### 3. 组件封装 (Component Extraction)

- 将 `Mycircle` 和 `Mysquare` 提取到单独的文件中，使用 `import` 导入。
    
- **优点**: 提高代码可读性，方便复用。
    
- **数据传递**: 使用构造函数 `Mycircle({required this.child})` 将数据从父组件传给子组件。


---
# 📱 Flutter 学习笔记：用户输入与状态管理 (Text Input)

**日期**: 2026年2月6日
**核心概念**: `StatefulWidget`, `TextField`, `TextEditingController`, `setState`

---

## 📂 1. 代码实例 (`Homepage`)

本节重点在于如何获取用户输入的内容，并将其显示在屏幕上。

```dart
import 'package:flutter/material.dart';

// 1. 必须使用 StatefulWidget，因为界面内容会发生改变
class Homepage extends StatefulWidget {
  const Homepage({Key? key}) : super(key: key);

  @override
  _HomepageState createState() => _HomepageState();
}

class _HomepageState extends State<Homepage> {
  // 2. 控制器：相当于 TextField 的“遥控器”
  final _textController = TextEditingController();

  // 3. 变量：用来存储并显示在屏幕上的文字
  String userPost = "";

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.end, // 按钮靠右
          mainAxisAlignment: MainAxisAlignment.center, // 整体垂直居中
          children: [
            // --- 显示区域 ---
            Expanded(
              child: Container(
                color: Colors.deepPurple[200],
                child: Center(
                  child: Text(
                    userPost, // 显示变量中的内容
                    style: TextStyle(fontSize: 35, color: Colors.white),
                  ),
                ),
              ),
            ),

            // --- 输入区域 ---
            TextField(
              controller: _textController, // 绑定控制器
              decoration: InputDecoration(
                border: const OutlineInputBorder(), // 边框
                hintText: 'What\'s on your mind?', // 提示文字
                // 尾部图标：点击清除文字
                suffixIcon: IconButton(
                  onPressed: () {
                    _textController.clear(); // 调用控制器清除输入
                  },
                  icon: const Icon(Icons.clear),
                ),
              ),
            ),

            // --- 按钮区域 ---
            MaterialButton(
              onPressed: () {
                // 4. 核心：使用 setState 更新状态
                setState(() {
                  userPost = _textController.text; // 把输入框的字赋值给显示变量
                  _textController.clear(); // 发布后清空输入框
                });
              },
              color: Colors.blue,
              child: const Text(
                "Post",
                style: TextStyle(color: Colors.white),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```
## 📝 2. 关键知识点解析

### A. StatefulWidget vs StatelessWidget

- **之前**: 我们一直用 `StatelessWidget`，因为界面画好就不动了。
    
- **现在**: 我们需要用 **`StatefulWidget`**。
    
- **原因**: 用户输入文字后，屏幕上显示的文字需要“变”。只要界面需要**动态更新**，就必须用 Stateful。
    
---
### B. TextEditingController (文本控制器)

- **作用**: 它就像 `TextField` 的大脑或遥控器。
    
- **获取文字**: `_textController.text` 可以拿到用户当前输入的字符串。
    
- **清空文字**: `_textController.clear()` 可以把输入框清空。
    

### C. setState(() {}) (刷新界面)

这是 Flutter 中最重要的函数之一。

- **代码逻辑**:

``` Dart
setState(() {
   userPost = _textController.text;
});
```
- **原理**: 当你调用 `setState` 时，你是在告诉 Flutter：“嘿，数据变了（`userPost` 变了），请**重新运行 `build` 函数**，把界面重新画一遍！”
    
- **如果不写 setState**: 变量 `userPost` 的值确实会变，但屏幕上显示的字**不会变**，因为界面没有刷新。


---
### D. Expanded

- 在 `Column` 中使用 `Expanded` 包裹显示区域的 `Container`，是为了让它占据屏幕上方**剩余的所有空间**，把输入框和按钮挤到底部。

---

## 🚀 3. 交互流程总结

1. 用户在 `TextField` 打字 -> 文字暂存在 `_textController` 里。
    
2. 用户点击 **Post** 按钮。
    
3. 触发 `onPressed` 事件 -> 执行 `setState`。
    
4. **第一步**: 将 `_textController.text` (输入的内容) 赋值给 `userPost` (显示的变量)。
    
5. **第二步**: 调用 `_textController.clear()` 清空输入框。
    
6. **结果**: 界面刷新，紫色方块里出现了刚才打的字，输入框变空了。



---


# 📱 Flutter 学习笔记：弹出对话框 (Alert Dialog)

**日期**: 2026年2月6日
**核心概念**: `showDialog`, `AlertDialog`, `Navigator.pop`

---

## 📂 1. 代码实例 (`Homepage`)

本节重点在于如何创建一个模态对话框（用户必须处理后才能回到主界面的弹窗）。

```dart
import 'package:flutter/material.dart';

class Homepage extends StatefulWidget {
  const Homepage({Key? key}) : super(key: key);

  @override
  _HomepageState createState() => _HomepageState();
}

class _HomepageState extends State<Homepage> {
  
  // 1. 定义显示对话框的函数
  void _showDialog() {
    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          backgroundColor: Colors.pink[300], // 自定义背景色
          title: Text("Title"), // 标题
          content: Text("This is the content of the dialog box."), // 正文内容
          actions: [
            // --- 按钮 1: 确认 ---
            MaterialButton(
              onPressed: () {
                // 这里可以写确认后的逻辑，比如保存数据
              },
              child: Text("Ok"),
            ),
            // --- 按钮 2: 取消 ---
            MaterialButton(
              onPressed: () {
                // 2. 核心：关闭对话框
                Navigator.pop(context); 
              },
              child: Text("Cancel"),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.deepPurple[200],
      body: Center(
        child: MaterialButton(
          color: Colors.deepPurple[100],
          onPressed: _showDialog, // 点击按钮调用函数
          child: Padding(
            padding: const EdgeInsets.all(15.0),
            child: Text(
              "SHOW DIALOG",
              style: TextStyle(fontSize: 30),
            ),
          ),
        ),
      ),
    );
  }
}
```
## 📝 2. 关键知识点解析

### A. `showDialog` 函数

- 这是 Flutter 提供的内置函数，用来在当前页面之上覆盖一层“蒙版”并显示内容。
    
- **`context`**: 必须传入上下文，告诉 Flutter 这个弹窗属于哪个页面。
    
- **`builder`**: 这是一个构建器，你需要在这里返回你想显示的组件（通常是 `AlertDialog`）。
    
---
### B. `AlertDialog` 组件

这是一个标准的 Material Design 对话框结构，包含三个主要部分：

1. **`title`**: 顶部的粗体标题。
    
2. **`content`**: 中间的详细信息或正文。
    
3. **`actions`**:底部的操作按钮列表（如“确定”、“取消”）。
    
---
### C. 关闭对话框 (`Navigator.pop`)

- 对话框本质上是被“推”入导航栈的一个新页面。
    
- 要关闭它，我们使用 `Navigator.pop(context)`。
    
- 这行代码的意思是：“把当前最上面的这个页面（也就是对话框）移除掉，回到前一个页面。”