---
layout: post
title: "Parameterized type"
date: 2017-01-05 21:47
comments: true
categories: scala
---

#### Generic

scala支持`parameterized types`,其实和java中的`generic`是一个含义，java用`<T>`,scala用`[T]`

```scala
val strings: List[String] = List("one", "two", "three")
val ints: List[Int] = List(1, 2, 3)
val somes: List[Some[Int]] = List(Some(1), Some(2), None)
```

#### Variance

```scala
class X[+A] {...}   		//covariant
class X[-A] {...}   		//contravariant
class X[A] {...}    		//invariant
class X[-A, B, +C] {...}	//mixed
```

`+`表示如果`A`是`B`的子类，则`X[A]`是`X[B]`的子类

`-`表示如果`A`是`B`的子类，则`X[B]`是`X[A]`的子类

如果既不是`+`也不是`-`，则`X[A]`和`X[B]`没有关系

##### Covariant

`String`是`AnyRef`的子类，`List[String]`也是`List[AnyRef]`的子类。`List`在scala中的声明如下

```scala
sealed abstract class List[+A]
```

##### Cotravariant

最简单的contravariant是traits `FuntionN`, `N`从0到22，代表函数参数的数量，比如`scala.Function2`。scala使用这些traits来实现匿名函数

```scala
List(1, 2, 3, 4) map (i => i + 3) // List(4, 5, 6, 7)
```

函数表达式`i => i + 3`其实是`scala.Funtion1`的语法糖

```scala
val f: Int => Int = new Function1[Int, Int] {
  def apply(i: Int): Int = i + 3
}
List(1, 2, 3, 4) map f
```

由于Java 8之前的jvm不支持匿名函数，所以scala采用匿名类的方式来实现匿名函数，Java 8中添加了lambdas支持

traits `scala.Function1`的声明为

```scala
trait Function1[-T, +R] extends AnyRef
```

第一个类型参数为函数参数，是contravariant，第二个类型参数为函数返回值，是covariant。对于其他`FunctionN` ，函数参数都是contravariant

```scala
class CSuper { def msuper() = println("CSuper") }
class C extends CSuper { def m() = println("C") }
class CSub extends C { def msub() = println("CSub") }

var f: C => C 	= 	(c: C)		=> new C
    f:			=	(c: CSuper)	=> new CSub
    f:			=	(c: CSuper) => new C
    f: 			=	(c: C)		=> new CSub
    f:			=	(c: Csub)	=> new CSuper		//编译错误！
```

为什么?

<!-- more -->

> Liskov Substitution Principle (require less, provide more)

通俗解释 `(c: CSuper) => new CSub`是安全的：

我们将会使用函数`f: C => C`，所以任何的有效的`C`类型的值可以作为函数调用的参数，并且 `f`会返回`C`类型的值。

当我们调用函数`f`的时候，实际上调用的是 `(c: CSuper) => new CSub`。这个函数不仅接受`C`类型的值，并且可以接受`C`的父类`CSuper`。所以当我们仅仅用`C`类型的值作为参数的时候，保证是安全的，因为函数`(c: CSuper) => new CSub`里只会使用`CSuper` 的方法。同样地，对于返回值，调用者期望得到类型`C`的值，函数返回`CSub`是安全的。

##### Invariant

对于mutable的类型参数，只能允许invariant

```scala
class ContainerPlus[+A](var value: A) 
// error, covariant type A occurs in contravariant position
```

对于mutable的变量，内部实现是把它当做一个private变量，然后自动提供getter和setter函数

```scala
class ContainerPlus[+A](var a: A) {
  private var _value: A = a
  def value_=(newA: A): Unit = _value = newA
  def value: A = _value
}
```

A同时出现在函数参数和函数返回值的位置上，既不能covariant也不能contravariant，所以只能invariant。

scala中的`Array`是mutable的，所以是invariant的，`Array[String]` 和`Array[AnyRef]`之间没有subtype的关系。

java中的`Array`是variant的，虽然可以通过编译，但是会有运行时异常。

```java
public class JavaArrays {
  public static void main(String[] args) {
    Integer[] array1 = new Integer[] {new Integer(1), new Integer(2) };
    Number[] array2 = array1;
    array2[1] = new Double(3.14);      // runtime error
  }
}
```

#### Lower Type Bounds

```scala
class Queue[+T] {
  def enqeue[T](x: T): Queue[T] = {...}    //编译错误
}
```

由于类型参数`T`出现在函数参数的位置，`Queue`不能被设计成covariant，但是我们可以利用`Lower Bounds`来突破限制。

```scala
class Queue[+T] {
  def enqueue[U >: T](x: U): Queue[U] = {...}
}
```

`U >: T`表示传入的参数类型`U`必须是`T`的supertype。`T`是本身的supertype，所以也可以用类型`T`传入函数。

```scala
val q: Queue[Apple] = ...
val orange: Orange = ...
val nq: Queue[Fruit] = q.enqueue(orange) //得到了一个拥有多种水果的队列
```

#### Upper Type Bounds

```scala
class Person(val firstName: String, val lastName: String) extends Ordered[Person] {
  def compare(that: Persion) {
    lastName.compareToIngnoreCase(that.lastName)
  }
  override def toString = firstName + " " + lastName
}
```

通过实现traits `Ordered`的`compare`方法, 我们可以用`>`,`<`,`=`对`Person`进行比较。

如果定义一个排序函数对一个List进行排序，要求List中的元素具备了通过`>`,`<`,`=`进行比较的功能，我们可以通过`Upper Type Bounds`来做。

```scala
def orderedMergeSort[T <: Ordered[T]](xs: List[T]): List[T] = {
  def merge(xs: List[T], ys: List[T]): List[T] = 
    (xs, ys) match {
      case (Nil, _) => ys
      case (_, Nil) => xs
      case (x :: xs1, y :: ys1) =>
      	if (x < y) x :: merge(xs1, ys)
        else y :: merge(xs, ys1)
  	}
  val n = xs.length / 2
  if (n == 0) xs
  else {
    val (ys, zs) = xs splitAt n 
    merge(orderedMergeSort(ys), orderedMergeSort(zs))
  }
}
```

`T <: Ordered[T]`表示传入排序函数的List元素必须是`Ordered[T]`的subtype，`T`是本身的subtype。

```scala
val people = List(new Person("aa", "bb"), new Person("cc", "dd"))
orderMergeSort(people)
  
orderMergeSort(List(3,2,1))    //编译错误，Int不是Ordered[Int]的subtype
```

参考书籍：

1. Programming in Scala, 3rd edition
2. Programming Scala 