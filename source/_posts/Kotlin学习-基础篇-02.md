---
title: Kotlin学习-基础篇-02
date: 2020-11-04 19:50:00
tags:
    - kotlin
    - 笔记
    - 阅读
categories: 
    - [Kotlin]
excerpt: Lambda编程 集合的创建与遍历 集合函数式 API 常用函数式 API maxBy() map() filter() any() all() Java 函数式 API 空指针检查 ? 操作符 ?. 操作符...
---
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cadc0205934a440cb769ef810dac9eb1~tplv-k3u1fbpfcp-watermark.image)
# Lambda编程
>本文是我在学习Kotlin的过程中记录的一些知识点，如有错误，欢迎指正
## 集合的创建与遍历
**List集合**  
使用Kotlin内置的 `listof()` 简化初始化集合，示例如下：
```kotlin
val list = listOf("Apple","Banana","Orange","Pear","Grape")
```
`listOf()`创建的是一个不可变集合，不可变集合指只能用于读取，不能进行添加、修改、删除等操作  
使用 `mutableListOf()` 函数创建可变集合，示例如下：
```kotlin
val list = mutableListOf("Apple","Banana","Orange","Pear","Grape")
list.add("Watermelon") //可以进行添加等操作
```

**Set集合**  
`set`集合的创建和List类似，也有 `setOf()` 和 `muatbleSetOf()`,使用方法相同。  
`set`集合底层时使用hash映射机制来存放数据的，因此集合中各大元素无法保证有序

**Map集合**  
`Map`集合是一种键值对形式的数据结构  
Kotlin中不建议使用`put()`和`get()`方法来对Map进行添加和读取数据操作  
`Msp`集合基础实现，如下:
```kotlin
val map = HashMap<String,Int>
map["Apple"] = 1
map["Banana"] = 2
...
```
`mapOf()`和`mutableMapOf()`使用和前面一致，如下：
```kotlin
val map = mapOf("Apple" to 1, "Banana" to 2, ...)
```
注，`to`不是关键字，而是一个`infix`函数

## 集合函数式 API
`Lambda`就是一小段可以作为参数传递的代码  
语法结构如下：
```
{参数名1:参数类型, 参数名2:参数类型 -> 函数体}
```
解释：如果有参数传入到 Lambda 表达式中的话，需要声明参数列表，参数列表结尾使用一个 `->` 符号，表示参数列表的结束以及函数体的开始，函数体中最后一行代码为自动作为 Lambda 表达式的返回值  
示例如下：
```kotlin
var maxLengthString = list.maxBy({string:String -> string.length})
```
1、当Lambda参数是函数的最后一个参数时，可将Lambda表达式移到函数括号的外面  
2、当Lambda参数是函数的唯一一个参数时，可将括号省略  
3、当Lambda表达式的参数列表中只有一个参数时，可不必声明参数名，可以使用it关键字代替  
最后如下：
```kotlin
var maxLengthString = list.maxBy{it.length}
```
### 常用函数式 API
#### maxBy()
接收一个 Lambda 类型参数，在遍历集合时将每次遍历的值作为参数传给 Lambda 表达式。
根据我们传入的条件来遍历集合，从而找到该条件下的最大值  
示例，找到集合中字符串最长的一个：
```kotlin
var maxLengthString = list.maxBy{it.length}
```

#### map()
接收一个 Lambda 类型参数，将集合中的每个元素都映射成一个另外的值，映射的规则在 Lambda 表达式中指定，最终生成一个新的集合。  
示例：将集合中的字母都变成大写：
```kotlin
var upperCase = list.map{it.toUpperCase() }
```

#### filter()
用于过滤集合中的数据可单独使用，也可配合其他函数（如`map`函数）一起使用  
示例，保留5个字母以内的单词：
```kotlin
var newList = list.filter{it.length <= 5 }
                  .map{it.toUpperCase() }
```

#### any()
用于判断集合中是否至少存在一个元素满足条件  
示例，是否存在5个字母以内的单词
```kotlin
val list = listOf("Apple","Banana","Orange","Pear","Grape")
var anyResult = list.any{it.length <=5 }
//运行结果为 true
```

#### all()
用于判断集合中是否所有元素都满足指定条件  
示例，是否所有单词都在5个字母以内
```kotlin
val list = listOf("Apple","Banana","Orange","Pear","Grape")
var anyResult = list.all{it.length <=5 }
//运行结果为 false
```

## Java 函数式 API
当在 Kotlin 中调用一个 Java 方法，并且该方法接收一个**Java单抽象方法接口参数**，就可使用函数式 API  
示例，Java 中最常见的单抽象方法接口--`Runnable`接口，在 Kotlin 中的写法如下：
```kotlin
Thread(object : Runnable{  //创建匿名类实例改用object关键字
    override fun run(){....}
}).start()
```
因为 Thread 类的构造方法符合 Java 函数式 API 的使用条件，可简化为：
```kotlin
Thread(Runnable {  //Runnable类中只有一个待实现方法，故可不显示重写run()方法
    ....
}).start()
```
当一个 Java 方法的**参数列表不存在一个以上 Java 单抽象方法接口**时可以将接口名省略，如下所示：
```kotlin
Thread{....}.start()
```
Android 中最为常用的 Java 单抽象方法接口，`OnClickListener`，在 Kotlin 中可这样写：
```kotlin
button.setOnClickListener{.....}
```
Java 函数式 API 的使用都**仅限于从 Kotlin 中调用 Java 方法，并且单抽象方法接口也是用 Java 定义**的。

## 空指针检查
### `?` 操作符
 Kotlin 中**默认所有参数和变量都不可为空。**  
可为空类型系统可以让需要的变量或参数为空，如下：
```kotlin
var int:Int?
var string:String?
var c:ClassName?
```
 `?` 符号表示该类型的变量可为空  
**注意，使用可为空类型的参数或变量时需将空指针异常进行处理**

###  `?.` 操作符
结合上述的 `?` 操作符一起使用时，如下：
```kotlin
fun doSomething(study:Study?){
    study?.doHomework()
    study?.readBook()
}
```
当对象不为空时正常调用相应方法，为空时什么都不做  

### `?:` 操作符
这个操作符左右两边都接收一个表达式，如果**左边的表达式不为空则返回左边的结果**，**否则返回右边的表达式的结果**，示例如下：
```kotlin
var c = if(a! = null){
    a
} else {
    b
}
//以上运用?:操作符可简化为
var c = a ?: b
```
将 ?. 和 ?: 结合使用，示例如下
```kotlin
fun getTextLength(text:String?) = text?.length() ?: 0
```

### 非空断言工具 `!!`
在一些情况下，当你非常确定变量不会为空时，为了通过编译器的限制，在对象后面加上 `!!` 可断言该对象不为空，强行通过编译

更多文章：  
[Kotlin学习知识点整理-基础篇-01](https://juejin.im/post/6891231633702125576)  
[Kotlin学习知识点整理-基础篇-02](https://juejin.im/post/6891260438219227143)  
[Kotlin学习知识点整理-基础篇-03](https://juejin.im/post/6891621855204786184)  
[Kotlin学习知识点整理-进阶篇-04](https://juejin.im/post/6892393981695819789)
>写在最后  
>分享一些话：有些时候，当你所处的环境，都像一艘船一样沉下去的时候，你仍需知道，你最大，最优先的责任，是要先救起你自己