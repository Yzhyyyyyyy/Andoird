# 🏗️ Compose 学习笔记09 - 第九课 (进阶布局：脚手架与高性能列表)

在 `efootball_main` 界面中，我们脱离了简单的上下排列，开始构建真正的“App 级”页面结构。本节课重点理解 Compose 的**状态管理**、**插槽设计哲学**以及**懒加载家族**。

---

## 1. Scaffold (脚手架) 与 核心组件 🏠

### 🤔 1.1 为什么 Scaffold 要设计成“插槽 (Slot)”？
初学者常疑惑：为什么 `Scaffold` 不像 `Card` 一样直接把内容包裹起来，或者通过传字符串来设置标题？
*   **传统思维的局限**：如果 `Scaffold(title = "首页")` 只能传字符串，那当你想在标题栏放一张 Logo 图片或搜索框时，就束手无策了。
*   **Compose 的插槽哲学 (Slot API)**：`Scaffold` 就像一个**预留了标准格子的便当盒**。它提供了 `topBar = {}`、`bottomBar = {}` 等大括号（插槽）。它只负责排版，至于你往大括号里塞入官方组件还是自定义的图片、按钮，**完全由你自由决定**，这赋予了极大的灵活性。

### 🧩 1.2 TopAppBar (顶部标题栏) 结构解析
`TopAppBar` 是 Material Design 标准的顶部栏组件。
*   **`@OptIn(ExperimentalMaterial3Api::class)`**：因为 Material 3 的部分组件还在不断打磨中，官方要求你加上这句“免责声明”，表示你知晓这是一个**实验性 API**，未来可能会有微调。
### Scaffold（脚手架）三大核心区域

`Scaffold` 就像一个标准的大便当盒，为你划分好了“上、中、下”三个主要区域，每个区域都有自己专属的插槽和参数：

#### 1. `topBar = {}`：顶部导航栏区域
通常在这里放入 `TopAppBar` 组件。它内部又细分为三个核心插槽：
*   `navigationIcon = {}`：**最左侧插槽**。通常放返回键 `<-` 或抽屉菜单图标。
*   `title = {}`：**中间主标题插槽**。地位重要，通常放文字 `Text` 作为页面大标题，也可以放图片 `Image` 作为 Logo。
*   `actions = {}`：**最右侧操作区插槽**。默认水平排列，可塞入多个图标按钮，如搜索 🔍、设置 ⚙️、分享等。

#### 2. `bottomBar = {}`：底部导航栏区域
通常在这里放入 `NavigationBar` 组件。它内部会循环创建多个 `NavigationBarItem`（底部按钮），每个按钮包含以下核心参数/插槽：
*   `icon = {}`：**图标插槽**。通常放 `Icon` 组件，展示该页面的图形（如 🏠、👤）。
*   `label = {}`：**标签插槽**。通常放 `Text` 组件，作为图标下方的辅助说明小字（注意与顶部的 `title` 区分）。
*   `selected = ...`：**选中状态**。接收一个布尔值（`true`/`false`），决定该按钮当前是否需要变成高亮颜色。
*   `onClick = {}`：**点击动作**。接收一段代码，当用户点击时执行，通常用于修改大脑里的“状态变量”（如 `selectedTab`）。

#### 3. `content = { innerPadding -> }`：中间核心内容区
这是屏幕中间占地最大的区域，通常放入 `Box`、`Column` 或 `LazyColumn` 等布局。
*   `innerPadding`：**安全内边距**。极其重要！必须将其应用到内部布局上（`Modifier.padding(innerPadding)`），它的作用是防止中间的内容被顶部栏或底部栏挡住。
*   **页面切换逻辑**：在这个区域内，我们会通过 `if-else` 或 `when` 语句，读取当前选中的状态变量（如 `selectedTab`），来决定屏幕中间具体画出哪个页面的 UI 内容。

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
tabs.forEach { tab -> NavigationBarItem(...) }

