# 零基础 Compose 学习笔记 - 第三课：Column 布局与 UI 积木块

本节课目标：学习如何将文字、按钮等组件从上到下整齐地排列在屏幕正中间，并进行精细的美化。

## 1. Column：垂直布局容器 🍡

`Column` 是 Compose 的三大基础布局之一。放在 `Column` 内部的组件，会像**串糖葫芦**一样，从上到下依次排列。

**核心代码：**
```kotlin
Column(
    modifier = Modifier
        .fillMaxSize()    // 填满整个屏幕
        .padding(24.dp),  // 四周留出 24dp 的安全距离
    horizontalAlignment = Alignment.CenterHorizontally, // 水平方向：全部居中对齐
    verticalArrangement = Arrangement.Center            // 垂直方向：整体挤在正中间
) {
    // 里面放 Logo、标题、按钮等...
}
```
*   **`Alignment` (对齐)**：控制组件在交叉轴（这里是水平方向）的位置。
*   **`Arrangement` (排列)**：控制组件在主轴（这里是垂直方向）的分布方式。

## 2. Spacer：隐形的排版积木 👻

如果不加控制，`Column` 里的组件会紧紧贴在一起。我们使用 `Spacer` 来撑开间距。
你可以把它想象成一块**完全透明的乐高积木**。

```kotlin
// 在两个组件之间塞入一块高度为 32dp 的透明积木
Spacer(modifier = Modifier.height(32.dp))
```

## 3. Text：文字的精细雕刻 🅰️

`Text` 是专门用来显示文字的组件。通过各种参数，可以实现极具设计感的排版。

```kotlin
Text(
    text = "抽卡模拟器",
    fontSize = 44.sp,                  // 字号大小
    fontWeight = FontWeight.Black,     // 字重：Black 是比 Bold 更粗的终极加粗
    color = Color(0xFF0F172A),         // 字体颜色：极深灰
    letterSpacing = 2.sp               // 字间距：拉开间距增加呼吸感
)
```

## 4. Button：高对比度按钮 🔘

按钮不仅仅是一个点击区域，它本身也是一个**容器**（大括号里可以放 Text，也可以放 Icon）。

```kotlin
Button(
    onClick = { onStartClick() }, // 点击通电，触发回调函数
    modifier = Modifier
        .fillMaxWidth(0.8f)       // 宽度占屏幕的 80% (两边自然留白)
        .height(56.dp),           // 高度固定 56dp
    shape = RoundedCornerShape(16.dp), // 形状：圆角矩形，圆角弧度 16dp
    colors = ButtonDefaults.buttonColors(
        containerColor = Color(0xFF0F172A), // 按钮底色
        contentColor = Color.White          // 按钮内部内容(文字/图标)的默认颜色
    )
) {
    // 这里的 Text 不需要再写 color = Color.White，它会自动继承 contentColor！
    Text(text = "选择游戏", fontSize = 18.sp, fontWeight = FontWeight.Bold)
}
```

### ❓ 为什么 Button 的颜色设置这么复杂（ButtonDefaults）？
不要直接在 `Text` 里写死颜色，而是使用 `ButtonDefaults.buttonColors` 的 `contentColor`，原因有两个：
1.  **大喇叭广播（统一变色）**：`contentColor` 会向按钮内部广播，里面的所有 `Text` 和 `Icon` 都会自动变成这个颜色，不用挨个设置。
2.  **智能状态管理（防呆设计）**：`ButtonDefaults` 内部自带了“禁用状态（Disabled）”的颜色配置。如果按钮不可点击，它会自动把背景和文字都变成灰色。如果你在 `Text` 里把颜色写死为白色，禁用时文字依然是刺眼的白色，会非常违和！

## 5. Icon：矢量小图标 🎨

`Icon` 是专门用来显示简单、纯色形状的组件（如返回箭头、设置齿轮、小爱心等）。它使用的是**矢量图**，无论怎么放大都不会模糊，而且可以通过代码一秒换色！

**核心代码：**
```kotlin
Icon(
    imageVector = Icons.Default.Star, // 1. 选图案：Compose 自带的星星图标
    contentDescription = "收藏",      // 2. 盲人辅助：告诉读屏软件这是什么（必填项，不写会报黄线）
    tint = Color.Yellow,              // 3. 一秒上色：将图标刷成黄色
    modifier = Modifier.size(24.dp)   // 4. 大小：设置图标宽高为 24dp
)
```

**有哪些自带的图标可以用？**
Compose 内置了 Material Design 图标库，你可以通过输入 `Icons.Default.` 然后让代码编辑器自动提示来选择。常用的有：
*   `Icons.Default.ArrowBack`：返回箭头 ⬅️
*   `Icons.Default.Settings`：设置齿轮 ⚙️
*   `Icons.Default.Favorite`：爱心 ❤️
*   `Icons.Default.Home`：主页 🏠
*   `Icons.Default.Search`：放大镜搜索 🔍
*   `Icons.Default.Person`：用户头像 👤
*   `Icons.Default.Add`：加号 ➕
*   `Icons.Default.Check`：打勾 ✔️

## 💡 核心知识点：dp 与 sp 的区别
在 Android 开发中，有两个非常重要的尺寸单位，绝不能混用：
*   **`dp` (Density-independent Pixels)**：用于**宽度、高度、间距、圆角**等。它能保证在不同分辨率的手机上，看起来物理大小差不多。
*   **`sp` (Scale-independent Pixels)**：**专门用于文字大小 (`fontSize`)**。它除了具备 `dp` 的特性外，还会根据用户在手机系统设置里调整的“字体大小”自动缩放（对长辈或视力不好的用户非常友好）。
