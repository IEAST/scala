---
title: Basic Declarations & Definitions
layout: default
chapter: 4
---

# 基本声明和定义

```ebnf
Dcl         ::=  ‘val’ ValDcl
              |  ‘var’ VarDcl
              |  ‘def’ FunDcl
              |  ‘type’ {nl} TypeDcl
PatVarDef   ::=  ‘val’ PatDef
              |  ‘var’ VarDef
Def         ::=  PatVarDef
              |  ‘def’ FunDef
              |  ‘type’ {nl} TypeDef
              |  TmplDef
```

*声明*引入了名称并为其指定了类型。它可以构成[类定义](05-classes-and-objects.html#templates)的一部分或[复合类型](03-types.html#compound-types)中的细化。

*定义*引入了表示术语或类型的名称。它可以构成对象或类定义的一部分，也可以是局部的数据块。 声明和定义都产生*数据绑定*，它将类型名称与类型定义或范围相关联，并将项目名称名称与类型相关联。

由声明或定义引入的名称范围是包含数据绑定的整个语句序列。但是，数据块中的向前引用存在限制:在语句序列$s_1 \ldots s_n$中构成数据块，如果$s_i$中的简名是指由$s_j$定义的实体，其中$j \geq i$，那么对于$s_i$ 和 $s_j$之间的所有的$s_k$：

- $s_k$ 不能是变量定义
- 如果$s_k$ 是值定义, 那么它必须是有延迟的.

<!--
Every basic definition may introduce several defined names, separated
by commas. These are expanded according to the following scheme:
\bda{lcl}
\VAL;x, y: T = e && \VAL; x: T = e \\
                 && \VAL; y: T = x \\[0.5em]

\LET;x, y: T = e && \LET; x: T = e \\
                 && \VAL; y: T = x \\[0.5em]

\DEF;x, y (ps): T = e &\tab\mbox{expands to}\tab& \DEF; x(ps): T = e \\
                      && \DEF; y(ps): T = x(ps)\\[0.5em]

\VAR;x, y: T := e && \VAR;x: T := e\\
                  && \VAR;y: T := x\\[0.5em]

\TYPE;t,u = T && \TYPE; t = T\\
              && \TYPE; u = t\\[0.5em]
\eda

All definitions have a ``repeated form`` where the initial
definition keyword is followed by several constituent definitions
which are separated by commas.  A repeated definition is
always interpreted as a sequence formed from the
constituent definitions. E.g. the function definition
`def f(x) = x, g(y) = y` expands to
`def f(x) = x; def g(y) = y` and
the type definition
`type T, U <: B` expands to
`type T; type U <: B`.
}
\comment{
If an element in such a sequence introduces only the defined name,
possibly with some type or value parameters, but leaves out any
additional parts in the definition, then those parts are implicitly
copied from the next subsequent sequence element which consists of
more than just a defined name and parameters. Examples:

- []
The variable declaration `var x, y: Int`
expands to `var x: Int; var y: Int`.
- []
The value definition `val x, y: Int = 1`
expands to `val x: Int = 1; val y: Int = 1`.
- []
The class definition `case class X(), Y(n: Int) extends Z` expands to
`case class X extends Z; case class Y(n: Int) extends Z`.
- The object definition `case object Red, Green, Blue extends Color`~
expands to
```scala
case object Red extends Color
case object Green extends Color
case object Blue extends Color
```
-->

## 值的声明和定义

```ebnf
Dcl          ::=  ‘val’ ValDcl
ValDcl       ::=  ids ‘:’ Type
PatVarDef    ::=  ‘val’ PatDef
PatDef       ::=  Pattern2 {‘,’ Pattern2} [‘:’ Type] ‘=’ Expr
ids          ::=  id {‘,’ id}
```

值声明 `val $x$: $T$` 引入 $x$ 作为 $T$类型的值的名称.

值定义`val $x$: $T$ = $e$`将$x$定义为由$e$赋值产生的值的名称。如果值定义不是递归的，则可以省略类型$T$，在这种情况下，就说$e$是[包类型](06-expressions.html#expression-typing).如果给出类型$T$,那么$e$应该符合它。

赋值定义意味着赋值给右侧$e$，除非他具有修饰符`lazy`.值定义的作用是将$x$绑定到$e$装换为$T$类型的值。`lazy`值定义在第一次访问值时赋值其右侧$e$.

*常量*的定义如下

```scala
final val x = e
```

其中`e`是[常量表达式](06-expressions.html#constant-expressions)。必须存在`final`修饰符，并且不能给出注释类型。对常量值`x`的引用本身被视为常量表达式，在生成的代码中，他们被定义在右侧的e替换。

值定义可以具有左侧的[样式](08-pattern-matching.html#patterns)，如果$p$不是简单的名称或后面跟冒号和类型的名称，而是某种样式，那么`val $p$ = $e$`的值定义将展开如下:

1.如果样式 $p$ 已绑定变量 $x_1 , \ldots , x_n$, 其中 $n > 1$:

```scala
val $\$ x$ = $e$ match {case $p$ => ($x_1 , \ldots , x_n$)}
val $x_1$ = $\$ x$._1
$\ldots$
val $x_n$ = $\$ x$._n
```

在这里，$\$ x$ 是一个新名称.

2. 如果 $p$ 有一个唯一的绑定变量 $x$:

```scala
val $x$ = $e$ match { case $p$ => $x$ }
```

3. 如果 $p$ 没有绑定变量:

```scala
$e$ match { case $p$ => ()}
```

###### 例

以下是值定义的例子

```scala
val pi = 3.1415
val pi: Double = 3.1415   // 相当于第一个定义
val Some(x) = f()         // 模式定义
val x :: xs = mylist      // 中缀模式定义
```

最后两个定义具有以下扩展
```scala
val x = f() match { case Some(x) => x }

val x$\$$ = mylist match { case x :: xs => (x, xs) }
val x = x$\$$._1
val xs = x$\$$._2
```

任何声明或者定义的值的名称可以不以 `_=`结尾.

值声明`val $x_1 , \ldots , x_n$: $T$`是值声明序列`val $x_1$: $T$; ...; val $x_n$: $T$`的简写。值定义`val $p_1 , \ldots , p_n$ = $e$` 是值定义`val $p_1$ = $e$; ...; val $p_n$ = $e$`序列的简写。值定义`val $p_1 , \ldots , p_n: T$ = $e$`是值定义`val $p_1: T$ = $e$; ...; val $p_n: T$ = $e$`的序列的简写。

## 变量的声明和定义

```ebnf
Dcl            ::=  ‘var’ VarDcl
PatVarDef      ::=  ‘var’ VarDef
VarDcl         ::=  ids ‘:’ Type
VarDef         ::=  PatDef
                 |  ids ‘:’ Type ‘=’ ‘_’
```
变量声明`var $x$: $T$`等同于*getter函数*$x$*和*setter函数*`$x$_=`的声明。

```scala
def $x$: $T$
def $x$_= ($y$: $T$): Unit
```
类的实现可以使用变量*定义*或通过定义相应的setter和getter方法来定义声明的变量。

变量定义`var $x$: $T$ = $e$`引入了一个类型位$T$的可变变量和由$e$给出的初始值。将定$e$的类型，在这种情况下类型$T$可以省略。如果给出$T$，那么$e$应该[符合它](06-expressions.html#expression-typing)。

或者，变量定义可以具有左侧的[样式](08-pattern-matching.html#patterns)。变量定义`var $p$ = $e$`,期中$p$是除简单名称之外的模式或名称后跟冒号和类型的模式，以与[值定义](#值声明和定义)`val $p$ = $e$`相同的方式进行扩展，除了$p$中的不受约束的名称呗引入为可变变量，而不是值。

任何声明或定义的变量的名称可以不以 `_=`结尾.

变量定义`var $x$: $T$ = _`只能作为模板的成员出现。它引入了一个觉定类型为$T$的可变字段和一个默认的初始值。默认值取决于类型$T$，如下所示：

| default  | type $T$                           |
|----------|------------------------------------|
|`0`       | `Int` 或其子范围类型之一              |
|`0L`      | `Long`                             |
|`0.0f`    | `Float`                            |
|`0.0d`    | `Double`                           |
|`false`   | `Boolean`                          |
|`()`      | `Unit`                             |
|`null`    |所有其他类型                          |

当他们作为模板的成员出现时，两种形式的变量定义还引入了一个getter函数$x$，它返回当前赋给变量的值。还有一个setter函数`$x$_=` ，它更改当前赋给变量的值。

###### 例

以下例子展示了如何在Scala中模拟*属性*。它定义了一个时间类型`TimeOfDayVar`，其中小时，分钟和秒是可以更新的整数字段。其实现包涵的测试只允许将合法值分配给这些字段。另一方面，用户代码就能够像普通变量一样访问这些字段。

```scala
class TimeOfDayVar {
  private var h: Int = 0
  private var m: Int = 0
  private var s: Int = 0

  def hours              =  h
  def hours_= (h: Int)   =  if (0 <= h && h < 24) this.h = h
                            else throw new DateError()

  def minutes            =  m
  def minutes_= (m: Int) =  if (0 <= m && m < 60) this.m = m
                            else throw new DateError()

  def seconds            =  s
  def seconds_= (s: Int) =  if (0 <= s && s < 60) this.s = s
                            else throw new DateError()
}
val d = new TimeOfDayVar
d.hours = 8; d.minutes = 30; d.seconds = 0
d.hours = 25                  // throws a DateError exception
```

变量声明`var $x_1 , \ldots , x_n$: $T$`是变量声明`var $x_1$: $T$; ...; var $x_n$: $T$`序列的简写。  
变量定义`var $x_1 , \ldots , x_n$ = $e$` 是变量定义`var $x_1$ = $e$; ...; var $x_n$ = $e$`序列的简写。  
变量定义`var $x_1 , \ldots , x_n: T$ = $e$`是变量定义`var $x_1: T$ = $e$; ...; var $x_n: T$ = $e$`序列的简写。

## 键入声明和类型别名

<!-- TODO: Higher-kinded tdecls should have a separate section -->

```ebnf
Dcl        ::=  ‘type’ {nl} TypeDcl
TypeDcl    ::=  id [TypeParamClause] [‘>:’ Type] [‘<:’ Type]
Def        ::=  ‘type’ {nl} TypeDef
TypeDef    ::=  id [TypeParamClause] ‘=’ Type
```
*类型声明*`type $t$[$\mathit{tps}\,$] >: $L$ <: $U$`
声明的$t$为抽象类型，下限类型为$L$，上限类型为$U$。入股省略类型参数字句`[$\mathit{tps}\,$]`,则$t$接受一阶类型，否则$t$代表一个构造函数类型，它接受类型参数子句所描述的类型参数。

如果一个类型声明显示为一个类型的成员声明，则该类型的实现可以使用任何类型$T$实现$t$，其中$L <: T <: U$。如果$L$不符合$U$，就会在编译时报错。可以省略任意一个或两个边界。如果缺少下限$L$，则假设底部类型为`scala.Nothing`。如果缺少上限$U$,则假定顶部类型为`scala.Any`。

类型构造函数声明对$t$可能存在的类型增加了额外的限制。除了边界$L$和$U$之外，类型参数自居可以强加更高阶级的边界和方差，有[类型构造函数的一致性](03-types.html#conformance)决定。

类型参数的范围扩展到边界`>: $L$ <: $U$`和类型参数子句$\mathit{tps}$本身。高阶类型参数自居(抽象类型构造函数$tc$)具有相同类型的作用域，仅限于类型参数$tc$的声明。

为了说明嵌套作用域，这些声明都是等效的，`type t[m[x] <: Bound[x], Bound[x]]`, `type t[m[x] <: Bound[x], Bound[y]]` 和 `type t[m[x] <: Bound[x], Bound[_]]`。因为 例如$m$的类型参数的范围仅限于$m$的声明。在所有的这些中，$t$是一个抽象类型成员，它抽象了两个类型构造函数:$m$代表一个类型构造函数，它接受一个类型参数，并且必须是$Bound$的子类型，$t$类型构造参数。$t$的有效用途是`t[MutableList, Iterable]`。

*类型别名*type $t$ = $T$`将$t$定义为$T$类型的别名。类型别名的左侧可以具有类型参数子句，例如`ype $t$[$\mathit{tps}\,$] = $T$`。类型参数的范围扩展到右侧$T$和类型参数子句$\mathit{tps}$本身。

[定义](#基本声明和定义)和[类型参数](#函数声明和定义)的范围规范了使用类型名称可以出现在其自己的右侧和边界。但是，如果类型别名以递归的方式应用定义的类型构造函数本身，则它是静态的，是错误的。也就是说,类型别名`type $t$[$\mathit{tps}\,$] = $T$`中的类型$T$可能不会直接或间接引用名称$t$。如果抽象类型直接或间接的具有其自己的上限或下限，则也是错误的。

###### 例

以下是合法的类型声明和定义：

```scala
type IntList = List[Integer]
type T <: Comparable[T]
type Two[A] = Tuple2[A, A]
type MyCollection[+X] <: Iterable[X]
```

以下是非法的声明和定义:

```scala
type Abs = Comparable[Abs]      // recursive type alias(递归类型别名)

type S <: T                     // S, T are bounded by themselves.（ST由他们自身限制)
type T <: S

type T >: Comparable[T.That]    // Cannot select from T.(无法从T中选择)
                                // T is a type, not a value(T是一中类型，而不是一中价值)
type MyCollection <: Iterable   // Type constructor members must explicitly
                                // state their type parameters.(类型构造函数成员必须显式声明其类型参数。)
```

如果类型别名`type $t$[$\mathit{tps}\,$] = $S$`引用类型$S$，名称$t$也可以用作类型对象的构造函数$S$.

###### 例

假设我们将`Pair`作为参数类型`Tuple2`的别名，如下

```scala
type Pair[+A, +B] = Tuple2[A, B]
object Pair {
  def apply[A, B](x: A, y: B) = Tuple2(x, y)
  def unapply[A, B](x: Tuple2[A, B]): Option[Tuple2[A, B]] = Some(x)
}
```

因此，对于任何两种类型$S$和$T$，Pair[$S$, $T\,$]` 等价于`Tuple2[$S$, $T\,$]`。`Pair`
也可以用作构造函数而不是`Tuple2`,
就像:

```scala
val x: Pair[Int, String] = new Pair(1, "abc")
```

## 类型参数

```ebnf
TypeParamClause  ::= ‘[’ VariantTypeParam {‘,’ VariantTypeParam} ‘]’
VariantTypeParam ::= {Annotation} [‘+’ | ‘-’] TypeParam
TypeParam        ::= (id | ‘_’) [TypeParamClause] [‘>:’ Type] [‘<:’ Type] [‘:’ Type]
```

类型参数显示在类型定义，类定义和函数定义中。在本节中，我们只考虑具有下限`>: $L$`和上限`<: $U$` 。而对于上下文边界$U$和视图边界`<% $U$`的讨论被推迟到[这](07-implicits.html#context-bounds-and-view-bounds).

类型参数的一般形式是`$@a_1 \ldots @a_n$ $\pm$ $t$ >: $L$ <: $U$`。在这里面，$L$和$U$是限制参数的类型参数上限和下限。如果$L$不符合$U$，则会编译错误。$\pm$是一个*方差*，即`+`或`-`为一个可选前缀。一条或多条注释可以在类型参数之前。

<!--
The upper bound $U$ in a type parameter clauses may not be a final
class. The lower bound may not denote a value type.

TODO: Why
-->

<!--
TODO: this is a pretty awkward description of scoping and distinctness of binders
-->

在继承封闭类型参数子句中，所有类型参数的名称必须两两不相同。类型参数的范围在每种情况下都包括整个类型参数子句。因此，类型参数可能在同一子句中作为其自己的边界的一部分或其他类型参数的边界出现。但是，类型参数本身不能直接或间接限制。

类型构造函数参数将嵌套类型参数子句添加到类型参数。类型构造函数参数最常见的形式是`$@a_1\ldots@a_n$ $\pm$ $t[\mathit{tps}\,]$ >: $L$ <: $U$`。

上面的作用域限制适用于嵌套类型参数子句，这些子句声明高阶类型参数。高阶类型参数(类型为$t$的类型参数)仅在其紧邻的参数子句中可见(可能包括更深级别嵌套的子句)和$t$的边界。因此，他们的名称只能与其他可见参数的名称两两不同。由于高阶类型参数的名称通常不相关的，因此他们可以用`‘_’`表示，这是不可见的。

###### 例
下面是一些格式正确的类型参数子句：

```scala
[S, T]
[@specialized T, U]
[Ex <: Throwable]
[A <: Comparable[B], B <: A]
[A, B >: A, C >: A <: B]
[M[X], N[X]]
[M[_], N[_]] // equivalent to previous clause(相当于前一条款)
[M[X <: Bound[X]], Bound[_]]
[M[+X] <: Iterable[X]]
```

以下类型参数子句是非法的:

```scala
[A >: A]                  // illegal, `A' has itself as bound(非法，`A`本身就是受约束的)
[A <: B, B <: C, C <: A]  // illegal, `A' has itself as bound(非法，`A`本身就是受约束的)
[A, B, C >: A <: B]       // illegal lower bound `A' of `C' does
                          // not conform to upper bound `B'.
```

## 方法注释

方法注释指示参数化类型的实例随着[子类型](03-types.html#conformance)方面的变化。`+`方差表示协变依赖性，`-`方差表示逆变依赖性，确实方差表示不变依赖性、

方差注释约束了戴注释的类型变量在绑定类型参数的类型或类型中出现的方法。在类型定义`type $T$[$\mathit{tps}\,$] = $S$`或`type $T$[$\mathit{tps}\,$] >: $L$ <: $U$`中，标记为‘+’的 类型参数必须仅出现在协变位置，而标记为 ‘-’ 的类型参数必须只出现在逆变位置。类似的，对于类定义`class $C$[$\mathit{tps}\,$]($\mathit{ps}\,$) extends $T$ { $x$: $S$ => ...}`，标记为‘+’的类型参数必须仅出现在自身类型$S$和模板$T$中的协变位置，而标记为'-'的类型参数必须仅出现在逆变位置。

类型或模板中类型参数的方差位置定义如下。让协方差的与逆变性对立，让它本身和不变性对立。 类型或模板的顶级始终处于协变位置。方差位置在以下构造函数发生改变。

- 方法参数的方差位置与封闭参数子句的方差位置相反。
- 类型参数的方差位置与封闭类型参数子句的方差位置相反。
- 类型声明或类型参数的下限的方差位置与类型声明或参数的方差位置相反。
- 可变变量的类型始终处于不变位置。
- 类型别名的右侧始终处于不变位置。
- 类型选择 `$S$#$T$` 的前缀$S$始终处于不变的位置。
- 对于`$S$[$\ldots T \ldots$ ]`类型的类型参数$T$:如果相应的类型参数斯不变的，则$T$处于不
  变位置。如果相应的类型参数是逆变的，这$T$的方差位置与封闭类型`$S$[$\ldots T \ldots$ ]`的方差位置相反。


<!-- TODO: handle type aliases -->

不检查
[对象私有或受对象保护的值，类型，变量或方法](05-classes-and-objects.html#modifiers) of 中的类型参数的引用的方差位置。在这些成员中，类型参数可以出现在任何地方，而不会限制其合法的方差注释。

###### 例
以下方差注释是合法的。

```scala
abstract class P[+A, +B] {
  def fst: A; def snd: B
}
```
使用此方差注释，可以相当于对其参数同义地键入$P$子类型的实例，例如：

```scala
P[IOException, String] <: P[Throwable, AnyRef]
```

如果$P$的成员是可变变量，则相同的方差注释变为非法

```scala
abstract class Q[+A, +B](x: A, y: B) {
  var fst: A = x           // ****错误：非法差异
  var snd: B = y           // `A', `B' occur in invariant position.(“A”，“B”出现在不变的位置。)
}
```

如果可变变量是对象私有的，则类定义再次变为合法：

```scala
abstract class R[+A, +B](x: A, y: B) {
  private[this] var fst: A = x        // OK
  private[this] var snd: B = y        // OK
}
```

###### 例

以下方差注释是非法的，因为$a$出现在`append``参数的逆变位置:

```scala
abstract class Sequence[+A] {
  def append(x: Sequence[A]): Sequence[A]
                  // **** error: illegal variance:
                  // `A' occurs in contravariant position.
                  //（错误：非法方差：“A”出现在逆变位置。）
}
```

通过下限概括`append`的类型可以避免这个问题：

```scala
abstract class Sequence[+A] {
  def append[B >: A](x: Sequence[B]): Sequence[B]
}
```

###### 例

```scala
abstract class OutputChannel[-A] {
  def write(x: A): Unit
}
```

使用该注释，我们得到`OutputChannel[AnyRef]`符合`OutputChannel[String]`。也就是说，可以在骑上写入任何对象的通道可以替代只能写入字符串的通道。

## 函数声明和定义

```ebnf
Dcl                ::=  ‘def’ FunDcl
FunDcl             ::=  FunSig ‘:’ Type
Def                ::=  ‘def’ FunDef
FunDef             ::=  FunSig [‘:’ Type] ‘=’ Expr
FunSig             ::=  id [FunTypeParamClause] ParamClauses
FunTypeParamClause ::=  ‘[’ TypeParam {‘,’ TypeParam} ‘]’
ParamClauses       ::=  {ParamClause} [[nl] ‘(’ ‘implicit’ Params ‘)’]
ParamClause        ::=  [nl] ‘(’ [Params] ‘)’
Params             ::=  Param {‘,’ Param}
Param              ::=  {Annotation} id [‘:’ ParamType] [‘=’ Expr]
ParamType          ::=  Type
                     |  ‘=>’ Type
                     |  Type ‘*’
```
*函式声明*的形式是`def $f\,\mathit{psig}$: $T$`，其中$f$是函数名称，$\mathit{psig}$是其参数签名，$T$是其结果类型。函数定义`def $f\,\mathit{psig}$: $T$ = $e$`还包括函数体$e$,即定义函数结果的表达式。参数签名由可选的类型参数子句`[$\mathit{tps}\,$]`组成，后面不跟或跟多个值参数子句`($\mathit{ps}_1$)$\ldots$($\mathit{ps}_n$)`。这样的声明或定义引入了一个带有(可能是多态的)方法类型的值，其参数类型和结果类型是给定的。  

如果给出函数体的类型，则期望函数体的[类型](06-expressions.html#expression-typing)符合函数的声明结果类型。如果函数定义不是递归的，则可以省略结果类型，在这种情况下，它是根据函数体的包类型确定的。

*类型参数子句*$\mathit{tps}$由一个或多个类型声明组成，它引入了类型参数，可能带有边界。类型参数的范围包括整个签名，包括任何类型参数边界以及函数体(如果存在)。

*值参数子句*$\mathit{ps}$由零个或多个心事参数绑定组成。例如`$x$: $T$` or `$x: T = e$`，它绑定值参数并将它们与他们的类型相关联。

### 默认参数

每个值参数声明可以选择性地定义默认参数。默认参数表达式$e$通过使用未定义类型替换$T$中函数的所有类型参数而获得的预提类型$T'$进行类型检查.

对于带有默认参数的每个参数 $p_{i,j}$，会生成一个名为`$f\$$default$\$$n`的方法，该方法计算默认参数表达式。这里面，$n$表示参数在方法生声明中的位置.这写方法由类型参数子句`[$\mathit{tps}\,$]`参数化，并且`$f\$$default$\$$n`方法之前所有值的参数子句`($\mathit{ps}_1$)$\ldots$($\mathit{ps}_{i-1}$)`对于用户程序来说都是不可访问的。

###### 例子
在方法中

```scala
def compare[T](a: T = 0)(b: T = a) = (a == b)
```
使用未定义的与其类型对缺省表达式`0`进行类型检查。应用`compare()`，将插入默认值`0`
，并将`T`实例化为`Int`。计算默认参数的方法具有以下形式:

```scala
def compare$\$$default$\$$1[T]: Int = 0
def compare$\$$default$\$$2[T](a: T): T = a
```

正式值参数名称$x$的范围包括所有后续参数子句，以及方法返回类型和函数题(如果给出).类型参数名称和值参数名称必须是两不相同的。

依赖于早期参数的默认值使用实际参数(如果提供)，而不是默认参数。

```scala
def f(a: Int = 0)(b: Int = a + 1) = b // OK
// def f(a: Int = 0, b: Int = a + 1)  // "error: not found: value a"
f(10)()                               // returns 11 (not 1)
```

如果隐式搜索未找到[隐式参数](07-implicits.html#implicit-parameters)，则可以使用提供的默认参数。

```scala
implicit val i: Int = 2
def f(implicit x: Int, s: String = "hi") = s * x
f                                     // "hihi"
```

### 按名称参数

```ebnf
ParamType          ::=  ‘=>’ Type
```

值参数的类型可以以`=>`为前缀例如`$x$: => $T$`.这样的参数类型是无参数方法`=> $T$`.这表示相应的参数不在函数应用程序点评估，而是在函数内的每次使用时进行评估。也就是说，使用*按名称调用*来评估参数。

对于带有`val` 或 `var`前缀的类的参数，不允许使用by-name修饰符，包括隐式生成`val`前缀的case类参数。

###### 例
声明

```scala
def whileLoop (cond: => Boolean) (stat: => Unit): Unit
```

表明使用call-by-name来评估`whileLoop`的两个参数。

### Repeated Parameters

```ebnf
ParamType          ::=  Type ‘*’
```

参数部分的最后一个值可以以`'*'`结尾，例如`(..., $x$:$T$*)`。方法中的这种重复参数的类型是序列类型`scala.Seq[$T$]`。具有重复参数`$T$*`的方法接受类型为$T$的可变数量的参数。也就是说，如果将具有`($p_1:T_1 , \ldots , p_n:T_n, p_s:S$*)$U$`的方法$m$应用与参数$(e_1 , \ldots , e_k)$ where $k \geq n$。则在改程序中采用具有类型$(p_1:T_1 , \ldots , p_n:T_n, p_s:S , \ldots , p_{s'}S)U$的$m$。$k - n$包涵类型$S$，除了$p_s$之外的任何参数名称都是新定义的。此规定的唯一例外是，如果最后一个参数为*序列参数*且通过 `_*`标注.如果将$m$应用于参数`($e_1 , \ldots , e_n, e'$: _*)`之上，那么该应用程序中$m$的类型将被视为`($p_1:T_1, \ldots , p_n:T_n,p_{s}:$scala.Seq[$S$])`.  
The last value parameter of a parameter section may be suffixed by
`'*'`, e.g. `(..., $x$:$T$*)`.  The type of such a
_repeated_ parameter inside the method is then the sequence type
`scala.Seq[$T$]`.  Methods with repeated parameters
`$T$*` take a variable number of arguments of type $T$.
That is, if a method $m$ with type
`($p_1:T_1 , \ldots , p_n:T_n, p_s:S$*)$U$` is applied to arguments
$(e_1 , \ldots , e_k)$ where $k \geq n$, then $m$ is taken in that application
to have type $(p_1:T_1 , \ldots , p_n:T_n, p_s:S , \ldots , p_{s'}S)U$, with
$k - n$ occurrences of type
$S$ where any parameter names beyond $p_s$ are fresh. The only exception to
this rule is if the last argument is
marked to be a _sequence argument_ via a `_*` type
annotation. If $m$ above is applied to arguments
`($e_1 , \ldots , e_n, e'$: _*)`, then the type of $m$ in
that application is taken to be
`($p_1:T_1, \ldots , p_n:T_n,p_{s}:$scala.Seq[$S$])`.


不允许在重复参数的参数部分定义任何默认参数。

###### 例
以下方法定义计算可变数量的整数参数的和。

```scala
def sum(args: Int*) = {
  var result = 0
  for (arg <- args) result += arg
  result
}
```
该程序按以下方法顺序生成“0”，“1”，“6”。

```scala
sum()
sum(1)
sum(1, 2, 3)
```

此外，假设定义：

```scala
val xs = List(1, 2, 3)
```

方法`sum`的以下应用是不正确的：

```scala
sum(xs)       // ***** 错误: 预期: Int, 找到: List[Int]
```

相比之下，以下程序是可行的，并再次产生结果6.

```scala
sum(xs: _*)
```

### 过程(程序)

```ebnf
FunDcl   ::=  FunSig
FunDef   ::=  FunSig [nl] ‘{’ Block ‘}’
```

程序存在特殊语法，即返回`Unit`值`()`的函数。*过程声明*是一个函数声明，其中省略了结果类型。然后结果类型隐式完成`Unit`类型。例如`def $f$($\mathit{ps}$)`相当于`def $f$($\mathit{ps}$): Unit`。

*过程声明*是一个函数定义，其中省略了一个结果类型和等号；他的定义表达式必须是一个块。例如：`def $f$($\mathit{ps}$) {$\mathit{stats}$}`相当于`def $f$($\mathit{ps}$): Unit = {$\mathit{stats}$}`.

###### 例

这是一个名为`write`的过程的声明和定义：

```scala
trait Writer {
  def write(str: String)
}
object Terminal extends Writer {
  def write(str: String) { System.out.println(str) }
}
```

以上代码展开是以下代码：

```scala
trait Writer {
  def write(str: String): Unit
}
object Terminal extends Writer {
  def write(str: String): Unit = { System.out.println(str) }
}
```

### 方法返回类型推断

在$C$基类中覆盖其他函数$m'$的类成员定义$m$可能会忽略返回类型，即他是递归的。在这种情况下，被覆盖$m'$的返回类型$R'$，被视为$C$的成员，被视为$m$的每个递归调用的$m$的返回类型。这样，可以确定$m$右侧的$R$类型，然后将其作为$m$的返回类型。注意，只能说$R$符合$R'$,$R$和$R'$可能不同。


###### 例
假设定义如下:

```scala
trait I {
  def factorial(x: Int): Int
}
class C extends I {
  def factorial(x: Int) = if (x == 0) 1 else x * factorial(x - 1)
}
```

在这里，可以省略'C'中的‘factorial’的结果类型，即该方法是递归的。


## 导入条款

```ebnf
Import          ::= ‘import’ ImportExpr {‘,’ ImportExpr}
ImportExpr      ::= StableId ‘.’ (id | ‘_’ | ImportSelectors)
ImportSelectors ::= ‘{’ {ImportSelector ‘,’}
                    (ImportSelector | ‘_’) ‘}’
ImportSelector  ::= id [‘=>’ id | ‘=>’ ‘_’]
```

导入子句的格式为`import $p$.$I$`。其中$p$是一个[稳定标识符](03-types.html#paths)$I$是导入表达式。导入表达式确定$p$的可导入成员的一组名称，这些名称在没有限定条件的情况下可用。如果[可以访问](05-classes-and-objects.html#modifiers)，则$p$的成员$m$是可导入的。导入表达式的最常见形式是*导入选择器*列表。


```scala
{ $x_1$ => $y_1 , \ldots , x_n$ => $y_n$, _ }
```

对于$n \geq 0$,其中缺少最终的通配符`‘_’`.它使每个可导入成员`$p$.$x_i$`在非限定名称的$y_i$下可用。即每个导入选择器`$x_i$ => $y_i$`将`$p$.$x_i$`重命名为$y_i$.如果存在最终通配符，则除`$x_1 , \ldots , x_n,y_1 , \ldots , y_n$`之外的所有的$p$的可导入成员$z$也可以在其的非限定名称下使用。

导入选择器的工作方式和类型成员的工作方式相同。例如：import子句`import $p$.{$x$ => $y\,$}`将`$p$.$x$`项重命名为项$y$。类型名称 `$p$.$x$`重命名为类型名称 $y$.这两个名称中至少有一个必须引用$p$的可导入成员。

如果导入选择器中的目标是通配符，则导入选择器将隐藏对源成员的访问权限。例如，导入选择器`$x$ => _` “renames”将 $x$"重命名"为通配符符号(用户程序中的名称将无法访问)，从而有效的防止对$x$的不合格的访问。如果在相同的导入选择器列表中有一个最终通配符，该通配符将导入前面导入选择器中没有提到的所有成员，那么这将非常有用。

import子句引入的绑定范围在import子句之后立即开始，并延伸到封闭块，模板，包子句或编译单元的末尾，以先出现的为准。

存在几个简称。导入选择器可能只是一个简单的名称$x$.在这种情况下，导入$x$而不重名，因此导入选择器相当于`$x$ => $x$`。此外，可以通过单个标识符或通配符替换整个导入选择器列表。import子`import $p$.$x$`相当于导入`import $p$.{$x\,$}`，使得他无条件的提供$p$的成员$x$.import子句`import $p$._`相当与`import $p$.{_}`,即它无条件的获得$p$的所有成员(类似于java中的`import $p$.*`)。

具有多个导入表达式`import $p_1$.$I_1 , \ldots , p_n$.$I_n$`的import子句被解释为导入子句`import $p_1$.$I_1$; $\ldots$; import $p_n$.$I_n$`的数列。

###### 例
考虑到对象定义：

```scala
object M {
  def z = 0, one = 1
  def add(x: Int, y: Int): Int = x + y
}
```

然后

```scala
{ import M.{one, z => zero, _}; add(zero, one) }
```

相当于

```scala
{ M.add(M.z, M.one) }
```
