# Jetpack
Jetpack 用于给安卓设计 UI 界面，使用kotlin风格编程。可以跨平台设计UI，到跨平台性不如react。

Jetpack 四大组件分别是架构组件(Architecture)、行为组件(Behavior)、UI 组件和基础组件(Foundation)。后面三部分是对之前已有的功能进行的整理，下图是2018年版本。
![[Jetpack-four-components-collections.png]]
```java
class Aechitecture{
/**
	Lifecycle：具备宿主生命周期感知能力的组件特性：持有组件(如 Activity 或 Fragment)生命周期状态的信息，并且允许其他对象观察此状态；
	
	LiveData：新一代具备生命周期感知能力的数据订阅、分发组件特性：支持共享资源、支持黏性事件的分发、不再需要手动处理生命周期、确保界面符合数据状态；
	
	ViewModel：具备生命周期感知能力的数据存储组件特性：页面因配置变更导致的重启，此时数据不丢失；可以实现跨页面(跨 Activity)的数据共享；
	
	SavedState 架构组件原理解析特性：因内存不足，电量不足导致页面被回收时可以搭配 ViewModel 实现数据存储与恢复；
	
	Room：轻量级 orm 数据库，本质上是一个 SQLite 抽象层特性：使用简单(类似于 Retrofit 库)，通过注解的方式实现相关功能，编译时自动生成相关实现类
	
	DataBinding：只是一种工具，解决的是 View 和数据之间的双向绑定特性：支持数据与视图双向绑定、数据绑定空安全、减少模板代码、释放 Activity/Fragment 压力；
	
	Paging: 列表分页组件，可以轻松完成分页预加载以达到无限滑动的效果特性：巧妙融合 LiveData、提供多种数据源加载方式；不足之处：不支持列表数据增删改，列表添加 HeaderView，FooterView 定位不准确；
	
	Navigation 组件原理分析：端内统一路由组件特性：能够为 Activity，Fragment，Dialog，FloatWindow 提供统一的路由导航服务，可以传递参数，指定导航动画，还支持深度链接等主要能力；不足：十分依赖 xml 配置文件不利于组件化，模块化
	
	WorkManager：新一代后台任务管理组件，service 能做的事情它都能做特性：支持周期性任务调度、链式任务调度、丰富的任务约束条件、程序即便退出，依旧能保证任务的执行；
*/
}
class Foundation {
/**
	Android KTX：优化了供 Kotlin 使用的 Jetpack 和 Android 平台 API，帮助开发者以更简洁、更愉悦、更惯用的方式使用 Kotlin 进行 Android 开发；
	
	AppCompat：帮助较低版本的 Android 系统进行兼容；
	
	Auto：开发 Android Auto 应用的组件，提供了适用于所有车辆的标准化界面和用户交互；
	
	检测：从 AndroidStudio 中快速检测基于 Kotlin 或 Java 的代码；
	
	多 Dex 处理：为具有多个 Dex 文件应用提供支持；
	
	安全：安全的读写加密文件和共享偏好设置；
	
	测试：用于单元和运行时界面测试的 Android 测试框架；
	
	TV：构建可让用户在大屏幕上体验沉浸式内容的应用；
	
	Wear OS：开发 Wear 应用的组件；
*/
} 
class Behavior {
/**
	CameraX：帮助开发简化相机应用的开发工作，提供一致且易于使用的界面，适用于大多数 Android 设备，并可向后兼容至 Android 5.0(API 21)；
	
	DownloadManager：处理长时间运行的 HTTP 下载的系统服务；
	
	媒体和播放：用于媒体播放和路由(包括 Google Cast)的向后兼容 API；
	
	通知：提供向后兼容的通知 API，支持 Wear 和 Auto；
	
	权限：用于检查和请求应用权限的兼容性 API；
	
	设置：创建交互式设置，建议使用 AndroidX Preference Library 库将用户可配置设置集成到应用中；
	
	分享操作：可以更轻松地实现友好的用户分享操作；
	
	切片：切片是一种 UI 模板，创建可在应用外部显示应用数据的灵活界面元素；
*/
}
class UI {
/**
	Animation and Transition：该框架包含用于常见效果的内置动画，并允许开发者创建自定义动画和生命周期回调；
	
	Emoji Compatibility：即便用户没有更新 Android 系统也可以获取最新的表情符号；
	
	Fragment：组件化界面的基本单位；
	
	布局：用 XML 中声明 UI 元素或者在代码中实例化 UI 元素；
	
	调色板：从调色板中提取出有用的信息；
*/
}
```

