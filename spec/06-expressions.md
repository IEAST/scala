---
title: Expressions
layout: default
chapter: 6
---

# 表达式

```ebnf
Expr         ::=  (Bindings | id | ‘_’) ‘=>’ Expr
               |  Expr1
Expr1        ::=  ‘if’ ‘(’ Expr ‘)’ {nl} Expr [[semi] ‘else’ Expr]
               |  ‘while’ ‘(’ Expr ‘)’ {nl} Expr
               |  ‘try’ (‘{’ Block ‘}’ | Expr) [‘catch’ ‘{’ CaseClauses ‘}’] [‘finally’ Expr]
               |  ‘do’ Expr [semi] ‘while’ ‘(’ Expr ‘)’
               |  ‘for’ (‘(’ Enumerators ‘)’ | ‘{’ Enumerators ‘}’) {nl} [‘yield’] Expr
               |  ‘throw’ Expr
               |  ‘return’ [Expr]
               |  [SimpleExpr ‘.’] id ‘=’ Expr
               |  SimpleExpr1 ArgumentExprs ‘=’ Expr
               |  PostfixExpr
               |  PostfixExpr Ascription
               |  PostfixExpr ‘match’ ‘{’ CaseClauses ‘}’
PostfixExpr  ::=  InfixExpr [id [nl]]
InfixExpr    ::=  PrefixExpr
               |  InfixExpr id [nl] InfixExpr
PrefixExpr   ::=  [‘-’ | ‘+’ | ‘~’ | ‘!’] SimpleExpr
SimpleExpr   ::=  ‘new’ (ClassTemplate | TemplateBody)
               |  BlockExpr
               |  SimpleExpr1 [‘_’]
SimpleExpr1  ::=  Literal
               |  Path
               |  ‘_’
               |  ‘(’ [Exprs] ‘)’
               |  SimpleExpr ‘.’ id
               |  SimpleExpr TypeArgs
               |  SimpleExpr1 ArgumentExprs
               |  XmlExpr
Exprs        ::=  Expr {‘,’ Expr}
BlockExpr    ::=  ‘{’ CaseClauses ‘}’
               |  ‘{’ Block ‘}’
Block        ::=  BlockStat {semi BlockStat} [ResultExpr]
ResultExpr   ::=  Expr1
               |  (Bindings | ([‘implicit’] id | ‘_’) ‘:’ CompoundType) ‘=>’ Block
Ascription   ::=  ‘:’ InfixType
               |  ‘:’ Annotation {Annotation}
               |  ‘:’ ‘_’ ‘*’
```

表达式由运算符和操作数组成。 随后以递减的优先顺序讨论表达形式。

## 表达式类型

_表达式类型_ 通常与某些预期类型相关(可能是未定义的)。当我们写“表达式$e$预计符合$T$类型”时，我们的意思是：
  1.  $e$ 的预期类型是 $T$,
  2. 表达式 $e$ 必须符合 $T$.

