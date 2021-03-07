---
title: Kotlin学习-基础篇-01
date: 2020-11-04 19:43:48
tags:
    - kotlin
    - 笔记
    - 阅读
categories: 
    - [Kotlin]

excerpt: Kotlin学习-基础篇 1、变量 1. 变量延迟初始化 2、函数 3、if 条件语句 4、when 条件语句 5、循环语句 5.1. 区间 5.2. for-in 循环 6、面向对象 6.1. 类与对象 6.2. 类与构造对象 6.3. 接口 6.4. 修饰符 6.5. 数据类与单例类
top_image: https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b51ce98510514b319952f3713eba566c~tplv-k3u1fbpfcp-watermark.image
# tags:  设置文章标签
#  - PlayStation
#  - Games
# categories: 嵌套分类，Life 成为 Diary 的子分类
#  - Diary
#  - Life
#categories: Life 和 Diary 成为并列分类。
#  - [Diary]
#  - [Life]
#categories: PlayStation 和 Games 同为 Diary 的子分类，而 Life 和 Diary 是并列分类
#  - [Diary, PlayStation]
#  - [Diary, Games]
#  - [Life]
# top_image: https://xxxxx.jpg 头图
# comments: false 是否启用评论
# excerpt: 这是一段文章摘要，是通过 Front-Matter 的 excerpt 属性设置的。
# link: 设置标题链接
# photos: 设置页面插图
# sidebar: true  是否显示侧边栏（默认所有页面生效）
# reward: true  是否启用打赏（默认只对所有文章页面生效）
# copyright: true  是否启用版权信息(默认只对所有文章页面生效) 
# top: false 是否置顶文章 
# 设置图片大小 ![](https://xxxxx.png?size=200x100)
# 行内图片 ![](https://xxxxx.png?show=inline)
# 图片的大小和是否显示为行内图片 ![](https://xxxxx.png?size=200x100&show=inline)
# quicklink: true 使浏览器在空闲时间预取可视区内的链接，以加快后续页面的加载速度
---


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b51ce98510514b319952f3713eba566c~tplv-k3u1fbpfcp-watermark.image)

# Kotlin学习-基础篇
>本文是本人在学习 Koitlin 期间记录的一些知识点，仅作参考，如有错误，欢迎指正
## 1、变量
`val` (value的简写) 用来声明一个不可变的变量，对应Java中的 `final` 变量  
`var` (variable的简写) 用来声明一个可变的变量，对应Java中的非 `final` 变量  
当Kotlin无法自动推导变量类型时，需要显示声明变量类型，如：
```kotlin
val a: Int = 10
```
Java和Kotlin数据类型对照表：
| Java基本数据类型 | Kotlin对象数据类型 | 数据类型说明 |
| :------------: | :----------------: | :---------: |
| int | Int | 整型 |
| long | Long | 长整型 |
| short | Short | 短整型 |
| float | Float | 单精度浮点型 |
| double | Double | 双精度浮点型 |
| boolean | Boolean | 布尔型 |
| char | Char | 字符型 |
| byte | Byte | 字节型 |

### 变量延迟初始化
`lateinit`可以告诉`Kotlin`编译器我会晚些时候对其进行初始化，避免一开始声明变量时需赋值为空，示例如下：
```kotlin
class Class1 {
    private lateinit var params: Params
    fun doSomthing(){
        if(!::params.isInitialized){
        params = Params(...)
        }
    }
}
```
为了避免因变量没有初始化导致程序崩溃，故通过，`::params.isInitialized`来判断变量是否已经初始化

## 2、函数
自定义函数，语法规则如下：
```kotlin
fun methodName(param1:Int, param2:Int):Int{
    return 0
}
//Kotlin函数语法糖,当函数只有一行代码时，可省去return关键字，等号可表示返回值的意思
fun methodName(param1:Int, param2:Int):Int = 0
```

