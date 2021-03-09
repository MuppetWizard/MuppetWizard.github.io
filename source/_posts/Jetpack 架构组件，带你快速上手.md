---
title: Jetpack 架构组件，带你快速上手
date: 2020-11-18 19:50:00
tags:
    - kotlin
    - Android
    - JetPack
categories: 
    - [Android]

excerpt: 1. Jetpack 2. ViewModel 2.1. 接下来简单使用一下： 2.2. 向 ViewModel 传递参数 3. Lifecycles 3.1. 简单使用： 4. LiveData 4.1. 简单使用 4.2. LiveData 中的 map 和 switchMap 4.2.1. map() 4.2.2. switchMap() 5. Room 5.1. Room 的具体用法 5.1.1. 定义 Entity： 5.1.2. 定义 Dao： 5.1.3. 定义 Database 5.1.4. Room 的数据库升级 6. WorkManager 6.1. 使用 WorkManager 处理复杂任务 6.2. 链式任务
---
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93847c64524e486f929351c2dd8bb87f~tplv-k3u1fbpfcp-watermark.image)
# Jetpack
>本文是我在学习Jetpack的过程中做的一些记录，如有错误，欢迎指正  
>本文包含了 ViewModel、Lifecycles、LiveData、Room、WorkManager 的相关用法，你可以通过目录直接跳转到你想了解的地方
# ViewModel
简单介绍下 `ViewModel`：`ViewModel` 类旨在以注重生命周期的方式存储和管理界面相关的数据。ViewModel 类让数据可在发生屏幕旋转等配置更改后继续留存。  
这是官网给出的介绍，简单解释一下 `ViewModel` 就是将界面 (Activity或Fragment) 中显示的数据从其中分离出来，单独进行处理，减少界面的逻辑复杂程度，减轻界面的负担。  

## 接下来简单使用一下：  
添加依赖：
```gradle
implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
```
给对应的 `Activity` 创建一个对应的 `XXXViewModel` 类，并让它继承自 `ViewModel` （不管是 `Activity` 还是 `Fragment` 都最好给每一个都创建一个对应的 `ViewModel` ）
```kotlin
class MainViewModel : ViewModel(){
    var counter = 0 //用于计数
}
```

在 Activity 中使用：
```kotlin
class MainActivity : AppCompatActivity() {

    lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        viewModel = ViewModelProvider(this).get(viewModel::class.java)
        btn_plus.setOnClickListener{
            viewModel.counter++
        }
        refreshCounter()
    }

    private fun refreshCounter() {
        tv_info.text = viewModel.counter.toString()
    }
}
```
现在我们实现了一个计数器，在这里需要注意，我们不能直接去创建 `ViewModel` 的实例，而是要通过 `ViewModelProvider` 来获取 `ViewModel` 的实例，之所以这样是因为 `ViewModel` 有独立的生命周期，且其生命周期长于 `Activity` 的生命周期，如果在 `onCreat()` 中创建 `ViewModel` 的实例，那么每次 `onCreat()` 执行时，`ViewModel` 都会创建一个实例，这样就无法保存其中的数据了。  

`ViewModel` 对象存在的时间范围是获取 `ViewModel` 时传递给 `ViewModelProvider` 的 `Lifecycle`。`ViewModel` 将一直留在内存中，直到限定其存在时间范围的 `Lifecycle` 永久消失，如下图是 `Activity` 的生命周期和 `VIewModle` 的生命周期对应的情况
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5619d7bcc394e1faf99efef9a746d28~tplv-k3u1fbpfcp-watermark.image)

## 向 ViewModel 传递参数
借助 `ViewModelProvider.Factory` ，下面我们来实现退出程序后再打开，数据仍然不会消失的效果
先修改 `MainViewModel` 的代码，如下：
```kotlin
class MainViewModel(counterReserved: Int) : ViewModel(){
    var counter = counterReserved //用于计数
}
```

接着创建 `MainViewModelFactory` 类，并实现 `ViewModelProvider.Factory` 接口，如下：
```kotlin
class MainViewModelFactory(private val countReserved: Int) : ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return MainViewModel(countReserved) as T
    }
}
```
我们这里实现了接口要求我们的 `create` 方法，在方法里面我们创建并返回了一个 `MainViewModel` 的实例，为什么我们这里就可以创建 `MainViewModel` 的实例了呢？因为 `create()` 方法的执行时机和 `Activity` 的生命周期无关，所以不会产生之前提到的问题。

