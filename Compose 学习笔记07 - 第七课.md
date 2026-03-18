# Compose 学习笔记07 - 第七课

# 页面跳转的使用 (Navigation)

在之前的项目中，我们学会了使用 `when(状态)` 来实现底部菜单的局部切换。但是，如果我们需要从当前页面**完全跳转**到一个全新的页面（比如从“商城列表页”跳转到“商品详情页”，并且按手机的返回键能退回来），我们就必须使用 Compose 的官方导航组件 —— **Navigation**。

在 Compose 中，页面跳转不再需要像老 Android 那样新建 `Activity` 或使用 `Intent`，一切都在 `@Composable` 函数之间完成！

---

## 一、 导航的“三剑客”

要把页面跳转玩转，你只需要记住三个核心概念，我们可以把它们想象成**“打车”**的过程：

1. **`NavController`（导航控制器）**：相当于**“出租车司机”**。它负责执行真正的跳转动作，以及记录你走过的路线（返回栈）。
2. **`NavGraph` / `NavHost`（导航地图）**：相当于**“城市地图”**。在这里规定了你的 App 一共有哪些页面，每个页面的“名字（路由）”是什么。
3. **`Destination`（目的地）**：相当于地图上的**“具体景点”**。也就是你写的每一个 `@Composable` 页面。

---

## 二、 基础使用步骤

### 1. 准备工作：引入依赖
在使用之前，需要确保 `build.gradle` 中引入了 Navigation 的库（如果新建项目时选了 Empty Compose Activity 通常需要手动加）：
```gradle
implementation("androidx.navigation:navigation-compose:2.7.7") // 版本号可能随时间更新
```

### 2. 核心代码实现

我们来写一个最简单的例子：从“主页 (Home)”跳转到“详情页 (Detail)”。

```kotlin
import androidx.compose.runtime.Composable
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.NavController

// ================= 1. 定义总的 App 入口 =================
@Composable
fun MyApp() {
    // 第一步：请一位“司机” (创建 NavController)
    val navController = rememberNavController()

    // 第二步：展开“地图” (创建 NavHost)
    // startDestination 指定了 App 打开时的第一个页面
    NavHost(navController = navController, startDestination = "HomeScreen") {
        
        // 在地图上标记第一个目的地：主页
        composable("HomeScreen") {
            // 把司机传给主页，主页里才有能力按按钮叫司机开车
            HomeScreen(navController)
        }

        // 在地图上标记第二个目的地：详情页
        composable("DetailScreen") {
            DetailScreen(navController)
        }
    }
}

// ================= 2. 具体的页面 =================

@Composable
fun HomeScreen(navController: NavController) {
    Column(modifier = Modifier.fillMaxSize()) {
        Text("这里是主页")
        
        Button(
            onClick = { 
                // 第三步：告诉司机目的地，开始跳转！
                navController.navigate("DetailScreen") 
            }
        ) {
            Text("跳转到详情页")
        }
    }
}

@Composable
fun DetailScreen(navController: NavController) {
    Column(modifier = Modifier.fillMaxSize()) {
        Text("这里是详情页")
        
        Button(
            onClick = { 
                // 让司机掉头，返回上一个页面
                navController.popBackStack() 
            }
        ) {
            Text("返回上一页")
        }
    }
}
```

---

## 三、 核心避坑指南 (⚠️ 重点)

1. **不要在底层组件里创建 `NavController`**：
   `rememberNavController()` **绝对应该**放在整个 App 的最顶层（比如 `MainActivity` 调用的第一个 Composable 里）。然后通过参数 `navController: NavController` 一层层传给需要跳转的子页面。

2. **路由名字 (Route) 是个字符串**：
   代码里的 `"HomeScreen"` 和 `"DetailScreen"` 就是路由（Route）。它们就像网址一样，千万不要拼错，拼错了司机就找不到路，App 会直接崩溃。

3. **状态驱动 vs Navigation 跳转**：
   * **底部导航栏切换（微信底部的 4 个 Tab）**：推荐用 `when(状态)` 切换，性能更好，不需要加入返回栈。
   * **层级递进跳转（点开朋友圈 -> 点开某条动态的详情）**：**必须**用 `Navigation`，因为你需要按返回键能一层层退回来。