| 类别        | 说明                              | 代表性组件                                                                                         |
| :-------- | :------------------------------ | :-------------------------------------------------------------------------------------------- |
| **架构组件**  | **最核心的部分**，用于构建稳健、可测试、可维护的应用架构。 | `ViewModel`, `LiveData`, `Room`, `DataBinding`                                                |
| **UI 组件** | 用于构建和优化应用界面。                    | `Fragment`, `Layout`（ConstraintLayout等）, `EmojiCompat`,`RecyclerView`、`Paging`、`Navigation` 等 |
| **行为组件**  | 让应用能更好地与 Android 系统及其他应用交互。     | `WorkManager`, `Notifications`                                                                |
| **基础组件**  | 提供向下兼容的功能和 Kotlin 语言支持。         | `AppCompat`, `Android KTX`, `Multidex`, `Test`, 安全等                                           |
组件之间的关系图：
![[Jetpack-four-components-relation.png]]



## 架构组件
![[Jetpack-core-components.png]]
### 1. ViewModel 和 LiveData
#### a. ViewModel
关于ViewModel的生命周期就一句话：**在Activity、Fragment等组件整个生命周期过程中，ViewModel的实例有且只有一个。**
*   **作用**：该类负责为界面准备数据，以注重生命周期的方式存储和管理界面相关的数据。
*   **解决痛点**：
    *   屏幕旋转时，数据不会丢失。
    *   将数据和逻辑从 Activity/Fragment 中分离出来，使它们更专注于绘制 UI。
    *   能很好的实现多个Fragment之间的数据共享。
#### b. LiveData
LiveData只通知活跃状态( `STARTED` or `RESUMED` )的Observer更新，并在 `DESTROYED` 状态时自动移除Observers，来避免内存泄漏。
*   **作用**：一种可观察的数据存储器类，具有生命周期感知能力。这意味着它只在 Activity、Fragment 或 Service 处于活动状态时才更新 UI。
*   **解决痛点**：
	*   避免手动更新 UI 时可能引发的内存泄漏。
	*   确保 UI 与数据状态同步。
```alert type=note
注意：
1. 您必须调用 setValue(T) 方法以从主线程更新 LiveData 对象。
2. 如果在 worker 线程中执行代码，则您可以改用 postValue(T) 方法来更新 LiveData 对象。
```
##### b.1 LiveData、MutableLiveData、MediatorLiveData

|       父类        |      Class       | 作用                           |
| :-------------: | :--------------: | :--------------------------- |
|        -        |     LiveData     | 一个具有生命周期感知的可观察的数据保持类。        |
|    LiveData     | MutableLiveData  | 在LiveData基础上打开了修改Value的方法权限。 |
| MutableLiveData | MediatorLiveData | 可管理多个LiveData。               |
##### b.2 Transformations
通过使用 Transformations的map()和switchMap()，可以实现一个值刷新之后，带动其它值刷新。
```java
// 示例代码1：观察将被转换LiveData<User>，待其数据源变更后转换为LiveData<String>并通知订阅者。
LiveData<User> userLiveData = ...;
LiveData<String> userName = Transformations.map(userLiveData, user -> {
	user.name + " " + user.lastName
});

// 示例代码2: addressInput 刷新会带动 postalCode 从数据库中刷新
private final PostalCodeRepository repository;
private final MutableLiveData<String> addressInput = new MutableLiveData();
public final LiveData<String> postalCode =
	Transformations.switchMap(addressInput, (address) -> {
		return repository.getPostCode(address);
	});
```

