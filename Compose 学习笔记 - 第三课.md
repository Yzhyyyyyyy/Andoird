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
        contentColor = Color.White          // 按钮内部内容(文字)的颜色
    )
) {
    Text(text = "选择游戏", fontSize = 18.sp, fontWeight = FontWeight.Bold)
}
```

## 💡 核心知识点：dp 与 sp 的区别
在 Android 开发中，有两个非常重要的尺寸单位，绝不能混用：
*   **`dp` (Density-independent Pixels)**：用于**宽度、高度、间距、圆角**等。它能保证在不同分辨率的手机上，看起来物理大小差不多。
*   **`sp` (Scale-independent Pixels)**：**专门用于文字大小 (`fontSize`)**。它除了具备 `dp` 的特性外，还会根据用户在手机系统设置里调整的“字体大小”自动缩放（对长辈或视力不好的用户非常友好）。