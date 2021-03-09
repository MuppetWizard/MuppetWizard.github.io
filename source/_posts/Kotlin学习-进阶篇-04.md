---
title: Kotlin学习-进阶篇-04
date: 2020-11-07 19:50:00
tags:
    - kotlin
    - 笔记
    - 阅读
categories: 
    - [Kotlin]

excerpt: Kotlin知识点整理进阶 一、字符串内嵌表达式 二、函数的参数默认值 三、定义静态方法 1、单例类 2、companion object 3、注解：@JvmStatic 4、顶层方法 四、密封类 五、扩展函数
---
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac070a7c9c2a45369fe732e7f887723d~tplv-k3u1fbpfcp-watermark.image)
# Kotlin知识点整理进阶
>本文是我在学习Kotlin的过程中整理的一些知识点，如有错误，欢迎指正
## 一、字符串内嵌表达式
语法规则：
```kotlin
"hello ${obj.name}, nice to meet you"
```
当表达式中只有一个变量时可将 `{}` 大括号省去

## 二、函数的参数默认值
示例如下：
```kotlin
//给第二个参数赋默认值
fun printParam(num:Int,str:String = "hello"){
    println("num is $num, str is $str")
}
//调用
printParam(123)
```
因为 `str` 参数已经有了一个默认值 `”hello“` 所以**当调用时不传**第二个参数的话便会使用该参数的默认值
```kotlin
//给第一个参数赋默认值时
fun printParam(num:Int = 123, str:String){
    println("num is $num, str is $str")
}
//调用 
printParam(str = "hello")
```

## 三、定义静态方法
### 1、单例类
示例如下：
```kotlin
object Util{
    fun doAction(){
        println("do action")
    }
}
//调用方法
Util.doAction
```
事实上这里的 `doAction()` 并不是真正意义上静态方法，使用单例类的写法会将整个类中的所有方法**全部变成类似于静态方法**的调用方式。

### 2、companion object
示例如下：
```kotlin
class Util {
    fun doAction1(){
        println("do action 1")
    }
    companinon object{
        fun doAction2(){
            println("do action 2")
        }
    }
}
```
`doAction1()` 方法是一定要先创建 `Util` 类的实例才能调用， `doAction2()` 方法可以直接使用`Util.doAction2()` 方式调用  
TIps：`doAction2()` 方法其实也并不是静态方法，`companinon object` 这个关键字实际上会在 `Util` 类的内部创建一个伴生类，而 `doAction2()`方法就是定义在这个伴生类中的实例方法，Kotlin会保证 `Util` 类始终只会存在一个伴生类对象，所以实际上就是调用了 `Util` 类中伴生对象的`doAction2()`方法

### 3、注解：@JvmStatic
Kotlin 编译器会将使用该注解的方法编译成真正的静态方法，示例如下：
```kotlin
class Util {
    fun doAction1(){
        println("do action 1")
    }
    companinon object{
        @JvmStatic  //该注解只能加在单例类或 companion object 中的方法上
        fun doAction2(){
            println("do action 2")
        }
    }
}
```
现在 `doAction2()` 方法已经是真正的静态方法了，现在不管在Java或是Kotlin中都可以使用 `Util.doAction2()`的写法来调用

### 4、顶层方法
**Kotlin 编译器会将所有的顶层方法全部编译成静态方法，故只要定义了一个顶层方法，那么它就一定是静态方法。**  
创建方法：  
1、创建 Kotlin 文件，创建类型要选择File。  
2、在创建的文件中定义一个方法，如 `doSomething()` 方法，如下：
```kotlin
fun doSomething(){
    println("do something")
}
//在Kotlin中调用
doSomething()
//在java中调用
FileNameKt.doSomething()
```
Kotlin 编译器会自动创建一个叫做 `FileNameKt` 的Java类

## 四、密封类
为了解决由于要满足编译器的要求而编写无用的条件分支的情况，在Kotlin中可使用密封类  
密封类关键字是：`sealed class`  示例如下：
```kotlin
sealed class Result
class Success(val msg:String) : Result()
class Failed(val error:Exception) : Result()
//由于密封类是一个可继承类，因此在继承它时需要在后面加一对括号
```
**注意**：密封类及其所有子类只能定义在同一文件的顶层位置

## 五、扩展函数
扩展函数表示即使在不修改某个类的源码的情况下，任然可以打开这个类，向该类添加新函数  
扩展函数语法结构，示例如下：
```kotlin
fun className.methodName(params:Int, params:Int) : Int{
    return 0
}
```
例子:统计字符串中的字母的数量，如下：
```kotlin
fun String.letterCount() : Int {
    var count = 0
    for(char in this){
        if(char.isLetter()){
            count++
        }
    }
    return count
}
var count = "abcd12345EFG".letterCount()
```
更多文章：  
[Kotlin学习知识点整理-基础篇-01](https://juejin.im/post/6891231633702125576)  
[Kotlin学习知识点整理-基础篇-02](https://juejin.im/post/6891260438219227143)  
[Kotlin学习知识点整理-基础篇-03](https://juejin.im/post/6891621855204786184)  
[Kotlin学习知识点整理-进阶篇-04](https://juejin.im/post/6892393981695819789)
>写在最后  
>《一日重生》佳句分享：  
>I could barely make it move. But her arm went across my chest and I sensed her carrying me once more, air passing over my face. I saw only darkness, as if we were traveling behind the length of a curtain. The the dark pulled away and there were stars. Thousands of them. She was laying me down in wet grass, returning my ruined soul to this world.  
>我几乎不能动弹。但妈妈的手臂环绕着我，我再一次感觉她抱起了我，空气划过我的脸庞。我看到的只有黑暗，好像我们在一个长长的黑幕布后飞行。突然间，黑幕布拉开了，我的眼前出现了星星。成千上百的星星。她放下我，让我躺在湿漉漉的草地上，把我受伤的心还给这个世界。
