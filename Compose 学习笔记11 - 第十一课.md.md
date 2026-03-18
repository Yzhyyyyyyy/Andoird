# 🎇 Compose 学习笔记11 - 第十一课 (炫酷动效与手势拦截)

在第二课中，你已经掌握了 `Canvas` 的基础用法。这节课我们聚焦于如何让画面**动起来**。Compose 的动画系统非常强大且易用，我们将学习两种最常用的动画：无限循环动画和目标值动画，以及如何处理动画期间的**手势拦截**。

---

## 1. 无限循环动画 (rememberInfiniteTransition) 🎡

### 🌀 1.1 什么是无限循环动画？
像雷达扫描、呼吸灯、抽卡时的光阵旋转，这些动画一旦开始就不会停止，直到组件被销毁。在 Compose 中，我们使用 `rememberInfiniteTransition()` 来管理这类动画。

### ⚙️ 1.2 核心参数拆解
*   **`initialValue` / `targetValue`**: 动画的起点和终点。比如缩放从 0.8 倍变到 1.3 倍。
*   **`animationSpec` (动画规格)**:
    *   `tween(durationMillis)`: 补间动画，指定单次动画耗时（毫秒）。
    *   `easing`: 缓动曲线。`LinearEasing` 是匀速（适合旋转）；`FastOutSlowInEasing` 是快出慢进（适合呼吸缩放，看起来更自然）。
*   **`repeatMode` (循环模式)**:
    *   `RepeatMode.Restart`: 到达终点后，瞬间回到起点重新开始（适合 0 -> 360 度的旋转）。
    *   `RepeatMode.Reverse`: 到达终点后，原路返回到起点（适合变大再变小的呼吸效果）。

### 💻 1.3 详细代码实例：旋转的呼吸光阵
```kotlin
@Composable
fun MagicCircleAnimation() {
    // 1. 引擎：创建无限动画引擎
    val infiniteTransition = rememberInfiniteTransition()

    // 2. 呼吸：定义缩放动画 (0.8 -> 1.3 -> 0.8 循环)
    val scale by infiniteTransition.animateFloat(
        initialValue = 0.8f, 
        targetValue = 1.3f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000, easing = FastOutSlowInEasing), 
            repeatMode = RepeatMode.Reverse // 🪀 反转模式
        )
    )

    // 3. 旋转：定义旋转动画 (0 -> 360 匀速循环)
    val rotation by infiniteTransition.animateFloat(
        initialValue = 0f, 
        targetValue = 360f,
        animationSpec = infiniteRepeatable(
            animation = tween(2000, easing = LinearEasing), 
            repeatMode = RepeatMode.Restart // 🔄 重启模式
        )
    )

    // 4. 应用：将计算出的动画值应用到 Modifier 上
    Box(
        modifier = Modifier
            .size(100.dp)
            .scale(scale)       // 🪄 应用缩放
            .rotate(rotation)   // 🪄 应用旋转
            .background(Color.Cyan, shape = CircleShape),
        contentAlignment = Alignment.Center
    ) {
        Text("光阵", color = Color.White)
    }
}
```

---

## 2. 目标值动画 (Animatable) 🎯

### 🏁 2.1 什么是目标值动画？
与无限循环不同，有些动画是有明确起止时间的。比如抽卡时，进度条从 0% 涨到 100%，或者彩虹光效在 3 秒内逐渐显现。这种**从 A 变到 B 就结束**的动画，使用 `Animatable`。

### 🧠 2.2 核心机制
`Animatable` 必须配合协程（`LaunchedEffect`）使用，因为它的 `animateTo()` 方法是一个挂起函数（会随时间推移逐步执行）。

### 💻 2.3 详细代码实例：延迟出现的彩虹特效
```kotlin
@Composable
fun RainbowUpgradeEffect() {
    // 1. 定义一个初始值为 0f 的动画变量
    val animProgress = remember { Animatable(0f) }

    // 2. 组件加载时，启动动画
    LaunchedEffect(Unit) {
        // ⏳ 在 3000 毫秒内，将值从 0f 匀速变到 1f
        animProgress.animateTo(
            targetValue = 1f, 
            animationSpec = tween(3000, easing = LinearEasing)
        )
    }

    // 3. 动态计算透明度 (Alpha)
    // 假设我们希望前 1.5 秒不显示(alpha=0)，后 1.5 秒渐渐显示(alpha 0->1)
    val rainbowAlpha = if (animProgress.value > 0.5f) {
        (animProgress.value - 0.5f) * 2f // 🧮 数学映射
    } else {
        0f
    }

    Box(
        modifier = Modifier
            .size(200.dp)
            .background(Color.Gray) // 底色
    ) {
        // 🌈 覆盖在上面的特效层，透明度随动画变化
        Box(
            modifier = Modifier
                .fillMaxSize()
                .alpha(rainbowAlpha)
                .background(Color.Red) // 这里用红色代替彩虹渐变演示
        )
        
        Text(
            text = "当前进度: ${(animProgress.value * 100).toInt()}%",
            modifier = Modifier.align(Alignment.Center)
        )
    }
}
```

---

## 3. 拦截用户手势 (pointerInput) 🛡️

### 🤔 3.1 为什么需要拦截手势？
在抽卡动画播放的 3 秒内，如果不做处理，用户可以乱点底部的 Tab 切换页面，或者点击返回按钮。这会导致动画还没放完，页面就没了，程序逻辑极易崩溃。

### 🛑 3.2 解决方案：透明护盾
我们可以在整个屏幕的最上层，盖一个半透明（或全透明）的 `Box`，并让这个 `Box` **强行消费掉所有的点击事件**。这样，底层的按钮就接收不到点击了。

### 💻 3.3 详细代码实例：动画期间禁止操作
```kotlin
@Composable
fun BlockTouchScreen() {
    var isAnimating by remember { mutableStateOf(false) }
    val coroutineScope = rememberCoroutineScope()

    Box(modifier = Modifier.fillMaxSize()) {
        // 👇 底层 UI
        Column(modifier = Modifier.align(Alignment.Center)) {
            Button(onClick = { 
                isAnimating = true
                coroutineScope.launch {
                    delay(3000) // 模拟动画 3 秒
                    isAnimating = false
                }
            }) {
                Text("开始 3 秒动画")
            }
            
            Spacer(modifier = Modifier.height(20.dp))
            
            Button(onClick = { println("我被点击了！") }) {
                Text("测试底层按钮")
            }
        }

        // 🛡️ 顶层遮罩（只有在动画时才出现）
        if (isAnimating) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .background(Color.Black.copy(alpha = 0.5f)) // 半透明黑底
                    // ✨ 核心魔法：拦截所有点击手势，相当于“吞掉”了点击
                    .pointerInput(Unit) { 
                        detectTapGestures(
                            onTap = { /* 吞掉点击 */ },
                            onDoubleTap = { /* 吞掉双击 */ },
                            onLongPress = { /* 吞掉长按 */ }
                        ) 
                    } 
            ) {
                Text(
                    "动画播放中，禁止操作...", 
                    color = Color.White,
                    modifier = Modifier.align(Alignment.Center)
                )
            }
        }
    }
}
```