#### c. ViewModel 和 LiveData 最佳实践
```kotlin
class CounterViewModel : ViewModel() {
	// 使用 MutableLiveData 作为可变的底层数据
	private val _count = MutableLiveData<Int>().apply { value = 0 }
	// 对外暴露不可变的 LiveData，防止外部篡改
	val count: LiveData<Int> = _count
	
	fun increment() {
		_count.value = _count.value?.plus(1) // 更新 LiveData 的值
	}
}
  ```

```kotlin
class MainActivity : AppCompatActivity() {
	private val viewModel: CounterViewModel by viewModels()
	
	override fun onCreate(savedInstanceState: Bundle?) {
		// ... 初始化视图
		
		// 观察者1 LiveData！
		viewModel.count.observe(this) { newCount ->
			// 这个 lambda 表达式会在 count 的值改变时被调用
			// `this` 是 LifecycleOwner (Activity/Fragment)
			countTextView.text = newCount.toString()
		}
		// 观察者2 LiveData！
		viewModel.count.observe(this) { newCount ->
			// ... 多次调用 observe() 可以设置多个观察者
		}
		
		incrementButton.setOnClickListener {
			viewModel.increment() // 只需调用方法，UI 会自动更新
		}
	}
}
 ```

### 2. Data Binding
**在纯 Jetpack Compose 项目中，传统的 View 系统 Data Binding 库不被使用**。Compose 内置了响应式数据绑定的能力，不需要额外的 Data Binding 库。
Compose 中数据绑定存在于**数据与界面绑定**和**数据与ViewModel绑定**。Compose 数据绑定最佳实践：
#### a. Compose 内置数据绑定
```kotlin
@Composable
fun Counter() {
    // State 就是被观察的数据
    var count by remember { mutableStateOf(0) }
    
    Column {
        // 当 count 变化时，Text 会自动重新绘制 - 这就是数据绑定！
        Text(text = "Count: $count")
        
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}
```
#### b. Compose 与 ViewModel 的数据绑定
```kotlin
class MyViewModel : ViewModel() {
    // LiveData 或 StateFlow 作为数据源
    // !!!这里分两步声明变量用户名，不带下划线的变量把用户名都修改属性给隐藏掉了
    private val _userName = MutableLiveData("John")
    val userName: LiveData<String> = _userName
    
    // 或者使用 StateFlow（更推荐用于 Compose）
    private val _email = MutableStateFlow("john@example.com")
    val email: StateFlow<String> = _email
    
    fun updateUser(newName: String) {
        _userName.value = newName
    }
}

@Composable
// 使用 Hilt 框架自动注入 MyViewModel
fun UserProfile(viewModel: MyViewModel = hiltViewModel()) {
    // 方式1：观察 LiveData
    val userName by viewModel.userName.observeAsState()
    
    // 方式2：观察 StateFlow（更推荐）
    val email by viewModel.email.collectAsState()
    
    Column {
        // 自动绑定到 LiveData/StateFlow
        Text("Name: $userName")  // 当 LiveData 变化时自动更新
        Text("Email: $email")    // 当 StateFlow 变化时自动更新
        
        Button(onClick = { 
            viewModel.updateUser("Jane") 
        }) {
            Text("Update Name")
        }
    }
}
```
#### c. 传统数据绑定与 Compose 数据绑定对比
- 传统方式
XML 文件配置：
```xml
<!-- activity_main.xml -->
<layout>
    <data>
        <variable name="viewModel" type="com.example.MyViewModel" />
    </data>
    
    <LinearLayout>
        <TextView
            android:text="@{viewModel.userName}"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <Button
            android:onClick="@{() -> viewModel.updateUser()}"
            android:text="Update" />
    </LinearLayout>
</layout>
```
Kotlin 代码：
```kotlin
// MainActivity.kt
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.viewModel = ViewModelProvider(this).get(MyViewModel::class.java)
        binding.lifecycleOwner = this
    }
}
```