最后修改 activity 中的代码，如下：
```kotlin
class MainActivity : AppCompatActivity() {

    lateinit var viewModel: MainViewModel
    lateinit var sp: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        sp = getSharedPreferences("count_reserved",Context.MODE_PRIVATE)
        val countReserved = sp.getInt("count_reserved",0)
        viewModel = ViewModelProvider(this,MainViewModelFactory(countReserved))
            .get(viewModel::class.java)
        ......
        btn_clear.setOnClickListener {
            viewModel.counter = 0
            refreshCounter()
        }
        refreshCounter()
    }

    override fun onPause() {
        super.onPause()
        val edit = sp.edit()
        edit.putInt("count_reserved",viewModel.counter)
        edit.apply()
    }
    ......
}
```

# Lifecycles
顾名思义，`Lifecycles` 是一个用来感知 `Activity` 生命周期的组件，下面来学习下简单用法。
## 简单使用：
新建一个 `MyObserver` 类，并实现 `LifecycleObserver` 接口
```kotlin
class MyObserver : LifecycleObserver{
}
```
`LifecycleObserver` 这是一个空方法接口，我们可以在 `MyObserver` 中定义任何方法，如果需要感知 `Activity` 的生命周期就需要为方法添加注解，如下所示：
```kotlin
class MyObserver : LifecycleObserver{

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun activityStart() {
        Log.d("MyObserver","activityStart")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun activityStop() {
        Log.d("MyObserver","activityStop")
    }
}
```
这里使用 `@OnLifecycleEvent` 注解，并传入了一种生命周期事件，生命周期事件一共有 7 种，分别是：`ON_CREATE` 、`ON_START` 、`ON_RESUME` 、`ON_PAUSE` 、`ON_STOP` 、`ON_DESTROY` 、`ON_ANY` 。前六种分别匹配 `Activity` 中相应的的生命周期回调，最后一种表示可以匹配 `Activity` 的任何生命周期回调。

接下来就是需要 `LifecycleOwner` 去通知 `MyObserver` 生命周期发生了变化，它可以使用如下的语法结构去通知 `MyObserver`
```kotlin 
lifecycleOwner.lifecycle.addObserver(MyObserver())
```
这里 `LifecycleOwner` 调用了 `getLifecycle` 方法，得到一个 `Lifecycle` 对象，接着调用 `addObserver` 来观察 `LifecycleOwner` 的生命周期，再把 `MyObserver` 传进去。  

`LifecycleOwner` 是个什么？如何让获取一个 `LifecycleOwner` 的实例？  
大多数情况下，只要 `Activity` 是继承自 `AppCompatActivity` 的，或者 `Fragment` 是继承自 `androidx.fragment.app.Fragment` 的，那么它们本身就是一个 `LifecycleOwner` 的实例，所以我们在 `Activity` 中就可以这样写
```kotlin
class MainActivity : AppCompatActivity() {
    ......
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        ......
        lifecycle.addObserver(MyObserver())
    }
    ......
}
```
现在程序可以感知到 `Activity` 的生命周期变化，但没法主动获知当前的生命周期状态，解决这个问题，只需要在 `MyObserver` 的构造函数中将 `Lifecycle` 对象传进去即可，如下：
```kotlin
class MyObserver(val lifecycle: Lifecycle) : LifecycleObserver{......}
```
有了 `Lifecycle` 对象后，就可以在任何地方调用 `lifecycle.currntState` 来主动获取当前的生命周期状态。  
`lifecycle.currntState` 返回的生命周期状态时一个枚举类型，一共有 5 种状态类型，如下：
```
INITIALIZED
DESTROYED
CREATED
STARTED
RESUMED
```
它们与 `Activity` 的生命周期回调所对应的关系如图：
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1664e9bd88f2492dba7ab10f7be9316f~tplv-k3u1fbpfcp-watermark.image)

# LiveData
`LiveData` 是一种可观察的数据存储器类。与常规的可观察类不同，`LiveData` 具有生命周期感知能力，意指它遵循其他应用组件（如 `Activity`、`Fragment` 或 `Service`）的生命周期。这种感知能力可确保 `LiveData` 仅更新处于活跃生命周期状态的应用组件观察者。
## 简单使用
`LiveData` 可以包含任何类型的数据，并在数据发生变法的时候通知给观察者  

