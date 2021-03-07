---
title: Kotlin学习-进阶篇-05
date: 2021-03-04 19:50:00
tags:
    - kotlin
    - 笔记
    - 阅读
categories: 
    - [Kotlin]
top_image: https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16bc39bfcece47bd96edee5511fb1ca8~tplv-k3u1fbpfcp-watermark.image
excerpt: Kotlin知识点整理进阶 一、运算符重载 二、扩展函数 + 运算符重载使用示例 三、高阶函数 1、定义高阶函数 2、内联函数
---
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16bc39bfcece47bd96edee5511fb1ca8~tplv-k3u1fbpfcp-watermark.image)
# Kotlin知识点整理进阶
>本文是我在学习Kotlin的过程中整理的一些知识点，如有错误，欢迎指正
## 一、运算符重载
运算符重载使用的是 `operator` 关键字  
以加号运算符为例，实现让两个对象相加的功能，语法结构如下：
```kotlin
class OBj{
    operator fun plus(obj: Obj): Obj{
     //处理相加逻辑
    } 
}
//调用方法如下
val obj1 = Obj()
val obj2 = Obj()
val obj3 = obj1 + obj2 
```
<table width="400">
    <caption>语法糖表达式和实际调用函数对照表</caption>
    <tr>
        <th>语法糖表达式</th>
        <th>实际调用函数</th>
    </tr>
    <tr>
        <td align="center">a + b</td>
        <td align="center">a.plus(b)</td>
    </tr>
    <tr>
        <td align="center">a - b</td>
        <td align="center">a.minus(b)</td>
    </tr>
    <tr>
        <td align="center">a * b</td>
        <td align="center">a.times(b)</td>
    </tr>
    <tr>
        <td align="center">a / b</td>
        <td align="center">a.div(b)</td>
    </tr>
    <tr>
        <td align="center">a % b</td>
        <td align="center">a.rem(b)</td>
    </tr>
    <tr>
        <td align="center">a++</td>
        <td align="center">a.inc()</td>
    </tr>
    <tr>
        <td align="center">a--</td>
        <td align="center">a.dec()</td>
    </tr>
    <tr>
        <td align="center">+a</td>
        <td align="center">a.unaryPlus()</td>
    </tr>
    <tr>
        <td align="center">- a</td>
        <td align="center">a.unaryMinus()</td>
    </tr>
    <tr>
        <td align="center">! a</td>
        <td align="center">a.not()</td>
    </tr>
    <tr>
        <td align="center">a == b</td>
        <td align="center">a.equals(b)</td>
    </tr>
    <tr>
        <td align="center">a > b </td>
        <td align="center" rowspan="4">a.compareTo(b)</td>
    </tr>
    <tr>
        <td align="center">a < b</td>
    </tr>
    <tr>
        <td align="center">a >= b</td>
    </tr>
    <tr>
        <td align="center">a <= b</td>
    </tr>
    <tr>
        <td align="center">a..b</td>
        <td align="center">a.rangeTo(b)</td>
    </tr>
    <tr>
        <td align="center">a[b]</td>
        <td align="center">a.get(b)</td>
    </tr>
    <tr>
        <td align="center">a[b] = c</td>
        <td align="center">a.set(b, c)</td>
    </tr>
    <tr>
        <td align="center">a in b</td>
        <td align="center">b.contains(a)</td>
    </tr>
</table>

注：最后一个 `a in b`,表示判断 `a` 是否在 `b` 中，而 `b.contains(a)` 表示判断 `b` 是否包含 `a`，因此这两种表达方式是等价的。

## 二、扩展函数 + 运算符重载使用示例
下面来看个示例，现在准备用扩展函数+运算符重载来实现一个让字符串重复打印指定次数的功能，类似 `str * n`这种形式，代码如下：  
```kotlin
operator fun String.tines(n : Int) : String {
    val builder = StringBuilder()
    repeat(n) {
        builder.append(this)
    }
    return builder.toString
}
```
`operator`关键字是必不可少的，接着参考上面的表可知函数名是 `tines`，然后由于我们是给字符串添加的扩展函数，故在方法名前面加上了 `String` ，最后借助 `repeat()` 将字符串重复 `n` 次。  

现在，字符串就有了可以和数字相乘的能力，比如执行如下代码：
```kotlin
val str = "abc" * 3
println(str)

//最终打印结果就是：abcabcabc
```
其实 `Kotlin` 中的 `String` 类中已经提供了一个 `repeat()` 函数，所以我们还可以简化上面的函数，如下：
```kotlin
operator fun String.tines(n : Int) = repeat(n)
```


## 三、高阶函数
### 1、定义高阶函数
定义：如果一个函数接收另一个函数作为参数，或者返回值的类型是另一个函数，那么该函数就称为高阶函数。  

定义一个函数类型：
```kotlin
(String, Int) -> Unit
```
`->` 左边的部分就是用来声明该函数接收什么参数的，多个参数之间使用逗号隔开，如果不接受任何参数，写一对空括号就可以。  
`->` 右边的部分用于声明该函数的返回值是什么类型，如果没有返回值就使用 Unit，它大致相当于Java中的 void  

定义一个高阶函数：
```kotlin
fun example(func: (String, Int) -> Unit){
    func("hello",123)
}
```
用途：高阶函数允许让函数类型的参数来决定函数的执行逻辑。即使是用一个高阶函数，只要传入不同的函数类型参数，那么它的执行逻辑和最终的返回结果就可能是完全不同的。  
我们来看一个示例：
```kotlin
fun StringBuilder.build(block: StringBuilder.() -> Unit): StringBuilder{
    block()
    return this
} 
```
我们给 `StringBuilder` 定义了一个扩展函数 `build` ，它接收一个函数类参数。并且如果在函数类型前面加上了 `ClassName.` 的语法结构，就表示这个函数类型是定义在哪个类中的。故这里的参数加上了 `StringBuilder.` 的语法结构，就表示这个函数类型是定义在 `StringBuilder` 这个类中的，这样的作用是当我们调用 `build` 函数时传入的 `Lambda` 表达式可以自动拥有 `StringBuilder` 的上下文，类似于 `apply` 函数，不用的地方在于 `apply` 可以作用到所有类上，这里的只能作用在 `StringBuilder` 类上。  
用法如下：
```kotlin
val result = StringBuilder.build{
    append("Start here \n")
    append("str1 str2 \n")
    append("End here")
}
```

### 2、内联函数
内联函数主要是为了解决运行Lambda表达式时带来的运行时开销。  
内联函数用法 在定义高阶函数时加上 `inline` 关键字，如下所示：
```kotlin
inline fun num1AndNum2(num1:Int,num2:Int,operation: (Int,Int) -> Int) : Int{
    val result = operation(num1, num2)
    return result
}
```
`noinline` 只想内联其中一个Lambda表达式：
```kotlin
inline fun num1AndNum2(block1 : () -> Unit, noinline block2 : () -> Unit){
}
```
Q：为何Kotlin要提供一个 `noinline` 关键字来排除内联功能？  
A：内联函数的函数类型参数在编译时会进行代码替换，因此它没有真正的参数属性。非内联的函数类型参数可以自由地传递给其它任何函数，因为它就是一个真实的参数，而内联的函数类型参数只允许传递给另一个内联函数。  
（补充：内联函数与非内联函数还有一个重要的区别：内联函数所引用的Lambda表达式中是可以使用 return 关键字来进行函数返回的，而非内联函数只能局部返回）  

上一篇 [Kotlin学习知识点整理-进阶篇-04](https://juejin.cn/post/6892393981695819789)