- Jetpack Compose 方式
```kotlin
@Composable
fun MainScreen(viewModel: MyViewModel = hiltViewModel()) {
    Column {
        Text(text = viewModel.userName)
        Button(onClick = { viewModel.updateUser() }) {
            Text("Update")
        }
    }
}

// Activity 中：
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                MainScreen()
            }
        }
    }
}
```
#### d. Compose 数据绑定特性：单向、双向、自定义数据绑定
- 单项数据绑定 - 可用于一个负责展示数据，其它组件对这个数据进行修改的场景。
```kotlin
@Composable
fun TodoApp() {
    var todos by remember { mutableStateOf(emptyList<Todo>()) }
    
    Column {
        // 数据显示
        TodoList(todos = todos, onTodoClick = { todo ->
            // 事件处理
            todos = todos.map { if (it.id == todo.id) it.copy(completed = !it.completed) else it }
        })
        
        // 数据输入
        AddTodoButton { newTodo ->
            todos = todos + newTodo
        }
    }
}
```
- 双向数据绑定  - 可应用于类似文本框等实时显示的组件。
```kotlin
@Composable
fun LoginForm() {
    var username by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    
    Column {
        // 类似双向绑定：UI 变化更新状态，状态变化更新 UI
        OutlinedTextField(
            value = username,
            onValueChange = { username = it },  // 双向绑定
            label = { Text("Username") }
        )
        
        OutlinedTextField(
            value = password, 
            onValueChange = { password = it },  // 双向绑定
            label = { Text("Password") }
        )
    }
}
```
- 自定义数据绑定
	在传统 Data Binding 中用 `@BindingAdapter`，在 Compose 中直接用 Composable 函数：
```kotlin
// 传统 Data Binding：
@BindingAdapter("imageUrl")
fun loadImage(view: ImageView, url: String) {
    Glide.with(view.context).load(url).into(view)
}

<!-- XML 中使用 -->
<ImageView app:imageUrl="@{user.avatarUrl}" />
```

```kotlin
// Compose 方式：
@Composable
fun NetworkImage(url: String, modifier: Modifier = Modifier) {
    var imageBitmap by remember { mutableStateOf<ImageBitmap?>(null) }
    
    LaunchedEffect(url) {
        // 加载图片逻辑
        imageBitmap = loadImageBitmap(url)
    }
    
    imageBitmap?.let { bitmap ->
        Image(bitmap = bitmap, contentDescription = null, modifier = modifier)
    }
}

// 使用
@Composable
fun UserProfile(avatarUrl: String) {
    NetworkImage(url = avatarUrl)  // 直接使用自定义组件
}
```

### 3. Lifecycle
Lifecycle是一个类，它包含组件 ( Activity 或 Fragment ) 生命周期状态的信息，并允许其他对象观察此状态。它通过 **生命周期所有者 (LifecycleOwner)** 和 **生命周期观察者 (LifecycleObserver)** 这两个角色来实现。
![[Jetpack-lifecycle.png]]
上图中 **states** 表示组件状态， **events** 表示组件生命周期事件。其实Lifecycle代码内部使用了两个主要枚举 ( **Event**、**State** ) 来跟踪其关联组件的生命周期状态。