以下斯科伦化规则通用于所有表达式。假定表达式的类型是存在类型 $T$，则表达式的类型被假定为$T$的[斯科伦范式](03-types.html#existential-types)。

通过类型打包来逆转斯克伦范式，假设类型$T$的表达式$e$并且让$t_1[\mathit{tps}\_1] >: L_1 <: U_1 , \ldots , t_n[\mathit{tps}\_n] >: L_n <: U_n$为$e$的某些部分的斯克伦范式创建的所有类型变量，这些变量在$T$中是可以直接使用的。

Skolemization is reversed by type packing. Assume an expression $e$ of
type $T$ and let $t_1[\mathit{tps}\_1] >: L_1 <: U_1 , \ldots , t_n[\mathit{tps}\_n] >: L_n <: U_n$ be
all the type variables created by skolemization of some part of $e$ which are free in $T$.

那么$e$的包类型是:

```scala
$T$ forSome { type $t_1[\mathit{tps}\_1] >: L_1 <: U_1$; $\ldots$; type $t_n[\mathit{tps}\_n] >: L_n <: U_n$ }.
```

## 字面值

```ebnf
SimpleExpr    ::=  Literal
```

描述文字的键入以及它们的[词法语法](01-lexical-syntax.html#literals);他们的求值是即时的

## *空* 值

`null`值的类型为 `scala.Null`，因此符合每个引用类型。它表示引用类型。他表示引用特殊`null`对象的参考值。该对象实现了类`scala.AnyRef`中的方法，如下所示：

- `eq($x\,$)` 和 `==($x\,$)` 如果参数$x$也是"null"对象，则返回 `true`.
- `ne($x\,$)` 和 `!=($x\,$)` 如果参数 x 不是"null"对象，则返回true。
- `isInstanceOf[$T\,$]` 总是返回 `false`.
- `asInstanceOf[$T\,$]` 返回$T$类型的[默认值](04-basic-declarations-and-definitions.html#value-declarations-and-definitions)。
- `##` 返回 ``0``.

对 "null" 对象的任何成员引用会导致抛出 `NullPointerException` 。

## 指示器

```ebnf
SimpleExpr  ::=  Path
              |  SimpleExpr ‘.’ id
```

指示符指的是命名的术语。他可以是 _简单的名称_ 或 _选择_ 。

简单名称$x$指的是[此处](02-identifiers-names-and-scopes.html#identifiers,-names-and-scopes)的值。如果$x$被封闭类或对象$C$中的定义或声明绑定，则它被认为等同于选择`$C$.this.$x$` ,qizhong $C$用于应用包含$x$的类，即使类型名$C$在$x$出现时被[隐藏](02-identifiers-names-and-scopes.html#identifiers,-names-and-scopes) .

如果$r$是$T$类型[稳定标识符](03-types.html#paths),则选择 $r.x$ 静态地引用$r$的术语成员$m$,其在$T$中由名称$x$标识。

对于其他表达式$e$, $e.x$的输入就好像是 `{ val $y$ = $e$; $y$.$x$ }`，$y$是一个新成员。

指示器前缀的期望类型总是未定义的。指示符的类型是它引用的实体的类型$T$,但是以下情况除外：在需要[稳定类型](03-types.html#singleton-types)的上下文中出现的[路径](03-types.html#paths)$p$的类型是单例类型`$p$.type`.

需要稳定类型的上下文需要满足以下条件:

1. 路径$p$作为选择的前缀出现，并且它不指定常量
1. 期望类型 $\mathit{pt}$ 是一种稳定类型
1. 期望类型 $\mathit{pt}$ 是一个抽象类型，其类型为稳定类型，$p$引用的实体类型$T$不符合$\mathit{pt}$。
1. 路径$p$指定一个模块。

选择$e.x$在限定表达式$e$第一次求值时求值，同时产生一个对象$r$,选取的结果是$r$的成员，该成员是由$m$定义或覆盖$m$的定义定义。

## This 和 Super

```ebnf
SimpleExpr  ::=  [id ‘.’] ‘this’
              |  [id ‘.’] ‘super’ [ClassQualifier] ‘.’ id
```

表达式`this`可以出现在模板或符合类型的语句部分。它表示由包含引用的最内层模板或复合类型定义的对象。如果是复合类型，则`this`的类型是该化合物类型。如果他们是具有简单名称$C$的类或对象定义的模板，则其类型与`$C$.this`类型相同。

表达式`$C$.this`在封闭类或对象定义的语句部分是合法的，简称为$C$。它代表最内层的定义定义的对象。如果表达式的期望类型是稳定类型，或`$C$.this`作为选择的前缀出现，则其类型为`$C$.this.type`，否则它是$C$的自身类型。

引用`super.$m$`静态引用包含该引用的最里层模板的最小合理超类型的方法或类型$m$。它的求值结果等于$m$或重载$m$的该模板的实际超类型中的成员$m'$。静态引用的成员$m$必须是类型或方法。

如果它是一个方法，那么它必须是具体的，或者包含引用的模板必须有一个成员$m'$覆盖$m$并且标记为`abstract override`.

引用`$C$.super.$m$`静态引用一个方法或类型$m$,在最内层的封闭类或对象定义的最不合适的超类型中，它包含引用。它求值的结果是$m$或重载$m$的该类或对象的实际超类中的成员$m'$。被静态引用成员$m$必须是类型或方法。如果静态引用的成员$m$是一个方法，它必须是具体的，或者名为$C$的最内存的封闭类或对象定义必须有一个成员$m'$,它覆盖$m$并且标记为`abstract override`.

前缀`super`后面可以跟特征限定符`[$T\,$]`,例如 `$C$.super[$T\,$].$x$`。这被称为 *静态超级引用*。在这种情况下，引用的是简单名称$T$的$C$的父特征中$x$的类型或方法。该成员必须是唯一定义的。如果这是一种方法，那么该方法必须是实体的。


###### 例
考虑一下类定义

```scala
class Root { def x = "Root" }
class A extends Root { override def x = "A" ; def superA = super.x }
trait B extends Root { override def x = "B" ; def superB = super.x }
class C extends Root with B {
  override def x = "C" ; def superC = super.x
}
class D extends A with B {
  override def x = "D" ; def superD = super.x
}
```

 `C`类的线性化是`{C, B, Root}`，`D`类型的线性化是 `{D, B, A, Root}`。然后我们有

```scala
(new A).superA == "Root"

(new C).superB == "Root"
(new C).superC == "B"

(new D).superA == "Root"
(new D).superB == "A"
(new D).superD == "B"
```

要注意，`superB`方法返回不同的结果，具体取决于`B`是否与类 `Root`或`A`混合在一起.

## 函数应用

```ebnf
SimpleExpr    ::=  SimpleExpr1 ArgumentExprs
ArgumentExprs ::=  ‘(’ [Exprs] ‘)’
                |  ‘(’ [Exprs ‘,’] PostfixExpr ‘:’ ‘_’ ‘*’ ‘)’
                |  [nl] BlockExpr
Exprs         ::=  Expr {‘,’ Expr}
```

应用程序`$f(e_1 , \ldots , e_m)$` 将函数`$f$`应用于参数表达式 `$e_1, \ldots , e_m$`。为了使该表达式具有良好类型，该函数必须 *适用* 于其参数，接下来通过对$f$类型的分析来定义该函数。

如果$f$具有方法类型`($p_1$:$T_1 , \ldots , p_n$:$T_n$)$U$`，则每个参数表达式$e_i$的类型必须与对应的参数类型$T_i$一致。设$S_i$是 $e_i$ $(i = 1 , \ldots , m)$的参数类型。方法$f$必须 *适用* 于其参数s $e_1, \ldots , e_n$或类型$S_1 , \ldots , S_n$。我们说参数表达式$e_i$是一个 *命名* 参数，如果他的形式为`$x_i=e'_i$`，`$x_i$`是参数名称 `$p_1, \ldots, p_n$`之一。

一旦确定了类型$S_i$，如果满足以下所有条件，则认为上述方法对方法类型$f$适用：
  - 对于每个命名参数 $p_j=e_i'$类型$S_i$与参数类型 $T_j$[兼容](03-types.html#compatibility)。
  - 对于每个位置参数$e_i$，$S_i$类型与$T_i$[兼容](03-types.html#compatibility)。
  - 如果定义了与其类型，则结果类型$U$与之[兼容](03-types.html#compatibility)。
如果$f$是一个多态方法，则使用[本地类型推](#局部类型推断)断来实例化$f$的类型参数。如果类型推断可以确定可以确定类型参数，那么多态方法是适用的，以便实例化方法适用。

如果$f$具有某种值类型，则应用程序等同于`$f$.apply($e_1 , \ldots , e_m$)`,即应用由$f$定义的`apply`方法。如果`$f$.apply` 适用，则 `$f$`值适用于给定的参数。

`$f$($e_1 , \ldots , e_n$)`的求值通常必须顺序计算$f$和 $e_1 , \ldots , e_n$。每个参数表达式都转换为其相应形式的参数类型。字后，应用程序被重写到函数的右侧，实际参数替换形式参数。计算重写的右侧结果最终转换为函数声明的结果类型(如果有的话)。

具有无参方法类型`=>$T$`的形式参数的情况会被特殊处理。在这种情况下，在应用程序之前不会计算相应的实际参数表达式$e$。相反，在重写规则的右侧使用形式参数需要重新计算$e$。换句话说，`=>`参数的计算顺序是*按名称调用*，而正常参数的计算顺序是*按值调用*。此外，要求$e$的[打包类型](#表达式类型化)符合参数类型$T$.如果由于named或default参数将将应用程序转换为块，则会保留by-name参数的行为。在这种情况下，该参数的本地值的格式为 `val $y_i$ = () => $e$`，传递给该函数的参数为`$y_i$()`。

应用中最后一个参数可以标记为序列参数，例如`$e$: _*`。这样的参数必须与一个类型`$S$*`的[重复参数](04-basic-declarations-and-definitions.html#repeated-parameters)一致，也必须是唯一与该参数匹配的参量(即形式参数和实际参数的数量必须相同)。此外，$e$的类型必须符合`scala.Seq[$T$]`，对于符合$S$的某些类型$T$。在这种情况下，参数类型的最终形式是用其元素来替换$e$.当应用程序使用命名参数时，可变参数的参数必须有明确的指定。

函数应用通常在程序运行时堆栈上分配一个新帧。但是，如果本地方法或者fina方法将自身分配为最后一个操作的化，该调用将在调用者的堆栈帧中执行。

###### 例
假设以下方法计算可变数量的参数总和：
```scala
def sum(xs: Int*) = (0 /: xs) ((x, y) => x + y)
```

那么

```scala
sum(1, 2, 3, 4)
sum(List(1, 2, 3, 4): _*)
```

都会得到结果 `10`，然而

```scala
sum(List(1, 2, 3, 4))
```

无法通过类型检查.

### 命名和默认参数

如果应用程序要使用命名参数$p = e$或默认参数，必须满足一下条件。
- 对于参数列表$e_1 \ldots e_m$中位置参数左侧出现的每个命名参数$p_i = e_i$，参数位置$i$与应用方法的参数列表中参数$p_i$的位置一致。
- 所有命名参数的名称 $x_i$ 是成对不同的，并且没有命名的参数定义了由位置变量指定的参数。
- 每个正式参数$p_j:T_j$都没有由positional或named参数指定，他有一个默认参数

如果应用程序使用命名参数或默认参数，则使用以下转换将其转换为没有命名参数或默认参数的应用程序。

如果方法 $f$的格式为 `$p.m$[$\mathit{targs}$]` 则转换为块：

```scala
{ val q = $p$
  q.$m$[$\mathit{targs}$]
}
```

如果方法$f$本身就是一个应用程序表达式，则转换将以递归的方式应用于$f$.转换$f$的结果是以下块：

```scala
{ val q = $p$
  val $x_1$ = expr$_1$
  $\ldots$
  val $x_k$ = expr$_k$
  q.$m$[$\mathit{targs}$]($\mathit{args}_1$)$, \ldots ,$($\mathit{args}_l$)
}
```

其中$(\mathit{args}\_1) , \ldots , (\mathit{args}\_l)$中的每个参数都是对$x_1 , \ldots , x_k$的引用。要将当前程序集成到块中，首先为$e_1 , \ldots , e_m$中的每个参数创建一个使用新名称$y_i$的值定义，初始化化为$e_i$用于位置参数和$e'_i$表示`$x_i=e'_i$`形式的命名参数。然后，对于参数裂变未指定的参数，将创建使用名称为 $z_i$ 的值定义，该值使用计算此参[数默认参](04-basic-declarations-and-definitions.html#function-declarations-and-definitions)数的方法进行初始化。_

假设\mathit{args}$是生成的名称 $y_i$ 和 $z_i$的序列，这样每个名称的位置与与方法类型`($p_1:T_1 , \ldots , p_n:T_n$)$U$`中相应参数的位置相匹配。转换的最终结果是一个块形式：

```scala
{ val q = $p$
  val $x_1$ = expr$_1$
  $\ldots$
  val $x_l$ = expr$_k$
  val $y_1$ = $e_1$
  $\ldots$
  val $y_m$ = $e_m$
  val $z_1$ = $q.m\$default\$i[\mathit{targs}](\mathit{args}_1), \ldots ,(\mathit{args}_l)$
  $\ldots$
  val $z_d$ = $q.m\$default\$j[\mathit{targs}](\mathit{args}_1), \ldots ,(\mathit{args}_l)$
  q.$m$[$\mathit{targs}$]($\mathit{args}_1$)$, \ldots ,$($\mathit{args}_l$)($\mathit{args}$)
}
```

### 签名多态方法

对于目标平台`$f$($e_1 , \ldots , e_m$)`的签名多态方法的调用，被调用的方法在每个调用位置有不同的方法类型`($p_1$:$T_1 , \ldots , p_n$:$T_n$)$U$` 。参数类型`$T_ , \ldots , T_n$` 是参数表达式`$e_1 , \ldots , e_m$`的类型，而`$U$`是调用位置的预期的类型。如果期望的类型未定义，那么`$U$`是 `scala.AnyRef`。参数名称`$p_1 , \ldots , p_n$`是新的。

###### 注意

在java7及更高版本中，类`java.lang.invoke.MethodHandle` 中的方法`invoke` 和`invokeExact`是签名多态。

## 方法值

```ebnf
SimpleExpr    ::=  SimpleExpr1 ‘_’
```

如果$e$是方法类型或$e$是按照名称调用参数，则表达式`$e$ _`的格式正确。如果$e$是带参数的方法，
`$e$ _`通过[eta扩展](#eta扩展)得到函数类型$e$。如果$e$是无参方法或者类型`=>$T$`的call-by-name参数，`$e$ _`表示类型为`() => $T$`的函数，且将应用于空参数列表`()`时求$e$的值。

###### 例
左列中的方法值分别等同于右侧的[eta扩展](#eta扩展)表达式。
| placeholder syntax(占位符语法)  | eta-expansion(eta扩展表达式)                                                               |
|------------------------------ | ----------------------------------------------------------------------------|
|`math.sin _`                   | `x => math.sin(x)`                                                          |
|`math.pow _`                   | `(x1, x2) => math.pow(x1, x2)`                                              |
|`val vs = 1 to 9; vs.fold _`   | `(z) => (op) => vs.fold(z)(op)`                                             |
|`(1 to 9).fold(z)_`            | `{ val eta1 = z; val eta2 = 1 to 9; op => eta2.fold(eta1)(op) }`            |
|`Some(1).fold(??? : Int)_`     | `{ val eta1 = () => ???; val eta2 = Some(1); op => eta2.fold(eta1())(op) }` |

注意，方法名称和尾随下划线之间需要一个空格，否则下划线将被视为名称的一部分。

## 类型应用

```ebnf
SimpleExpr    ::=  SimpleExpr TypeArgs
```

_类型应用_`$e$[$T_1 , \ldots , T_n$]`实例化了具有类型`[$a_1$ >: $L_1$ <: $U_1, \ldots , a_n$ >: $L_n$ <: $U_n$]$S$`和参数类型`$T_1 , \ldots , T_n$`的多态$e$.每个参数类型$T_i$必须遵守相应的边界$L_i$和$U_i$。也就是说，对于每个$i = 1, \ldots , n$，我们必须有$\sigma L_i <: T_i <: \sigmaU_i$，其中$\sigma$是替换$[a_1 := T_1 , \ldots , a_n:= T_n]$。应用的类型是$\sigma S$。

如果函数部分$e$是某种值类型，该类型应用将等价于`$e$.apply[$T_1 , \ldots ,$ T$_n$]`,比如由$e$定义的`apply`方法应用。

如果[本地类型推断](#本地类型推断)可以通过实际函数参数类型和期望结果类型来得到一个多态函数的最佳类型参数，则类型应用可以忽略。

## 元组

```ebnf
SimpleExpr   ::=  ‘(’ [Exprs] ‘)’
```
_元组_ 表达式`($e_1 , \ldots , e_n$)`是类实例创建`scala.Tuple$n$($e_1 , \ldots , e_n$)`的别名，其中$n \geq 2$.空元组`()`是 `scala.Unit`类型的唯一值。

## 实例创建表达式

```ebnf
SimpleExpr     ::=  ‘new’ (ClassTemplate | TemplateBody)
```

一个_简单的实例创建表达式_的形式是`new $c$`，其中$c$是一个[构造函数调用](05-classes-and-objects.html#constructor-invocations)。设$T$是$c$的类型。那么$T$必须表示`scala.AnyRef`的一个非抽象子类(的类型实例)。更进一步，表达式的_具体的自我类型_必须与$T$表示的类型的[自我类型](05-classes-and-objects.html#templates)一致。具体的自我类型通常为$T$,除非表达式`new $c$`在值定义的右侧出现：

```scala
val $x$: $S$ = new $c$
```

(类型注释`: $S$`可以缺失)。在后一种情况下，表达式的具体自我类型是复合类型`$T$ with $x$.type`。

表达式通过创建一个$T$类型的新对象来求值，该对象通过计算$C$来初始化。表达式的类型为$T$。

对于某些[类的模板](05-classes-and-objects.html#templates)$t$,_通过实例创建表达式的形式为_`new $t$` 。这样的表达式等同于块。

```scala
{ class $a$ extends $t$; new $a$ }
```

其中$a$是用户程序无法访问的_匿名类_的新名称。

还有一种用于创建结构类型值的简写形式:如果`{$D$}`是一个类体，那么`new {$D$}` 等同于一般实例创建表达式`new AnyRef{$D$}`。

###### 例
请考虑一下结构实例创建表达式:

```scala
new { def getName() = "aaron" }
```

这是一般实例创建表达式的简写：

```scala
new AnyRef{ def getName() = "aaron" }
```

后者又是该块的简写：

```scala
{ class anon\$X extends AnyRef{ def getName() = "aaron" }; new anon\$X }
```

其中`anon\$X`是一些新创建的名称。

## 块

```ebnf
BlockExpr  ::=  ‘{’ CaseClauses ‘}’
             |  ‘{’ Block ‘}’
Block      ::=  BlockStat {semi BlockStat} [ResultExpr]
```

_块表达式_`{$s_1$; $\ldots$; $s_n$; $e\,$}`由一个块语句序列$s_1 , \ldots , s_n$和一个最终表达式构$e$构成。语句序列中不能有两个定义或声明绑定到同一命名空间的同一命名上。最终表达式可忽略，默认为单元值 `()` 。

最终表达式$e$的预期类型是块的预期类型。所有先前语句的预期类型都未定义。

块`$s_1$; $\ldots$; $s_n$; $e$`的类型是`$T$ forSome {$\,Q\,$}`，其中$T$是$e$的类型，$Q$包含在$T$中中的每个自由和在语句 $s_1 , \ldots , s_n$中局部定义值或类型命名的[既存类型](03-types.html#existential-types).我们说存在子句 _绑定_ 了值或类型命名，需要特别指出:

- 一个本地定义的类型定义 `type$\;t = T$`由存在子句`type$\;t >: T <: T$`绑定，如果$t$携带参数子句，则会出错。
- 本地定义的值定义`val$\;x: T = e$`由存在子句`val$\;x: T$`绑定。
- 本地定义的类定义`class$\;c$ extends$\;t$`由存在子句`type$\;c <: T$`绑定，其中$T$是最小类类型或修饰类型，且他是一个适当的超类型$c$。如果$c$带有参数，则会出错。
- 本地定义的`object$\;x\;$extends$\;t$`由存在子句`val$\;x: T$`绑定，其中$T$是最小类类型或修饰类型，他是`$x$.type`的适当超类型。

对块的评估需要评估其语句序列，然后评估最终表达式$e$,它定义了块的结果。

###### 例

假设一个类`Ref[T](x: T)`，代码块

```scala
{ class C extends B {$\ldots$} ; new Ref(new C) }
```
具有类型`Ref[_1] forSome { type _1 <: B }`，代码块

```scala
{ class C extends B {$\ldots$} ; new C }
```

的类型仅是`B`，因为[这里](03-types.html#simplification-rules)的规则有存在限定类型`_1 forSome { type _1 <: B }` 可以简化为`B`。

## 前缀，中缀和后缀运算

```ebnf
PostfixExpr     ::=  InfixExpr [id [nl]]
InfixExpr       ::=  PrefixExpr
                  |  InfixExpr id [nl] InfixExpr
PrefixExpr      ::=  [‘-’ | ‘+’ | ‘!’ | ‘~’] SimpleExpr
```
表达式由运算符和操作数构成。

### 前缀运算

前缀运算$\mathit{op};e$ 由前缀运算符$\mathit{op}$组成，它必须是标识符‘`+`’, ‘`-`’,
‘`!`’ 或 ‘`~`’之一。表达式 $\mathit{op};e$等同于后缀方法应用`e.unary_$\mathit{op}$`.

前缀运算符与普通方法应用程序的不同之处在于他们的操作数表达式不一定是原子的。例如，输入序列`-sin(x)`被解读为`-(sin(x))`，而方法应用程序`negate sin(x)`将被解析为应用中缀运算符 `sin`到操作数`negate` 和 `(x)`.

### 后缀运算

后缀运算符可以是任意标识符。后缀操作$e;\mathit{op}$被解读为$e.\mathit{op}$。

### 中缀运算

中缀运算符可以是任意标识符。中缀运算符的优先级和关联性定义如下：

中缀运算的_优先级_由运算符的第一个字符决定。下面按优先级的递增顺序列出字符，同一行上的字符具有相同的优先级。

```scala
(all letters)
|
^
&
= !
< >
:
+ -
* / %
(all other special characters)
```

也就是说，以字母开头的运算符具有最低优先级，然后就是以‘`|`’开头的运算符。

这条规则有一个例外，它涉及到[_赋值运算符_](#赋值运算符)。赋值运算符的优先级于简单赋值`(=)`的优先级相同。也就是说，它低于任何其他运算符的优先级。

运算符的_相关性_由运算符的最后一个字符确定。由‘`:`’结尾的运算符是右相关的。其他的所有运算符是左相关的。

运算符的优先级和关联性决定了表达式各部分的分组如下：

- 如果表达式中有多个中缀操作，则优先级较高的运算符比优先级较低的运算符绑定的更紧密。
- 如果有连续的中缀运算$e_0; \mathit{op}\_1; e_1; \mathit{op}\_2 \ldots \mathit{op}\_n; e_n$与运算符$\mathit{op}\_1 , \ldots , \mathit{op}\_n$具有相同的优先级，然后这些运算符必须具有相同的关联性。如果所有运算符都是左关联的，则序列被解释为$(\ldots(e_0;\mathit{op}_1;e_1);\mathit{op}_2\ldots);\mathit{op}_n;e_n$。或者，如果所有运算符都是右关联的，则序列被解释为$e_0;\mathit{op}_1;(e_1;\mathit{op}_2;(\ldots \mathit{op}_n;e_n)\ldots)$。
- 后缀运算符的优先级始终低于中缀运算符。例如， $e_1;\mathit{op}\_1;e_2;\mathit{op}\_2$ 始终等同于$(e_1;\mathit{op}\_1;e_2);\mathit{op}\_2$.

左关联运算符的右侧操作数可能包含在括号中的几个参数，例如$e;\mathit{op};(e_1,\ldots,e_n)$。然后将此表达式解释为$e.\mathit{op}(e_1,\ldots,e_n)$。

左关联二进制运算$e_1;\mathit{op};e_2$被解释为$e_1.\mathit{op}(e_2)$。如果$\mathit{op}$ is是右侧关联的并且其参数是按名称传递的，则相同的操作将会被解释为$e_2.\mathit{op}(e_1)$。如果$\mathit{op}$是右关联的，并且其参数是按值传递的，则他被解释为`{ val $x$=$e_1$; $e_2$.$\mathit{op}$($x\,$) }`，其中$x$是一个新成员。

### 赋值运算

_赋值运算_ 是一个运算符符号([标识符](01-lexical-syntax.html#identifiers)中的语法类别`op` )，以“`=`”结尾，但是其中包含一下条件之一的运算符除外：

1. 运算符以等号开头
1. 运算是其中之一 `(<=)`, `(>=)`, `(!=)`.

则赋值运算符可以特殊处理，如果没有其他有效解释，则扩展为赋值、

我们考虑一个赋值运算符，比如`+=`。在中缀运算`$l$ += $r$`中，$l$和$r$是表达式。该运算符可以重新解释为负责赋值的运算：

```scala
$l$ = $l$ + $r$
```

除了操作的左侧$l$仅被计算一次。

如果满足以下俩个条件，则重新计算：
1. 左侧的$l$没有名为`+=`的成员，也无法通过[隐式转换](#隐式转换)转换为名为`+=`的成员变量。
1. 赋值运算`$l$ = $l$ + $r$`的类型是正确的。特别此处暗含了$l$应用了一个变量或对象，且该变量或对象可以赋值，且可转变为一个具有名为`+`的成员的值。

## 类型化的表达式

```ebnf
Expr1              ::=  PostfixExpr ‘:’ CompoundType
```

_类型化的表达式_ $e: T$的类型为$T$。表达式$e$期望符合$T$.表达式的结果是$e$的值转换为$T$类型。

###### 例

以下是良好类型和不良类型表达式的事例：

```scala
1: Int               // 合法的Int类型。
1: Long              // 合法的Long类型。
// 1: string         // ***** 非法
```

## 标注表达式

```ebnf
Expr1              ::=  PostfixExpr ‘:’ Annotation {Annotation}
```
_标注表达式_`$e$: @$a_1$ $\ldots$ @$a_n$`将[标注](11-annotations.html#user-defined-annotations)$a_1 , \ldots , a_n$附加到表达式$e$。

## 赋值

```ebnf
Expr1        ::=  [SimpleExpr ‘.’] id ‘=’ Expr
               |  SimpleExpr1 ArgumentExprs ‘=’ Expr
```

对简单变量`$x$ = $e$`的赋值的解释取决于$x$的定义。如果$x$表示一个可变变量，然后赋值将$x$的当前值更改为计算表达式$e$的结果。$e$的类型被期望与$x$的类型一致。如果$x$是在某个模板中定义的无参方法，并且该模板包含setter方法`$x$_=` 作为成员，则赋值`$x$ = $e$` 被解释为调用`$x$_=($e\,$)`这个setter方法。类似的，无参函数$x$的赋值`$f.x$ = $e$`被解释为调用`$f.x$_=($e\,$)`。

使用‘`=`’运算符左侧的方法应用程序赋值`$f$($\mathit{args}\,$) = $e$`被解释为`$f.$update($\mathit{args}$, $e\,$)`，即调用$f$调用的`update`方法。

###### 例
这是一些赋值表达式及其等效扩展：

| 赋值                     | 扩展                  |
|--------------------------|----------------------|
|`x.f = e`                 | `x.f_=(e)`           |
|`x.f() = e`               | `x.f.update(e)`      |
|`x.f(i) = e`              | `x.f.update(i, e)`   |
|`x.f(i, j) = e`           | `x.f.update(i, j, e)`|

###### 示例矩阵乘法代码

这是矩阵乘法的常用命令代码

```scala
def matmul(xss: Array[Array[Double]], yss: Array[Array[Double]]) = {
  val zss: Array[Array[Double]] = new Array(xss.length, yss(0).length)
  var i = 0
  while (i < xss.length) {
    var j = 0
    while (j < yss(0).length) {
      var acc = 0.0
      var k = 0
      while (k < yss.length) {
        acc = acc + xss(i)(k) * yss(k)(j)
        k += 1
      }
      zss(i)(j) = acc
      j += 1
    }
    i += 1
  }
  zss
}
```

去掉数据访问和赋值的语法糖，就是下面这个扩展版本：

```scala
def matmul(xss: Array[Array[Double]], yss: Array[Array[Double]]) = {
  val zss: Array[Array[Double]] = new Array(xss.length, yss.apply(0).length)
  var i = 0
  while (i < xss.length) {
    var j = 0
    while (j < yss.apply(0).length) {
      var acc = 0.0
      var k = 0
      while (k < yss.length) {
        acc = acc + xss.apply(i).apply(k) * yss.apply(k).apply(j)
        k += 1
      }
      zss.apply(i).update(j, acc)
      j += 1
    }
    i += 1
  }
  zss
}
```

## 条件表达式

```ebnf
Expr1          ::=  ‘if’ ‘(’ Expr ‘)’ {nl} Expr [[semi] ‘else’ Expr]
```

_条件表达式_ `if ($e_1$) $e_2$ else $e_3$`根据$e_1$的值来选择值$e_2$或$e_3$。条件$e_1$期望与类型 `Boolean`一致。Then部分$e_2$和else部分$e_3$都期望与条件表达式的期望一致。条件表达式的类型是$e_2$和$e_3$类型[的弱最小上限](03-types.html#weak-conformance)。j将忽略条件表达式`else`字符前的分号。

通过先计算$e_1$来计算条件表达式。如果此计算结果为`true`,则返回计算$e_2$的结果，否则则计算$e_3$的结果。

条件表达式的简短形式消除了else部分。条件表达式`if ($e_1$) $e_2$` 求值的方法为`if ($e_1$) $e_2$ else ()`.

## While循环表达式

```ebnf
Expr1          ::=  ‘while’ ‘(’ Expr ‘)’ {nl} Expr
```

_While循环表达式_ `while ($e_1$) $e_2$`的类型化求值方式类似于函数`whileLoop ($e_1$) ($e_2$)`的应用，假定函数的 `whileLoop`定义如下：

```scala
def whileLoop(cond: => Boolean)(body: => Unit): Unit  =
  if (cond) { body ; whileLoop(cond)(body) } else {}
```

## Do循环表达式

```ebnf
Expr1          ::=  ‘do’ Expr [semi] ‘while’ ‘(’ Expr ‘)’
```

_do循环表达式_`do $e_1$ while ($e_2$)` 执行的输入和计算和表达式`($e_1$ ; while ($e_2$) $e_1$)`一样。do循表达式的`while`前面的分号将被忽略。


## for循环

```ebnf
Expr1          ::=  ‘for’ (‘(’ Enumerators ‘)’ | ‘{’ Enumerators ‘}’)
                       {nl} [‘yield’] Expr
Enumerators    ::=  Generator {semi Generator}
Generator      ::=  Pattern1 ‘<-’ Expr {[semi] Guard | semi Pattern1 ‘=’ Expr}
Guard          ::=  ‘if’ PostfixExpr
```

_for循环_ `for ($\mathit{enums}\,$) $e$`对于由枚举类型$\mathit{enums}$产生的每个绑定执行表达式$e$。_for comprehension_ `for ($\mathit{enums}\,$) yield $e$` 为枚举类型
 $\mathit{enums}$生成的每个绑定计算表达式$e$的值并搜集结果。 枚举类型序列始终以生成类型开始；后面可接其它生成器，值定义或守卫。_生成器_ `$p$ <- $e$`程表达式$e$生成绑定，它以某种方式与模式$p$匹配。_值定义_ `$p$ = $e$`将值名称$p$(或模式$p$中的多个名称)绑定到计算表达式$e$的求值结果上。_守卫_`if $e$`$e$包涵一个限制枚举类绑定的布尔表达式。生成器和防护的精确含义是通过转换为四种方法的调用来定义的:`map`,`withFilter`, `flatMap`, 和 `foreach`。这些方法可以根据不同的携带类型具有不同的实现。

翻译框架如下。在第一步中，每个生成器 `$p$ <- $e$`，对于$e$的类型被替换成如下形式，$p$不是[不可反驳的](08-pattern-matching.html#patterns)。


```scala
$p$ <- $e$.withFilter { case $p$ => true; case _ => false }aaaaaaaaaaaa
```

然后，重复应用如下规则，直到所有的语段被消耗完毕为止：

  - for comprehension `for ($p$ <- $e\,$) yield $e'$`被翻译为`$e$.map { case $p$ => $e'$ }`。
  - for 循环
    `for ($p$ <- $e\,$) $e'$`
    被翻译为
    `$e$.foreach { case $p$ => $e'$ }`.
  - for comprehension

    ```scala
    for ($p$ <- $e$; $p'$ <- $e'; \ldots$) yield $e''$
    ```
    其中`$\ldots$` 是一个(可能为空的)生成器，定义或守卫序列，被翻译为

    ```scala
    $e$.flatMap { case $p$ => for ($p'$ <- $e'; \ldots$) yield $e''$ }
    ```

  - for循环

    ```scala
    for ($p$ <- $e$; $p'$ <- $e'; \ldots$) $e''$
    ```

    其中`$\ldots$` 是一个(可能为空的)生成器，定义或守卫序列，被翻译为

    ```scala
    $e$.foreach { case $p$ => for ($p'$ <- $e'; \ldots$) $e''$ }
    ```

  - 后跟守卫`if $g$`的生成器`$p$ <- $e$`翻译成单个生成器 `$p$ <- $e$.withFilter(($x_1 , \ldots , x_n$) => $g\,$)`，其中$x_1 , \ldots , x_n$是$p$的自由变量。
  - 后跟值定义的`$p'$ = $e'$`的产生器`$p$ <- $e$`翻译为一下值的生成器，这里的  $x$ 和 $x'$是新名称：

    ```scala
    ($p$, $p'$) <- for ($x @ p$ <- $e$) yield { val $x' @ p'$ = $e'$; ($x$, $x'$) }
    ```'

###### 例

以下代码参数$1$和$n-1$之间的所有和为素数的数值对
```scala
for  { i <- 1 until n
       j <- 1 until i
       if isPrime(i+j)
} yield (i, j)
```

该for comprehension 翻译为

```scala
(1 until n)
  .flatMap {
     case i => (1 until i)
       .withFilter { j => isPrime(i+j) }
       .map { case j => (i, j) } }
```

###### 例
For comprehensions可以用来简洁的表达矢量和矩阵算法。例如，下列是一种计算给定矩阵的转置的方法：

```scala
def transpose[A](xss: Array[Array[A]]) = {
  for (i <- Array.range(0, xss(0).length)) yield
    for (xs <- xss) yield xs(i)
}
```
这是一种计算两个向量的无向量积的方法：

```scala
def scalprod(xs: Array[Double], ys: Array[Double]) = {
  var acc = 0.0
  for ((x, y) <- xs zip ys) acc = acc + x * y
  acc
}
```

最后，这是一个计算两个矩阵的乘积的方法，与[命令版式](#示例矩阵乘法代码)比较


```scala
def matmul(xss: Array[Array[Double]], yss: Array[Array[Double]]) = {
  val ysst = transpose(yss)
  for (xs <- xss) yield
    for (yst <- ysst) yield
      scalprod(xs, yst)
}
```
上面的代码使用了`scala.Array`中以定义的成员`map`, `flatMap`,`withFilter`, 和 `foreach`为类

## Return表达式

```ebnf
Expr1      ::=  ‘return’ [Expr]
```

_return表达式_`return $e$`必须出现在闭合的用户方法的主体内。源程序中最内层的封闭方法$m$必须具有显式声明的结果类型，并且$e$的类型必须符合它。

返回表达式计算表达式$e$并返回其值作为$m$的结果。对return表达式之后的语句或表达式将会被忽略求值。return表达式的类型是`scala.Nothing`。

表达式$e$可以省略。`return`以`return ()`的形式左类型检查和求值。

从嵌套函数内的方法返回可以`scala.runtime.NonLocalReturnException`来实现抛出和捕获。返回点和封闭方法之间的任何异常捕获都可能看到并捕获该异常。key确保此异常仅由返回终止的方法类型捕获。

如果return表达式本身就是匿名函数的一部分，则在执行返回表达式之前，封闭方法$m$可能已经返回。在这种情况下，抛出的`scala.runtime.NonLocalReturnException`将不会被捕获，并将向上传播调用堆栈。

## Throw表达式

```ebnf
Expr1      ::=  ‘throw’ Expr
```

_throw表达式_ `throw $e$`计算表达式$e$。次表达式的类型必须符合`Throwable`。如果$e$的计算结果为异常引用，则将会停止计算，抛出异常。如果$e$的计算值为`null`，则终止计算并抛出`NullPointerException`.如果此处有一个活动的[`try`表达式](#try表达式)，且要处理抛出的异常，则求值在该处理中继续执行；否则执行`throw`的线程将被终止。`throw` 表达式的类型是`scala.Nothing`。

## Try表达式

```ebnf
Expr1 ::=  ‘try’ (‘{’ Block ‘}’ | Expr) [‘catch’ ‘{’ CaseClauses ‘}’]
           [‘finally’ Expr]
```
_try表达式_ 的形式为`try { $b$ } catch $h$`，其中处理程序$h$是是一个[匹配匿名函数的模式](08-pattern-matching.html#pattern-matching-anonymous-functions)。

```scala
{ case $p_1$ => $b_1$ $\ldots$ case $p_n$ => $b_n$ }
```
通过计算块$b$来计算表达式。如果$b$的计算不会导致抛出异常，则返回$b$的结果。否则，处理程序$h$将应用于异常的抛出。如果处理程序包含一个case与抛出的异常匹配，则调用第一个该类case。如果没有与抛出的异常匹配的case，则异常被重新抛出。

设$\mathit{pt}$是try表达式的期望类型。代码块$b$的预期符合$\mathit{pt}$。处理程序$h$预计符合`scala.PartialFunction[scala.Throwable, $\mathit{pt}\,$]`类型。try表达还是的类型是$b$类型和$h$的结果类型的[弱最小上限](03-types.html#weak-conformance)。

try表达式 `try { $b$ } finally $e$`计算块$b$。如果$b$的评估不会导致异常的抛出，则会计算表达式$e$。如果在计算$e$的期间抛出异常，则try表达式的求值终止且抛出异常。如果在对$e$求值的期间没有异常抛出，则返回$b$的结果作为try表达式的结果。

如果在计算$b$的期间抛出异常，则还会计算finally代码块$e$。如果在计算$e$期间抛出另外一个异常$e$，则抛出异常并且终止try表达式的计算。如果在计算$e$的期间没有抛出异常，则一旦$e$的计算完成，就会重新抛出$b$中抛出的原有异常。代码块$b$应该符合try表达式的预期类型。最终表达式$e$预计符合类型`Unit`。

try表达式`try { $b$ } catch $e_1$ finally $e_2$`是`try { try { $b$ } catch $e_1$ } finally $e_2$`的简写。


## 匿名函数

```ebnf
Expr            ::=  (Bindings | [‘implicit’] id | ‘_’) ‘=>’ Expr
ResultExpr      ::=  (Bindings | ([‘implicit’] id | ‘_’) ‘:’ CompoundType) ‘=>’ Block
Bindings        ::=  ‘(’ Binding {‘,’ Binding} ‘)’
Binding         ::=  (id | ‘_’) [‘:’ Type]
```

arity的匿名函数$n$，`($x_1$: $T_1 , \ldots , x_n$: $T_n$) => e`将类型$T_i$的参数$x_i$映射为由表达式$e$给出的结果。每个形式参数$x_i$的范围是$e$。形式参数必须具有两两不同的名称。

在具有单个未类型化的正式参数时，`($x\,$) => $e$`可以缩写为`$x$ => $e$`。如果匿名函数`($x$: $T\,$) => $e$`有单个类型化的参数作为一个代码块的结果表达式出现，则可以缩写为`$x$: $T$ => e`.

形式参数也可以是由下划线`_`表达式的通配符。在这种情况下，可以任意选择参数的一个新名称。

匿名函数的命名参数可以选择在前面加上`implicit`修饰符。在这种情况下，参数被标记为[`implicit`](07-implicits.html#implicit-parameters-and-views)。但是参数部分本身不算作[implicit 参数部分](07-implicits.html#implicit-parameters)。因此，必须明确给出匿名函数的参数。

### 解答

如果匿名函数的期望类型具有`scala.Function$n$[$S_1 , \ldots , S_n$, $R\,$]`的形式，或则[SAM 转换](#SAM转换)为这样的函数的函数类型，则$e$的期望类型为$R$,每个参数`$x_i$`的类型`$T_i$`可以忽略，可假定 `$T_i$ = $S_i$`。

如果匿名函数的期望类不存在，则所有正式参数的类型`$T_i$`必须明确指定，$e$的预期类型是未定义的。匿名函数的类型是 `scala.Function$n$[$T_1 , \ldots , T_n$, $R\,$]`，其中$R$是$e$的[打包类型](#表达式类型)。$R$必须等同于不引用任何形式参数$x_i$的类型。

匿名函数的最终运行时值由期望类型决定：

  - 一个内置函数类型的子类， `scala.Function$n$[$S_1 , \ldots , S_n$, $R\,$]` (完全定义了$S_i$ 和 $R$ ),
  - 一个[抽象方法(SAM)类型](#SAM转换)
  - `PartialFunction[$T$, $U$]`,如果函数形如`x => x match { $\ldots$ }`
  - 其他一些类型

标准匿名函数的计算方法与胰腺癌实例创建表达式的计算方法相同：
```scala
new scala.Function$n$[$T_1 , \ldots , T_n$, $T$] {
  def apply($x_1$: $T_1 , \ldots , x_n$: $T_n$): $T$ = $e$
}
```

同样的计算适用于SAM类型，除了实例化类型由SAM类型给出，并且实现的方法是此类型的单个抽象成员方法。

底层平台可以提供构建这些实例的更有效的方法，例如java8的`invokedynamic`字节码和 `LambdaMetaFactory`类。

`PartialFunction`的值接收一个额外的`isDefinedAt`成员，该成员派生自函数文字中的匹配模式。每个case的主体被替换为`true`，并且添加的默认模式(如果没有给出)计算结果为`false`。

###### 例
匿名函数的示例

```scala
x => x                             // 恒等函数

f => g => x => f(g(x))             // 柯里化的函数组合

(x: Int,y: Int) => x + y           // 求和函数

() => { count += 1; count }        // 该函数的列表为 $()$,
                                   // 递增非局部变量`count`并返回新值。

_ => 5                             // 该函数忽略楼其他参数，总是返回5
```

### 匿名函数的占位符语法

```ebnf
SimpleExpr1  ::=  ‘_’
```

表达式（语法类别为`Expr`）可以在合法的标识符位置包含嵌入的下划线符号`_`。这样的表达式表示一个下划线的后续出现表示连续的参数的匿名函数，。

将 _下划线部分_ 定义为`_:$T$`形式的表达式，其中$T$是一个类型，或者形式为`_`.下划线并不是类型归属`_:$T$`的表达式部分。

如果以下两个条件成立，则句法类别是`Expr`的表达式$e$_绑定_ 下划线部分$U$:(1)$e$合理包含$u$。(2)没有其他的具有`Expr`句法归类的表达式合理包含于$e$且自身合理包含$u$.

如果表达式$e$按照既定顺序绑定道下划线段$u_1 , \ldots , u_n$，则等价与匿名函数`($u'_1$, ... $u'_n$) => $e'$`，每个$u_i'$是将$u_i$中下划线替换为新的标识符的结果，$e'$则是将$e$中的每个下划线段$u_i$ 替换为 $u_i'$的结果。

###### 例
左列中的匿名函数使用占位符语法。 这些中的每一个都相当于其右侧的匿名函数。

| | |
|---------------------------|----------------------------|
|`_ + 1`                    | `x => x + 1`               |
|`_ * _`                    | `(x1, x2) => x1 * x2`      |
|`(_: Int) * 2`             | `(x: Int) => (x: Int) * 2` |
|`if (_) x else y`          | `z => if (z) x else y`     |
|`_.map(f)`                 | `x => x.map(f)`            |
|`_.map(_ + 1)`             | `x => x.map(y => y + 1)`   |

## 常数表达式

常量表达式是Scala编译器可以计算为常量的表达式。 “常量表达式”的定义取决于平台，但它们至少包括以下形式的表达式：
- 值类的文字，例如整数
- 字符串文字
- 用[`Predef.classOf`](12-the-scala-standard-library.html#the-predef-object)构造的类
- 来自底层平台的枚举元素
- 一个文字数组，格式为`Array$(c_1 , \ldots , c_n)$`,其中所有 $c_i$本身都是常量表达式
- [由常量值定义](04-basic-declarations-and-definitions.html#value-declarations-and-definitions)的定义标识符。.

## 语法

```ebnf
BlockStat    ::=  Import
               |  {Annotation} [‘implicit’] [‘lazy’] Def
               |  {Annotation} {LocalModifier} TmplDef
               |  Expr1
               |
TemplateStat ::=  Import
               |  {Annotation} {Modifier} Def
               |  {Annotation} {Modifier} Dcl
               |  Expr
               |
```

语句作为块和模板的一部分出现。_语句_ 可以是import，定义或表达式，也可以是空的。 类定义模板中使用的语句也可以是声明。用作语句的表达式可以具有任意值类型。表达式语句$e$的求值是对表达哈斯$e$求值然后丢弃求值的结果。

代码块语句可以是在代码块中绑定本地命名的定义。代码块本地定义中允许的修饰符是`implicit`。
在为类或对象定义定义添加前缀是，还允许使用`abstract`, `final`和 `sealed`。

对语句序列的计算需要按照它们的编写顺序计算语句。

## 隐含转换

隐式转换可以应用于类型与其预期类型不匹配的表达式，选择中的限定符以及未应用的方法。可用的隐式转换在接下来的两个子部分中给出。

### 值转换

一下七个隐式转换可以应用于表达式$e$,他具有一些值类型$T$,并且使用某些预期类型$\mathit{pt}$进行类型计算。

###### 静态重载解析
如果表达式表示类的几个可能成员，则应用[重载解析](#重载解析)来选择唯一成员。

###### 类型实例化
表达式$e$的多态类型

```scala
[$a_1$ >: $L_1$ <: $U_1 , \ldots , a_n$ >: $L_n$ <: $U_n$]$T$
```
并不作为类型应用的函数部分，根据类型变量`$T_1 , \ldots , T_n$`通过[本地类型推断](#本地类型推断)来确定实例类型`$T_1 , \ldots , T_n$`，并隐式将$e$嵌入[类型应用](#类型应用)`$e$[$T_1 , \ldots , T_n$]`的方式转换为$T$的类型实例。

###### 数字扩张

如果$e$具有[弱符合](03-types.html#weak-conformance)预期类型的原始数字类型，则使用[定义](12-the-scala-standard-library.html#numeric-value-types).的`toShort`, `toChar`, `toInt`, `toLong`,`toFloat`, `toDouble` 转换方法之一将其扩展为预期类型。

###### 数字字面值缩减

如果期望的类型是 `Byte`, `Short` 或 `Char`，并且表达式$e$是符合该类型范围的整数文字，则它将转换为该类型中的同样的字面值。

###### 值丢弃

如果$e$具有某种值类型且期望类型为 `Unit`，则将$e$转换为期望类型，方法是将其嵌入到术语`{ $e$; () }`.

###### SAM转换

如果满足以下条件，则函数类型`(T1, ..., TN) => T`的表达式`(p1, ..., pN) => body`可以转换为期望的类型`S`.
  - `S`的`C`类声明一个带有签名`(p1: A1, ..., pN: AN): R`的抽象方法`m`;
  - 除了`m`, `C` 不得声明或继承任何其他递延价值成员;
  - 方法`m`必须有一个参数列表;
  - 必须有一个类型 `U` 是 `S`的子类型, 所以表达式
    `new U { final def m(p1: A1, ..., pN: AN): R = body }` 是良好类型 (符合预期的类型`S`);
  - 为了确定范围，`m`应该被认为是一个静态成员（`U`的成员不在体内范围内）;
  - `(A1, ..., AN) => R`是`(T1, ..., TN) => T` 的子类型（在`S`中，满足这一条件会驱动未知类型参数的类型推断）;

请注意，针对一个SAM函数文本不一定编译上述实例创建表达式。 这与平台有关。

它遵循：
  - 如果`C`类定义了一个构造函数，它必须是可访问的，并且必须只定义一个空的参数列表;
  - 类 `C` 不是 `final` 或 `sealed` (为简单起见，我们忽略了在与密封类相同的编译单元中进行SAM转换的可能性);
  - `m` 不是多态的;
  - 必须可以通过推断`C`的任何未知类型参数从`S`派生完全定义的类型`U`.

最后，我们施加了一些实施限制（这些可能会在以后的版本中解除）：
  - `C` 不能嵌套或本地（它不能捕获其环境，因为它会导致非零参数构造函数）
  - `C` 的构造函数不能有隐式参数列表（这简化了类型推断）;
  - `C` 不能声明自我类型（这简化了类型推断）
  - `C` 必须不是`@specialized`.

###### 视图应用
如果以前的转换都不适用，并且$e$的类型不符合预期类型$\mathit{pt}$,则会尝试将$e$转换为带有[视图](07-implicits.html#views)的预期类型。

###### `Dynamic`选择
如果以前的转换都不适用，且$e$是选择器$e.x$的前缀，并且$e$的类型符合`scala.Dynamic`，则根据[动态成员选择](#动态成员选择)的规则重写选择。

### 方法转换

以下四个隐式转换可以应用于无法应用于某些参数列表的方法。

###### 求值
具有类型 `=> $T$` 的无参方法$m$总是通过对$m$绑定的表达式求值来转换道类型$T$.

###### 隐式应用
如果该方法仅采用隐式参数，则遵循[此处](07-implicits.html#implicit-parameters)规则传递隐式参数。

###### eta扩展
否则，如果方法不是构造函数，并且期望类型$\mathit{pt}$是函数类型$(\mathit{Ts}') \Rightarrow T'$，则对表达式执行[eta1扩展](#eta扩展)。

###### 空应用
否则，如果$e$具有方法类型$()T$，则它隐式应用于空参数列表，产生$e()$。

### 重载解析

如果一个标识符或选择$ e$引用了类的多个成员，则将使用引用的上下文来推断唯一的成员。使用的方法将依赖与$e$是否被用作一个函数，设$\mathscr{A}$是$e$引用的成员的集合。

先假设$e$在应用程序中显示为函数，如`$e$($e_1 , \ldots , e_m$)`.

首先根据参数的 _模型_ 确定可能[适用](#函数的应用)的函数集。

参数表达式$e$的 *模型*，写成 $\mathit{shape}(e)$，是一种定义如下的类型：

  - 对于函数表达式 `($p_1$: $T_1 , \ldots , p_n$: $T_n$) => $b$: (Any $, \ldots ,$ Any) => $\mathit{shape}(b)$`,
    其中`Any` 在参数类型中出现了$n$次.
  - 对于模式匹配的匿名函数定义`{ case ... }`: `PartialFunction[Any, Nothing]`.
  - 对于命名参数`$n$ = $e$`: $\mathit{shape}(e)$.
  - 对于所有其他表达式： `Nothing`.

让 $\mathscr{B}$ 成为 $\mathscr{A}$ 中 [_适用_](#函数的应用) 于表达式$(e_1 , \ldots , e_n)$ 类型为 $(\mathit{shape}(e_1) , \ldots , \mathit{shape}(e_n))$.
如果 $\mathscr{B}$中只有一个替代选项，则选择该替代方案。

否则，让$S_1 , \ldots , S_m$成为通过键入每个参数获得的类型列表，如下所示。

通常，除非尝试传播更多类型信息以帮助推断更高阶函数参数类型，否则将在没有期望类型的情况下键入参数，如下所述。直观上，所有参数必须是一个函数样型（`PartialFunction`, `FunctionN`或一些相应的[SAM型](#SAM转换)），而这又必须定义相同的一组更高阶的参数类型的，使得它们可以安全地被用作预期重载方法的给定参数的类型，不过度排除任何替代方案。 目的不是引导重载解析，而是保留足够的类型信息来引导参数的类型推断（函数文字或eta扩展方法）到这个重载方法。

注意，预期的类型的驱动器ETA-
扩张（未除非函数样型预期执行），以及省略参数类型函数文本的推理。

更确切地说，一个参数`$e_i$`是用从每个替代中找到的第 `$i$`个参数类型派生的期望类型键入的（将`$T_ {ij}$`称为代替`$j$`和参数位置`$i$`) 当所有`$T_{ij}$`都是某个arity`$n$`的函数类型`$(A_{1j},..., A_{nj}) => ?$`（或等效的`PartialFunction`或SAM）时，它们的参数类型`$A_{kj}$` 在给定`$k$`的所有重载 `$j$`中是相同的。

`$k$`. 最后`e_i$` 的预期类型派生如下：
   - 我们使用 `$PartialFunction[A_{1j},..., A_{nj}, ?]$` 如果对于某些重载 `$j$`, `$T_{ij}$`的类型符号是 `PartialFunction`;
   - 或者，如果对于某些 `$j$`, `$T_{ij}$` 是 `FunctionN`, 则预期的类型是`$FunctionN[A_{1j},..., A_{nj}, ?]$`;
   - 或则, 如果对于所有`$j$`, `$T_{ij}$` 是同一类的SAM类型，定义参数类型 `$A_{1j},..., A_{nj}$`(以及可能变化的结果类型),期望的类型编码这些参数类型和SAM类。

对于$\mathscr{B}$中的每个成员$m$确定它是否适用$S_1, \ldots , S_m$类型的表达式（$e_1 , \ldots , e_m$）。

如果$\mathscr{B}$中没有成员适用，则会出错。 如果有一个单一的适用替代方案，则选择该替代方案。否则，让$\mathscr{CC}$成为适用的替代方案的集合，这些替代方案不会在应用程序中使用任何默认参数$e_1 , \ldots , e_m$.

如果$\mathscr{CC}$为空，则再次出错。否则，在$\mathscr{CC}$中选择_最具体的_替代方案，根据以下定义为“同样具体”，“更具体”：
- 参数话方法$m$类型`($p_1:T_1, \ldots , p_n:T_n$)$U$` _特用于_ 某些其他成员 $m'$类型$S$,如果$m'$[适用](#功能应用)于参数 `($p_1 , \ldots , p_n$)` 类型$T_1 , \ldots , T_n$。
- 类型`[$a_1$ >: $L_1$ <: $U_1 , \ldots , a_n$ >: $L_n$ <: $U_n$]$T$` 的多态方法与$S$类型类型的其他成员一样具体，如果$T$与$S$一样具体，假设对于$i = 1 , \ldots , n$每个$a_i$是一个抽象类型名称，由下面的$L_i$和上面的$U_i$界定。
- 任何其他类型的成员始终与参数化方法或多态方法一样具体。
- 给定$T$和$U$类型的两个成员既不是参数化也不是多态方法类型，$T$类型的成员与$U$类型的成员一样具体，如果$T$的存在对偶符合 $U$的存在对偶。这里，多态类型  `[$a_1$ >: $L_1$ <: $U_1 , \ldots , a_n$ >: $L_n$ <: $U_n$]$T$` 的存在对偶是  `$T$ forSome { type $a_1$ >: $L_1$ <: $U_1$ $, \ldots ,$ type $a_n$ >: $L_n$ <: $U_n$}`。每种其他类型的存在对偶都是类型本身。

另一个$A$代替$B$的_相对权重_是从0到2的数字，定义为的总和

- 如果$A$特定于$B$，则为1，否则为0
- 如果$A$定义在一个类或对象中，该类或对象派生自定义$B$的类或对象,则为1，否则为0。

如果以下之一成立，则类或对象$C$_派生_ 自类或对象$D$：

- $C$ 是 $D$的子类，
- $C$ 是从 $D$派生的类的伴随对象
- $D$ 是一个类的伴随对象，从中派生出 $C$ .

如果$A$相对于$B$的相对权重大于$B$相对于$A$的相对权重，则替代$A$比替代$B$_更具体_。

如果$\mathscr{CC}$中没有替代方法，那么这是一个错误，它比$\mathscr{CC}$中的所有其他选项更具体。

接下来假设$e$在类型应用程序中显示为函数，如 `$e$[$\mathit{targs}\,$]`。然后选择$\mathscr{A}$中的所有替代项，它们采用与$\mathit{targs}$中的类型参数相同数量的类型参数。如果不存在这样的替代方案则是错误的。 如果有几个这样的替代方案，重载解析将再次应用于整个表达式 `$e$[$\mathit{targs}\,$]`。


最后我们假定$e$没有在应用或类型应用中做为函数出现。如果给出了期望类型,设是$\mathscr{A}$中与其[兼容](03-types.html#compatibility)的该类可选项的集合。否则,设 B 为 A。在此情况下我们在 B 的所有可选项中选择最具体的可选项。如果 B 中没有可选项比 B 中其他所有的可选项更具体则将导致错误。



最后我们假定$e$没有在应用或类型应用中做为函数出现。如果给出了期望的类型，那么让$\mathscr{B}$成为$\mathscr{A}$中与之[兼容](03-types.html#compatibility)的那些备选方案的集合。否则，让$\mathscr{B}$与$\mathscr{A}$相同。在此情况下我们在$\mathscr{B}$的所有可选项中选择最具体的可选项。 如果$\mathscr{B}$中没有可选项比$\mathscr{B}$中其他所有的可选项更具体则将导致错误。


###### 例
考虑以下定义：

```scala
class A extends B {}
def f(x: B, y: B) = $\ldots$
def f(x: A, y: B) = $\ldots$
val a: A
val b: B
```
则应用`f(b, b)`指向$f$的第一个定义，应用`f(a, a)`指向第二个，假设我们添加了第三个重载定义


```scala
def f(x: B, y: A) = $\ldots$
```

则应用程序`f(a, a)` 因模糊定义而被拒绝，因为不存在更具体的可用签名。

### 本地类型推断
地类型推断推断要传递给多态类型表达式的类型参数。假设$e$的类型为[$a_1$ >: $L_1$ <: $U_1, \ldots , a_n$ >: $L_n$ <: $U_n$]$T$并且没有显示类型的参数给出。

本地类型推断将此表达式转换为类型应用程序`$e$[$T_1 , \ldots , T_n$]`。类型参数 $T_1 , \ldots , T_n$的选择取决于表达式出现的上下文以及期望类型$\mathit{pt}$. 有三种情况。


###### 第一种情况：选择
如果表达式显示为名称为$x$的命名的前缀出现，则类型推断将_延迟_到整个表达式$e.x$。也就是说，如果$ ex $的类型为$S$,且现在处理的形式有类型[$a_1$ >: $L_1$ <: $U_1 , \ldots , a_n$ >: $L_n$ <: $U_n$]$S$,且本地类型推断应用到在$e.x$出现处的上下文中推断类型参量$a_1 , \ldots , a_n$。

###### 第二种情况：值

如果表达式$e$显示为值而未应用于值参数，则通过求解约束系统来推断类型参数，该约束系统将表达式的类型$T$与期望类型$\mathit{pt}$相关联。不失一般性，我们可以假定$T$是一种值类型; 如果它是方法类型，我们应用[eta扩展](#eta扩展)将其转换为函数类型。求解意味着为类型参数$a_i$的一个类型$T_i$的替换$\sigma$ ，这样

- 推断类型$T_i$都不是[单例类型](03-types.html#singleton-types)，除非它是对应于对象或常量值定义的单例类型或相应的绑定$U_i$是`scala.Singleton`的子类型。
- 尊重所有类型参数边界都,既$\sigma L_i <: \sigma a_i$ 和 $\sigma a_i <: \sigma U_i$ for $i = 1 , \ldots , n$.
- 表达式的类型符合预期的类型，即$\sigma T <: \sigma \mathit{pt}$.

如果不存在这样的替换，则编译时会出现错误。如果存在多个替换，则本地类型推断将为每个类型变量$a_i$选择解决方案空间的最小或最大类型$T_i$。如果类型参数$a_i$在表达式的$T$类型中出现[逆变](04-basic-declarations-and-definitions.html#variance-annotations)，则将选择_最大类型_$T_i$。 在所有其他情况下，将选择_最小类型_$T_i$，即，如果变量在$T$类型中出现共变，非变化或根本不出现。 我们称这种替换是$T$类型的给定约束系统的_最优解_。

###### 第三种情况： 方法
如果表达式$e$出现在应用程序$e(d_1 , \ldots , d_m)$中，则最后一种情况适用。在这种情况下，$T$是一种方法类型$(p_1:R_1 , \ldots , p_m:R_m)T'$。 在不失一般性的情况下，我们可以假设结果类型$T'$是一个值类型; 如果它是方法类型，我们应用[eta-扩张](#eta扩张)将其转换为函数类型。首先使用两个替代方案计算参数表达式$d_j$的$S_j$类型。首先使用期望类型$R_j$键入每个参数表达式$d_j$，其中类型参数$a_1 , \ldots , a_n$被视为类型常量。如果失败，则通过替换$a_1 , \ldots , a_n$中_未定义_的每个类型参数来输入参数$d_j$而不是预期类型$R_j'$，这是$R_j$的结果。

在第二步中，通过求解约束系统来推断类型参数，该约束系统将方法的类型与期望类型$\mathit{pt}$和参数类型$S_1 , \ldots , S_m$相关联。 解决约束系统意味着为类型参数$a_i$找到类型$T_i$的替换$\sigma$，这样

- 推断类型$T_i$都不是[单例类型](03-types.html#singleton-types)，除非它是对应于对象或常量值定义的单例类型或相应的绑定$U_i$是 `scala.Singleton`的子类型。
- 所有类型参数边界都被尊重,即 $\sigma L_i <: \sigma a_i$ 且
  $\sigma a_i <: \sigma U_i$ for $i = 1 , \ldots , n$.
- 方法的结果类型$T'$符合预期的类型，即 $\sigma T' <: \sigma \mathit{pt}$.
- 每个参数类型[弱符合](03-types.html#weak-conformance)相应的形式参数类型。即$\sigma S_j <:_w \sigma R_j$ for $j = 1 , \ldots , m$._

如果不存在这样的替换，则是编译时错误。 如果存在多种解决方案，则选择$T'$类型的最佳解决方案。

预期类型$\mathit{pt}$的全部或部分可能未定义
。通在此[一致性](03-types.html#conformance)规则有所扩展，对于任意类型$T$一下两个语句都是
正确的 $\mathit{undefined} <: T$ 和 $T <: \mathit{undefined}$。

对于给定类型变量,可能不存在最小解或最大解,这将导致编译时错误。 因为$<:$是前序的,因此一个类型的解集中可以有多个最优解。在此情况下 Scala 编译器将自由选取其中某一个。


###### 例
考虑一下两个方法

```scala
def cons[A](x: A, xs: List[A]): List[A] = x :: xs
def nil[B]: List[B] = Nil
```
以及定义

```scala
val xs = cons(1, nil)
```
`cons`的应用首先由一个未定义的期望类型进行类型化。该应用程序通过本地类型推断`cons[Int](1, nil)`.这里，使用以下推理来推断类型参数`a`的类型参数`Int`：

首先，输入参数表达式。第一个参数`1`具有`Int`类型，而第二个参数`nil`本身是多态的。 一个尝试使用期望的类型`List[a]`来键入`nil`。 这导致了约束系统

```scala
List[b?] <: List[a]
```
我们在哪里标记了`b?` 用问号表示它是约束系统中的变量。 因为类`List` 是协变的，所以这个约束的最优解是

```scala
b = scala.Nothing
```
在第二步中，一个解决了`cons`的类型参数`a`的以下约束系统：

```scala
Int <: a?
List[scala.Nothing] <: List[a?]
List[a?] <: $\mathit{undefined}$
```

该约束系统的最优解是

```scala
a = Int
```

因此`Int`是`a`的类型的推断结果。

###### 例

现在考虑一下定义

```scala
val ys = cons("abc", xs)
```

其中`xs`像以前一样定义了`List[Int]`类型。 在这种情况下，本地类型推断如下进行。

首先，输入参数表达式。第一个参数`"abc"`具有`String`类型。第一个参数`xs`首先尝试输入预期类型`List[a]`。这失败了，因为 `List[Int]`不是`List[a]`的子类型。因此尝试第二种策略;将使用期望类型 `List[$\mathit{undefined}$]`来类型化`xs`。这成功并产生参数类型`List[Int]`.

在第二步中，一个解决了`cons`的类型参数`a`的以下约束系统：

```scala
String <: a?
List[Int] <: List[a?]
List[a?] <: $\mathit{undefined}$
```
该约束系统的最优解是

```scala
a = scala.Any
```

所以 `scala.Any` 就是 `a` 的类型推结果。


### Eta扩展

_Eta扩展_ 将方法类型的表达式转换为函数类型的等效表达式。 它分两步进行。

首先,标识出$e$的最大子表达式;比如 $e_1 , \ldots , e_m$。对于其中的每一项，都会创建新命名$x_i$。设设$e'$表示将$e$中的每个最大子表达式$e_i$替换为相应的新名称$x_i$。其次，为方法的每个参数类型$T_i$创建一个新名称$y_i$（$i = 1 , \ldots ,n$）。 eta转换的结果是：

```scala
{ val $x_1$ = $e_1$;
  $\ldots$
  val $x_m$ = $e_m$;
  ($y_1: T_1 , \ldots , y_n: T_n$) => $e'$($y_1 , \ldots , y_n$)
}'
```
在eta扩展中保留了[按名称调用参数](#功能应用)的行为：在扩展块中不评估相应的实际参数表达式，即无参数方法类型的子表达式。

### 动态成员选择

标准Scala库定义标记特征`scala.Dynamic`。此特征的子类能够通过定义名称 `applyDynamic`, `applyDynamicNamed`, `selectDynamic`, 和 `updateDynamic`的方法来拦截其实例上的选择和应用程序。

假设$e$的类型符合`scala.Dynamic`，则执行以下重写。动态，且原始表达式不按常规规则进行类型检查，详见[隐式转换](#隐式转换)相关小节:

 *  `e.m[Ti](xi)` 变成 `e.applyDynamic[Ti]("m")(xi)`
 *  `e.m[Ti]`     变成 `e.selectDynamic[Ti]("m")`
 *  `e.m = x`     变成 `e.updateDynamic("m")(x)`


如果在应用程序中命名了任何参数(其中一个`xi`的形式为`arg = x`),则它们的名称将保留为传递给`applyDynamicNamed`的对的第一个组件（对于缺少的名称，使用`""` ）：

 *  `e.m[Ti](argi = xi)` 变成 `e.applyDynamicNamed[Ti]("m")(("argi", xi))`

结果:

 *  `e.m(x) = y` 变成 `e.selectDynamic("m").update(x, y)`

这些方法都不是在`scala.Dynamic`中实际定义的，因此用户可以使用或不使用类型参数或隐式参数自由定义它们。
