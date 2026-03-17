# 🚀 Compose 学习笔记06 - 第六讲（点击事件与状态提升终极备忘录）

在 Jetpack Compose 中，处理点击事件和传递参数的核心思想是：**“数据向下流，事件向上抛”**（状态提升 State Hoisting）。

一个完整的交互闭环由 **“打工人（底层组件）”** 和 **“老板（主页面）”** 共同完成。

---

## 💡 核心三步曲

### 第一步：打工人“挖坑”（声明参数）
在封装的卡片组件（如 `GameCard`）的函数签名中，定义一个函数类型的参数 `onClick`。
*   **无参数：** `onClick: () -> Unit`
*   **带参数：** `onClick: (String) -> Unit`（承诺点击时会传出一个字符串）

### 第二步：打工人“触发”（Modifier.clickable）
给卡片的最外层容器（如 `Card` 或 `Row`）添加 `Modifier.clickable` 属性。
当用户点击时，调用老板传进来的 `onClick`，并把自己的数据（如 `gameName`）塞进去传给老板。

### 第三步：老板“填坑”（接收并处理）
在主页面（如 `MainScreen`）调用 `GameCard` 时，在大括号 `{}` 中接收打工人抛上来的参数，并执行真正的业务逻辑（如跳转页面、修改状态）。

---

## 🌟 核心代码剖析（含参数传递）

只要看懂这三行灵魂代码，带参数的点击事件就彻底通关了：

**1. 声明坑位（打工人内部）：**
```kotlin
onClick: (String) -> Unit
```
> **解释**：这叫“函数类型”。它就像一个带有入口的管道，`(String)` 代表必须往里面塞一个字符串，`-> Unit` 代表不需要返回结果。打工人告诉老板：“给我传一段代码，这段代码得能接收一个 String”。

**2. 触发并传参（打工人内部的 clickable 中）：**
```kotlin
onClick(gameName)
```
> **解释**：当玩家手指点下时，打工人拿出老板给的管道，并**把自己的数据（gameName）塞进括号里**。这一瞬间，字符串就像子弹一样顺着管道发射回了主页面！

**3. 填坑并接收（老板 Main 页面中）：**
```kotlin
onClick = { name -> 
    clickedGameName = name 
}
```
> **解释**：老板在调用组件时，把具体的处理逻辑（管道）传下去。`{ name -> ... }` 里的 `name`，就是用来**精准接住**打工人发射上来的那个字符串！接住后，老板就可以拿它去修改状态或跳转页面了。

---

## 💻 完整代码模板

### 1. 底层组件：GameCard.kt (打工人)

```kotlin
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Card
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun GameCard(
    gameName: String,       // 外部传入的数据（打工人自己的数据）
    description: String,    // 外部传入的数据
    onClick: (String) -> Unit // 👈 核心1：挖坑（声明带有 String 参数的点击事件）
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            // 👈 核心2：触发（万物皆可点击）
            .clickable { 
                // 将自己的 gameName 作为参数，向上抛给老板
                onClick(gameName) 
            }
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(text = gameName)
            Text(text = description)
        }
    }
}
```

### 2. 主页面：MainScreen.kt (老板)

```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.material3.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun MainScreen() {
    // 记录当前点击的游戏名称（状态）
    var clickedGameName by remember { mutableStateOf("还没点击任何游戏") }

    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "老板收到的汇报：$clickedGameName",
            modifier = Modifier.padding(bottom = 16.dp)
        )

        // 👈 核心3：填坑（调用组件并处理事件）
        GameCard(
            gameName = "原神",
            description = "开放世界冒险RPG",
            onClick = { name -> 
                // 接收到底层传来的 name，并更新状态
                clickedGameName = name 
                // 真实项目中这里通常是：navController.navigate("detail/$name")
            }
        )

        GameCard(
            gameName = "崩坏：星穹铁道",
            description = "银河冒险策略游戏",
            onClick = { name -> 
                clickedGameName = name 
            }
        )
    }
}
```

---

## 🎯 总结与口诀

*   **`()` 决定性质**：修饰符 `Modifier`、颜色、大小、点击事件（`onClick`）都在小括号里。
*   **`{}` 决定内容**：UI 的嵌套结构（如 `Column` 里放 `Text`）都在大括号里。
*   **组件要纯粹**：底层组件不要自己做决定（比如直接写死跳转路由），要把决定权通过 `onClick` 交给调用它的父页面。这就是**状态提升**！
