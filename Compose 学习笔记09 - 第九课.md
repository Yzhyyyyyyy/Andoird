# 🏗️ Compose 学习笔记09 - 第九课 (进阶布局：脚手架与网格列表)

在 `efootball_main` 界面中，我们脱离了简单的上下排列，开始构建真正的“App 级”页面结构。本节课重点理解 Compose 的**状态管理**、**插槽设计哲学**以及**高性能列表**。

---

## 1. Scaffold (脚手架) 与 核心组件 🏠

### 🤔 1.1 为什么 Scaffold 要设计成“插槽 (Slot)”？
初学者常疑惑：为什么 `Scaffold` 不像 `Card` 一样直接把内容包裹起来，或者通过传字符串来设置标题？
*   **传统思维的局限**：如果 `Scaffold(title = "首页")` 只能传字符串，那当你想在标题栏放一张 Logo 图片或搜索框时，就束手无策了。
*   **Compose 的插槽哲学 (Slot API)**：`Scaffold` 就像一个**预留了标准格子的便当盒**。它提供了 `topBar = {}`、`bottomBar = {}` 等大括号（插槽）。它只负责排版，至于你往大括号里塞入官方组件还是自定义的图片、按钮，**完全由你自由决定**，这赋予了极大的灵活性。

### 🧩 1.2 TopAppBar (顶部标题栏) 结构解析
`TopAppBar` 是 Material Design 标准的顶部栏组件。
*   **`@OptIn(ExperimentalMaterial3Api::class)`**：因为 Material 3 的部分组件还在不断打磨中，官方要求你加上这句“免责声明”，表示你知晓这是一个**实验性 API**，未来可能会有微调。
*   **三大核心插槽**：它内部也像便当盒一样分了三个区：
    1.  `navigationIcon = {}`：最左侧（通常放返回键 `<-` 或抽屉菜单）。
    2.  `title = {}`：中间主标题（可以放文字 `Text`，也可以放图片 `Image`）。
    3.  `actions = {}`：最右侧操作区（默认水平排列，可塞入多个图标如搜索 🔍、设置 ⚙️）。

### 🧠 1.3 状态记忆与列表遍历 (底部导航栏核心)
在构建底部导航栏时，我们通常会用到以下核心语法：

```kotlin
// 1. 状态触发器与记忆卡
var selectedTab by remember { mutableStateOf("首页") }
// 2. 准备数据列表
val tabs = listOf("首页", "我的")
```
*   **`mutableStateOf`**：触发器。值一变，Compose 就会重新渲染 UI（重组）。
*   **`remember`**：记忆卡。保证 UI 重新渲染时，不会忘记你上次选中的状态。
*   **`val`**：表示这是一个不可变的只读变量。

**关于遍历生成 UI：**
你可以使用 Kotlin 优雅的 `forEach`，也可以使用最经典的 `for` 循环，两者效果完全一样！
```kotlin
// 写法 A：优雅的 forEach (Compose 常用风格)
tabs.forEach { tab -> 
    NavigationBarItem(label = { Text(tab) }, ...) 
}

// 写法 B：经典的 for 循环 (直观易懂，完全没问题！)
for (tab in tabs) {
    NavigationBarItem(label = { Text(tab) }, ...)
}
```

### ⚠️ 1.4 致命避坑：PaddingValues
`Scaffold` 会计算顶部和底部组件占据的高度，并通过 `innerPadding` 传给你。**你必须将它应用到主体内容的 Modifier 上**，否则你的内容会被导航栏死死挡住！

---

## 2. LazyVerticalGrid (高性能网格列表) 🗂️

### 🚀 2.1 为什么不用 Column + Row 嵌套？
如果要展示 100 个卡片，普通的 `Column` 会在瞬间渲染全部 100 个，导致严重卡顿。
`LazyVerticalGrid` 拥有**懒加载 (Lazy)** 魔法：它只渲染屏幕上能看到的卡片，滑出屏幕的会被销毁复用，性能极高！

### 💻 2.2 核心骨架代码
```kotlin
// 模拟数据
val playerList = List(50) { "球员 ${it + 1}" }

LazyVerticalGrid(
    columns = GridCells.Fixed(3), // 固定 3 列 (也可选 Adaptive 自适应)
    horizontalArrangement = Arrangement.spacedBy(12.dp), // 列间距
    verticalArrangement = Arrangement.spacedBy(12.dp),   // 行间距
    modifier = Modifier.fillMaxSize()
) {
    // 🔄 必须在 items 内部遍历数据
    items(playerList) { playerName ->
        // 这里定义单个网格的 UI
        Box(
            modifier = Modifier.aspectRatio(1f).background(Color.LightGray),
            contentAlignment = Alignment.Center
        ) {
            Text(playerName)
        }
    }
}
```