## 3、if 条件语句
`Kotlin` 中的 `if`  语句相比于 Java 有一个额外功能，它可以有返回值  
`if` 语句中每一个条件中最后一行代码可作为返回值，如下：
```kotlin
fun methodName(param1:Int, param2:Int):Int{
    return if(param1 > param2){
        param1
    } else{
        param2
    }
}
//Kotlin语法糖..简化为
fun methodName(param1:Int, param2:Int):Int = if (param1 > param2) param1 else param2
```

## 4、when 条件语句
`when`语句允许掺入一个任意类型的参数，然后可以在`when`的结构体中定义一系列条件。
格式为：  
```
匹配值 -> {执行逻辑}
```
示例，不同学生对应不同分数：
```kotlin
fun getScore(name:String) = when(name){
    "Tom" -> 60
    "Jim" -> 30
    "Tommy" -> 50
    "Lily" -> 70
    else -> 0
}
//当执行逻辑里只有一行代码时可省去{}，且when语句也将最后一行作为返回值
```
`when` 不带参数的用法，示例：
```kotlin
fun getScore(name:String) = when{
    name == "Tom" -> 60
    name == "Jim" -> 30
    name == "Tommy" -> 50
    name == "Lily" -> 70
//    name.startWith("Tom") -> 80 //将以Tom开头的名字，分数都为80
    else -> 0
}
```
when语句也可用作类型匹配，示例：
```kotlin
fun checkNumber(num:Number){
    when(num){
        is Int -> println("Number is Int")
        is Double -> println("Number is Bouble")
        ....
        else -> ("Number not support")
    }
}
```

## 5、循环语句
### 区间
Kotlin中创建一个区间：  
双端闭区间，如下：
```kotlin
val range = 0..10
//表示创建了一个从0到10 的区间，并且两端都是闭区间，数学式为[0,10]
//其中 .. 是创建双端闭区间的关键字
```
单端闭区间，如下：
```kotlin
val range = 0 until 10
//表示创建了一个 从0到10的左闭右开区间，数学式为[1,10)
//其中 until 是创建单端闭区间的关键字
```
注意：使用 `..` 和 `until` 关键字都要求左端小于等于右端

降序区间，如下：
```kotlin
val range = 10 downTo 1
//表示创建了一个从10到1的区间，数学式表示为[10,1]
//downTo是创建降序区间的关键字
```

### for-in 循环
示例如下：
```kotlin
for (i in 0..10){
    ....
}
//表示遍历这个从0到10的闭区间
```
由于 `for-in` 循环每次执行循环时区间范围递增 1，所以可使用 `step` 关键字来跳过一些元素
示例如下：
```kotlin
for (i in 0..10 step 2){
    ....
}
//表示每次执行循环递增2
```

## 6、面向对象
### 类与对象
Kotlin中创建一个对象，如下：
```kotlin
class ClassName{
}
```
对这个类进行实例化，如下：
```kotlin
val c = ClassName()
```

### 类与构造对象
在Kotlin中任何一个非抽象类默认都是不可被继承的  
使类可以被继承，如下：
```kotlin
open class ClassName{
}
//加上open关键字使类可被继承
```
让一个类去继承另一个类，如下：
```kotlin
class class1 : class2(){
}
```

Kotlin中的构造函数分为**主构造函数**和**次构造函数**  
每一个类默认都会有一个不带参数的主构造函数，可显式地指定参数。 

主构造函数特点没有函数体，直接定义在类名后面即可，如下：
```kotlin
class class1(val params1:String, val params2:Int){
}
```
实例化，如下：
```kotlin
val c = class1("Hello",10)
```

当需要在主构造函数中编写一些逻辑时，  
Kotlin提供了一个 `init` 结构体，可将主构造函数逻辑写在里面，如下：
```kotlin
class class1(val params1:String, val params2:Int){
    init{
        prinln(params1 + params2)
    }
}
```

子类的构造函数必须调用父类中的构造函数，如下：
```kotlin
//父类
open class Class2(val name:String, val age:Int){
}
//子类
class Class1(val son:String, val grade:Int, name:String, age:Int) : 
        Class2(name, age){
}
//创建实例
val c = Class1("a123",2,"Tom",12)
```
注意，在`Class1`类中不能将`name`和`age`声明成`val`，因为在主构造函数中声明成`var`或`val`的参数将自动成为该类的字段。不加任何关键字，让它的作用域仅限定在主构造函数当中即可。

