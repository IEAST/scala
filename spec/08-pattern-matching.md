---
title: Pattern Matching
layout: default
chapter: 8
---

# 模式匹配

## 模式

```ebnf
  Pattern         ::=  Pattern1 { ‘|’ Pattern1 }
  Pattern1        ::=  boundvarid ‘:’ TypePat
                    |  ‘_’ ‘:’ TypePat
                    |  Pattern2
  Pattern2        ::=  id [‘@’ Pattern3]
                    |  Pattern3
  Pattern3        ::=  SimplePattern
                    |  SimplePattern {id [nl] SimplePattern}
  SimplePattern   ::=  ‘_’
                    |  varid
                    |  Literal
                    |  StableId
                    |  StableId ‘(’ [Patterns] ‘)’
                    |  StableId ‘(’ [Patterns ‘,’] [id ‘@’] ‘_’ ‘*’ ‘)’
                    |  ‘(’ [Patterns] ‘)’
                    |  XmlPattern
  Patterns        ::=  Pattern {‘,’ Patterns}
```

模式由常量，构造函数，变量和类型测试组成的。模式匹配测试给定值(或值序列)是否具有由模式定义的形式，如果是，则将模式中变量绑定到该值或该组值(或值序列)的相关部分。一个模式中的同一个变量名不能多次被绑定。


###### 例
一些模式的例子是：
 1.  模式`ex: IOException`匹配类`IOException`的所有实例，将变量`ex`绑定到实例。
 1.  模式`Some(x)`匹配`Some($v$)`形式的值，将`x`绑定到`Some`构造函数的参数值$v$.
 1.  模式`(x,_)`匹配值对，将`x`绑定到该对的第一个组件。第二组件与通配符模式匹配。
 1.  模式`x :: y :: xs`匹配长度$\geq 2$的列表，将`x`绑定到列表的第一个元素，将`y`绑定到列表的第二个元素，将`xs`绑定到剩下的元素。
 1.  模式`1 | 2 | 3`匹配1到3之间的整数。

模式匹配总是在提供预期类型的模式的上下文中完成，我们区分一下几种模式。

### 变量模式

```ebnf
  SimplePattern   ::=  ‘_’
                    |  varid
```

_变量模式_$x$是一个简单的标识符，以小写字母开头。它匹配任意值，并将变量名称绑定到该值。$x$的类型是从外部给出的模式的预期类型。一种特殊情况是通配符`_`，它每次出现都被作为一个新变量使用。

### 字面值模式

```ebnf
  Pattern1        ::=  varid ‘:’ TypePat
                    |  ‘_’ ‘:’ TypePat
```

