---
title: "Spark笔记(一)"
date: 2020-05-07T10:27:18+08:00
draft: true
---

## Scala

---
### 简介 :
Scala是一门多范式的编程语言,一种类似Java的编程语言,设计初衷是实现可伸缩的语言,并集成面向对象编程和函数式编程的各种特性
### 特征 :
1. Java和scala可以无缝混编,都是运行在JVM之上的
2. 类型推测(自动推测类型),不用指定类型
3. 并发和分布式(Actor,类似Java多线程Thread)
4. 特质Trait(类似Java中interfaces和abstract结合)
5. 模式匹配,match case(类似Java中的switch case)
6. 高阶函数(函数的参数是函数,函数的返回值和函数),可进行函数式编程

### 数据类型
- Unit表示无值,和其他语言中void等同
- Null空值或者空引用
- Nothing所有其他类型的子类型,表示没有值
- Any 所有类型的超类,任何实例都属于Any类型
- AnyRef所有引用类型的超类
- AnyVal所有值类型的超类
- Boolean布尔类型
- String 字符串
- char 16bit Unicode字符
- double 双精度浮点型
- Float 单精度浮点型
- Long 有符号数字
- Int 整性
- Short 16 bit有符号数字
- Byte 8bit的有符号数字
![image-20200507102812424](https://i.loli.net/2020/05/07/uyBOGRKa4w2oNvm.png)

| Null    | Trait,其唯一实例为null,是AnyRef的子类,不是AnyVal的子类 |
| ------- | ------------------------------------------------------ |
| Nothing | Trait,所有类型(包括AnyRef和AnyVal)的子类,没有实例      |
| None    | Option的两个子类之一,另一个是Some,用于安全的函数返回值 |
| Unit    | 无返回值的函数的类型,和Java的void对应                  |
| Nil     | 长度为0的List                                          |

### 函数
> 普通函数 

```scala
  // 1.普通函数 : def 方法名称(参数列表):(返回值)={方法体}
  def fun (a:Int,b:Int):Unit={
    println(a+b)
  }
  def fun1(a:Int,b:Int)=a+b
```
> 递归函数 

```scala
  // 2.递归函数
  def fun2(num:Int):Int={ // 必须写返回值类型
    if (num==1)
      num
    else
      num * fun2(num-1)
  }
```
> 包含参数默认值的函数

```scala
  // 3. 包含参数默认值的函数
  //3.1 函数的参数有默认值,在调函数的时候可以传参,也可以不传参
  //3.2 不传参 : 使用的是默认值, 传参 : 会覆盖默认值
  //3.3 传参的时候 可以指定参数名进行传值
  def fun3(num1:Int=10,num2:Int=20)={
    num1 + num2
  }
//  println(fun3())
//  println(fun3(100))
//  println(fun3(num2=1000))
```
> 可变参数个数的函数

```scala
 // 4. 可变参数个数的函数:函数参数可以是一个也可以是多个,随机灵活的
  def fun4(args:Double*)={
    /**
      * 在scala中
      *     +=前后的数据类型必须一致
      *     +前后的数值类型可以不一致
      */
    var sum = 0.0
    for (arg <- args) sum += arg
    sum
  }
```
> 匿名函数

```scala
 /**
    * 5. 匿名函数 : 没有函数名的函数
    * 5.1 有参数的匿名函数
    * 5.2 没有参数的匿名函数
    * 5.3 有返回值的匿名函数
    * 注意 :
    * 可以将匿名函数返回给定义的一个变量
    * 匿名函数不能显式声明函数的返回类型
    */
  // 有参数的匿名函数
  val value1 = (a: Int) => {
    println(a)
  }
  // 无参数的匿名函数
  val value2 =()=>{
    println("天干物燥")
  }
  // 有返回值的匿名函数
  val value3 =(a:Int,b:Int) => {
    a+b
  }
```
> 嵌套函数

```scala
/**
    * 6. 嵌套函数 : 在函数体里面又定义了一个函数
    */
  def fun5(num:Int)={
    def fun6(a:Int,b:Int):Int={
      if (a==1){
        b
      }else{
        fun6(a-1,a*b)
      }
    }
    fun6(num,1)
  }
```
> 高阶函数

```scala
  /**
    * 高阶函数 :
    * 函数的参数是函数,
    * 或者函数的返回类型是函数,
    * 或者函数的参数和函数的返回类型是函数的函数
    */
    // 函数的参数是函数
    def heightFun(f:(Int,Int)=>Int,a:Int):Int = {
      f(a,100)
    }
    def f(v1:Int,v2:Int)={
      v1+v2
    }
    //println(heightFun(f, 100))
     // 函数的返回值是函数
  def highrtFun2( a:Int,b:Int ):( Int,Int )=>Int={
    def f2 (v1:Int,v2:Int)={
      v1+v2+a+b
    }
    f2
  }
  //println(highrtFun2(1, 2)(3, 4))

  //函数的参数是函数,函数的返回值是函数
  def highrtFun3(f:(Int,Int)=> Int):(Int,Int)=>Int={
    f
  }
  //println(highrtFun3(f)(100, 200))
```
> 柯里化函数
```scala

  /**
    * 柯里化函数
    *       可以理解为高阶函数的简化
    *       柯里化函数就是对嵌套函数这种情况的高阶函数的简化版
    */
  def klhFun(a:Int)(b:Int) = a * b

  // 具体
  def klhAllFun(a:Int):(Int)=>Int={
    val fun = (b:Int)=>a*b
    fun
  }

```

### 常用
#### yield

 for循环中的 yield 会把当前的元素记下来，
保存在集合中，循环结束后将返回该集合。
Scala中for循环是有返回值的。如果被循环的是Map，返回的就是Map，
被循环的是List，返回的就是List
```scala
 for (i <- 1 to 5) yield i
scala.collection.immutable.IndexedSeq[Int] = Vector(1, 2, 3, 4, 5)

```
#### getOrElse

getOrElse用于当集合或者option中有可能存在空值或不存在要查找的值的情况，其作用类似于
```scala
val result = Option(myType) match {
  case Some(value) =>
    // 有值
    // todo
  case None =>
    // 没有值时的处理
}
```
- Map中的用法
```scala
myMap.getOrElse("myKey", "no such key")
```
当不存在"myKey"时，则直接返回"no such key"
- Option中的用法
```scala
val op1 = Option[String]("value exists")
val op2 = None
println(op1.getOrElse("no value here"))
println(op2.getOrElse("no value here"))
```
则，上面会输出value exists而下面则输出no value here.

