---
标题: 标识符，名称和范围
布局: 默认
章节: 2
---

# 标识符，名称和范围

Scala中的名称标识统称为 *实体* 的类型，值，方法和类。名称由本地[定义和声明](04-basic-declarations-and-definitions.html#basic-declarations-and-definitions)，
[继承](05-classes-and-objects.html#class-members)，
[import引入](04-basic-declarations-and-definitions.html#import-clauses)或
[packages引入](09-top-level-definitions.html#packagings)，这些子句统称为 _bindings_ (绑定)。

不同类型的绑定具有在定义上的不同优先级：

1. 定义和说明，这是本地的，遗传的。或者由package子句提供，并且在引用它们的同一编译单元中定义，具有最高优先级。
1. Explicit imports 具有次高的优先级。
1. Wildcard imports 具有次高的优先级。
1. 由package子句提供的定义，但不与引用定义在同一编译单元中，以及由编译器提供但未显式写入源代码的导入具有最低优先级。

有两个不同的名称空间，一个用于[类型](03-types.html#types)，
一个用于[术语](06-expressions.html#expressions)。同名可以指定类型和术语，具体取决于使用名称的上下文。

绑定具有一个 *范围*，在该范围内，可以使用简单名称访问由单个名称定义的实体。范围是嵌套的。某些内部作用域中的绑定会 *影响* 同一作用域中较低优先级的绑定以及外部作用域中相同或较低优先级的绑定。

请注意，影响只是部分顺序。在下面的示例中，`x`的绑定都不会对另一个有影响。因此，在块的最后一行中对`x`的引用是不确定的。

```scala
val x = 1
locally {
  import p.X.x
  x
}
```

对非限定（类型或术语）标识符 $x$ 的引用受唯一绑定的限制，which

- 在与标识符相同的命名空间中定义名称为 $x$ 的实体
- 隐藏在该命名空间中定义名称为 $x$ 的实体的所有其他绑定。

如果不存在此类绑定，则为错误。如果 $x$ 由import子句绑定，那么简单名称 $x$ 将等同于import子句映射到 $x$ 的限定名称。如果 $x$ 由定义或声明绑定，则 $x$ 指的是该绑定引入的实体。在这种情况下， $x$ 的类型是引用实体的类型。

对限定（类型或术语）标识符 $e.x$ 的引用是指 $e$ 类型的成员，其名称 $x$ 与标识符在同一名称空间中。如果 $T$ 不是[值类型](03-types.html#value-types)，则会出错。$e.x$ 的类型是 $T$ 中引用实体的成员类型。

绑定优先级意味着,源文件被捆绑的方式会影响名称解析。尤其是，导入的名称比在其他文件中定义的名称具有更高的优先级，这些名称可能在其他情况下可见，因为它们是在当前包或封闭包中定义的。

请注意，包定义被视为最低优先级，因为包是开放的，可以跨任意编译单元定义。

```scala
package util {
  import scala.util
  class Random
  object Test extends App {
    println(new util.Random)  // scala.util.Random
  }
}
```

编译器在每个源文件的前导码中提供导入。 此前言在概念上具有以下形式，其中大括号表示嵌套范围：

```scala
import java.lang._
{
  import scala._
  {
    import Predef._
    { /* source */ }
  }
}
```

这些导入被视为最低优先级，因此它们始终被用户代码遮蔽，用户代码可能包含竞争导入和定义。它们还会增加嵌套深度，如图所示，以便后面的导入会影响先前的嵌套深度。

为了方便起见，允许将类型标识符的多个绑定到同一基础类型。当import子句引入具有相同绑定优先级的成员类型别名的绑定时，这是可能的，通常通过通配符导入。这允许在不引入歧义的情况下导入冗余类型别名。

```scala
object X { type T = annotation.tailrec }
object Y { type T = annotation.tailrec }
object Z {
  import X._, Y._, annotation.{tailrec => T}  // OK, all T mean tailrec
  @T def f: Int = { f ; 42 }                  // error, f is not tail recursive
}
```

同样，允许使用由package语句引入的名称的导入别名，即使名称严格不明确：

```scala
// c.scala
package p { class C }

// xy.scala
import p._
package p { class X extends C }
package q { class Y extends C }
```
在`X`的定义中对`C`的引用是严格不明确的，因为`C`是通过不同文件中的package子句可用的，并且不能隐藏导入的名称。但由于引用是相同的，因此定义被视为影响了导入。

###### 例子

假设在单独的编译单元中包`p`和`q`中名为`X`的对象的以下两个定义。

```scala
package p {
  object X { val x = 1; val y = 2 }
}

package q {
  object X { val x = true; val y = false }
}
```

以下程序说明了它们之间的不同类型的绑定和优先级。

```scala
package p {                   // `X' bound by package clause
import Console._              // `println' bound by wildcard import
object Y {
  println(s"L4: \$X")          // `X' refers to `p.X' here
  locally {
    import q._                // `X' bound by wildcard import
    println(s"L7: \$X")        // `X' refers to `q.X' here
    import X._                // `x' and `y' bound by wildcard import
    println(s"L9: \$x")        // `x' refers to `q.X.x' here
    locally {
      val x = 3               // `x' bound by local definition
      println(s"L12: \$x")     // `x' refers to constant `3' here
      locally {
        import q.X._          // `x' and `y' bound by wildcard import
//      println(s"L15: \$x")   // reference to `x' is ambiguous here
        import X.y            // `y' bound by explicit import
        println(s"L17: \$y")   // `y' refers to `q.X.y' here
        locally {
          val x = "abc"       // `x' bound by local definition
          import p.X._        // `x' and `y' bound by wildcard import
//        println(s"L21: \$y") // reference to `y' is ambiguous here
          println(s"L22: \$x") // `x' refers to string "abc" here
}}}}}}
```
