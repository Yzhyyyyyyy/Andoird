# Compose 学习笔记07 - 第七课

# 页面切换与跳转 (状态驱动 vs Navigation)

在 Compose 中，让屏幕显示不同的内容通常有两种方式：一种是**“局部切换”**（比如微信底部的 4 个 Tab），另一种是**“完全跳转”**（比如从列表点进详情页）。今天我们把这两种方式一次性搞懂！🎉

---

## 一、 📱 状态驱动的局部切换 (底部导航栏最佳实践)

对于 App 的主界面（带有底部导航栏的页面），最经典、性能最好的写法**不是**用复杂的导航组件，而是全靠一个**“状态变量”**来当总司令。

### 1. 核心三步曲 🎵
*   **第一步：定义状态 (总司令)**
    使用 `mutableStateOf` 记住当前选中的是哪个页面。
*   **第二步：点击修改状态 (发号施令)**
    点击底部按钮时，把状态改成对应的页面。
*   **第三步：根据状态渲染 (执行命令)**
    使用 `when` 表达式，状态是什么，屏幕中间就画什么。

### 2. 代码模板 💻
```kotlin
// 1. 定义页面的枚举类（代表底部的几个分类）
enum class AppTab { Home, Message, Mine }

@Composable
fun MainScreen() {
    // 2. 定义状态，默认是首页 (总司令)
    var currentTab by remember { mutableStateOf(AppTab.Home) }

    Scaffold(
        bottomBar = {
            NavigationBar {
                // 3. 点击时修改状态 (向总司令报告)
                NavigationBarItem(
                    selected = currentTab == AppTab.Home,
                    onClick = { currentTab = AppTab.Home },
                    icon = { Icon(Icons.Default.Home, contentDescription = null) },
                    label = { Text("首页") }
                )
                // ... 其他按钮省略
            }
        }
    ) { paddingValues ->
        // 4. 根据状态，决定屏幕中间显示哪个组件！(执行命令)
        Box(modifier = Modifier.padding(paddingValues)) {
            when (currentTab) {
                AppTab.Home -> HomeScreen()
                AppTab.Message -> MessageScreen()
                AppTab.Mine -> MineScreen()
            }
        }
    }
}
```
**💡 优点：** 性能极高，代码简单！因为没有加入“返回栈”，所以按手机返回键不会在这几个 Tab 之间来回退，非常符合主流 App 的底部导航逻辑。

---

## 二、 🚕 真正的页面跳转 (Navigation 组件)

如果我们需要从当前页面**完全跳转**到一个全新的页面（比如从“商城列表页”跳转到“商品详情页”，并且按手机的返回键能退回来），我们就必须使用 Compose 的官方导航组件 —— **Navigation**。

### 1. 导航的“三剑客” ⚔️
我们可以把页面跳转想象成**“打车”**的过程：
1. **`NavController`（导航控制器）**：相当于**“出租车司机”**。负责执行真正的跳转动作，以及记录你走过的路线（返回栈）。
2. **`NavGraph` / `NavHost`（导航地图）**：相当于**“城市地图”**。规定了 App 一共有哪些页面，每个页面的“名字（路由）”是什么。
3. **`Destination`（目的地）**：相当于地图上的**“具体景点”**。也就是你写的每一个 `@Composable` 页面。

### 2. 基础使用步骤 🛠️

**引入依赖：**
```gradle
implementation("androidx.navigation:navigation-compose:2.7.7")
```

**核心代码实现：**
```kotlin
@Composable
fun MyApp() {
    // 第一步：请一位“司机” (创建 NavController)
    val navController = rememberNavController()

    // 第二步：展开“地图” (创建 NavHost)
    NavHost(navController = navController, startDestination = "HomeScreen") {
        
        // 在地图上标记第一个目的地：主页
        composable("HomeScreen") {
            HomeScreen(navController)
        }

        // 在地图上标记第二个目的地：详情页
        composable("DetailScreen") {
            DetailScreen(navController)
        }
    }
}

@Composable
fun HomeScreen(navController: NavController) {
    Button(onClick = { 
        // 第三步：告诉司机目的地，开始跳转！
        navController.navigate("DetailScreen") 
    }) {
        Text("跳转到详情页")
    }
}
```

---

## 三、 ⚠️ 核心避坑指南 (新手必看)

### 1. 🎯 到底用哪个？(状态 vs Navigation)
* **底部导航栏切换（如微信底部的 4 个 Tab）**：推荐用 `when(状态)` 切换，轻量且符合交互逻辑。
* **层级递进跳转（如点开朋友圈 -> 点开某条动态的详情）**：**必须**用 `Navigation`，因为你需要按返回键能一层层退回来。

### 2. 📍 路由名字 (Route) 是个字符串
Navigation 代码里的 `"HomeScreen"` 就是路由（Route）。它们就像网址一样，千万不要拼错，拼错了司机就找不到路，App 会直接崩溃。

### 3. 📦 状态管理大坑：列表不刷新怎么办？(复习)
* 监听**普通变量**变化：用 `mutableStateOf()`
* 监听**列表增删改**：**千万别用** `mutableStateOf(mutableListOf())`，因为大爷（Compose）只认箱子不认里面的东西，增删数据不会刷新 UI！**必须直接用** `mutableStateListOf()`！
