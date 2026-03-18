# ⚡ Compose 学习笔记10 - 第十课 (异步操作与弹窗交互)

在抽卡游戏中，点击抽卡后通常会有几秒钟的“网络请求”或“动画播放”，然后再弹出结果。这就涉及到了 Compose 中的“协程（异步等待）”和“弹窗控制”。

---

## 1. 协程在 Compose 中的使用 (Coroutine) ⏳

在 Android 开发中，**主线程（UI 线程）绝对不能被阻塞**。如果你在按钮的点击事件里直接写 `Thread.sleep(3000)`，整个 App 就会卡死 3 秒钟，甚至导致崩溃（ANR）。
因此，我们需要使用**协程（Coroutine）**和挂起函数 `delay(3000)`。在 Compose 中，启动协程有两大神器：

### 👆 1.1 方式一：`rememberCoroutineScope()` (用于用户主动触发)
**适用场景**：当你想在**点击按钮、滑动屏幕**等用户主动操作时，触发一个耗时任务（如网络请求、等待动画）。

```kotlin
@Composable
fun DrawCardButton() {
    // 1. 获取当前组件的协程作用域
    val coroutineScope = rememberCoroutineScope()
    var buttonText by remember { mutableStateOf("点击抽卡") }
    var isDrawing by remember { mutableStateOf(false) }

    Button(
        onClick = {
            if (isDrawing) return@Button // 防止重复点击
            
            isDrawing = true
            buttonText = "正在连线经纪人..."
            
            // 🚀 2. 在点击事件中启动协程
            coroutineScope.launch {
                // 这里的代码在后台异步执行，不会卡死 UI
                delay(3000) // ⏳ 模拟耗时 3 秒
                
                // 3 秒后，继续执行以下代码
                buttonText = "抽卡完成！"
                isDrawing = false
            }
        }
    ) {
        Text(buttonText)
    }
}
```

### 🔄 1.2 方式二：`LaunchedEffect` (用于生命周期/状态驱动触发)
**适用场景**：当组件**刚刚出现在屏幕上**，或者**某个状态发生变化时**，你希望*自动*触发一个耗时任务。

> 💡 **核心参数 `key`：**
> `LaunchedEffect(key1)` 中的 `key1` 是触发器。
> *   **传 `Unit`**：`LaunchedEffect(Unit)`，这段协程只会在组件**第一次显示**时执行一次。
> *   **传变量**：`LaunchedEffect(someState)`，每次 `someState` 改变，旧协程会被取消，并启动新协程。

```kotlin
@Composable
fun AutoTimerScreen() {
    var timeLeft by remember { mutableStateOf(5) }

    // 🎬 因为传入了 Unit，所以这个协程只会在刚显示时启动一次
    LaunchedEffect(Unit) {
        while (timeLeft > 0) {
            delay(1000) // 等待 1 秒
            timeLeft -= 1 // 倒计时减 1
        }
    }

    Text("距离页面关闭还有: $timeLeft 秒", fontSize = 24.sp)
}
```

---

## 2. 状态驱动的弹窗 (AlertDialog) 💬

在传统的 Android (View 系统) 中，弹窗是命令式的：`dialog.show()` 和 `dialog.dismiss()`。
但在 Compose 中，UI 是状态的反映。弹窗的存在与否，完全取决于一个 Boolean 类型的状态变量。

### 🧠 2.1 核心思想
我们不“弹出”对话框，我们只是把 `showDialog` 设为 `true`。Compose 监测到状态变化，重新渲染 UI，发现 `if(showDialog)` 成立，于是把对话框**画**在了屏幕上。

### 💻 2.2 详细代码实例：金币不足提示窗
```kotlin
@Composable
fun RechargePrompt() {
    // 🎛️ 控制弹窗显示隐藏的核心状态
    var showDialog by remember { mutableStateOf(false) }

    Column {
        Button(onClick = { showDialog = true }) {
            Text("购买 10000 金币的球员")
        }

        // 👁️ 只有当状态为 true 时，才渲染 AlertDialog
        if (showDialog) {
            AlertDialog(
                // 🔙 onDismissRequest 是当用户点击弹窗外部，或按返回键时触发的回调
                onDismissRequest = { showDialog = false },
                title = { 
                    Text("余额不足", fontWeight = FontWeight.Bold) 
                },
                text = { 
                    Text("您的金币不足以支付本次签约，是否前往充值？") 
                },
                confirmButton = {
                    Button(onClick = { 
                        println("跳转到充值页面...")
                        showDialog = false // 关闭弹窗
                    }) {
                        Text("去充值")
                    }
                },
                dismissButton = {
                    TextButton(onClick = { showDialog = false }) {
                        Text("残忍拒绝", color = Color.Gray)
                    }
                }
            )
        }
    }
}
```

---

## 3. 动态列表状态：mutableStateListOf 📝

### ❌ 3.1 为什么不用普通的 mutableStateOf？
如果你用 `var list by remember { mutableStateOf(mutableListOf<String>()) }`，当你调用 `list.add("新球员")` 时，Compose **不会**刷新 UI。因为列表的“内存地址”没有变，Compose 以为数据没变。

### ✅ 3.2 正确姿势
对于需要频繁增删元素的列表（如背包、购物车），必须使用 `mutableStateListOf`。它内部做了特殊处理，只要列表内容发生增删改，就会立刻通知 UI 刷新。

```kotlin
@Composable
fun InventoryTest() {
    // ✨ 正确声明动态列表的方式
    val inventory = remember { mutableStateListOf<String>() }

    Column {
        Button(onClick = { inventory.add("梅西") }) {
            Text("抽中梅西")
        }
        
        Text("背包球员数量：${inventory.size}")
        inventory.forEach { player ->
            Text("- $player")
        }
    }
}
```
