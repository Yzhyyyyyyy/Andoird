# 🚀 Compose 学习笔记07 - 第七课（系统交互：Context、Intent 与异常兜底）

在前面的课程中，我们学习的都是如何“画 UI”（比如用 Canvas 画光晕，用 Column 排版）。
但在 `help.kt` 界面中，我们遇到了一个新需求：**点击按钮，拉起手机里的邮件 App**。这已经超出了纯 UI 的范畴，我们需要和 Android 系统底层打交道了！

---

## 新技能一：LocalContext.current (万能通行证)

在 Compose 中，UI 组件只是“图纸”，它们本身没有权力去调用系统的弹窗或拉起其他 App。要想做这些事，必须拿到 Android 系统的**“万能通行证” —— `Context`（上下文）**。

**获取方式极其简单：**
在 `@Composable` 函数的开头，加上这句咒语：
```kotlin
val context = LocalContext.current
```
有了这个 `context`，你就可以在 Compose 里为所欲为了（比如发广播、查电量、弹原生提示等）。

---

## 新技能二：Intent (跨应用任务委派)

怎么让我们的 App 去打开别人家的 App（比如 QQ 邮箱）？
在 Android 中，这需要通过发送 **`Intent`（意图/公函）** 来实现。

### 💡 “发公函”的比喻
1.  **写公函标题 (`Intent.ACTION_SENDTO`)**：告诉系统，我要执行“发送”任务。
2.  **写收件人 (`data = Uri.parse("mailto:...")`)**：明确这是一个发邮件的任务，并填上目标邮箱。
3.  **写邮件主题 (`putExtra`)**：顺便把邮件的默认标题也写好。
4.  **盖章发出 (`context.startActivity`)**：把公函交给系统，系统会自动去手机里找能处理这封公函的 App。

**核心代码：**
```kotlin
val intent = Intent(Intent.ACTION_SENDTO).apply {
    data = Uri.parse("mailto:3242752034@qq.com") // 目标邮箱
    putExtra(Intent.EXTRA_SUBJECT, "关于模拟抽卡模块的数据提供") // 默认邮件标题
}
context.startActivity(intent) // 👈 必须用 context 这个通行证来发送！
```

---

## 新技能三：try-catch (给程序穿上防弹衣)

上面的发邮件代码有一个**致命隐患**：万一用户的手机里**根本没有安装任何邮件 App** 怎么办？
如果系统找不到能处理 `Intent` 的 App，程序就会直接**崩溃闪退**！

为了防止这种情况，我们需要使用 `try-catch` 语法来“买保险”。

*   **`try { ... }` (尝试执行)**：把可能有风险的代码放进去。
*   **`catch (e: Exception) { ... }` (兜底方案)**：如果 `try` 里面的代码搞砸了（抛出异常），程序不会崩溃，而是立刻跳到这里执行兜底逻辑。

```kotlin
try {
    context.startActivity(intent) // 尝试拉起邮件 App
} catch (e: Exception) {
    // 兜底方案：如果没装邮件App拉起失败，就弹个原生的文字提示框告诉用户
    Toast.makeText(context, "请发送邮件至: 3242752034@qq.com", Toast.LENGTH_LONG).show()
}
```

---

## 新技能四：Toast (原生轻提示)

在 `catch` 块中，我们用到了 `Toast`。这是 Android 最经典的原生提示框（就是屏幕底部弹出的那种黑色半透明小药丸，几秒后自动消失）。

**使用公式：**
```kotlin
Toast.makeText(通行证, "要提示的文字", 显示时长).show()
```
*   `Toast.LENGTH_LONG`：显示时间较长（约 3.5 秒）。
*   `Toast.LENGTH_SHORT`：显示时间较短（约 2 秒）。
*   **千万别忘了最后的 `.show()`！** 否则提示框造出来了但不会显示。

---

## 新技能五：Surface 与 Divider (UI 质感小组件)

除了系统交互，`help.kt` 里还用到了两个提升 UI 质感的新组件：

### 1. `Surface` (带阴影的纸张)
之前我们做卡片用的是 `Card`，这里用的是 `Surface`。它们很像，但 `Surface` 是更底层的组件，适合用来做大面积的白色背景板。
```kotlin
Surface(
    shape = RoundedCornerShape(24.dp), // 大圆角
    color = Color.White,
    shadowElevation = 8.dp // 👈 核心：8dp 的阴影，让这块白色区域瞬间悬浮起来！
) { ... }
```

### 2. `Divider` (分割线)
不用自己画细线了，系统自带的分割线组件，专门用来隔开不同的内容区域。
```kotlin
Divider(
    color = Color(0xFFE2E8F0), 
    thickness = 1.dp // 线条粗细
)
```

---
**🎯 总结：**
从这节课开始，你的 App 不再是一个封闭的单机页面了。通过 `Context` 和 `Intent`，你打通了与 Android 系统以及其他 App 交互的桥梁！