修改 `MainViewModel` 中的代码，如下：
```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
    var counter = MutableLiveData<Int>()
    init {
        counter.value = countReserved
    }

    fun plusOne() {
        val count = counter.value ?: 0
        counter.value = count + 1
    }
    fun clear() {
        counter.value = 0
    }
}
```
这里将 `counter` 变量修改成了一个 `MutableLiveData` 对象，这是一种可变的 `LiveData` 。它主要有三种读写数据的方法，分别是：  
* `getvalue()` //用于获取 `LiveData` 中包含的数据
* `setValue()` //用于给 `LiveData` 设置数据，但是只能在主线程中调用
* `postValue()` //用于在非主线程中给 `LiveData` 设置数据

下面来修改 MainActivity 中的代码：
```kotlin
class MainActivity : AppCompatActivity() {
    ......
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ......
        btn_plus.setOnClickListener{
            viewModel.plusOne()
        }
        btn_clear.setOnClickListener {
            viewModel.clear()
        }

        viewModel.counter.observe(this, Observer { count ->
            tv_info.text = count.toString()
        })
    }

    override fun onPause() {
        super.onPause()
        val edit = sp.edit()
        edit.putInt("count_reserved",viewModel.counter.value ?: 0)
        edit.apply()
    }
}
```
这里 `counter` 变量已经变成了一个 `LiveData` 对象，任何 `LiveData` 对象都可以调用它的 `observe()` 方法来观察数据的变化。`observer()` 方法接收两个参数：第一个参数是一个 `LifecycleOwner` 对象，这里也就是 `Activity` 自己。第二个参数是一个 `Observer` 接口，当 `counter` 中包含的数据发生变化时，就会回调到这里。  

关于 `observe()` 方法，Google 官方在专门面向 Kotlin 语言的 API 中提供了很多好用的语法扩展，要使用它需添加依赖：
```gradle
implementation 'androidx.lifecycle:lifecycle-livedata-ktv:2.2.0'
```
之后我们就可以使用如下结构的 `observe()` 方法了
```kotlin
viewModel.counter.observe(this) { count -> 
    tv_info.text = count.toString()
}
```
以上是 `LiveData` 基本用法，可以正常使用，但仍然不是最规范的用法， 主要问题是我们将 `counter` 这个可变的 `LiveData` 暴露给了外部，这样在 `ViewModel` 外面也是可以给 `counter` 设置数据，从而破坏了 `LiveData` 数据的封装性  

比较推荐的做法是，永远只暴露不可变的 `LiveData` 给外部，下面来改造下 `MainViewModel` ，如下：
```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
    
    val counter : LiveData<Int>
        get() =_counter
    private val _counter = MutableLiveData<Int>()
    
    init {
        _counter.value = countReserved
    }
    fun plusOne() {
        val count = _counter.value ?: 0
        _counter.value = count + 1
    }
    fun clear() {
        _counter.value = 0
    }
}
```
这里 `_counter` 变量对于外部便是不可见的，而我们又定义了一个 `counter` 变量，类型声明为不可变的 `LiveData` ，并在它的 `get()` 属性方法中返回 `_counter` 变量。  
这样当外界调用 `counter` 变量时，实际上获得的是 `_counter` 的实例，但是无法给 `counter` 设置数据，从而保证了 `LiveData` 数据的封装性  

## LiveData 中的 map 和 switchMap
`LiveData` 为了能够应对各种不同需求场景，提供了两种转换方法：`map()` 和 `switchMap()` 方法

### map() 
这个方法的作用是将实际包含数据的 `LiveData` 和仅用于观察数据的 `LiveData` 进行转换，实例如下：  
比如有一个 `User` 类，其中包含用户的姓名、年龄，如下：
```kotlin
data class User(var firstName: String,var lastName:String, var age: Int) {}
```
这里包含了三个数据，但如果我们的 `Activity` 明确了只需要用户的姓名，不关心年龄时 还将整个 `User` 暴露出去就不太合适，这时候就可以使用 `map()` 方法，将 `User` 类型的 `LiveData` 自由的转型成任意其他类型的 `LiveData`，如下：
```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
    private val userLiveData = MutableLiveData<User>()

    val userName : LiveData<String> = Transformations.map(userLiveData){ user ->
        "${user.firstName} ${user.lastName}"
    }
......
}
```
`map()` 方法接收两个参数：
* 第一个：参数是原始的 `LiveData` 对象；
* 第二个：参数是一个转换函数