// 写法 B：经典的 for 循环 (直观易懂，完全没问题！)
for (tab in tabs) { NavigationBarItem(...) }
```

### 💻 1.4 完整代码样例：标准 App 骨架
> ⚠️ **致命避坑**：`Scaffold` 会计算顶部和底部组件占据的高度，并通过 `innerPadding` 传给你。**你必须将它应用到主体内容的 Modifier 上**，否则你的内容会被导航栏死死挡住！

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Home
import androidx.compose.material.icons.filled.Person
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun StandardAppScreen() {
    var selectedTab by remember { mutableStateOf("首页") }
    val tabs = listOf("首页", "我的")

    Scaffold(
        topBar = {
            TopAppBar(title = { Text("我的应用") })
        },
        bottomBar = {
            NavigationBar {
                for (tab in tabs) { // 使用经典的 for 循环
                    NavigationBarItem(
                        icon = { 
                            Icon(
                                imageVector = if (tab == "首页") Icons.Filled.Home else Icons.Filled.Person, 
                                contentDescription = tab
                            ) 
                        },
                        label = { Text(tab) },
                        selected = selectedTab == tab,
                        onClick = { selectedTab = tab }
                    )
                }
            }
        }
    ) { innerPadding ->
        // ⚠️ 必须加上 padding
        Box(modifier = Modifier.padding(innerPadding).fillMaxSize()) {
            Text(text = "当前选中的是：$selectedTab")
        }
    }
}
```

---

## 2. 高性能列表：LazyColumn 与 LazyVerticalGrid 🗂️

### 🤔 2.1 它们俩是一个东西吗？
它们**本质上是同一个东西**，都属于 Compose 的“懒加载 (Lazy) 家族”！
*   **相同点（核心魔法）**：它们都具备**懒加载**特性。如果要展示 100 个卡片，它们都只会渲染屏幕上能看到的几个。当你滑动时，滑出屏幕的卡片会被销毁，即将进入屏幕的卡片才会被实时创建。这叫“视图回收复用”，性能极高。
*   **不同点（排版方式）**：
    *   **`LazyColumn`**：**单列**垂直滑动。就像微信朋友圈、新闻列表、聊天记录。一行只有一个元素。
    *   **`LazyVerticalGrid`**：**多列**垂直滑动。就像手机相册、游戏里的球员背包。一行可以有多个元素（网格状）。

### 🚀 2.2 为什么不用普通的 Column 或 Row？
普通的 `Column` 没有“懒加载”魔法。如果你在 `Column` 里放 100 个元素，它会在页面打开的一瞬间**把 100 个元素全部渲染出来**，即使有 90 个元素在屏幕外面根本看不到。这会严重消耗手机内存，导致页面卡顿甚至崩溃。

### 💻 2.3 完整代码样例：网格列表 (球员背包)
```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.grid.GridCells
import androidx.compose.foundation.lazy.grid.LazyVerticalGrid
import androidx.compose.foundation.lazy.grid.items
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp

@Composable
fun InventoryGridScreen() {
    // 模拟生成 50 个球员数据
    val playerList = List(50) { "球员 ${it + 1}" }

    LazyVerticalGrid(
        // 核心参数：固定 3 列 (如果是 LazyColumn 则没有这个参数)
        columns = GridCells.Fixed(3), 
        contentPadding = PaddingValues(16.dp), // 整个网格的外边距
        horizontalArrangement = Arrangement.spacedBy(12.dp), // 列与列之间的间距
        verticalArrangement = Arrangement.spacedBy(12.dp),   // 行与行之间的间距
        modifier = Modifier.fillMaxSize()
    ) {
        // 🔄 必须使用 items() 函数来遍历数据
        items(playerList) { playerName ->
            // 这里定义单个网格长什么样
            Box(
                modifier = Modifier
                    .aspectRatio(1f) // 📐 强制宽高比为 1:1（正方形卡片）
                    .background(Color(0xFFE2E8F0))
                    .padding(8.dp),
                contentAlignment = Alignment.Center
            ) {
                Text(text = playerName, color = Color.Black)
            }
        }
    }
}
```
