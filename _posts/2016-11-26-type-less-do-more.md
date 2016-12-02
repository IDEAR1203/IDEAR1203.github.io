---
layout: post
title: 《Scala程序设计》笔记 - 更简洁，更强大
---

本章将继续探究Scala的特性，重点关注它如何为我们提供简洁且灵活的语法代码。我们将探讨如何组织文件和包，如何导入其他类型、变量、方法声明，以及一些非常有用的数据类型和和各种约定俗成的语法习惯。

<!--more-->

## 分号

可以将多个表达式放在同一行中，表达式之间用分号隔开。

> 如果需要将多行代码解释为同一表达式，却被系统视为多个表达式，可以用REPL的`:paste`模式。输入`:paste`，然后输入你的代码，最后用`Ctrl-D`结束。

## 变量声明

一个不可变的“变量”用`val`关键字声明：

```scala
val array: Array[String] = new Array(5)
```

一个可变的变量用`var`关键字声明：

```scala
var stockPrice: Double = 100.0
```

在Java中，所谓的原生类型，即`char`、`byte`、`short`、`int`、`long`、`float`、`double`和`boolean`,与其他引用类型有着本质的不同。这些类型确实既不是对象，也没有引用，是“原始”值。Scala尽力使其面向对象特性更加一致，因此这些类型在Scala中是包含方法的对象，就像引用类型一样。然而，Scala编译时将这些类型尽可能地转为原生类型，使你可以得到原生类型的运行效率。

> var和val关键字只标识引用本身是否可以指向另一个不同的对象，他们并未表明其所引用的对象是否可变。

为了减少可变性引起的bug，应该尽可能地使用不可变变量。

例如，在散列映射中，可变对象是非常危险的。如果对象发生改变，`hashCode`方法的输出就会发生变化，在散列映射表中原来的位置就无法找到对应的值了。

更为常见的是，当你正在使用的对象被其他人修改时，将引起对象产生不可预见的行为。借助量子力学的名词，这是一种“幽灵般的超距作用”，本地的所有操作都无法解释这种不可预见行为，因为这是由其他某处的操作引起的。

这在多线程程序中是最致命的bug。在多线程程序中，对共享的可变状态进行读写之前要使用同步操作，但实践中往往很难实现正确的同步。

这个时候，如果使用的是不可变的值，就可以减少这类问题。

## Range

有时我们需要一个数字序列，从某个起点到某个终点。而`Range`能满足这个需要。以下实例将展示如何创建`Range`，支持`Range`的类型包括`Int`、`Long`、`Float`、`Double`、`Char`、`BigInt`、`BigDecimal`。

```scala
// range.sc

1 to 10                 // Int类型的Range，包括区间上限，步长为1（从1到10）
1 until 10              // Int类型的Range，不包括区间上限，步长为1（从1到9）
1 to 10 by 3            // Int类型的Range，包括区间上限，步长为3
10 to 1 by -3           // Int类型的Range，包括区间上限，步长为-3
1L to 10L by 3          // Long类型
1.1f to 10.3f by 3.1f   // Float类型的Range，步长可以不等于1
1.1f to 10.3f by 0.5f   // Float类型的Range，步长可以小于1
1.1 to 10.3 by 3.1      // Double类型
'a' to 'g' by 3         // Char类型
BigInt(1) to BigInt(10) by 3                // BigInt类型
BigDecimal(1.1) to BigDecimal(10.3) by 3.1  // BigDecimal类型
```

脚本执行结果：

```
scala> :load test/range.sc
Loading test/range.sc...
res0: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
res1: scala.collection.immutable.Range = Range(1, 2, 3, 4, 5, 6, 7, 8, 9)
res2: scala.collection.immutable.Range = Range(1, 4, 7, 10)
res3: scala.collection.immutable.Range = Range(10, 7, 4, 1)
res4: scala.collection.immutable.NumericRange[Long] = NumericRange(1, 4, 7, 10)
res5: scala.collection.immutable.NumericRange[Float] = NumericRange(1.1, 4.2, 7.2999997)
res6: scala.collection.immutable.NumericRange[Float] = NumericRange(1.1, 1.6, 2.1, 2.6, 3.1, 3.6, 4.1, 4.6, 5.1, 5.6, 6.1, 6.6, 7.1, 7.6, 8.1, 8.6, 9.1, 9.6, 10.1)
res7: scala.collection.immutable.NumericRange[Double] = NumericRange(1.1, 4.2, 7.300000000000001)
res8: scala.collection.immutable.NumericRange[Char] = NumericRange(a, d, g)
res9: scala.collection.immutable.NumericRange[BigInt] = NumericRange(1, 4, 7, 10)
res10: scala.collection.immutable.NumericRange.Inclusive[scala.math.BigDecimal] = NumericRange(1.1, 4.2, 7.3)
```

## 偏函数

偏函数（PartialFunction）之所以“偏”，原因在于它们并不处理所有可能的输入，而只处理那些能与至少一个`case`匹配的输入。

