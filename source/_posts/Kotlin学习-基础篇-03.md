---
title: Kotlin学习-基础篇-03
date: 2020-11-05 19:50:00
tags:
    - kotlin
    - 笔记
    - 阅读
categories: 
    - [Kotlin]
excerpt:  Kotlin 的内置扩展函数 1、let 2、with 3、run 4、apply 5、repeat
top_image: https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1aa72c86f8849039b9b4d4c87cf1bf2~tplv-k3u1fbpfcp-watermark.image
---
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1aa72c86f8849039b9b4d4c87cf1bf2~tplv-k3u1fbpfcp-watermark.image)
# Kotlin 的内置扩展函数
>本文是我在学习Kotlin的过程中整理的一些知识点，如有错误，欢迎指正
### 1、let

示例代码如下：
```kotlin
obj.let { obj2 ->
    //业务逻辑
}
```
将原始调用对象作为参数传递到 Lambda 表达式中，可配合 `?.`操作符进行辅助判空处理

### 2、with

**`with`** 接收两个参数，第一个参数可以是一个任意类型的对象，第二个参数是一个**Lambda**表达式，  
**`with`** 函数会在 **Lambda** 表达式中提供第一个参数对象的上下文，并使用**Lambda**表达式中的最后一行作为返回值返回，示例如下：

```kotlin
val result = with(obj) {
    //这里式obj的上下文
    "value" //最后一行 with 函数的返回值
}
```
可连续调用同一个对象的多个方法

### 3、run

与 **`with`** 函数较为相似，**`run`** 函数不能直接嗲用，而是一定要调用某个对象的 **`run`** 函数才行，**`run`** 函数只接受一个 **Lambda** 参数，并且会在 **Lambda **表达式中提供调用对象的上下文，会使用 **Lambda** 表达式的最后一行作为返回值，示例代码如下：

```kotlin
val result = obj.run{
     //这里式obj的上下文
    "value" //最后一行 with 函数的返回值
}
```

### 4、apply

与 **`run`** 函数极为相似，需在某个对象上调用，只接收一个 **Lambda** 参数，在**Lambda**表达式中提供调用对象的上下文，**`apply`** 函数无法指定返回值，**而是会自动返回调用对象本身，**示例代码如下：

```kotlin
val result = obj.apply{
    //这里式obj的上下文
}
// result == obj
```

### 5、repeat

**`repeat`** 接收两个参数，第一个参数是一个 **`Int`** 类型的参数，用于指定循环次数。第二个参数是一个 Lambda 表达式。  
**`repeat`** 可以将其中的 **Lambda** 表达式重复执行指定的次数。  
示例如下：

```kotlin
 repeat(4) { it ->
       println(" it = $it")
 }
//运行结果
it = 0
it = 1
it = 2
it = 3
```
更多文章：  
[Kotlin学习知识点整理-基础篇-01](https://juejin.im/post/6891231633702125576)  
[Kotlin学习知识点整理-基础篇-02](https://juejin.im/post/6891260438219227143)  
[Kotlin学习知识点整理-基础篇-03](https://juejin.im/post/6891621855204786184)  
[Kotlin学习知识点整理-进阶篇-04](https://juejin.im/post/6892393981695819789)
>写在最后  
>《一日重生》佳句分享：  
>I feel ashamed now that I tried to take my life. It is such a precious thing. I had no one to talk me out of my despair, and that was a mistake. You need to keep people close. You need to give them access to your heart.  
>我曾经试图结束自己的生命，我对此感到羞愧。生命宝贵无比。我没有找任何人谈心，排解我的绝望，这是不对的。你需要与人亲近，你得让他们接近你的心。

