---
title: Implicits
layout: default
chapter: 7
---

# Implicits

## 隐式修饰符

```ebnf
LocalModifier  ::= ‘implicit’
ParamClauses   ::= {ParamClause} [nl] ‘(’ ‘implicit’ Params ‘)’
```

使用`implicit`修饰符标记的模板成员和参数可以传递给[隐式参数](#隐式参数)，并且可以在隐式转换中使用，这种情况被称为[视图](#视图)。`implicit`不能用于所有的类型成员和[顶级对象](09-top-level-definitions.html#packagings).。

###### Monoid事例
下面的代码定义了一个monoids的抽象类和两个具体的实现，`StringMonoid`和`IntMonoid`。这两个实现通常标记为隐式的：

```scala
abstract class Monoid[A] extends SemiGroup[A] {
  def unit: A
  def add(x: A, y: A): A
}
object Monoids {
  implicit object stringMonoid extends Monoid[String] {
    def add(x: String, y: String): String = x.concat(y)
    def unit: String = ""
  }
  implicit object intMonoid extends Monoid[Int] {
    def add(x: Int, y: Int): Int = x + y
    def unit: Int = 0
  }
}
```

## 隐含参数

_隐式参数列表_`(implicit $p_1$,$\ldots$,$p_n$)` 将参数$p_1 , \ldots , p_n$标记为隐式的。方法或构造函数只能有一个隐式参数列表，且必须是一个给定的参数列表的最后一个。

具有隐式参数的方法可以像普通方法一样应用于参数。在这种情况下，`implicit`标签无效。但是，如果此方法没有隐式参数的参量，则会自动提供此类参数。

能够传递给类型$T$的隐式参数的实际参数分为两类。第一类，所有标识符$x$都符合在没有前缀的方法调用中访问，并且该方法表示[隐式定义](#隐式修饰符)或隐式参数。因此，符合条件的标识符可以是本地名称，也可以是封闭模版的一个成员，也可以是通过[import子句](04-basic-declarations-and-definitions.html#import-clauses)使其不用前缀也可以访问。如果此规则下没有符合条件的标识符，那么第二个符合条件的也就是隐式参数类型$T$的隐含作用域中的对象的`implicit`成员。

$T$类型的_隐式作用域_由隐含参数的类型相关联的类的所有[伴随模块](05-classes-and-objects.html#object-definitions)构成。类$C$与类型$T$相 _关联_ 的条件条件是$C$是$T$的某个部分[基类](05-classes-and-objects.html#class-linearization)。

类型$T$的部件是：

- 如果$T$是复合类型`$T_1$ with $\ldots$ with $T_n$`，则是$T_1 , \ldots , T_n$的部件以及$T$自身的合集。
- 如果$T$是参数话类型`$S$[$T_1 , \ldots , T_n$]`，则是$S$和 $T_1 , \ldots , T_n$的部件的合集。
- 如果类型$T$是单例类型 `$p$.type`,则是$p$类型的部分。
- 如果$T$是一个类型预测 `$S$#$U$`，则是$S$的部分以及$T$本身。
- 如果$T$是一个类型别名，则是他的扩展部分。
- 如果$T$是抽象类型，则是其上限部分。
- 如果$T$表示隐式转换具参数类型为$T_1 , \ldots , T_n$和结果类型$U$的方法的类型，则是 $T_1 , \ldots , T_n$和$U$部分的合集。
- 量化(存在或通用)和注释类型的部分被定义为基础类型的部分(例如，`T forSome { ... }`的部分是`T`部分。)
- 在其他所有情况下，就是$T$本身。

需要注意的是包在内部表示为包含包成员的伴随模块的类。因此，包在对象中定义的implicits是包含前缀的类型的隐式范围的一部分。

如果有几个符合条件的参数与隐式参数的类型匹配，则使用静态[重载解析](06-expressions.html#overloading-resolution)的规则选择最具体的一个。如果参数具有默认参数且未找到隐式参数，则使用默认参数。

###### 例

假设来自[`Monoid`示例](Monoid事例)的类，这里是一个使用monoid的`add`和`unit`操作计算元素列表总和的方法，

```scala
def sum[A](xs: List[A])(implicit m: Monoid[A]): A =
  if (xs.isEmpty) m.unit
  else m.add(xs.head, sum(xs.tail))
```

有问题的monoid被标记为隐式参数，因此可以根据列表的类型来推断。例如，考虑调用`sum(List(1,2,3))`在上下文中`stringMonoid`和`IntMonoid`都是可见的。我们知道`sum`的形式类型参数`a`需要初始化为`Int`。与隐式形式参数类型`Monoid[Int]`的唯一匹配参数是`intMmonoid`,因此该对象会被作为隐藏参数来传递。

这也同样说明了隐式参数在所有类型参数[推断](06-expressions.html#local-type-inference)之后才开始推断。

隐式方法本身有隐式参数。一个例子是来自模块`scala.List`的以下方法，他将列表注入到`scala.Ordered`类中，前提是列表的元素类型也可以转换为此类型。

```scala
implicit def list2ordered[A](x: List[A])
  (implicit elem2ordered: A => Ordered[A]): Ordered[List[A]] =
  ...
```

另外假设一种方法

```scala
implicit def int2ordered(x: Int): Ordered[Int]
```

将整数注入`Ordered`类。我们可以在有序列表上定义`sort`  方法。

```scala
def sort[A](xs: List[A])(implicit a2ordered: A => Ordered[A]) = ...
```

我们可以将`sort`应用于整数列表`yss: List[List[Int]]`，如下所示

```scala
sort(yss)
```

上面的调用将通过传递两个嵌套的隐式参数来完成：

```scala
sort(yss)((xs: List[Int]) => list2ordered[Int](xs)(int2ordered))
```

将隐式参量传递给隐式参量可能会导致死循环。例如，可以尝试定义如下方法，该方法将每种类型注入`Ordered`类：
```scala
implicit def magic[A](x: A)(implicit a2ordered: A => Ordered[A]): Ordered[A] =
  a2ordered(x)
```

现在，如果有人视图将`sort`应用于没有另外注入到`Ordered`类中的参量`arg`，则将得到无限扩展：

```scala
sort(arg)(x => magic(x)(x => magic(x)(x => ... )))
```

这种无限扩展应该作为错误进行检测和报告，但是为了支持递归值的隐式构造，我们允许隐式参数按名称进行标记。在调用站点中，如果他们出现在隐式的by-name参数中，则允许递归使用隐式值。

考虑如下定义:

```scala
trait Foo {
  def next: Foo
}

object Foo {
  implicit def foo(implicit rec: Foo): Foo =
    new Foo { def next = rec }
}

val foo = implicitly[Foo]
assert(foo eq foo.next)
```

与上面`magic`的情况一样，由于方法`foo`的递归隐式参数`rec`,在这种情况下有所不同.如果我们将隐式参数标记为按名称:

```scala
trait Foo {
  def next: Foo
}

object Foo {
  implicit def foo(implicit rec: => Foo): Foo =
    new Foo { def next = rec }
}

val foo = implicitly[Foo]
assert(foo eq foo.next)
```

该示例编译成功。

编译时，这种类型的按名称递归隐式参数被提取出来，作为调用站点上本地合成对象的val成员，如下所示


```scala
val foo: Foo = scala.Predef.implicitly[Foo](
  {
    object LazyDefns$1 {
      val rec$1: Foo = Foo.foo(rec$1)
                       //      ^^^^^
                       // recursive knot tied here
    }
    LazyDefns$1.rec$1
  }
)
assert(foo eq foo.next)
```
注意，`rec$1`的递归使用发生在`foo`的by-name参数中，因此被推迟。脱糖匹配程序将明确地构造这样的递归值。脱糖匹配程序将明确的构造这样的递归值。

为了避免这一类无限扩展，例如上面的`magic`示例，编译器会为当前正在搜索隐式参数创建一个"开放隐式类型"的堆栈。每当搜索道$T$类型的隐式参数时，$T$就会被添加到与产生他的隐式定义配对的堆栈中，以及是否需要慢住符名称的隐式参数。不管搜索成功还是失败，都会从堆栈中删除该类型。每次要将类型添加到堆栈时，都会根据由相同隐式定义生成的现有条目进行检查。

+ 如果它等同于某个已经在堆栈上的类型，并且该条目与堆栈顶部之间存在一个命名参数。在这种情况下，对该类型的搜索会立即成功，并且隐式参数将被编译为对found参数的递归引用。如果尚未添加该参数，则将其添加为合成隐式字典中的条目。
+ 或者，如果该类型的核心_主导_类型已经在堆栈上的_核心_，那么隐含扩展被称为_分歧_并且对该类型的搜索立即失败。
+ 否则它被添加到与产生它的隐式定义配对的堆栈中。 隐式解析继续使用该定义的隐式参数（如果有的话）。

在这$T$的_核心类型_是$T$,其中扩展了别名，删除了顶级类型[注释](11-annotations.html#user-defined-annotations)和[优化](03-types.html#compound-types)，并且顶层存在性绑定变量的出现被其上限替换。

这里一个核心类型$T$_影响_ 到一个类型$U$的情况是$T$[等价](03-types.html#equivalence)于$U$,或$T$和$U$的顶级类型构造器有共有元素且$T$比$U$更复杂,$T$和$U$的覆盖范围相等。

类型$T$的_顶级类型构造器_集合$\mathit{ttcs}(T)$与类型的形式有关:

- 对于类型指示器， $\mathit{ttcs}(p.c) ~=~ \{c\}$
- 对于参数话类型,  $\mathit{ttcs}(p.c[\mathit{targs}]) ~=~ \{c\}$;
- For a singleton type,  $\mathit{ttcs}(p.type) ~=~ \mathit{ttcs}(T)$, provided $p$ has type $T$;
- 对于复合类型, `$\mathit{ttcs}(T_1$ with $\ldots$ with $T_n)$` $~=~ \mathit{ttcs}(T_1) \cup \ldots \cup \mathit{ttcs}(T_n)$.

核心类型的_复杂度_$\operatorname{complexity}(T)$是一个整数，它还取决于类型的形式：

- 对于类型指示器, $\operatorname{complexity}(p.c) ~=~ 1 + \operatorname{complexity}(p)$
- 对于参数话类型, $\operatorname{complexity}(p.c[\mathit{targs}]) ~=~ 1 + \Sigma \operatorname{complexity}(\mathit{targs})$
- 对于表示包$P$的单例类型， $\operatorname{complexity}(p.type) ~=~ 0$
- 对于任何其他的单例类型，$\operatorname{complexity}(p.type) ~=~ 1 + \operatorname{complexity}(T)$, provided $p$ has type $T$;
- 对于复合类型, `$\operatorname{complexity}(T_1$ with $\ldots$ with $T_n)$` $= \Sigma\operatorname{complexity}(T_i)$

$T$类型的_覆盖集_$\mathit{cs}(T)$是类型中提到的类型指示符集合。例如，给定以下内容：

```scala
type A = List[(Int, Int)]
type B = List[(Int, (Int, Int))]
type C = List[(Int, String)]
```

相应的覆盖集是：

- $\mathit{cs}(A)$: List, Tuple2, Int
- $\mathit{cs}(B)$: List, Tuple2, Int
- $\mathit{cs}(C)$: List, Tuple2, Int, String

###### 例
对于某些类型为`List[List[List[Int]]]`的列表，`xs`，`sort(xs)`类型的隐含参量类型搜索序列是：

```scala
List[List[Int]] => Ordered[List[List[Int]]],
List[Int] => Ordered[List[Int]],
Int => Ordered[Int]
```

所有类型共享公共类型构造函数`scala.Function1`，但每种新类型的复杂性低于先前类型的复杂性。这就是代码的类型检查方式。

###### 例
设`ys`是某些不能转变为`Ordered`类型的list，例如

```scala
val ys = List(new IllegalArgumentException, new ClassCastException, new Error)
```

假设上面`magic`的定义在范围内。则隐式参数类型搜索的序列是

```scala
Throwable => Ordered[Throwable],
Throwable => Ordered[Throwable],
...
```

由于序列中的第二种类型等于第一种类型，编译器将参数一个发散隐含扩展的错误。

## 视图

隐式参数和方法还可以定义隐式转换，称作视图。从$S$类型道$T$类型的_视图_是一个隐式值定义，该隐式值具由一个函数类型为`$S$=>$T$`或`(=>$S$)=>$T$`的隐含值或一个可以转变为该类型的值定义。


视图适用于以下三种情况:

1.  如果表达式$e$的类型为$T$，并且$T$不符合表达式的预期类型$\mathit{pt}$。在这种情况下将会搜索一个隐含的$v$,$v$可以应用道$e$且结果类型与$\mathit{pt}$一致。搜索继续进行，如隐式参数的情况，其中隐式范围是`$T$ => $\mathit{pt}$`。如果找到这样的视图，表达式$e$将转换为`$v$($e$)`。
1.  选择$e.m$中,$e$的类型为$T$,如果选择器$m$并不表示$T$的成员。在这种情况下，搜索视图$v$,该视图适用于$e$，其结果包含名为$m$的成员。搜索按照隐式参数的情况进行，其中隐式范围是$T$中的某一个。如果找到这样的视图。则选择器$e.m$将转换为`$v$($e$).$m$`。
1.  在选择 $e.m(\mathit{args})$和$e$类型为$T$的情况下，如果选择器$m$表示$T$的某些成员，但这些成员都不适用于参数$\mathit{args}$。在这种情况下，搜索的视图$v$适用于$e$,其结果包含$m$的方法，适用于$\mathit{args}$。搜索按照隐式参数的情况进行，其中隐式范围是$T$中的一个。如果找到这样的视图，则选择器$e.m$将转换为`$v$($e$).$m(\mathit{args})$`。

隐式视图(如果找到)可以接受其参数$e$作为按值调用或按名称调用参数。但是，按值调用的含义优先于按名称调用的含义。

对于隐式参数，如果存在多个可能的候选(按值调用或按名称调用类别)，则应用重载解析，

###### 有序示例

类`scala.Ordered[A]`包含一个方法

```scala
  def <= [B >: A](that: B)(implicit b2ordered: B => Ordered[B]): Boolean
```

假设两个类表类型为`List[Int]`的`xs` 和 `ys`，并假设[此](#隐含参数)处定义的`list2ordered`和`int2ordered`方法在作用域内。那么操作

```scala
  xs <= ys
```

是合法的，并扩展为：

```scala
  list2ordered(xs)(int2ordered).<=
    (ys)
    (xs => list2ordered(xs)(int2ordered))
```

`list2ordered`的第一个应用程序将将列表`xs`转变为类`Ordered`的一个实例，第二个则是传递给`<=`方法的隐含参数的一部分。

## 上下文边界和视图边界

```ebnf
  TypeParam ::= (id | ‘_’) [TypeParamClause] [‘>:’ Type] [‘<:’ Type]
                {‘<%’ Type} {‘:’ Type}
```

方法或非特征的类型参数$A$可能有一个或多个视图边界`$A$ <% $T$`。在这种情况下，类型参数可以实例化为任何类型$S$，可以通过视图应用于绑定的$T$来转换。

方法或非特征类的类型参数$A$也可能有一个或多个上下文边界`$A$ : $T$`。在这种情况下，类型参数可以实例化为任何类型$S$,只要实例化点上有_证据_表明$S$满足绑定的$T$。这些由$T[S]$类型的隐含值组成。

包含具有视图或上下文边界的类型参数的方法或类被视为等效于具有隐式参数的方法。首先考虑具有视图和/或上下文边界的单个参数的情况，例如：

```scala
def $f$[$A$ <% $T_1$ ... <% $T_m$ : $U_1$ : $U_n$]($\mathit{ps}$): $R$ = ...
```

然后将上面的方法定义扩展为

```scala
def $f$[$A$]($\mathit{ps}$)(implicit $v_1$: $A$ => $T_1$, ..., $v_m$: $A$ => $T_m$,
                       $w_1$: $U_1$[$A$], ..., $w_n$: $U_n$[$A$]): $R$ = ...
```

其中$v_i$ 和 $w_j$是新引入的隐式参数的新名称。这些参数称为_证据参数_。

如果类或方法具有多个视图或上下文绑定类型参数，则每个这样的类型参数按他们出现的顺序扩展为证据参数，并且所有的得到的证据参数在一个隐式参数部分中连接。由于traits不采用构造函数参数，因此该转换对他们不起作用。因此，特征中的类型参数可能不是视图或上下文限制的。

证据参数前置于现有隐式参数部分（如果存在）。

举个例子:

```scala
def foo[A: M](implicit b: B): C
// 扩展为
// def foo[A](implicit evidence$1: M[A], b: B): C
```

###### 例子

可以更简洁的声明[`Ordered`示例](#ordered实例)中的`<=`方法，如下所示

```scala
def <= [B >: A <% Ordered[B]](that: B): Boolean
```

## 清单

清单是类型描述符，可以有Scala编译器自动生成，作为隐式参数的参数。Scala标准库包含四个清单类型的层次结构`OptManifest`位于顶部。他们的签名遵循一下大纲。

```scala
trait OptManifest[+T]
object NoManifest extends OptManifest[Nothing]
trait ClassManifest[T] extends OptManifest[T]
trait Manifest[T] extends ClassManifest[T]
```

如果方法或构造函数的隐式参数属于类`OptManifest[T]`的子类型$M[T]$，则更具一下规则 _确定清单$M[S]$_。

首先，如果已经存在与$M[T]$匹配的隐式参数，则选择此参数。

否则，如果$M$是特征`Manifest`，$\mathit{Mobj}$是`scala.reflect.Manifest`的伴随对象，否则是 `scala.reflect.ClassManifest`的伴随对象。如果$M$是特征`Manifest`，那么$M'$成为特征`Manifest`,否则就是特征`OptManifest`。然后适用于以下规则

1.  如果$T$是一个值类或者是`Any`, `AnyVal`, `Object`,`Null`, 或 `Nothing`其中之一，则通过选择`Manifest`模块中存在的相应清单值`Manifest.$T$`。
1.  如果$T$是`Array[$S$]`的实例，则使用调用`$\mathit{Mobj}$.arrayType[S](m)`生成清单，其中$m$是为$M[S]$确定的清单。
1.  如果$T$是其他类类型$S$#$C[U_1, \ldots, U_n]$，其中前缀类型$S$不能从$C$中静态确认，则使用调用`$\mathit{Mobj}$.classType[T]($m_0$, classOf[T], $ms$)`生成清单，其中$m_0$是为$M'[S]$确认的清单，和$ms$是为 $M'[U_1], \ldots, M'[U_n]$确定的清单。
1.  如果$T$是类型参数$U_1 , \ldots , U_n$的其他类类型，则使用调用 `$\mathit{Mobj}$.classType[T](classOf[T], $ms$)`生成清单，其中$ms$是为$M'[U_1] , \ldots , M'[U_n]$确认的清单。
1.  如果$T$是单例类型`$p$.type`，则使用调用`$\mathit{Mobj}$.singleType[T]($p$)`生成清单。
1.  如果$T$是精炼类型$T' \{ R \}$，则为$T'$生成清单，（也就是说，改进从未反映在清单中）。
1.  如果$T$是交叉类型`$T_1$ with $, \ldots ,$ with $T_n$`，其中$n>1$,结果取决于是否要确定完整清单。如果$M$是特征`Manifest`,则使用调用`Manifest.intersectionType[T]($ms$)`生成清单，其中$ms$是为$M[T_1] , \ldots , M[T_n]$确定的清单。否则，如果$m$是特征 `ClassManifest`，则为$T_1 , \ldots , T_n$类型的[交集支配](03-types.html#type-erasure)生成清单。
1.  如果$T$是其他类型，那么如果$M$是特征 `OptManifest`，则从指示符 `scala.reflect.NoManifest`生成清单。如果$M$是与`OptManifest`不同的类型，则会参生静态错误。