在偏函数中只能使用`case`语句，而整个函数也必须用花括号包围。

如果偏函数被调用，而函数的输入与所有的语句都不匹配，系统就会抛出一个`MatchError`运行时错误。

我们可以使用`isDefineAt`方法测试特定输入是否与偏函数匹配，这样偏函数就可以避免抛出`MatchError`错误了。

偏函数可以“链式连接”：`pf1 orElse pf2 orElse pf3`。如果`pf1`不匹配，就会尝试`pf2`，接着是`pf3`，以此类推。如果都不匹配，才会抛出`MatchError`。

## 方法声明

### 方法默认值和命名参数列表

`copy`方法允许你在创建`case`类的新实例时，只给出与原对象不同部分的参数。

```scala
// test/methodDeclaration.sc

case class Point(x: Double = 0.0, y: Double = 0.0) {

  def shift(deltax: Double = 0.0, deltay: Double = 0.0) =
    copy (x + deltax, y + deltay)
}

val p1 = new Point(x = 3.3, y = 4.4)
val p2 = p1.copy(y = 6.6)
```

脚本输出如下：

```
scala> :load test/methodDeclaration.sc
Loading test/methodDeclaration.sc...
defined class Point
p1: Point = Point(3.3,4.4)
p2: Point = Point(3.3,6.6)
```

命名参数列表让客户端代码更具可读性。当参数列表很长，且有若干参数是同一类型时，使用命名参数列表容易避免bug，因为在这种情况下容易搞错参数传入的顺序。当然，更好的做法是一开始就避免过长的参数列表。

### 方法具有多个参数列表

```scala
abstract class Shape() {
  /**
   * Draw takes TWO argument LISTS, one list with an offset for drawing,
   * and the other list that is the function argument we used previously.
   */
  def draw(offset: Point = Point(0.0, 0.0))(f: String => Unit): Unit =
    f(s"draw(offset = $offset), ${this.toString}")
}
```

这里的`draw`方法有两个参数列表，每个参数列表带有一个参数。

参数列表的个数可以任意指定，但实际上很少有人使用两个以上的参数列表。

当最后一个参数列表只包含一个表示函数的参数时，多个参数列表的形式拥有整齐的块结构语法。以下是我们调用新的`draw`方法的表达形式：

```scala
s.draw(Point(1.0, 2.0))(str => println(s"ShapesDrawingActor: $str"))
```

Scala允许我们将参数列表两边的圆括号替换为花括号，因此，这行代码可以写为：

```scala
s.draw(Point(1.0, 2.0)){str => println(s"ShapesDrawingActor: $str")}
```

如果函数字面量不能在一行完成，可以写成如下形式：

```scala
s.draw(Point(1.0, 2.0)) {
    str => println(s"ShapesDrawingActor: $str")
}
```

如果第一个参数使用缺省参数，第一个圆括号就不能省略：

```scala
s.draw() {
    str => println(s"ShapesDrawingActor: $str")
}
```

第二个优势是在之后的参数列表中进行类型推断。如以下例子：

```scala
// test/typeInference.sc

def m1[A](a: A, f: A => String) = f(a)
def m2[A](a: A)(f: A => String) = f(a)
m1(100, i => s"$i + $i")
m2(100)(i => s"$i + $i")
```

脚本执行结果：

```
scala> :load test/typeInference.sc
Loading test/typeInference.sc...
m1: [A](a: A, f: A => String)String
m2: [A](a: A)(f: A => String)String
<console>:13: error: missing parameter type
       m1(100, i => s"$i + $i")
               ^
res12: String = 100 + 100
```

对于`m1`，Scala无法推断该函数的参数`i`，`m2`则可以。`m1`需要写成如下形式：

```scala
m1(100, (i: Int) => s"$i + $i")
```

使用多个参数列表的第三个优势是，我们可以用最后一个参数列表来推断隐含参数。隐含参数使用`implicit`关键字声明的参数。当相应方法被调用时，我们可以显式指定这个参数，或者可以不指定，这时编译器会在当前作用域中找到一个合适的值作为参数。 **隐含参数可以代替参数默认值，并且更加灵活。**

### Future简介

```scala
// src/main/scala/progscala2/typelessdomore/futures.sc
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def sleep(millis: Long) = {
  Thread.sleep(millis)
}

// Busy work ;)
def doWork(index: Int) = {
  sleep((math.random * 1000).toLong)
  index
}

(1 to 5) foreach { index =>
  val future = Future {
    doWork(index)
  }
  future onSuccess {
    case answer: Int => println(s"Success! returned: $answer")
  }
  future onFailure {
    case th: Throwable => println(s"FAILURE! returned: $th")
  }
}

sleep(1000)  // Wait long enough for the "work" to finish.
println("Finito!")
```

`Future.apply`是单例对象`Future`的“工厂”方法。在这个例子中，`Future.apply`传入了一个匿名函数，表示需要做的任务。

