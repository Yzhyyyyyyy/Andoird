# 零基础 Compose 学习笔记 - 第二课：Box 容器与 Canvas 魔法画笔

本节课目标：理解如何给整个屏幕铺上底色，并在上面画出高级的、朦胧的发光效果。

## 1. Box 布局：千层饼容器 📦

在 Compose 的三大基础布局（`Column`列、`Row`行、`Box`盒）中，`Box` 的特性是**“层叠”**。
你可以把它想象成做**千层饼**：写在 `Box` 大括号 `{}` 里的第一个组件在最底层，第二个组件会盖在第一个上面。

**核心代码结构：**
```kotlin
// 整个屏幕的底层容器
Box(
    modifier = Modifier
        .fillMaxSize()
        .background(Color(0xFFF8FAFC)) // 极简冷白/灰白背景
) {
    // 第 1 层：魔法光晕效果 (铺在最底层)
    Canvas(modifier = Modifier.fillMaxSize()) {
        // ... 画光晕的代码
    }
    
    // 第 2 层：核心内容区 (盖在光晕上面)
    Column(...) {
        // ... Logo、标题、按钮的代码
    }
}
```

## 2. Modifier：组件的魔法棒 🪄

代码里随处可见的 `modifier` 是 Compose 中最常用的词，它是每个组件的**“变装魔法棒”**或**“设置面板”**。

*   **`.fillMaxSize()`**：魔法棒一挥，告诉组件：“你要变得和手机屏幕一样大！”（填满最大尺寸）。
*   **`.background(Color(0xFFF8FAFC))`**：给组件刷上底漆。
    *   *小知识*：`0xFF` 代表完全不透明，后面的 `F8FAFC` 是设计师给的十六进制颜色代码（冷白色）。

## 3. Canvas：你的专属画板 🎨

当普通的组件（如按钮、文字）画不出不规则的光晕时，就需要请出 `Canvas`（画板）。
我们用 `Canvas` 画了右上角和左下角两团柔光。

**完整画板代码：**
```kotlin
Canvas(modifier = Modifier.fillMaxSize()) {
    // 1. 画右上角的浅蓝色柔光
    drawCircle(
        brush = Brush.radialGradient(
            colors = listOf(Color(0xFF7DD3FC).copy(alpha = 0.65f), Color.Transparent),
            center = Offset(size.width * 0.8f, size.height * 0.1f),
            radius = size.width * 0.75f
        )
    )
    
    // 2. 画左下角的浅紫色柔光
    drawCircle(
        brush = Brush.radialGradient(
            colors = listOf(Color(0xFFD8B4FE).copy(alpha = 0.6f), Color.Transparent),
            center = Offset(size.width * 0.2f, size.height * 0.8f),
            radius = size.width * 0.75f
        )
    )
}
```

## 4. 拆解魔法光晕的秘密 🔍

在 `Canvas` 内部，我们使用了 `drawCircle`（画圆）指令，但怎么画出“光晕”而不是“实心球”呢？秘密在于画笔的设置：

*   **`Brush.radialGradient` (径向渐变画笔)**：这是发光效果的核心！它能让颜色从中心点向外围一圈一圈地扩散。
*   **`colors` (颜色过渡)**：
    *   起点颜色：`Color(0xFF7DD3FC).copy(alpha = 0.65f)`（带有 65% 透明度的浅蓝色）。
    *   终点颜色：`Color.Transparent`（完全透明）。
    *   *效果：颜色逐渐变淡直到消失，形成边缘虚化。*
*   **`center` (圆心位置)**：使用 `Offset(x, y)` 设置坐标。
    *   `size.width * 0.8f`：屏幕宽度的 80% 处（偏右）。
    *   `size.height * 0.1f`：屏幕高度的 10% 处（偏上）。
    *   *合起来就是右上角。*
*   **`radius` (半径)**：`size.width * 0.75f`，光晕非常大，半径占了屏幕宽度的 75%。