我们就只需要在转换函数里写具体的逻辑即可  
另外，我们将 `userLiveData` 声明成了 `private` ，以保证数据的封装性，外部只需要观察 `userName` 就可以了，当 `userLiveData` 数据发生变化时，`map()` 方法会监听到变化并执行转换函数中的逻辑，然后将转换之后的数据通知给 `userName` 的观察者

### switchMap()
我们通过一个实例来学习。根据传入的 `userId` 参数去服务器请求或者到是数据库中查找相应的 `User` 对象，但是这里只是模拟示例，因此每次传入的 `userId` 当作用户姓名来创建一个新的 `User` 对象即可  

代码如下，先创建一个 `Repository` 单例类，模拟获取用户数据的功能：
```kotlin
object Repository {
    fun getUser(userId: String) : LiveData<User>{
        val liveData = MutableLiveData<User>()
        liveData.value = User(userId,userId,0)
        return liveData
    }
}
```

下面获取 `Repository` 的 `LiveData` 对象，
```kotlin
class MainViewModel(countReserved: Int) : Int{
    ...
    fun getUser(userId: String) : LiveData<User> {
        return Repository.getUser(userId)
    }
}
```
在 `Activity` 中观察 `LiveData`
```kotlin
viewModel.getUser(userId).observe(this) { -> 
}
```
以上的这种呢做法完全错误，因为每次调用 `getUser()` 方法返回的都是一个新的 `LiveData` 实例，而上述写法会一直观察老的 `LiveData` 实例，这种情况下，`LiveData` 是不可能观察的   

以下是正确做法，  
借助 `switchMap` ，它的使用场景比较固定：如果 `VIewModel` 中的某个 `LiveData` 对象是调用另外的方法获取的，那么就可以借助 `switchMap()` 方法，将这里 `LiveData` 对象转换成另外一个可观察的 `LiveData` 对象。修改 `MainViewModel` 中的代码，如下：
```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
    ....
    private val userIdLiveData = MutableLiveData<String>()

    val user: LiveData<User> = Transformations.switchMap(userIdLiveData) { userId ->
        Repository.getUser(userId)
    }
    fun getUser(userId: String) {
        userIdLiveData.value = userId
    }
}
```
`switchMap()` 方法接收两个参数：  
* 第一个：传入新增的 `userIdLiveData` ，`switchMap()` 方法会对它进行观察
* 第二个：是一个转换函数，我们必须在这个转换函数中返回一个 `LiveData` 对象，因为 `switchMap()` 方法的工作原理就是要将转换函数中返回的 `LiveData` 对象转换成另一个可观察的 `LiveData` 对象。

现在 `user` 对象就是一个可观察的 `LiveData` 对象了  
修改 `Activity` 中的代码，如下：
```kotlin
class MainActivity : AppCompatActivity() {
    ....
    override fun onCreate(savedInstanceState: Bundle?) {
        ....
        btn_getUser.setOnClickListener {
            val userId = (0..1000).random().toString()
            viewModel.getUser(userId)
        }
        viewModel.user.observe(this,{
        tv_info.text = it.firstName
        })
    }
}
```
现在已经实现了功能并且可以正常运行了。

当我们的 `ViewModel` 中某个获取数据的方法有可能是没有参数的，这个时候该怎么办呢？  
这里我们先创建一个空的 `LiveData` 对象：
```kotlin
class MyViewModel : ViewModel(){
    
    private val refreshLiveData = MutableLiveData<Any?>()
    
    val refreshResult = Transformatinos.switchMap(refreshLiveData) {
        Repository.refresh()
    }
    
    fun refresh() {
        refreshLiveData.value = refreshLiveData.value
    }
}
```
在 `refresh()` 方法中，只是将 `refreshLiveData` 原有的数据取出来（默认是空），再重新设置到 `refreshResult` 当中，这样就能触发一次数据变化。  
在 `LiveData` 内部不会判断即将设置的数据和原有数据是否相同，只要调用了 `setValue()` 或 `postValue()` 方法，就一定会触发数据变化事件，然后我们只需要在 `Activity` 中观察 `refreshResult` 这个 `LiveData` 对象即可

# Room
`Room` 是 Google 官方推出的一个 `ORM`（`Object Relational Mapping` 对象关系映射）框架，并将它加入了 `Jetpack` 中。

