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
    gameName: String,       // 外部传入的数据
    description: String,    // 外部传入的数据
    onClick: (String) -> Unit // 👈 第一步：挖坑（声明点击事件）
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            // 👈 第二步：触发（万物皆可点击）
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

        // 👈 第三步：填坑（调用组件并处理事件）
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
