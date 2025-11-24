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
            
            // 观察 LiveData！
            viewModel.count.observe(this) { newCount ->
                // 这个 lambda 表达式会在 count 的值改变时被调用
                // `this` 是 LifecycleOwner (Activity/Fragment)
                countTextView.text = newCount.toString()
            }
            
            incrementButton.setOnClickListener {
                viewModel.increment() // 只需调用方法，UI 会自动更新
            }
        }
    }
 ```

### 2. Lifecycle
Lifecycle是一个类，它包含组件(Activity或Fragment)生命周期状态的信息，并允许其他对象观察此状态。
![[Jetpack-lifecycle.png]]
上图中**states**表示组件状态，**events**表示组件生命周期事件。其实Lifecycle代码内部使用了两个主要枚举(**Event**、**State**)来跟踪其关联组件的生命周期状态。

- 其中枚举**Event**的值和Activity或Fragment组件的生命周期回调事件一一对应。
- 而枚举**State**则表示被跟踪组件的当前状态，其中 [`STARTED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#STARTED) 和 [`RESUMED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#RESUMED) 为活跃状态，配合**LiveData**使用时，只有组件处于活跃状态才能接受到数据更新通知。


## UI组件

## 行为组件

## 基础组件