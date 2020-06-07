+++
title= "Covariance and Contravariance"
date= "2017-08-17"
tags= ["concurrency"]
+++
Java的extends、Scala的+、Kotlin的out是协变，
Java的super、Scala的-、Kotlin的in是逆变。
<!--more-->
# 背景
定义继承结构
```scala
class A
class B extends A
class C extends B
class D extends B
```
使用泛型（无variance）
```scala
trait ProducerConsumer[T] {
    def produce(): T
    def consume(x:T)
}
```
如果以如下方式使用，则会抛出“Expression of type ProducerConsumer[C] doesn't conform to expected type ProducerConsumer[B]”的错误
```scala
var a:ProducerConsumer[B]
var b:ProducerConsumer[C]
a = b//抛出 Expression of type ProducerConsumer[C] doesn't conform to expected type ProducerConsumer[B]
```
编译器不允许这样泛型类型的直接赋值是因为下面这个例子：
```scala
var a:ProducerConsumer[B]
var b:ProducerConsumer[C]
var d:D
a = b//引用赋值
a.consume(d)//d为D类型，是B类型的子类，理应可以放入a中
b.produce()//这时，如果调用b的produce()函数，那么应该将d抛出，但是d是D类型，与b定义的C类型不兼容，会产生CastException
```
上面这样随意地按泛型类型的继承关系进行赋值的应用不应该被支持，但如果是一些限制性的使用方式，是应该被支持。

# 协变
如果泛型符号只在输出位置处使用（函数返回值），则可以扩展为协变泛型
```scala
trait Producer[+T] {
    def produce(): T
}
```
协变泛型允许如下赋值
```scala
var a:Producer[B]
var b:Producer[C]
a = b //因为C是B的子类，所以协变泛型下，Producer[C]就是Producer[B]的子类，可以赋值
var d:B = a.produce()//此处实际调用的是b.produce()，但由于返回的C类型是B类型的子类，所以可以合理的赋值给B类型的d
```
# 逆变
如果泛型符号只在输入位置处使用（函数参数），则可以扩展为逆变泛型
```scala
trait Consumer[-T] {
    def consume(x: T)
}
```
逆变泛型允许如下赋值
```scala
var a:Consumer[B]
var b:Consumer[A]
a = b //因为A是B的父类，所以逆变泛型下，Consumer[A]就是Consumer[B]的子类，可以赋值
var d:B
a.consume(d)//此处实际调用的是b.consume()，但由于接受的B类型是A类型的子类，所以可以传递给Consumer[A]。
```
协变、逆变的关系可由下面表示，空心三角表示继承关系，实心三角表示赋值方向。

![alt text](https://gitlab.com/wumo/wumo.gitlab.io/raw/master/static/variance.png)