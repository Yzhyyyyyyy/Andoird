# 🎒 Compose 学习笔记13 - 第十三课 (复杂状态提升与大转盘进阶动画)

> 本节课的内容提取自“和平精英大转盘”实战项目。重点攻克了多 Tab 切换时的数据防丢、转盘动画的数学公式、高级 Canvas 绘制以及 Kotlin 的集合魔法。

## 1. 🏗️ 状态提升的终极实战 (防数据丢失)

### 🤔 1.1 灾难现场：一切重头再来
在多 Tab（底部导航栏）的应用中，如果把“当前转盘星级”、“是否正在转动”这些状态写在 `PeRouletteTab`（转盘页面）内部，**当用户切到“充值”页面再切回来时，转盘页面会被销毁重建，玩家垫的保底和星级全丢了！**

### ✅ 1.2 正确姿势：状态提升 (State Hoisting)
必须把核心业务状态，提升到所有 Tab 的共同父组件（比如主界面的 `Scaffold` 之外）中去管理。
```kotlin
@Composable
fun PeaceEliteScreen() {
    // 1. 将核心状态定义在最顶层，只要主界面不销毁，数据就在
    var currentStar by remember { mutableIntStateOf(0) }
    var currentTab by remember { mutableStateOf(PeTab.Roulette) }

    Scaffold(...) {
        // 2. 将状态作为参数，向下传递给具体的 Tab 页面
        when (currentTab) {
            PeTab.Roulette -> PeRouletteTab(currentStar = currentStar)
            PeTab.Recharge -> PeRechargeTab()
        }
    }
}
```

---

## 2. 🧮 动画数学魔法：精准停靠的转盘

### 🌀 2.1 怎么让转盘转几圈后，精准停在目标位置？
大转盘不能瞎转，它需要一个通用的数学公式：**当前基础角度 + 多转的圈数 + 目标孔位的偏移角度**。

### 💻 2.2 核心公式拆解
```kotlin
// 1. 计算当前动画值除以 360 的余数（当前转到了哪）
val currentInnerRem = innerRotationAnim.value % 360f 

// 2. 算出基础的整圈角度（抹平余数，方便计算）
val innerBase = innerRotationAnim.value - currentInnerRem 

// 3. 目标角度 = 基础角度 + 狂转5圈(营造抽奖感) + 目标孔位需要的角度
val finalInner = innerBase + (5 * 360f) + innerTargetMod

// 4. 启动协程，让转盘转到最终角度
launch { 
    innerRotationAnim.animateTo(finalInner, tween(3500, easing = FastOutSlowInEasing)) 
}
```

---

## 3. 🎨 Canvas 进阶：扫描渐变与三角函数

### 💿 3.1 扫描渐变 (`Brush.sweepGradient`)
在第 12 课学了径向渐变（从中心向外发散）。这次转盘的“金属/CD光盘”质感，用的是**扫描渐变**（像雷达一样绕着圆心转一圈的渐变）。
```kotlin
drawCircle(
    brush = Brush.sweepGradient(
        colors = listOf(Color(0xFFF1F5F9), Color(0xFFE2E8F0), Color(0xFFF8FAFC), Color(0xFFE2E8F0), Color(0xFFF1F5F9))
    ),
    radius = size.width / 2
)
```

### 📐 3.2 终极画图：三角函数定位
如果要在圆周上均匀地画 8 条刻度线，必须用到初中数学的三角函数：
*   X 坐标公式：$$x = centerX + radius \cdot \cos(\theta)$$
*   Y 坐标公式：$$y = centerY + radius \cdot \sin(\theta)$$
```kotlin
// Kotlin 中的 Math.cos 和 Math.sin 需要传入弧度制 (Math.PI / 180f)
val angle = (i * 45f) * (Math.PI / 180f).toFloat()
val endX = size.width / 2 + (size.width / 2) * kotlin.math.cos(angle.toDouble()).toFloat()
val endY = size.height / 2 + (size.height / 2) * kotlin.math.sin(angle.toDouble()).toFloat()
```

---

## 4. 📜 轻量级滚动：`verticalScroll`

### 🤔 4.1 什么时候不用 `LazyColumn`？
`LazyColumn` 是为了**无限长**的列表设计的（比如几百条聊天记录），它有回收复用机制，但稍微重一点。
如果你的页面只有几个固定的按钮（比如充值页、设置页），超出了屏幕一点点，用普通的 `Column` 加上滚动修饰符最简单。

### ✅ 4.2 极简用法
```kotlin
// 给普通的 Column 加上 verticalScroll 即可实现上下滑动
Column(
    modifier = Modifier
        .fillMaxSize()
        .verticalScroll(rememberScrollState())
) {
    // 里面放固定的几个组件
    Text("充值中心")
    Button(...)
}
```

---

## 5. 🔪 Kotlin 集合魔法：`chunked()` 分块

### 🪄 5.1 不用网格布局实现网格效果
充值页面的按钮是“一行 3 个”排布的。正常思路是用 `LazyVerticalGrid`，但 Kotlin 提供了一个更暴力的魔法：`chunked()`。它可以直接把一个长列表“切”成指定大小的小块（把一维数组变成二维数组）。

### 💻 5.2 代码实例
```kotlin
val options = listOf(6, 30, 68, 128, 198, 328)

// 魔法：将列表按每 3 个一组进行切割
val chunkedOptions = options.chunked(3) 
// 结果变成了：[[6, 30, 68], [128, 198, 328]]

// 然后用普通的 Column 嵌套 Row 就能画出网格
Column {
    chunkedOptions.forEach { rowItems ->
        Row {
            rowItems.forEach { price ->
                Button(onClick = { /* 充值 */ }) { Text("¥$price") }
            }
        }
    }
}
```