- 其中枚举 **Event** 的值和 Activity 或 Fragment 组件的生命周期回调事件一一对应。这些事件映射到 Activity 和 Fragment 中的回调（例如 `ON_CREATE`, `ON_START`, `ON_RESUME` 等）。
- 而枚举 **State** 则表示被跟踪组件的当前状态，其中 [`STARTED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#STARTED) 和 [`RESUMED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#RESUMED) 为活跃状态，配合 **LiveData** 使用时，只有组件处于活跃状态才能接受到数据更新通知。

当 Activity 的 `onCreate` 被调用时，会分发 `ON_CREATE` **事件**，同时生命周期**状态**变为 `CREATED`。可以通过 `lifecycle.currentState` 来获取当前状态。

> **LiveData**：`LiveData` 本身就是一个生命周期感知的组件。当使用 `liveData.observe(lifecycleOwner) { ... }` 时，`LiveData` 只会将数据更新通知给处于 **活动状态**（`STARTED` 或 `RESUMED`）的观察者。当 `LifecycleOwner` 被销毁时，`LiveData` 会自动清理观察者，防止内存泄漏。
>**ViewModel**：`ViewModel` 的生存期范围是通过 `Lifecycle` 来定义的。它的存在时间是从首次创建直到其 `LifecycleOwner` 永久消失（对于 Activity，是 `onDestroy`；对于 Fragment，是 `onDestroy`）。

#### a. LifecycleOwner（生命周期所有者）
这是一个接口，表示其实现类拥有一个 Android 生命周期。`ComponentActivity` (AppCompatActivity 的父类) 和 `Fragment` 已经默认实现了这个接口。可以通过 `getLifecycle()` 方法来获取它们的 `Lifecycle` 对象。**简单来说：Activity 和 Fragment 就是 `LifecycleOwner`。**
#### b. Lifecycle（生命周期）
这是一个对象，它持有 `LifecycleOwner` 的当前生命周期状态（如 `CREATED`, `RESUMED`），并且允许其他对象观察该状态的变化。
#### c. LifecycleObserver（生命周期观察者）
这是一个标记接口，没有任何方法。可以在开发者的类中实现这个接口，然后使用 `@OnLifecycleEvent` 注解（在 Java 中）或**默认的生命周期方法**（在 Kotlin 中推荐）来声明哪些方法需要在生命周期发生变化时被调用。**简单来说：任何想监听生命周期的类，都可以成为 `LifecycleObserver`。**
#### d. Lifecycle 和 LifecycleObserver 配合使用
假设有一个 `MyLocationListener` 类，它负责在应用处于前台时监听位置更新，在应用进入后台时停止监听。我们不希望它在 Activity 被销毁后仍然保持运行。
```kotlin
// 1. 让 MyLocationListener 实现 LifecycleObserver
import androidx.lifecycle.DefaultLifecycleObserver
import androidx.lifecycle.LifecycleOwner

class MyLocationListener(
    private val context: Context,
    private val callback: (Location) -> Unit
) : DefaultLifecycleObserver { // 实现 DefaultLifecycleObserver 接口

    // 2. 使用注解或重写默认方法来响应生命周期事件
    //    实现接口之后，当 MainActivity 的生命周期状态发生变化时（从 CREATED 变为 STARTED），MyLocationListener 中对应的 onStart 方法会被自动调用。
    override fun onStart(owner: LifecycleOwner) {
        start() // 当 LifecycleOwner (如Activity) 进入 onStart 时，此方法被自动调用
    }

    override fun onStop(owner: LifecycleOwner) {
        stop() // 当 LifecycleOwner 进入 onStop 时，此方法被自动调用
    }

    fun start() {
        Log.d("Lifecycle", "开始监听位置")
    }
    fun stop() {
        Log.d("Lifecycle", "停止监听位置")
    }
}


// 3. 在 Activity 中，将观察者添加到生命周期中
class MainActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        myLocationListener = MyLocationListener(this) { location ->
            // 更新 UI
        }

        // 最关键的一步：将观察者 (MyLocationListener) 与所有者 (this Activity) 的生命周期关联起来
        lifecycle.addObserver(myLocationListener)
    }
    // 如果不使用Lifecycle，也可以实现该功能，只不过需要手动在 onStart 和 onStop 中调用 start() 和 stop()
}
```

### 4. Room 
Room 在 SQLite 之上提供的一个抽象层，让数据库访问变得更加简单、高效和类型安全。Room 能在编译时生成 SQLite 相关的样板代码，更直观的 Kotlin/Java 对象和注解来操作数据库。优点是：减少样板代码的编写、可在编译时验证数据库语句、更好集成到 LiveData ViewModel、更安全的类型转换。


## UI组件

## 行为组件

## 基础组件