次构造函数  
Kotlin规定，当一个类既有主构造函数也有次构造函数时，**所有的次构造函数都必须调用主构造函数**(包括间接调用)，示例如下：
```kotlin
class Class1(val son:String, val grade:Int, name:String, age:Int):
        Class2(name, age){
    constructor(name:String,age:Int) : this("",0,name,age){
    }
    constructor():this("",0){
    }
}
//分别调用构造函数进行实例化
val c1 = Class1()
val c2 = Class1("Tom",19)
val c3 = Class1("a123",4,"Tom",19)
```
次构造函数通过`constructor`关键字来定义，这里定义的两个次构造函数。第一个接收的参数时`name`和`age`参数，又通过`this`关键字调用了主构造函数，并将`son`和`grade`这两个参数赋值为初始值；第二个个次构造函数不接收任何参数，通过`thsi`关键字调用了第一个次构造函数，并将参数赋值为初始值。第二个次构造函数间接调用了主构造函数，所以也是合法的.

### 接口
Kotlin中让一个类实现一个接口以及实现接口中的方法，如下：
```kotlin
class Class1() : Class2() , InterfaceName{
    override fun doSomething(){
    }
}
```
实现接口使用逗号隔开，使用`override`关键字来重写父类或者接口中的函数

Kotlin中允许对接口中定义的函数进行默认实现，示例如下：
```kotlin
interface InterfaceName{
    fun doSomething(){
        .....//执行逻辑
    }
}
```
如果接口中的函数有了函数体，这个函数体中的内容就是它的默认实现，一个类去实现该接口时，**会强制要求实现没有函数体的函数，不会强制要求实现有函数体的函数，不实现时就会自动使用默认的实现逻辑。**

### 修饰符
Java 和 Kotlin 函数可见性修饰符对照表
| 修饰符 | Java | Kotlin |
| :---: | :---: | :---: |
| public | 所有类可见 | 所有类可见（默认）|
| private | 当前类可见 | 当前类可见 |
| protected | 当前类、子类、同一包路径下的类可见 | 当前类、子类可见 |
| default | 同一包路径下的类可见（默认） | 无 |
| internal | 无 | 同一模块中的类可见 |

### 数据类与单例类
数据类用于将服务器端或数据库中的数据映射到内存中，为编程逻辑提供数据模型的支持。  
Kotlin中创建数据类，示例如下：
```kotlin
data class Cellphone(val brand:String, val price:Double)
//使用上面这个数据类，如下
fun main() {
    val cellphone1 = Cellphone("Samsung", "1299.99")
    val cellphone2 = Cellphone("Samsung", "1299.99")
    println(cellphone1)
    println("cellphone1 equals cellphone2"+(cellphone1 == cellphone2))
}
//运行结果为
//Cellphone(brand=Samsung, price=1299.99)
//cellphone1 equals cellphone2 true
```
当在一个类前面声明`data`关键字时，便表明你希望这个类是一个数据类  
当一个类中没有任何代码时，可以将大括号省略。

单例类  
用于保证某个类在全局最多只能拥有一个实例，不同于Java中实现单例模式的方式，在 Kotlin 中实现非常简单。  
Kotlin中创建单例类，创建类型选择“Object”，示例代码如下：
```kotlin
object Singleton{
    fun singletonTest(){
        println("SingeltonTest is called")
    }
}
//调用单例类中的函数
Singleton.singletonTest()
```

更多文章：  
[Kotlin学习知识点整理-基础篇-01](https://juejin.im/post/6891231633702125576)  
[Kotlin学习知识点整理-基础篇-02](https://juejin.im/post/6891260438219227143)  
[Kotlin学习知识点整理-基础篇-03](https://juejin.im/post/6891621855204786184)  
[Kotlin学习知识点整理-进阶篇-04](https://juejin.im/post/6892393981695819789)
>写在最后：遨游在知识的海洋，永远对技术怀有一个敬畏之心
