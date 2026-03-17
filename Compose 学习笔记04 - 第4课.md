# Compose 学习笔记04 - 第4课：实战 MainActivity 与页面路由

## 1. 核心思想：状态驱动 UI (State-Driven UI)

在传统的 Android 开发中，页面跳转通常需要用到 `Intent` 和 `startActivity`，非常繁琐。
但在 Compose 中，**页面跳转本质上只是“状态的改变”**。

我们不需要“打开”一个新页面，我们只需要告诉 Compose：“现在的状态变了，请把屏幕清空，画出新状态对应的 UI”。

## 2. 代码架构拆解

在我们的 `MainActivity.kt` 中，整个 App 的导航系统由三个核心部分组成：

### 第一部分：定义“页面字典” (Enum Class)
```kotlin
enum class AppScreen {
    Start, Choose, Efootball, Help, StarRail, Genshin, WutheringWaves, Arknights, PeaceElite
}
```
**💡 为什么用 Enum 而不是直接用 String（字符串）？**
*   **防呆设计**：如果手滑把 `"Genshin"` 打成了 `"Genshin "`（多了一个空格），编译器会立刻报错，而不是等到运行 App 时才崩溃。
*   **代码提示**：输入 `AppScreen.` 时，IDE 会自动弹出所有可选的页面，开发体验极佳。

### 第二部分：定义“魔法变量” (State)
```kotlin
var currentScreen by remember { mutableStateOf(AppScreen.Start) }
```
这是 Compose 导航的灵魂代码：
*   `mutableStateOf`：创建了一个**响应式变量**。只要它的值发生改变，Compose 就会立刻触发**重组（Recomposition）**，重新执行这部分代码。
*   `remember`：告诉 Compose “记住”这个变量的值。如果不加 `remember`，每次屏幕刷新时，状态都会被重置回 `Start`。

### 第三部分：总指挥部 (The Router)
`MainApp()` 函数就像是一个聪明的“交通警察”，它通过 `when` 表达式来检查当前的 `currentScreen` 状态，并决定把哪个页面（打工人）派到屏幕上。

```kotlin
when (currentScreen) {
    AppScreen.Start -> {
        MyFirstScreen(
            onStartClick = { currentScreen = AppScreen.Choose }
        )
    }
    AppScreen.Choose -> {
        GameSelectionScreen(
            onNavigate = { gameId ->
                when (gameId) {
                    "genshin" -> currentScreen = AppScreen.Genshin
                    "peace_elite" -> currentScreen = AppScreen.PeaceElite
                    // ... 其他游戏路由
                }
            }
        )
    }
    // ... 其他页面
}
```

## 3. 巧妙的“对讲机”机制 (回调函数)

在 `Choose` 页面中，`GameSelectionScreen` 接收了一个带参数的回调函数 `onNavigate = { gameId -> ... }`。

*   **打工人 (GameSelectionScreen)**：只负责展示 UI 和捕获点击事件，当用户点击“原神”时，它通过对讲机喊话：`onNavigate("genshin")`。
*   **老板 (MainApp)**：收到呼叫后，根据传来的 `gameId`，在内部的 `when` 表达式中将 `currentScreen` 修改为 `AppScreen.Genshin`。
*   **结果**：状态改变，Compose 自动清空屏幕，重新执行 `MainApp`，由于状态变成了 `Genshin`，于是屏幕上瞬间渲染出了 `GenshinScreen`。

## 4. 总结

这种架构非常清晰地实现了**“UI 与逻辑分离”**（单向数据流 Unidirectional Data Flow）：
1.  **状态向下流动**：`MainApp` 决定显示哪个 Screen。
2.  **事件向上冒泡**：Screen 里的按钮被点击后，通过回调函数把事件传回给 `MainApp`。

这是一种非常经典且优雅的 Compose 页面路由写法，完美适用于中小型应用！你的代码结构已经完全达到了专业 Android 开发者的标准！