`Room` 的整体结构主要由 `Entity`、`Dao`、`Database` 这三个部分组成，每个部分都有自己明确的职责：
* `Entity`：用于定义封装实际数据的实体类，每个实体类都会在数据库中都有一张对应的表，并且表中的列是根据实体类中的字段自动生成的。
* `Dao`：`Dao` 是数据访问的意思，同长会在这里对数据库的各项操作进行封装。在实际编程中，逻辑层就不用和底层数据打交道了，直接和 `Dao` 层进行交互就可。
* `Database`：用于定义数据库中的关键信息，包括数据库的版本号、包含哪些实体类以及提供 `Dao` 层的访问实例。

## Room 的具体用法 
添加依赖：
```gradle
....
apply plugin: 'kotlin-kapt'

dependencies{
    ....
    implementation 'androidx.room:room-runtime:2.2.5'
    kapt 'androidx.room:room-compiler:2.2.5'
}
```
`kapt` 只能在 `Kotlin` 项目中使用，如果是 `Java` 项目的话，使用 `amotationProcessor` 即可

接下来按照刚才介绍的 `Room` 的三个部分来一一进行实现

### 定义 Entity：
我们直接就使用 上文定义的 User 类来改造
```kotlin
@Entity
data class User(var firstName: String,var lastName:String, var age: Int) {

    @PrimaryKey(autoGenerate = true) 
    var id: Long = 0  //给每个实体类都添加一个字段，并设为主键
}
```
`@Entity` 注解：将 `User` 声明成了一个实体类  
`@PrimaryKey` 注解：将 `id` 字段设为主键，将 `autoGenerate` 设置为 `true` ，使得主键的值自动生成

### 定义 Dao：
这一部分比较关键，因为所有访问数据库的操作都是在这里封装  
新建一个 UserDao 接口，如下：
```kotlin
@Dao
interface UserDao {
    
    @Insert
    fun insetUser(user: User) : Long
    
    @Update
    fun updateUser(newUser: User)
    
    @Query("select * from User")
    fun loadAllUser() : List<User>
    
    @Query("select * from User where age > :age")
    fun loadUsersOlderThan(age: Int) : List<User>
    
    @Delete
    fun deleteUser(user: User)
    
    @Query("delete from User where lastName = :lastName")
    fun deleteUserByLastName(lastName: String) : Int
}
```
`@Dao` 注解：让 `Room` 能够识别到 `UserDao` 是一个 `Dao`  
`@Insert` 注解：表示将参数传入 `User` 对象插入数据库中，插入完成后还会返回自动生成的主键 `id` 值  
`@Update` 注解：表示会将参数中传入的 `User` 对象更新到数据库当中  
`@Delete` 注解：表示会将参数传入的 `User` 对象从数据库中删除  
以上几种数据库操作都直接使用注解表示即可，不用编写 `SQL` 语句。如果想要从数据库中查询数据，或者使用非实体类参数来增删改查数据，就必须编写 `SQL` 语句，比如刚刚定义的 `loadAllUsers()` 方法，用于从数据库中查询所有用户。

`@Query` 注解：该注解中必须编写 `SQL` 语句，可以将方法中传入的参数指定到 `SQL` 语句当中，比如 `loadUserOlderThan()` 方法就可以查询所有年龄大于指定参数的用户。 

如果是使用非实体类参数来增删改查数据，也要编写 `SQL` 语句才行，而且只能使用 `@Query` 注解，比如 `deleteUserByLastName()` 方法

### 定义 Database
这部分一般只需要定义三个部分：数据库版本号、包含哪些实体类、提供 `Dao` 层的访问实例。  
新建一个 `AppDatabase.kt` 文件，如下：
```kotlin
@Database(version = 1, entities = [User::class])
abstract class AppDatabase : RoomDatabase() {

    abstract fun userDao() : UserDao
    companion object{
        private var instance: AppDatabase? = null

        @Synchronized
        fun getDatabase(context: Context): AppDatabase {
            instance?.let {
                return it
            }
            return Room.databaseBuilder(context.applicationContext,
                AppDatabase::class.java, "app_database")
                .build().apply {
                    instance = this
                }
        }
    }
}
```
`@Database` 注解：其中声明了数据库版本号以及包含的实体类，多个实体类之间用逗号隔开。  
`AppDatabase` 类必须继承自 `RoomDatabase` 类，并且一定要使用 `abstract` 关键字声明成抽象类，然后提供相应的抽象方法，用于获取之前的 `Dao` 实例，比如这里提供的 `userDao()` 方法。之后在 `companion object` 结构体中编写了一个单例模式，原则上全局应该只存在一个 `Appdatabase` 实例。  

