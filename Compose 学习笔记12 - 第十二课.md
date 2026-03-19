# 🎒 Compose 学习笔记12 - 第十二课 (字典状态与高级状态动画)

> 本节课的内容提取自“原神抽卡模拟器”实战项目。主要补充了复杂状态管理、极简动画 API 以及 Canvas 的高级画笔用法。

## 1. 字典状态：`mutableStateMapOf` 🗂️

### 🤔 1.1 为什么不用普通的 `mutableStateListOf`？
在第 10 课中，我们学过用 `mutableStateListOf` 来管理动态列表（比如任务列表）。
但是，在**背包系统**或**购物车系统**中，如果用户抽到了 10 把“无锋剑”，用 List 存的话，列表里会有 10 个重复的字符串，统计数量非常麻烦。
这时候，我们需要用**键值对（Key-Value）**的形式来存储：物品名字作为 Key，数量作为 Value。

### ✅ 1.2 正确姿势：可观察的字典
```kotlin
// 声明一个可观察的字典状态：Key 是 String (物品名)，Value 是 Int (数量)
val inventory = remember { mutableStateMapOf<String, Int>() }
```

### 💻 1.3 详细代码实例：背包物品添加逻辑
```kotlin
// 当抽到新物品时触发
val onItemPulled: (String) -> Unit = { item ->
    // 逻辑拆解：
    // 1. inventory[item] 获取当前数量，如果没抽到过就是 null
    // 2. ?: 0 表示如果是 null 就当做 0
    // 3. 最后 + 1，再存回字典里
    inventory[item] = (inventory[item] ?: 0) + 1 
}
```
**💡 核心机制**：当字典里的 Value 发生变化时，读取了 `inventory` 的 Compose 页面（比如背包的网格列表）会**自动触发重组（刷新 UI）**。

---

## 2. 极简状态动画：`animate*AsState` 🎨

### 🌀 2.1 什么是状态驱动动画？
在第 11 课中，我们学了 `Animatable`（目标值动画），它需要我们在协程里手动调用 `.animateTo()`。
而 `animateColorAsState`（以及 `animateDpAsState`、`animateFloatAsState` 等家族成员）是 Compose 中**最傻瓜式、最常用**的动画 API。你只需要改变状态变量的值，Compose 会自动接管过渡动画。

### ⚙️ 2.2 核心参数拆解
```kotlin
// 1. 定义一个普通的状态变量（比如当前颜色）
var currentColor by remember { mutableStateOf(Color.Blue) }

// 2. 声明动画状态（它会监听 currentColor 的变化）
val animatedColor by animateColorAsState(
    targetValue = currentColor, // 目标值：跟着 currentColor 走
    animationSpec = tween(600), // 动画配置：600毫秒平滑过渡
    label = "color_anim"        // 标签：用于在 Android Studio 动画预览中查看
)
```

### 💡 2.3 使用场景对比
*   **`Animatable`**：适合复杂的、需要精确控制（比如动画结束后的回调、物理弹簧效果）的场景。
*   **`animate*AsState`**：适合简单的状态切换（比如按钮点击后变色、展开/收起时的高度变化）。

---

## 3. 纯逻辑魔法：抽卡概率与保底机制 🎲

虽然这不是 Compose 特有的 API，但它是业务开发中极其重要的一环：**如何用 Kotlin 代码实现复杂的业务规则。**

### 🧠 3.1 核心逻辑拆解（原神抽卡机制）
1.  **软保底（Soft Pity）概率递增**：
    *   前 73 抽，出五星的概率是固定的 `0.6%`。
    *   从第 74 抽开始，每抽一次，概率增加 `6%`。
    ```kotlin
    val baseProb = 0.006f
    val softPity = 74
    // 计算当前这一抽的真实概率
    val prob5 = if (tempPity <= softPity) baseProb else baseProb + 0.06f * (tempPity - softPity)
    ```
2.  **大小保底切换（50/50 机制）**：
    *   需要一个布尔值状态 `var isGuaranteed by remember { mutableStateOf(false) }`。
    *   如果抽到了五星，判断 `isGuaranteed`。如果是 `true`，必定是当期限定（大保底），然后把状态重置为 `false`。
    *   如果是 `false`，则有 50% 概率歪常驻（小保底）。如果歪了，就把状态设为 `true`，保证下次必定不歪。
