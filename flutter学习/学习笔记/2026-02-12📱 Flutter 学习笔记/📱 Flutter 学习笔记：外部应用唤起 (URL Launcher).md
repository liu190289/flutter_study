#外部应用唤起 (URL Launcher)

**日期**: 2026年2月12日

**项目状态**: ✅ 已完成 (100%)

**核心技能**: `url_launcher`, `URI Schemes`, `Asynchronous Programming`, `Android Manifest`

---

## 📂 1. 原始源码记录 (Source Code)

这段代码展示了如何使用 `url_launcher` 插件，通过特定的协议头（Scheme）打开手机系统自带的短信应用。

**主要逻辑代码 (main.dart)**



```Dart
import 'package:flutter/material.dart';
import 'package:url_launcher/url_launcher.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  final String _phoneNumber = '+61426502918'; // 目标电话号码

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(
          child: MaterialButton(
            color: Colors.blue,
            // 1. 异步操作：唤起外部应用是一个耗时操作，需要 async/await
            onPressed: () async {
              // 2. URI 拼接：注意 'sms:' 后面不能有空格
              final _text = 'sms:$_phoneNumber'; 
              final Uri uri = Uri.parse(_text);

              // 3. 检查与执行：先检查系统能否处理该协议，再执行跳转
              if (await canLaunchUrl(uri)) {
                await launchUrl(uri);
              } else {
                // 建议：实际开发中这里应该加一个错误提示
                debugPrint("无法打开短信应用");
              }
            },
            child: const Text(
              "Text me",
              style: TextStyle(color: Colors.white),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 📝 2. 源码报告 (Source Code Report)

- **URI Scheme 机制**: 代码的核心在于字符串 `'sms:$_phoneNumber'`。Android 和 iOS 系统通过“协议头”（Scheme）来识别意图：
    
    - `sms:` -> 唤起短信
        
    - `tel:` -> 唤起拨号盘
        
    - `https:` -> 唤起浏览器
        
    - `mailto:` -> 唤起邮件 **注意**：代码中修正了之前提到的空格错误（`sms: ...` 是错的，必须是 `sms:...`），这是 URI 解析成功的关键。
        
- **安全检查 (`canLaunchUrl`)**: 在执行跳转前，代码使用了 `canLaunchUrl(uri)`。这是一个安全卫士，它会询问操作系统：“你有安装能处理 `sms:` 协议的 App 吗？”如果返回 `true`，才会执行跳转。这防止了在没有短信功能的设备（如某些纯 WiFi 平板）上导致 App 崩溃。
    
- **异步流程控制**: 因为与操作系统交互需要跨进程通信（IPC），这是一个异步过程。所以 `onPressed` 被标记为 `async`，并且必须使用 `await` 等待操作系统的反馈结果。
    

---

## 🔑 3. 核心知识点总结 (Key Takeaways)

### 1. 格式极其严格 (Strict Syntax)

URI Scheme 的格式容错率极低。

- ✅ 正确: `tel:+123456`
    
- ❌ 错误: `tel: +123456` (多了一个空格) 任何多余的空格或特殊字符都可能导致 `Uri.parse` 失败或系统无法识别。
    

### 2. Android 的隐性要求 (Android 11+ Queries)

虽然这段 Dart 代码是对的，但在 **Android 11 (API 30)** 及以上版本，出于隐私保护，Google 限制了 App 查询其他应用的能力。 **必须配置**: 需要在 `android/app/src/main/AndroidManifest.xml` 中添加 `<queries>` 标签，声明你需要使用的 `sms` 或 `tel` 协议，否则 `canLaunchUrl` 永远返回 `false`。

### 3. `canLaunch` vs 直接 `launch`

- **严谨做法**: 先 `canLaunchUrl` 再 `launchUrl`（如代码所示）。
    
- **偷懒做法**: 直接 `launchUrl` 并用 `try-catch` 捕获异常。 在处理电话和短信时，现代开发中常推荐直接尝试 launch 并捕获异常，因为某些定制 Android 系统对 `canLaunch` 的支持可能不稳定。