`databaseBuilder()` 方法接收三个参数：
* 第一个：参数一定要使用 `applicationContext` ，而不能使用普通 `context()` ，否则容易出现内存泄漏的情况。
* 第二个：参数是 `AppDatabase` 的 `Class` 类型。
* 第三个：参数是数据库名

修改 `MainActivity` 中的代码：
```kotlin
class MainActivity : AppCompatActivity() {
    ....
    override fun onCreate(savedInstanceState: Bundle?) {
        ....
        val userDao = AppDatabase.getDatabase(this).userDao()
        val user1 = User("Tony","ll",11)
        val user2 = User("Tom","SS",22)
        btn_addData.setOnClickListener {
            thread {
                user1.id = userDao.insetUser(user1)
                user2.id = userDao.insetUser(user2)
           }
        }
        btn_updateData.setOnClickListener {
            thread {
                user1.age = 33
                userDao.updateUser(user1)
            }
        }
        btn_deleteData.setOnClickListener {
            thread {
                userDao.deleteUserByLastName("SS")
            }
        }
        btn_queryData.setOnClickListener {
            thread {
                for (user in userDao.loadAllUser()) {
                    Log.d("MainActivity",user.toString())
                }
            }
        }   
    }
}
```
在 `AddData` 的点击事件中将 `insertUser()` 方法返回的主键 `id` 值赋值给了原来的 `User` 对象。之所以这样是因为使用 `@Update`、`@Delete` 注解去更新和删除数据时都是基于这个 `id` 值来操作的。  
由于数据库操作属于耗时操作，`Room` 默认不允许在主线程中进行数据库操作，因此上述的增删改查操作都放在了子线程中，不过为了方便调试，`Room` 还提供了一个更加简单的方法：
```kotlin
Room.databaseBuilder(context.applicationContext, AppDatabase::class.java, "app_database")
    .allowMainThreadQueries()
    .build()
```
这样 `Room` 就允许在主线程进行数据库操作了，**不过建议只在测试环境使用**

### Room 的数据库升级
`Room` 在数据库升级方面设计得比价繁琐，并没有比原生的 `SQLiteDatabase` 简单到哪儿去。如果你目前还只是在开发测试阶段，不想编写那么繁琐的数据库升级逻辑，`Room` 提供了一个简单粗暴的方法：
```kotlin
Room.databaseBuilder(aontext.applicationContext, AppDatabase::class.java. "app_database")
    .fallbackToDestructiveMigration()
    .build()
```
构建 `AppDatabase` 实例时加入了一个 `fallbackToDestructiveMigration()` 方法，这样只要数据库进行了升级，`Room` 就会将当前的数据库销毁，然后再重新创建，但这样做的问题也就显而易见，之前数据库中的数据也会全部丢失。

下面是正规用法  
我们先新建一个 Book 的实体类：
```kotlin
@Entity
data class Book(var nane:String, var price: Int) {

    @PrimaryKey(autoGenerate = true)
    var id : Long = 0
}
```
然后创建一个 BookDao 接口，并在其中随意定义一些 API
```kotlin
 @Dao
interface BookDao {

    @Insert
    fun insert(book: Book) : Long

    @Query("select * from Book")
    fun loadAllBooks() : List<Book>
}
```
修改 AppDatabase 中的代码：
```kotlin
@Database(version = 2, entities = [User::class, Book::class])
abstract class AppDatabase : RoomDatabase() {

    abstract fun userDao() : UserDao
    abstract fun bookDao() : Book

    companion object{

        val MIGRATION_1_2 = object : Migration(1,2) {
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL("create table Book (id integer primary key autoincrement not null, name text not null, price integer not null")
            }
        }

        private var instance: AppDatabase? = null

        @Synchronized
        fun getDatabase(context: Context): AppDatabase {
            instance?.let {
                return it
            }
            return Room.databaseBuilder(context.applicationContext,
                AppDatabase::class.java, "app_database")
                .addMigrations(MIGRATION_1_2)
                .build().apply {
                    instance = this
                }
        }
    }
}
```
我们在 `@Database` 注释中将版本号升级到了 **2** ，并将 `Book` 嘞添加到了实体类声明中。  
在`companion object` 结构体中，实现了一个 `Migration` 的匿名类，并传入了 **1** 和 **2** 这两个参数，表示当前数据库版本从 **1** 升级到 **2** 的时候就执行这个匿名类中的升级逻辑。这里是要新增一张 `Book` 表，所以需要在 `migrate()` 方法中编写相应的建表语句。`Book` 表的建表语句必须和 `Book` 实体类中声明的结构完全一致。最后在构建 `AppDatabase` 实例时，加入一个 `addMigrations()` 方法，并将参数传入即可。  

