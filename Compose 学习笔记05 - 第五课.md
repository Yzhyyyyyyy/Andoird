# 📝 Compose 学习笔记05 - 第五课 (Choose 界面核心技术拆解)

在前面的课程中，我们已经掌握了基础布局（Column/Box）和状态驱动路由。今天在这个 Choose（选择游戏）界面中，我们将学习如何构建**可滑动的列表**，以及如何利用**高级架构技巧**来复用页面背景。

---

## 新技能一：高级架构“插槽 API” (`@Composable () -> Unit`)

在第二课中，我们学会了用 `Box` 和 `Canvas` 画出梦幻背景。但如果每个页面都要把画背景的代码写一遍，就太累了。
这次的代码里，出现了一个极其优雅的招式：**插槽 API**。

注意看 `UnifiedBackground` 的参数：`content: @Composable () -> Unit`。

*   **通俗比喻：“挖坑填土”**
    你可以把 `UnifiedBackground` 想象成一个**精美的相框**。相框本身（背景图）是固定的，但相框中间**挖了一个洞**（`content`），留给你以后放照片。

```kotlin
// 1. 制造相框 (挖坑)
@Composable
fun UnifiedBackground(content: @Composable () -> Unit) {
    Box {
        Canvas(...) // 画好相框边缘 (复用第二课的知识)
        content()   // 👈 这里就是挖好的坑！
    }
}

// 2. 使用相框 (填土)
@Composable
fun GameSelectionScreen() {
    UnifiedBackground { // 👈 大括号里的内容，就会被塞进上面的 content() 里！
        LazyColumn { ... } // 把游戏列表塞进相框里
    }
}
```
*   **极致复用**：以后不管是“选择游戏页”还是“设置页”，只要想用这个背景，直接用 `UnifiedBackground { 你的页面 }` 包起来就行！这在 Compose 开发中是非常核心的组件封装思想。

---

## 新技能二：无限滑动的终极奥义 `LazyColumn`

之前我们排版用的是普通的 `Column`，但如果是几十上百个游戏的列表呢？

*   **普通 `Column` 的致命伤**：它会一次性把所有卡片全部渲染出来，哪怕屏幕上只能显示 5 个。结果就是：**手机卡死、内存爆炸**。
*   **`LazyColumn` (懒加载列表)**：它**只渲染屏幕上能看到的那几个卡片**。当你往下滑时，它会把滑出屏幕的卡片回收，用来显示新出现的卡片。这在 Android 开发中叫**“视图复用”**（相当于传统 View 体系里的 RecyclerView）。

**使用规范：**
在 `LazyColumn` 的大括号里，不能直接写组件，必须用 `item { ... }` 或者 `items() { ... }` 包起来：

```kotlin
LazyColumn(
    // verticalArrangement 可以统一设置每个 item 之间的间距，非常省事！
    verticalArrangement = Arrangement.spacedBy(16.dp) 
) {
    // 第 1 项：标题
    item { Text("选择游戏") }
    
    // 第 2 项：原神卡片
    item { GameCard(name = "原神") { ... } }
}
```

---

## 新技能三：UI 质感提升（Card 与 Clickable）

在第三课我们学了基础的 UI 积木，今天来看看 `GameCard` 是怎么做出高级质感的：

### 1. `Card` (卡片容器)
系统自带的高级容器。只要用 `Card` 包裹内容，它天生自带**圆角 (`shape`)** 和 **阴影 (`elevation`)**，立刻让 UI 产生立体的悬浮感。
```kotlin
Card(
    shape = RoundedCornerShape(16.dp), // 16dp 的圆角
    elevation = CardDefaults.cardElevation(defaultElevation = 2.dp) // 微微浮起的阴影
) { ... }
```

### 2. `Modifier.clickable` (万物皆可点)
系统自带的 `Button` 样式太固定了。如果你想让一个普通的卡片、图片甚至一段文字变得像按钮一样可以点击，只需要给它的 `Modifier` 加上 `.clickable { ... }` 即可！
```kotlin
Card(
    modifier = Modifier
        .fillMaxWidth()
        .clickable { onClick() } // 👈 魔法在这里！现在整个卡片都可以点击了
) { ... }
```
