# 🏗️ Compose 学习笔记09 - 第九课 (进阶布局：脚手架与网格列表)

在 `efootball_main` 界面中，我们脱离了简单的上下排列，开始构建真正的“App 级”页面结构。本节课主要学习标准页面骨架的搭建以及高性能网格列表的使用。

---

## 1. Scaffold (脚手架) 与 底部导航栏 🏠

### 🤔 1.1 为什么需要 Scaffold？
在之前的学习中，我们通常用 `Column` 或 `Box` 作为最外层。但一个标准的现代 App 页面通常包含：顶部标题栏（TopAppBar）、底部导航栏（BottomNavigationBar）、悬浮按钮（FAB）等。
如果全靠我们自己用 `Column` 和 `Box` 去拼凑这些元素，不仅麻烦，而且很难符合 Material Design 的标准规范。

> 💡 **通俗理解：**
> `Scaffold`（脚手架）就是 Compose 提供的一个**现成的页面模板**，它为你预留好了这些标准组件的“插槽”，你只需要把对应的积木塞进去即可。

### 🧩 1.2 Scaffold 的核心插槽与 PaddingValues
在使用 `Scaffold` 时，最容易犯的错误就是忽略 `paddingValues`。
因为底部导航栏或顶部标题栏会占据屏幕的空间，`Scaffold` 会计算出这些组件占据的高度，并通过 `paddingValues` 传给你。

> ⚠️ **致命避坑：**
> **你必须将这个 padding 应用到你的主体内容上**，否则你的主体内容就会被底部的导航栏死死挡住！

### 💻 1.3 详细代码实例：带底部导航栏的标准页面
```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Home
import androidx.compose.material.icons.filled.Person
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp

@Composable
fun StandardAppScreen() {
    // 记录当前选中的是哪个 Tab，默认选中 "首页"
    var selectedTab by remember { mutableStateOf("首页") }
    val tabs = listOf("首页", "我的")

    Scaffold(
        // 📍 插槽 1：顶部标题栏
        topBar = {
            ExperimentalMaterial3Api
            TopAppBar(
                title = { Text("我的应用") },
                colors = TopAppBarDefaults.topAppBarColors(containerColor = Color.LightGray)
            )
        },
        // 📍 插槽 2：底部导航栏
        bottomBar = {
            NavigationBar(containerColor = Color.White) {
                tabs.forEach { tab ->
                    NavigationBarItem(
                        icon = { 
                            Icon(
                                imageVector = if (tab == "首页") Icons.Filled.Home else Icons.Filled.Person, 
                                contentDescription = tab
                            ) 
                        },
                        label = { Text(tab) },
                        selected = selectedTab == tab, // 判断当前项是否被选中
                        onClick = { selectedTab = tab }, // 点击时更新状态
                        colors = NavigationBarItemDefaults.colors(
                            selectedIconColor = Color.Blue,
                            unselectedIconColor = Color.Gray
                        )
                    )
                }
            }
        }
    ) { innerPadding ->
        // ⚠️ 极其重要：必须把 innerPadding 应用到主体内容的 Modifier 上！
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding) // 🛡️ 保护内容不被遮挡
        ) {
            if (selectedTab == "首页") {
                Text("这里是首页的内容", modifier = Modifier.padding(16.dp))
            } else {
                Text("这里是个人中心", modifier = Modifier.padding(16.dp))
            }
        }
    }
}
```

---

## 2. LazyVerticalGrid (高性能网格列表) 🗂️

### 🚀 2.1 为什么不用 Column + Row 嵌套？
如果要在屏幕上展示 100 个球员卡片，使用普通的 `Column` 嵌套 `Row` 会导致这 100 个卡片在页面加载时**瞬间全部被渲染**，这会严重消耗手机内存，导致页面卡顿。

> 💡 **魔法特性：懒加载 (Lazy)**
> `LazyVerticalGrid` 只渲染当前屏幕上能看到的卡片。当你向上滑动时，滑出屏幕的卡片会被销毁，即将进入屏幕的卡片才会被实时创建。这种机制叫做“视图回收复用”，性能极高！

### ⚙️ 2.2 核心参数解析
*   **`columns` (列数定义)**:
    *   `GridCells.Fixed(4)`：强行规定每行显示 4 列，无论屏幕多宽。
    *   `GridCells.Adaptive(minSize = 100.dp)`：自适应模式。系统会根据屏幕宽度自动计算列数（推荐在适配平板时使用）。
*   **`Arrangement` (间距)**: `horizontalArrangement` / `verticalArrangement` 定义网格之间的水平和垂直间距。
*   **`items()`**: 这是一个特殊的 DSL 函数，必须在网格的大括号 `{}` 内使用，用于遍历你的数据列表。

### 💻 2.3 详细代码实例：球员背包展示
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

// 📦 模拟的数据模型
data class Player(val name: String, val rating: Int)

@Composable
fun InventoryGridScreen() {
    // 模拟生成 50 个球员数据
    val playerList = List(50) { index -> Player("球员 ${index + 1}", 80 + (index % 20)) }

    LazyVerticalGrid(
        columns = GridCells.Fixed(3), // 固定 3 列
        contentPadding = PaddingValues(16.dp), // 整个网格的外边距
        horizontalArrangement = Arrangement.spacedBy(12.dp), // 列与列之间的间距
        verticalArrangement = Arrangement.spacedBy(12.dp),   // 行与行之间的间距
        modifier = Modifier.fillMaxSize()
    ) {
        // 🔄 遍历数据列表
        items(playerList) { player ->
            // 这里定义单个网格长什么样
            Box(
                modifier = Modifier
                    .aspectRatio(1f) // 📐 强制宽高比为 1:1（正方形）
                    .background(Color(0xFFE2E8F0))
                    .padding(8.dp),
                contentAlignment = Alignment.Center
            ) {
                Column(horizontalAlignment = Alignment.CenterHorizontally) {
                    Text(text = player.name)
                    Text(text = "评分: ${player.rating}", color = Color.Red)
                }
            }
        }
    }
}
```