当我们的数据库可能不需要新建表，而是添加一个列时，可以使用 `alter` 语句修改表结构即可，如下：
```kotlin
@Entity
data class Book(var nane:String, var price: Int, var pages: Int) {
    @PrimaryKey(autoGenerate = true)
    var id : Long = 0
}
```
这里添加了一个页数 `pages` 的字段，之后修改 `AppDatabase` 的代码：
```kotlin
@Database(version = 2, entities = [User::class, Book::class])
abstract class AppDatabase : RoomDatabase() {
    ....
    companion object{
        ....
        var MIGRATION_2_3 = object : Migration(2, 3) {
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL("alter table Book add column pages integer not null default 'unknown'")
            }
        }
                private var instance: AppDatabase? = null

        @Synchronized
        fun getDatabase(context: Context): AppDatabase {
            ....
            return Room.databaseBuilder(context.applicationContext,
                AppDatabase::class.java, "app_database")
                .addMigrations(MIGRATION_1_2,MIGRATION_2_3)
                .build().apply {
                    instance = this
                }
        }
    }
}
```
升级步骤和之前差不多，就不多说了

# WorkManager
`WorkManager` 适合用于处理一些要求定时执行的任务，它可以根据操作系统的版本自动选择底层是使用 `AlarmManager` 实现还是 `JobScheduler` 实现，而且它还支持周期性任务、链式任务处理等功能。  

`WorkManager` 的基本用法主要分以下三步：  
1. 定义一个后台任务，并实现具体的任务逻辑；
2. 配置该后台任务的运行条件和约束信息，并构建后台任务请求；
3. 将改后台任务请求传入 `WorkManager` 的 `enqueue()` 方法中，系统会在合适的时间运行。

下面来按照以上步骤来实现。第一步，定义一个后台任务，新建一个 `SimpleWorker` 类，如下：
```kotlin
class SimpleWorker(context: Context, params: WorkerParameters) : Worker(context,params) {
    override fun doWork(): Result {
        Log.d("SimpleWorker", "do work in SimpleWorker")
        return Result.success()
    }
}
```
每一个后台任务都必须继承 `Work` 类。`doWork()` 方法不会运行在主线程中，因此可以在这里执行耗时操作，另外该方法要求返回一个 `Result` 对象，用于表示任务的运行结果。成功就是 `Result.success()` ，失败是 `Result.failure()` 。除了这两种还有一个 `Result.retry()` 方法，它其实也代表着失败，只是可以结合 `WorkRequest.Builder` 的 `setBackoffCriteria()` 方法来重新执行任务。

第二步，配置该后台任务的运行条件和约束信息。  
这里进行最基本的配置，代码如下：
```kotlin
var request= OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
```
`OneTimeWorkRequest.Builder` 是 `WorkRequest.Builder` 的子类，用于构建单词运行的后台任务请求，还有另一个 `PeriodicWorkRequest.Builder` 也是其子类，用于构建周期性运行的任务请求，但为了降低设备性能消耗，它的构造函数中传入的运行周期间隔不能短于 15 分钟，实例如下：
```kotlin
var request = PeriodicWorkRequest.Builder(SimpleWorker::class.java, 15,
    TimeUnit.MINUTES).build()
```

第三步，最后只需要将构建好的后台任务请求传入 `WorkManager` 的 `enqueue()` 方法中，系统就会在合适的时间去运行了：
```kotlin
WorkManager.getInstance(context).enqueue(request)
```
下面来测试下，修改 `MainActivity` 中的代码：
```kotlin
class MainActivity : AppCompatActivity() {
    ....
    override fun onCreate(savedInstanceState: Bundle?) {
        ....
        btn_doWork.setOnClickListener {
            val request= OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
            WorkManager.getInstance(this).enqueue(request)
        }
    }
}
```
这里后台任务的具体时间由我们所指定的约束以及系统的一些优化所决定的，由于这里没有指定任何约束，因此后台任务基本上会在点击按钮之后立刻运行。

## 使用 WorkManager 处理复杂任务
让后台任务在指定的延迟时间后运行，可以借助 `setInitialDelay()` 方法，如下：
```kotlin
var request = PeriodicWorkRequest.Builder(SimpleWorker::class.java)
    .setInitialDelay(5, TimeUnit.MINUTES)
    .build()
```
`setInitalDelay()` 接收两个参数，第一个是要延迟的时间，第二个是该时间的单位：可以选择毫秒、秒、分钟、小时、天都可以。