`Future.apply`返回一个新的`Future`对象，然后控制权就交还给循环了，该对象在另一个线程中执行`doWork`、接着，我们用`onSuccess`注册了一个回调函数，当`future`成功执行完毕后，该回调将会被执行。这个回调函数是一个偏函数。

类似地，我们用`onFailure`注册了一个回调函数来处理错误。

### 嵌套方法与递归

当你将一个很长的方法重构为几个更小的方法时，如果这些小的辅助方法只在该方法中调用，就可以用嵌套方法。我们将这些辅助函数嵌套定义在原方法中，它们便对其他外层函数不可见，包括类中的其他方法。

```scala
// src/main/scala/progscala2/typelessdomore/factorial.sc

def factorial(i: Int): Long = {
  def fact(i: Int, accumulator: Long): Long = {
    if (i <= 1) accumulator
    else fact(i - 1, i * accumulator)
  }

  fact(i, 1L)
}

(0 to 5) foreach ( i => println(factorial(i)) )
```

> 很容易忘记在原方法中调用嵌套的函数！这将导致原方法返回`Unit`。

必须为`fact`声明返回类型，因为这个是一个递归方法。Scala采用的是局部作用域类型推断，无法推断出递归函数的返回类型。

JVM和许多其他语言环境并不对尾递归做优化。对尾递归做优化的语言会将递归转换为循环，来避免栈溢出。

尾递归表示调用递归函数是该函数的最后一个表达式，该函数的返回值就是所调用的递归函数的返回值。

Scala编译器对尾递归做了有限的优化。它会对函数调用自身做优化，但是不会优化所谓的trampoline的情况，也就是“a调用b调用a调用b”的情形。

如果你使用`tailrec`关键字，编译器会告诉你代码是否正确地实现了尾递归。如果不是尾递归，编译器会抛出错误。

```scala
// test/fibonacci.sc

import scala.annotation.tailrec

@tailrec
def fibonacci(i: Int): Long = {
  if (i <= 1) 1L
  else fibonacci(i - 2) + fibonacci(i - 1)
}
```

脚本执行结果：

```
scala> :load test/fibonacci.sc
Loading test/fibonacci.sc...
import scala.annotation.tailrec
<console>:15: error: could not optimize @tailrec annotated method fibonacci: it contains a recursive call not in tail position
         else fibonacci(i - 2) + fibonacci(i - 1)
                               ^
```

我们有两个递归调用，然后又对递归的结果做计算，而不是只在结尾调用一次递归函数，因此这个函数不是尾递归。

外层方法所在作用域的一切在嵌套方法中都是可见的，包括传递给外层方法的参数。

## 推断类型信息

需要显式类型注解的情形：

* 声明了可变的`var`变量或者不可变的`val`变量，没有进行初始化。（例如在类中的抽象声明，如`val book: String, var count: Int`）。
* 所有的方法参数（如`def deposit(amount: String) = {...}`）。
* 方法的返回值类型，在以下情况中必须显式声明其类型。
  - 在方法中明显地使用了`return`（即使在方法结尾也是如此）。
  - 递归方法。
  - 两个或多个方法重载（拥有相同的函数名）,其中一个方法调用了另一个重载方法，调用者需要显式类型注解。
  - Scala推断出的类型比你期望的类型更为宽泛，如`Any`。

> 开发API时，如果与客户端分开构建，应该显式地声明返回类型，并尽可能地返回一个你所能返回的最通用的类型。

## 保留字

Scala的保留字中没有`break`和`continue`。Scala鼓励使用函数式编程的惯用法来实现相同的功能。

## 字面量

### 整型字面量

对于`Long`类型的字面量，除非你将该字面量赋值给一个`Long`类型的变量，否则需要在数字字面量后面加上`l`或者`L`。否则，字面量类型将默认推断为`Int`。

### 浮点数字面量

除非被赋值的变量是`Float`类型，或者在字面量后面加了`F`或`f`，否则字面量都将被推断为`Double`类型。

### 布尔型字面量

### 字符字面量

### 字符串字面量

字符串字面量是被双引号或者三重引号包围的字符串序列，如`"""..."""`。

用三重双引号包含的字符串字面量也被成为多行字符串字面量。这些字符串可以跨越多行，换行符是字符串的一部分。

* 可以包含任意字符，可以是一个双引号也可以是两个连续的双引号。但不允许出现三个连续的双引号。
* 反斜杠`\\`不用于构成Unicode字符，也不用于构成有效的转义序列。

### 符号字面量

Scala支持符号，符号是一些有规定的字符串。两个同名的符号会指向内存中的同一对象。

符号字面量是以单引号`'`后面跟上一个或多个数字、字母或下划线`_`，但第一个字符不能为数字。

符号字面量`'id`是表达式`scala.Symbol("id")`的简写形式。

### 函数字面量

### 元组字面量

表达式`t._n`提取元组`t`中的第`n`个元素。从1开始计数。

一个两元素的元组，有时也被简称为pair。定义pair的方法：

```scala
(1, "one")
1 -> "one"
1 → "one"
Tuple2(1, "one")
```

