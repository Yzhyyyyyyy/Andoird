# 💾 Compose 学习笔记14 - 第十四课 (三大秘密武器集结与数据流转架构)

## 🌟 核心奥义（一语道破天机）
> **“记忆的实现就是利用数据库和提前写好的 Dao 函数，在开始时提取数据，再利用 LaunchedEffect 画出这些持续性信息的 UI。”** 
> —— 这是对现代 Android 数据流转最精准的总结！

---

## 武器一：Room 数据库（永久记忆 / 地下金库）
存在手机内部存储（ROM）中，App 杀掉、手机重启数据都在。

### 1. 表结构定义 (`@Entity`)
```kotlin
@Entity(tableName = "habits")
data class Habit(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String,       // 习惯名称，如 "每天喝水"
    val exp: Int,           // 经验值奖励
    
    // 状态记录
    val isCompletedToday: Boolean = false, 
    val streakDays: Int = 0,
    
    // 🌟 核心时间锚点
    val lastCompletedDate: String = "" 
)
```

### 🤔 你的疑问 1：这相当于一个 struct，我要存的习惯在哪里？
**💡 解答**：`data class` 确实就是 Kotlin 里的 struct（结构体）。你要存的具体习惯（比如“每天喝水”、“跑步”），其实就是作为**字符串（String）**存放在了 `name` 这个字段里。我们是用这个模版去实例化一个个具体的习惯。

### 🤔 你的疑问 2：数据是不是需要自己更新？日期得变啊！
**💡 解答**：非常敏锐的架构思维！这就是著名的**“跨天重置”**问题。数据库本身是死的，所以我们加入了 `lastCompletedDate`（最后打卡日期）。当 `LaunchedEffect` 触发 ViewModel 去拿数据时，ViewModel 会检查这个日期。如果发现“最后打卡日期”不是“今天”，就会触发业务逻辑，把 `isCompletedToday` 重新设为 `false`。

---

## 武器二：操作手册 (`@Dao`)

### 🤔 你的疑问 3：Dao 一读取就把所有的都读取了，是不是需要分开写几个来读取不同的信息？
**💡 解答**：天生的性能优化大师！如果用户有 10000 条数据，全量读取会导致内存爆炸（OOM）。所以我们必须在 Dao 里写精细化的 SQL 查询，让数据库帮我们筛选好再拿出来：

```kotlin
@Dao
interface HabitDao {
    // 基础：读取全部（慎用）
    @Query("SELECT * FROM habits")
    suspend fun getAllHabits(): List<Habit>

    // 🌟 精细化读取：只拿今天没完成的（极大地节省内存，UI 直接用）
    @Query("SELECT * FROM habits WHERE isCompletedToday = 0")
    suspend fun getPendingHabits(): List<Habit>

    // 🌟 数据库直接计算：今天总共赚了多少经验？（不用拿回内存里用 for 循环加）
    @Query("SELECT SUM(exp) FROM habits WHERE isCompletedToday = 1")
    suspend fun getTodayTotalExp(): Int?
    
    @Insert
    suspend fun insertHabit(habit: Habit)
}
```

---

## 武器三：ViewModel 与 LaunchedEffect（大堂经理与迎宾员）

### 🤔 你的疑问 4：Dao 读取的文件怎么使用？是直接加 `.` 就可以访问了吗？
**💡 解答**：完全正确！Room 框架的魔法就在于，它会自动把底层冷冰冰的表格数据，翻译成你熟悉的 `Habit` 对象。拿到对象后，直接用 `.` 访问即可。

```kotlin
@Composable
fun HabitScreen(viewModel: HabitViewModel) {
    
    // 🛎️ 迎宾员：只要页面不销毁，里面的代码只执行一次！
    // 完美呼应总结：“在开始时提取数据”
    LaunchedEffect(Unit) {
        viewModel.loadHabits() 
    }

    LazyColumn {
        // 遍历 ViewModel 里的列表
        items(viewModel.habitList) { rowItem -> 
            // 🌟 拿到单个对象后，直接用 . 访问属性！
            // 完美呼应总结：“画出这些持续性信息的 UI”
            Row {
                Text(text = rowItem.name) 
                Text(text = "+ ${rowItem.exp} 经验") 
                
                if (rowItem.isCompletedToday) {
                    Icon(Icons.Default.Check, contentDescription = "已完成")
                }
            }
        }
    }
}
```

---

## 🔄 终极数据流转全景
1. **开机** -> `LaunchedEffect` 触发。
2. **跑腿** -> `ViewModel` 调用 `Dao`。
3. **查询** -> `Dao` 从 `Room` (SQLite) 中精准提取数据，打包成 `Habit` 对象。
4. **展示** -> `ViewModel` 拿到列表，Compose UI 遍历列表，用 `.` 提取 `name` 和 `exp` 画在屏幕上。
5. **操作** -> 用户点击打卡，`ViewModel` 更新 `lastCompletedDate` 并存回 `Room`。