_字面值模式_$x: T$由模式变量$x$和类型模式$T$组成。$x$的类型是类型模式$T$，其中每个类型变量和通配符都被一个新的未知类型替换。此模式匹配[字面值模式](#字面值模式)$T$;他将变量名绑定到该模式。

### Binders模式

```ebnf
  Pattern2        ::=  varid ‘@’ Pattern3
```

_绑定模式_`$x$@$p$`由模式变量$x$和模式$p$组成。变量$x$的类型是模式$p$隐含的静态类型$T$。此模式匹配由$p$匹配任意值模式$v$。

如果模式仅匹配$T$类型的值，则模式$p$_表示_ 类型$T$。

### 字面值模式

```ebnf
  SimplePattern   ::=  Literal
```

_字面值模式_$L$匹配任意与文字$L$$相等的值(用`==`表示)。$L$的类型必须复合模式的预期类型。

### 插值字符串模式

```ebnf
  Literal  ::=  interpolatedString
```
模式中插值字符串文字的扩展与表达式中的相同。如果它出现在一个模式中，则是任一形式的插值字符串文字。
```
id"text0{ pat1 }text1 … { patn }textn"
id"""text0{ pat1 }text1 … { patn }textn"""
```
相当于
```
StringContext("""text0""", …, """textn""").id(pat1, …, patn)
```

你可以定义自己的`StringContext`以隐藏`scala`包中的默认值。

如果成员`id`计算为提取器对象，则此扩展是良好的类型。如果提取器对象具有`apply`以及`unapply`或`unapplySeq`方法，则已处理的字符串可用作表达式或模式。

以XML为例
```scala
implicit class XMLinterpolation(s: StringContext) = {
    object xml {
        def apply(exprs: Any*) =
            // parse ‘s’ and build an XML tree with ‘exprs’
            //in the holes
        def unapplySeq(xml: Node): Option[Seq[Node]] =
          // match `s’ against `xml’ tree and produce
          //subtrees in holes
    }
}
```
然后，XML模式匹配可以表示如下：
```scala
case xml"""
      <body>
        <a href = "some link"> \$linktext </a>
      </body>
     """ => ...
```
其中linktext是由模式绑定的变量。

### 稳定标识符模式

```ebnf
  SimplePattern   ::=  StableId
```

_稳定标识符模式_ 为一个[稳定标识符](03-types.html#paths)$r$.$r$的类型要与模式的期望类型一致，该模式匹配所有任何值$v$.例如`$r$ == $v$` ([详情](12-the-scala-standard-library.html#root-classes))。

要解决与变量模式的语法重叠，稳定标识符模式可能不是以小写字母开头的简单名称。但是，可以在在引号中包含这样的变量名称，然后将其视为稳定的标识符模式。

###### 例
考虑以下函数定义：

```scala
def f(x: Int, y: Int) = x match {
  case y => ...
}
```

在这里，`y`是一个可变模式，他匹配任意值。如果我们想将该模式转为稳定的标识符模式，可以按如下方法实现：

```scala
def f(x: Int, y: Int) = x match {
  case `y` => ...
}
```
现在，模式匹配到函数`f`的参数`y`,只有当`f`的两个参数`x`和`y`相等，匹配才能成功。

### 构造函数模式

```ebnf
SimplePattern   ::=  StableId ‘(’ [Patterns] ‘)’
```

_构造函数模式_ 的形式为$c(p_1 , \ldots , p_n)$$n \geq 0$。它由一个稳定的标识符$c$组成，后面跟元素模式$p_1 , \ldots , p_n$。$c$或者是简单名字，或者是一个限定的名字标识符一个[`case`类](05-classes-and-objects.html#case-classes)。如果case类是单态的，他必须与模式的期望类型一致，$x$的[主构造函数](05-classes-and-objects.html#class-definitions)的形式参数类型被视为$p_1, \ldots ,p_n$的期望类型。如果case类是多态的，则实例化其类型参数，以便$c$的实例化复合模式的预期类型。然后将$c$的主构造函数的实例换形式参数类型作为组件模式$p_1, \ldots , p_n$的预期类型。该模式匹配从构造函数调用$c(v_1 , \ldots , v_n)$创建的所有对象，其中每个元素模式$p_i$匹配相应的值$v_i$。

当$c$的形式参数类型结尾是一个重复参数时，这将在[后面](#模式序列)中讨论。

### 元组模式

```ebnf
  SimplePattern   ::=  ‘(’ [Patterns] ‘)’
```

_元组模式_`($p_1 , \ldots , p_n$)`是构造函数模式`scala.Tuple$n$($p_1 , \ldots , p_n$)`$n \geq 2$的别名。空元组`()`是`scala.Unit`类型的唯一值。

### 提取类型

```ebnf
  SimplePattern   ::=  StableId ‘(’ [Patterns] ‘)’
```

_提取模式_$x(p_1 , \ldots , p_n)$ where $n \geq 0$$n \geq 0$与构成函数模式具有相同的语法形式。但是，稳定标识符$x$表示一个对象，它具有名为`unapply`的成员方法或与模式匹配`unapplySeq`,而不是case类。

提取模式不能与值`null`匹配。该实现确保`unapply`/`unapplySeq`方法不应用于`null`。

对象$x$中的`unapply`方法只接受单一参数并且符合以下任一条件时，我们说它_匹配_模式$x(p_1 , \ldots , p_n)$。

* $n=0$且`unapply`的结果类型是`Boolean`。在这种情况下，提取模式匹配所有值$c$,如果`$x$.unapply($v$)`的值为`true`。
* $n=1$且`unapply`的结果类型为`Option[$T$]`，对于某些类型$T$。在这种情况下，(唯一的)参数模式$p_1$依次键入预期类型$T$.提取模式匹配所有值$v$,其中`$x$.unapply($v$)`参数的格式为`Some($v_1$)`，$p_1$匹配$v_1$.
* $n>1$且`unapply`的结果类型是`Option[($T_1 , \ldots , T_n$)]`,对于某些类型$T_1 , \ldots , T_n$。在这种情况下，参数模式$p_1, \ldots , p_n$依次输入预期类型$T_1 , \ldots , T_n$。提取模式匹配所有值$v$,其中`$x$.unapply($v$)`产生一些形式的值  `Some(($v_1 , \ldots , v_n$))`，并且每个模式$p_i$匹配相应的值$v_i$。

对象$x$中的`unapplySeq`方法匹配模式$x(q_1 , \ldots , q_m, p_1 , \ldots , p_n)$，如果它只需要一个参数且其结果类型的格式为`Option[($T_1 , \ldots , T_m$, Seq[S])]`(如果`m=0`，也接受类型`Option[Seq[S]]`)。下面进一步讨论[这种](#模式序列)情况。

###### 例子

如果我们定义一个提取对象 `Pair`:

```scala
object Pair {
  def apply[A, B](x: A, y: B) = Tuple2(x, y)
  def unapply[A, B](x: Tuple2[A, B]): Option[Tuple2[A, B]] = Some(x)
}
```

这意味着可以使用名称`Pair`代替`Tuple2`来形成元组以及解构元祖中的元祖。因此，以下是可能的。

```scala
val x = (1, 2)
val y = x match {
  case Pair(i, s) => Pair(s + i, i * i)
}
```

### 模式序列

```ebnf
SimplePattern ::= StableId ‘(’ [Patterns ‘,’] [varid ‘@’] ‘_’ ‘*’ ‘)’
```

_模式序列_$p_1 , \ldots , p_n$出现在两个语境中。首先，在构造函数模式$c(q_1 , \ldots , q_m, p_1 , \ldots , p_n)$中，其中$c$是一个case类，拥有$m+1$个主要构造参数，最后一个参数是类型为`S*`的[重复参数](04-basic-declarations-and-definitions.html#repeated-parameters)。其次，在提取模式$x(q_1 , \ldots , q_m, p_1 , \ldots , p_n)$中，如果提取对象$x$没有`unapply`方法，但它确实定义楼一个`unapplySeq`方法，其结果类型符合`Option[(T_1, ... , T_m, Seq[S])]`(如果`m=0`,则椰接受类型`Option[Seq[S]]`)。模式$p_i$的预期类型为$S$。

模式序列中的最后一个模式可以是_序列通配符_`_*`。每个元素模式$p_i$都使用$S$作为预期类型进行类型检查，除非它是序列通配符。如果存在最终序列通配符，则该模式匹配所有值$v$，这些值是以匹配模式$p_1 , \ldots , p_{n-1}$的元素开头的序列。

### 中缀操作模式

```ebnf
  Pattern3  ::=  SimplePattern {id [nl] SimplePattern}
```

_中缀操作模式_$p;\mathit{op};q$是构造函数或提取模式的简写形式$\mathit{op}(p, q)$。模式中运算符的优先级和关联性与[表达式](06-expressions.html#prefix,-infix,-and-postfix-operations)中的相同。

中缀操作模式$p;\mathit{op};(q_1 , \ldots , q_n)$是构造函数或提取器模式$\mathit{op}(p, q_1, \ldots , q_n)$的简写。

### 模式选择

```ebnf
  Pattern   ::=  Pattern1 { ‘|’ Pattern1 }
```

_模式选择_`$p_1$ | $\ldots$ | $p_n$`由许多替代模式$p_i$组成。使用预期的模式类型对所偶替换模式进行检查。它们可能不会绑定除通配符之外的变量。如果至少有一个替代模式匹配$v$，则替代模式匹配值$v$。

### XML模式

XML模式的处理看[这](10-xml-expressions-and-patterns.html#xml-patterns).

### 正则表达式

从2.0版本开始，Scala中就停止使用正则表达式模式。

更高版本的Scala提提供了一个简化版的正则表达式模式，涵盖了大多数绯闻本序列处理的场景。_模式序列_ 是位于以下位置的模式：(1)预期符合某些`A`的`Seq[A]`的类型`T`模式。(2)遇有迭代形式参数的案例类构造函数`A*`。最右边位置的同为符星形图案`_*` 代表任意长序列。像往常一样，它可以使用`@`绑定道变量，在这种情况下，变量将具有`Seq[A]`类型。

### 恒等模式

当一个模式$p$满足下列条件之一，我们称为类型$T$的_恒等_类型模式:

1.  $p$ 是一个变量模式
1.  $p$ 是一个文字模式 $x: T'$, 并且 $T <: T'$,
1.  $p$ 是构造函数模式$c(p_1 , \ldots , p_n)$，类型$T$是类$c$的实例，$T$类型的[主构造函数](05-classes-and-objects.html#class-definitions)具有参数类型$T_1 , \ldots , T_n$，每个$p_i$是$T_i$的恒等模式。

## 类型模式

```ebnf
  TypePat           ::=  Type
```

类型模式由类型，类型变量和通配符组成。类型模式$T$是以下形式之一：

* 对类$C$，类$p.C$，或`$T$#$C$`的引用。匹配类$C$的所有非空实例。此类模式匹配给定类的任何非空实例。注意类的前缀(如果存在)与确定类实例相关。例如模式$p.C$仅匹配使用路径$p$作为前缀创建的$C$类的实例。这也适用于未在语法上给出的前缀。例如，如果$C$引用在最近的封闭类中定义的类，因此等于$this.C$，这认为它具有前缀。

  底部类型`scala.Nothing` 和 `scala.Null`不能用作类型模式，因为他们在任何情况下都不匹配。

* 单例类型`$p$.type`。此类型模式仅匹配路径$p$表示的值(`eq`方法用于判断与`p`的同一性)

* 文字类型`$lit$`。此类型模式仅匹配文字$list$表示的值(`==`方法用于将匹配值与$list$进行比较)。

* 复合类型模式`$T_1$ with $\ldots$ with $T_n$`其中每个$T_i$是一种类型模式。此类型模式匹配每个类型模式$T_i$匹配的所有值。

* 参数化类型模式$T[a_1 , \ldots , a_n]$，其中$a_i$是类型变量模式或通配符`_`。对于类型变量和通配符的某些任意实例化，此类型模式匹配与$T$匹配的所有值。这些类型变量的边界或别名类型在[这](#type-parameter-inference-in-patterns)讨论。
A parameterized type pattern $T[a_1 , \ldots , a_n]$, where the $a_i$
  are type variable patterns or wildcards `_`.
  This type pattern matches all values which match $T$ for
  some arbitrary instantiation of the type variables and wildcards. The
  bounds or alias type of these type variable are determined as
  described [here](#模式中的类型参数推断).

* 参数化类型模式`scala.Array$[T_1]$`，其中$T_1$是一种类型模式。此类型模式匹配任何类型为`scala.Array$[U_1]$`的非空实例，其中$U_1$ 是与$T_1$匹配的类型。

不属于上述形式之一的类型也被接受为类型模式。但是，这种类型模式将被转换为他们的[擦除](03-types.html#type-erasure).导致Scala编译器发出"unchecked"警告"，以标记可能的类型安全性丢失。

_类型变量模式_ 是一个简单的标识符，以小写字母开头。

## 模式中类型参数推断

类型参数 推断处理指定类型模式或构造器模式中的被绑定的类型变量的边界，从而确定模式的期望类型。

### 类型化模式的类型参数推断

假定一个模式为$p: T'$。将$T'$中的所有通配符用新的类型变量代替，我们得到类型$T$,其类型变量为$a_1 , \ldots , a_n$，这些类型变量在模式中被绑定。并令模式的期望类型为$\mathit{pt}$.

类型参数推断：首先在类型变量$a_i$上构造一组子类型约束$\mathcal{C}\_0$，初始约束集仅反映这些类型变量的范围。也就是说，假设$T$绑定类类型变量$a_1 , \ldots , a_n$,他们对应于类类型参数$a_1' , \ldots , a_n'$，下限为$L_1, \ldots , L_n$和上限$U_1 , \ldots , U_n$, $\mathcal{C}_0$包含约束：

$$
\begin{cases}
a_i &<: \sigma U_i & \quad (i = 1, \ldots , n) \\\\
\sigma L_i &<: a_i & \quad (i = 1, \ldots , n)
\end{cases}
$$

 $\sigma$ 是替换 $[a_1' := a_1 , \ldots , a_n' :=a_n]$.

然后对 $\mathcal{C}_0$进一步进行子类下约束，这一过程可能有两种情况：

###### 情况1
如果存在针对类型变量$a_i , \ldots , a_n$的变换$\sigma$，使得$\sigma T$与$\mathit{pt}$一致。可以确定一个针对$a_1, \ldots , a_n$的最弱子类型约束$\mathcal{C}\_1$，使得$\mathcal{C}\_0 \wedge \mathcal{C}_1$意味着$T$符合$\mathit{pt}$。

###### 情况2
否则，如果$T$不能通过实例化其类型变量来符合$\mathit{pt}$ ，则可以把$\mathit{pt}$ 中的所有类型变量定义为模式中的某个方法的类型参数，令这些类型参数为$b_1 , \ldots ,b_m$，且$\mathcal{C}\_0'$是反映类型变量$b_i$边界的子类下约束集。如果$T$是某final类的实例类型，那么有针对$a_1 , \ldots , a_n$和$b_1 , \ldots , b_m$的最弱子类型约束集$\mathcal{C}\_2$，使得{C}\_0 \wedge \mathcal{C}\_0' \wedge \mathcal{C}\_2$和$T$与$\mathit{pt}$一致。如果$T$不表示final类的实例类型，同样有$a_1 , \ldots , a_n$和$b_1 , \ldots , b_m$的最弱子类型约束集$\mathcal{C}\_2$，使得$\mathcal{C}\_0 \wedge \mathcal{C}\_0' \wedge \mathcal{C}\_2$使有可能构造出类型$T'$与$T$和$\mathit{pt}$都一致。如果没有符合该条建的约束集$\mathcal{C}\_2$，就会参生静态错误。

最后一步是为类型变量选择类型边界，这意味着建立的约束系统。对于上述两种情况， 该过程是不同的。

###### 情况1
我们取$a_i >: L_i <: U_i$，其中每个$L_i$是最小的，每个 $U_i$最大值为$<:$，这样$a_i >: L_i <: U_i$$i = 1, \ldots, n$可满足$\mathcal{C}\_0 \wedge \mathcal{C}\_1$。

###### 情况2
我们让$a_i >: L_i <: U_i$ 和 $b\_i >: L_i' <: U_i' $称其，其中每个$L_i$和$L_j'$都是最小的，$U_i$ 和 $U_j'$是最大的，这样$a_i >: L_i <: U_i$对于$i = 1 , \ldots , n$和$b_j >: L_j' <: U_j'$对于$j = 1 , \ldots , m$，可满足$\mathcal{C}\_0 \wedge \mathcal{C}\_0' \wedge \mathcal{C}_2$。

在这两种情况下，允许局部类型推断来限制推断边界的复杂性。必须相对于可接受复杂性的类型集来理解类型的最小性和最大性。


### 构造模式的类型参数推断
假设构造函数模式$C(p_1 , \ldots , p_n)$其中类$C$具有类型参数$a_1 , \ldots , a_n$。这些类型参数可以与类型化模式`(_: $C[a_1 , \ldots , a_n]$)`相同的方法推断出来。

###### 例如
考虑程序片段：

```scala
val x: Any
x match {
  case y: List[a] => ...
}
```
这，类型模式`List[a]`与预期类型`Any`匹配，模式绑定了类变量`a`，`List[a]`的任何类型参数斗使得`List[a]`和`Any`一致。因此`a`是没有边界的抽象类型。`a`的作用域就是case子句右边的代码：

另一方面，如果`x`的声明是这样：

```scala
val x: List[List[String]],
```
这会生成约束`List[a] <: List[List[String]]`，它简化为`a <: List[String]`，因为`List`是协变的。这样可知`a`具有类型上界`List[String]`。

###### 例
考虑程序片段：
```scala
val x: Any
x match {
  case y: List[String] => ...
}
```

Scala在运行时不维护有关有关类型参数的信息，因此无法检查`x`是否为字符串列表。相反，scala编译器会将模式[擦除](03-types.html#type-erasure)为`List[_]`;也就是说，它只会测试值`x`的顶级运行时类是否符合`List`，如果匹配则模式匹配成功。在列表`x`包含除字符串之外的元素的情况下，这可能导致稍后的类型转换异常。Scala编译器会针对这种情况给出"unchecked"警告信息，提醒用户潜在的类型安全缺陷。
###### 例
考虑一下程序：

```scala
class Term[A]
class Number(val n: Int) extends Term[Int]
def f[B](t: Term[B]): B = t match {
  case y: Number => y.n
}
```
模式`y: Number` 的期望类型是`Term[B]`，但类型`Number`并不与`Term[B]`一致，因此必须使用前面讨论的方式2,引入类型变量`B`来推断子类型约束。在这个例子中我们有约束`Number <: Term[B]`，限定了`B = Int`。因此`B`在case语句中被当作一个抽象类型，其上下限均为`INt`，因此，case语句的右边部分`y,n`的类型为`Int`,也就是与函数声明的类型`Number`一致。

## 模式匹配表达式

```ebnf
  Expr            ::=  PostfixExpr ‘match’ ‘{’ CaseClauses ‘}’
  CaseClauses     ::=  CaseClause {CaseClause}
  CaseClause      ::=  ‘case’ Pattern [Guard] ‘=>’ Block
```

一个 _模式匹配表达式_

```scala
e match { case $p_1$ => $b_1$ $\ldots$ case $p_n$ => $b_n$ }
```

由一个选择器表达式$e$和一组$n$个case表达式组成($n > 0$)。每个case由一个(可能有守卫)模式$p_i$和代码块$b_i$组成。每个$p_i$可以由守卫语句`if $e$`进一步限定，$e$为布尔表达式。$p_i$中的模式变量的作用域包括守卫语句和对应的代码块$b_i$。

令选择表达式$e$的类型为$T$,并且 $a_1 , \ldots , a_m$是拥有模式匹配的表达式内的方法的类型参数。对于每个$a_i$具有类型下界$L_i$和类型上界$U_i$。每个模式$p \in \{p_1, , \ldots , p_n\}$的类型有两种方法得到的。首先，视图令$T$为$p$的期望类型，如果失败，将$T$的每个类型参数$a_i$用\mbox{\sl undefined}取代，得到另一个期望类型$T'$。如果这一步也失败的话，将会抛出一个编译错误。否则令$T_p$为表达式$p$的类型，决定一组类型下界$L_11 , \ldots , L_m'$ 和类型上界$U_1' , \ldots , U_m'$使得对所有的$i$，关系$L_i <: L_i'$ 和$U_i' <: U_i$都成立，且满足下面的约束关系：

$$L_1 <: a_1 <: U_1\;\wedge\;\ldots\;\wedge\;L_m <: a_m <: U_m \ \Rightarrow\ T_p <: T$$

如果找不到这样的边界，则会发生编译错误。如果找到这样的边界，否则$a_i$的下边界将为$L_i'$而不是$L_i$,并且上限为$U_i'$而不是$U_i$。由此得到$p$开头的模式匹配子句的类型。

每个块$b_i$的预期类型就是整个模式匹配表达式的预期类型。然后，模式匹配表达式的类型是所有块$b_i$的类型的[弱最小上限](03-types.html#weak-conformance)。

将模式匹配表达式应用与选择器值时，将按照顺序模式尝试，直到找到与[选择器值](#模式)匹配的模式。假设这种匹配到`case $p_i \Rightarrow b_i$`。整个表达式的结果就是代码块$b_i$的求值结果。而$p_i$所包含的所有模式变量将会绑定到选择器的对应部分。如果匹配失败，将会抛出一个`scala.MatchError`异常。

case表达式中的模式可以有守卫后缀`if e`为布尔型表达式。匹配道该模式之后，对守卫表达式求值，值为`true`则匹配成功。如果保护模式的结算结果为`false`，则表示该情况下的模式不匹配，并继续搜索匹配模式。

为了效率，模式匹配表达式的苹果可以尝试除文本序列之外的其他顺序的模式。这时如果某些模式的搜为表达式包含副作用的话，就可能会对结果产生影响，但编译其能够确保只有模式被匹配到的时候，其守卫表达式才被求值。

如果模式匹配的选择器是[`sealed`类](05-classes-and-objects.html#modifiers)的实例,则模式匹配的编译可以发出警告。提醒给定待匹配模式枚举不完全，运行时可能会引发`MatchError`。

###### 例

考虑以下算数术语的定义

```scala
abstract class Term[T]
case class Lit(x: Int) extends Term[Int]
case class Succ(t: Term[Int]) extends Term[Int]
case class IsZero(t: Term[Int]) extends Term[Boolean]
case class If[T](c: Term[Boolean],
                 t1: Term[T],
                 t2: Term[T]) extends Term[T]
```

有些术语表示数字字面值，加一运算，0值测试和条件运算。每个运算符都是一个类型参数，表明运算的类型( `Int` 或 `Boolean`)。

这些术语的类型安全评估器可以写成如下形式：

```scala
def eval[T](t: Term[T]): T = t match {
  case Lit(n)        => n
  case Succ(u)       => eval(u) + 1
  case IsZero(u)     => eval(u) == 0
  case If(c, u1, u2) => eval(if (eval(c)) u1 else u2)
}
```
注意，类型参数可以通过模式匹配获得新的类型边界这一事实，是求值函数得以成立的关键。

例如，第二个case中，模式`Succ(u)`类型参数`T`的类型为`Int`，只有当`T`的类型上下界都是`Int`时，才符合选择器的期望类型。有了`Int <: T <: Int`这个假设，我们可以验证第二个的右侧的类型`Int`与期望类型`T`一致。

## 模式匹配匿名函数

```ebnf
  BlockExpr ::= ‘{’ CaseClauses ‘}’
```

匿名函数由一系列case定义：

```scala
{ case $p_1$ => $b_1$ $\ldots$ case $p_n$ => $b_n$ }
```

实际上是一个没有前缀`match`的表达式。表达式的期望类型必须部分被定义，要么是`scala.Function$k$[$S_1 , \ldots , S_k$, $R$]` $k > 0$，要么是`scala.PartialFunction[$S_1$, $R$]`，其中参数类型$S_1 , \ldots , S_k$必须明确，而结果类型是可以待定的。


如果预期类型是[SAM转换](06-expressions.html#sam-conversion)`scala.Function$k$[$S_1 , \ldots , S_k$, $R$]`,则表达式被认为等同于匿名函数：

```scala
($x_1: S_1 , \ldots , x_k: S_k$) => ($x_1 , \ldots , x_k$) match {
  case $p_1$ => $b_1$ $\ldots$ case $p_n$ => $b_n$
}
```
在这，每个$x_i$都是一个新名称。如[这](06-expressions.html#anonymous-functions)所示，上述匿名函数与下面的实例构造表达式等价(这里的$T$是所有$b_i$的最小共同类型上界)

```scala
new scala.Function$k$[$S_1 , \ldots , S_k$, $T$] {
  def apply($x_1: S_1 , \ldots , x_k: S_k$): $T$ = ($x_1 , \ldots , x_k$) match {
    case $p_1$ => $b_1$ $\ldots$ case $p_n$ => $b_n$
  }
}
```
如果期望类型是`scala.PartialFunction[$S$, $R$]`，表达式与下列实例构造表达式等价：

```scala
new scala.PartialFunction[$S$, $T$] {
  def apply($x$: $S$): $T$ = x match {
    case $p_1$ => $b_1$ $\ldots$ case $p_n$ => $b_n$
  }
  def isDefinedAt($x$: $S$): Boolean = {
    case $p_1$ => true $\ldots$ case $p_n$ => true
    case _ => false
  }
}
```

这里，$x$是一个新名称，$T$是所有$b_i$类型的最弱上限。`isDefinedAt`方法中，如果前面的模式$p_1 , \ldots , p_n$已经有了变量模式或通配符模式，最后缺省case将被忽略。

###### 例
这是一种使用左折号操作`/:`计算两个向量的标量积的方法：

```scala
def scalarProduct(xs: Array[Double], ys: Array[Double]) =
  (0.0 /: (xs zip ys)) {
    case (a, (b, c)) => a + b * c
  }
```

此代码中的case子句等效于以下匿名函数：

```scala
(x, y) => (x, y) match {
  case (a, (b, c)) => a + b * c
}
```