可以控制运行时间之后，再来添加点别的功能，比如给后台任务请求添加标签，如下所示：
```kotlin
var request = PeriodicWorkRequest.Builder(SimpleWorker::class.java)
    ....
    .addTag("simple")
    .build()
```
该功能最主要的一个功能就是通过标签来取消后台任务请求：
```kotlin
WorkManager.getInstance(this).cancelAllWorkByTag("simple")
```
使用标签可以将可以将同一标签名的所有后台请求全部取消  

如果没有标签，也可以通过 id 来来取消后台任务请求：
```kotlin
WorkManager.getInstance(this).cancelAllWorkById(request.id)
```

如果想要一次性取消所有后台任务：
```kotlin
WorkManager.getInstance(this).cancelAllWork()
```

在上文中提到，如果 `doWork()` 方法中返回了 `Result.retry()`，那么可以结合 `setBackoffCriteria()` 方法来重新执行任务，如下：
```kotlin
var request = PeriodicWorkRequest.Builder(SimpleWorker::class.java)
    ....
    .setBackoffCriteria(BackoffPolicy.LINEAR, 10, TimeUnit.SECONDS)
    .build()
```
`setBackoffCriteria()` 接收三个参数：
* 第一个：参数用于指定如果任务再次执行失败，下次重试的时间应该以什么样的形式延迟。该参数可选值有两种：
    * LINEAR：表示下次重试时间以线性的方式延迟。
    * EXPONENTIAL：表示下次重试的时间以指数的方式延迟。
* 第二个和第三个都用于指定在多久之后重新执行任务，时间最短不能少于 10 秒钟

接下来我们也可以对 `Result.success(`) 和 `Result.failure()` 任务结果进行监听，如下：
```kotlin
WorkManager.getInstance(this)
    .getWorkInfoByIdLiveData(request.id)
    .observe(this) { workInfo -> 
        if(workInfo.state == WorkInfo.State.SUCCEEDED) {
            Log.d("MainActivity", "do work succeeded")
        } else if(workInfo.state == WorkInfo.State.FAILED) {
            Log.d("MainActivity". "do work failed")
        }
    }
```
这里调用了 `getWorkInfoByIdLiveData()` 方法，并传入后台任务请求的 `id` ，会返回一个 `LiveData` 对象，接着就可以调用 `LiveData` 对象的 `observe()` 方法来观察数据变化了，以此监听后台任务的运行结果。

另外，调用 `getWorkInfoByTagLiveData()` 方法也可以监听同一标签名下所有后台任务请求的运行结果，用法差不多。

## 链式任务
假设定义了 3 个独立的后台任务：同步数据、压缩数据、上传数据，现在要实现先同步、再压缩、对吼上传的功能，就可以借助链式任务来实现，如下：
```kotlin
val sync = ....
val compress = ....
val upload = ....
WorkManager.getInstance(this)
    .beginWith(sync)
    .then(compress)
    .then(upload)
    .enqueue()
```
`beginWith()` 方法用于开启一个链式任务，后面要执行的任务只需要使用 `then()` 方法来连接即可。  
另外，`WorkManager` 还要求，必须在前一个后台任务运行成功之后，下一个后台任务才会运行，也就是说，如果某一个任务运行失败、或者取消了，之后的任务就都不能运行了

以上介绍的 `WorkManager` 的所有功能，在国产手机上都有可能得不到正确的运行，因为绝大多数的国产手机厂商在进行 `Android` 系统定制时会增加一个一键关闭的功能，允许用户一键杀死所有非白名单的应用程序，而被杀死的应用程序既无法接收广播，也无法运行 `WorkManager` 的后台任务，所有千万别依赖 `WorkManager` 去实现什么核心功能，因为它在国产手机上可能会非常不稳定

更多文章：  
[Kotlin学习知识点整理-基础篇-01](https://juejin.im/post/6891231633702125576)  
[Kotlin学习知识点整理-基础篇-02](https://juejin.im/post/6891260438219227143)  
[Kotlin学习知识点整理-基础篇-03](https://juejin.im/post/6891621855204786184)  
[Kotlin学习知识点整理-进阶篇-04](https://juejin.im/post/6892393981695819789)

>写在最后唠唠《一日重生》佳句分享  
>Going back to something is harder than you think.  
>回到过去，比你想象的更难。