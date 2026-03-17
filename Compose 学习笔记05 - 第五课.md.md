# 📝 Compose 学习笔记05 - 第五课 (Choose 界面核心技术拆解)

在这个界面中，我们不再是简单地把按钮堆在一起，而是开始构建**有层次、有质感、能滑动**的现代 App 页面，并且运用了高级的架构思想。

---

## 技能一：千层饼布局 `Box` (Z 轴叠放)

在之前的代码里，你用过 `Column`（垂直排）和 `Row`（水平排）。但如果你想让文字**浮在背景图上面**怎么办？这时候就要请出 `Box` 了。

**`Box` 的核心逻辑：像千层饼一样，先写的在最底层，后写的盖在上面。**

```kotlin
@Composable
fun UnifiedBackground(content: @Composable () -> Unit) {
    Box(modifier = Modifier.fillMaxSize()) { // 👈 开启千层饼模式
        // 第 1 层 (最底层)：Canvas 画的渐变光晕
        Canvas(modifier = Modifier.fillMaxSize()) { ... }
        // 第 2 层 (最顶层)：具体的页面内容 (比如游戏列表)
        content() 
    }
}
```
*   **沉浸式背景**：有了 `Box`，梦幻背景就会死死垫在最下面，不管上面的 `content()` 怎么滑动，背景都不会跟着跑。

---

## 技能二：高级架构“插槽 API” (`@Composable () -> Unit`)

这是这段代码里**最优雅、最显功底**的一招！
注意看 `UnifiedBackground` 的参数：`content: @Composable () -> Unit`。

*   **通俗比喻：“挖坑填土”**
    你可以把 `UnifiedBackground` 想象成一个**精美的相框**。相框本身（背景图）是固定的，但相框中间**挖了一个洞**（`content`），留给你以后放照片。

```kotlin
// 1. 制造相框 (挖坑)
@Composable
fun UnifiedBackground(content: @Composable () -> Unit) {
    Box {
        Canvas(...) // 画好相框边缘
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
*   **极致复用**：以后不管是“选择游戏页”还是“设置页”，只要想用这个背景，直接用 `UnifiedBackground { 你的页面 }` 包起来就行！

---

## 技能三：无限滑动的终极奥义 `LazyColumn`

为什么不用普通的 `Column`，非要在前面加个 `Lazy`（懒）？

*   **普通 `Column`**：一次性把所有卡片全部画出来，哪怕屏幕上只能显示 5 个。结果：**手机卡死、内存爆炸**。
*   **`LazyColumn`**：**只画屏幕上能看到的那几个卡片**。往下滑时，把滑出屏幕的卡片回收，用来显示新出现的卡片（视图复用）。

```kotlin
LazyColumn(
    verticalArrangement = Arrangement.spacedBy(16.dp) // 统一设置 item 间距
) {
    item { Text("选择游戏") }
    item { GameCard(name = "原神") { ... } }
}
```

---

## 技能四：UI 质感提升三件套

`GameCard` 组件好看的秘诀全靠这三样法宝：

1.  **`Card` (卡片)**：自带圆角 (`shape`) 和阴影 (`elevation`)，产生立体悬浮感。
2.  **`Spacer` (隐形占位符)**：像一块透明砖头，专门用来精准撑开元素之间的空间，替代繁琐的 padding 计算。
3.  **`Modifier.clickable` (万物皆可点)**：让普通的卡片、图片或文字变得像按钮一样可以点击。

```kotlin
Card(
    modifier = Modifier.clickable { onClick() }, // 👈 魔法在这里！整个卡片可点击
    shape = RoundedCornerShape(16.dp),
    elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
) { 
    Text("原神")
    Spacer(modifier = Modifier.height(4.dp)) // 👈 塞一块 4dp 高的透明砖头
    Text("HoYoverse")
}
```

---

## 技能五：架构核心之“对讲机”命名规则 (回调函数)

在 `GameSelectionScreen(onNavigate: (String) -> Unit)` 和 `GameCard(onClick: () -> Unit)` 中，我们大量使用了自定义的回调函数（就像给打工人发的“对讲机”）。

关于这些“对讲机频道”的命名，必须牢记以下规则：

### 1. 绝对不能重复：同一个页面（函数）内
在一个 Composable 函数的参数列表里，名字必须独一无二。如果一个页面有多个不同的点击动作，必须起不同的名字来区分。
```kotlin
// ✅ 正确示范：区分不同的动作
@Composable
fun MyFirstScreen(
    onStartClick: () -> Unit, // 点击开始
    onHelpClick: () -> Unit   // 点击帮助
) { ... }
```

### 2. 可以且强烈建议重复：不同的页面（函数）之间
在不同的 Composable 页面中，如果它们触发的动作逻辑相似（比如都是“返回上一页”），**强烈建议使用相同的命名**（如统一叫 `onBackClick`）。
这是一种非常专业和优雅的代码规范，能让老板（`MainApp`）在统一管理状态时，代码结构极其整齐。
```kotlin
// ✅ 优雅的规范：跨页面统一命名
@Composable
fun GenshinScreen(onBackClick: () -> Unit) { ... }

@Composable
fun PeaceEliteScreen(onBackClick: () -> Unit) { ... }
```