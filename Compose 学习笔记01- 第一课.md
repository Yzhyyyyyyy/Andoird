# 零基础 Compose 学习笔记 - 第一课：基础骨架与跳转机制

## 1. 代码的基础骨架
每个 Compose 界面的文件开头通常包含以下部分：

*   **`package` (门牌号)**：标明这段代码属于哪个项目（例如 `package com.example.demo001`）。
*   **`import` (工具箱)**：引入需要用到的现成工具（如 `Text`, `Button`, `Color`）。在 Android Studio 中，输入代码时通常会自动补全，或者使用快捷键 `Alt + Enter` (Mac: `Option + Return`) 自动导入。
*   **`@Composable` (魔法标签)**：**最核心的标记！** 加在函数上面，表示这是一个可以画在屏幕上的“UI 组件”（乐高积木），而不是普通的计算函数。

## 2. 核心机制：回调函数（事件上报）
在 Compose 中，UI 组件遵循**“只负责展示和喊话，不负责做决定”**的原则。

### 💡 “机器与电线”的比喻
*   **造机器（定义组件）**：
    ```kotlin
    @Composable
    fun MyFirstScreen(onStartClick: () -> Unit = {}) { 
        // ... 界面内容
    }
    ```
    这里的 `onStartClick` 就是给这个界面留的一根**外接信号线**。
    *   `() -> Unit`：表示这根线只负责传递“触发动作”，不传递具体数据。
    *   `= {}`：表示如果没有人接线，默认什么都不做（防止程序崩溃）。

*   **按开关（触发事件）**：
    ```kotlin
    Button(onClick = { 
        onStartClick() // 触发信号！
    }) { 
        Text("选择游戏") 
    }
    ```
    当用户点击按钮时，组件内部通电，顺着 `onStartClick` 这根电线向外发送信号。

*   **接线（外部调用与执行）**：
    ```kotlin
    // 在大管家 (MainActivity) 中使用这个组件时，接上真正的执行逻辑
    MyFirstScreen(
        onStartClick = {
            // 这里写真正的跳转代码，比如：
            // navController.navigate("选择游戏页面")
        }
    )
    ```

### 为什么要这么做？
为了**极高的复用性**！把“界面长什么样”和“点击后干什么”彻底分开。这样同一个精美的界面，可以在不同的地方重复使用，只需在外部接上不同的“电线”即可。
