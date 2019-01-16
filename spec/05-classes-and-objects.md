---
title: Classes & Objects
layout: default
chapter: 5
---

# 类和对象

```ebnf
TmplDef          ::= [‘case’] ‘class’ ClassDef
                  |  [‘case’] ‘object’ ObjectDef
                  |  ‘trait’ TraitDef
```

[类](#类定义) 和 [对象](#对象定义)
都是用*模板*定义的.

## 模板

```ebnf
ClassTemplate   ::=  [EarlyDefs] ClassParents [TemplateBody]
TraitTemplate   ::=  [EarlyDefs] TraitParents [TemplateBody]
ClassParents    ::=  Constr {‘with’ AnnotType}
TraitParents    ::=  AnnotType {‘with’ AnnotType}
TemplateBody    ::=  [nl] ‘{’ [SelfType] TemplateStat {semi TemplateStat} ‘}’
SelfType        ::=  id [‘:’ Type] ‘=>’
                 |   this ‘:’ Type ‘=>’
```

*模板* 定义了类的特征或类对象或单个对象的类型签名，行为和初始状态。模板是构成实例创建表达式，类定义和对象定义的一部分。模板`$sc$ with $mt_1$ with $\ldots$ with $mt_n$ { $\mathit{stats}$ }`包含构造函数调用$sc$,它定义了模板的超类，特征引用`$mt_1 , \ldots , mt_n$` $(n \geq 0)$，定义模板的特征。以及语句序列  $\mathit{stats}$，其中包含初始化代码和模板的其他成员定义。

每个特征引用 $mt_i$ 必须表示一个 [特征](#特征)。相比之下，超类构造函数$sc$通常是指一个不是特征的类。可以写出由特征引用开头的父类列表，例如`$mt_1$ with $\ldots$ with $mt_n$`.在这种情况下，父类列表被自动扩展为包含$mt_1$的超类。新的超类必须至少有一个无参的构造函数。在下文中，我们将始终假设以执行该自动扩展，一遍模板的第一个父类是常规超类构造函数，而不是特征引用。

模板的父类列表必须格式正确。这意味着由超类构造函数$sc$表示的类必须是所有特征$mt_1 , \ldots , mt_n$的超类的子类。换句话说，模板继承的非特征类在继承层次结构中形成一个链，该链以模板的超类开头。

模板的超类的 *最低要求* 是类类型或[复合类型](03-types.html#compound-types)由其所有的父类类型构成的。

语句序列 $\mathit{stats}$ 包含定义新成员或覆盖父类成员的成员定义。如果模板构成抽象类或特征定义的一部分，那么 $\mathit{stats}$ 也可能包含抽象成员的声明。如果模板构成实体类定义的一部分 $\mathit{stats}$ 仍然可能包含抽象类型成员的声明，但不包含抽象术语成员的声明。此外 $\mathit{stats}$ 在任何情况下都可能包含严格计算的表达式:这些表达式按照他们作为模板初始化一部分给出的顺序执行，即使他们出现在被覆盖成员的定义中。

模板语句的序列可以以形式参数定义和箭头为前缀，例如`$x$ =>`, 或者`$x$:$T$ =>`。如果给出了正式参数，则可以将其用作整个模板主体中的引用`this`的别名。如果正式参数带有 $T$类型，则次定义会影响基础类或对象本身的类型$S$.如下所示：设$C$是定义模板的类、特征或对象的类型。如果给定正式的自参数$T$类型，$S$是$T$和$C$的最大下限。如果没有给出$T$类型。$S$只是$C$。在模板内部，`this`的类型就是$S$.

类或对象的自身类型必须符合模板$t$继承的所有类的自身类型。

第二种形式的自身类型注释就是这样:`this: $S$ =>`。它为`this`规定了$S$类型而没有为他引入别名。

###### 例
考虑以下类的定义：

```scala
class Base extends Object {}
trait Mixin extends Base {}
object O extends Mixin {}
```

在这种情况下， `O`的定义扩展为:

```scala
object O extends Base with Mixin {}
```


**继承自java类型** 模板可以将Java类作为其超类，将Java接口作为其mixins(混入)。

**模板求值** 考虑一个模板 `$sc$ with $mt_1$ with $mt_n$ { $\mathit{stats}$ }`.


如果这是[特征](#特征)模板，则其*混入求值*包括对语句 $\mathit{stats}$的求值。

如果这不是特征模板，则其 *求值* 包括以下步骤:

- 首先，对超类构造函数 $sc$ [求值](#构造函数调用)。
- 然后，模板[线性化](##类线性化)中的所有基类，直到模板的超类(由$SC$表示)都被混合求值。在线性化过程中，混合求值的发生顺序是相反的。
- 最后，计算语句序列$\mathit{stats}\,$ 。

###### 延迟初始化
在超类构造函数调用之后的对象或类(不能是特征)的初始化代码以及模板基类的混合求值被传递到特殊的钩子，这是用户代码无法访问的。通常，该钩子只是执行传递给它的代码。但是继承 `scala.DelayedInit` 特征的模块可以通过重新实现`delayedInit`的方法来覆盖钩子，该方法定义如下：

```scala
def delayedInit(body: => Unit)
```

### 构造函数调用

```ebnf
Constr  ::=  AnnotType {‘(’ [Exprs] ‘)’}
```

构造函数调用定义由实例创建表达式创建的对象和类型，成员和初始状态，或定义由类或对象定义继承的对象定义部分。构造函数调用是一个函数应用程序 `$x$.$c$[$\mathit{targs}$]($\mathit{args}_1$)$\ldots$($\mathit{args}_n$)`，其中$x$是一个[稳定标识符](03-types.html#paths)，$c$是一个类型名称，它指定一个类或为一个类定义一个别名类型， $\mathit{targs}$是一个类型参数列表。$\mathit{args}_1 , \ldots , \mathit{args}_n$是参数列表，并且该类的构造函数[适用](06-expressions.html#function-applications)于给定的参数。如果构造函数调用使用命名参数或默认参数，则使用[此处](sec:named-default)描述的相同转换将其转换为块表达式。

前缀`$x$.`可以省略。仅当类$c$接受类型参数时，才能给出类型参数列表。即使这样，它也可以省略。在这种情况下，使用[本地类型推断](06-expressions.html#local-type-inference)类型参数列表。如果没有显式的给出参数，就会默认一个空参数列表`()`.

对于构造函数调用`$x$.$c$[$\mathit{targs}$]($\mathit{args}_1$)$\ldots$($\mathit{args}_n$)`的执行包含以下步骤:

- 首先计算前缀 $x$ 。
- 然后，从左到右计算参数 $\mathit{args}_1 , \ldots , \mathit{args}_n$ 。
- 最后，通过计算 $c$的引用类的模板来初始化正在创建的类.

### 类的线性化

通过从$C$类直接继承关系的传递闭包可到达的类称为$C$的 *基类*。由于混入，基类上的继承关系通常形成有向无环图。该图的线性化定义如下：


###### 线性化的定义
设$C$是一个模板`$C_1$ with ... with $C_n$ { $\mathit{stats}$ }`的类。$C$, $\mathcal{L}(C)$的 *线性化定义* 如下：

$$\mathcal{L}(C) = C, \mathcal{L}(C_n) \; \vec{+} \; \ldots \; \vec{+} \; \mathcal{L}(C_1)$$

这里$\vec{+}$表示串接，其中右操作数的元素替换左操作数的相同元素。

$$
\begin{array}{lcll}
\{a, A\} \;\vec{+}\; B &=& a, (A \;\vec{+}\; B)  &{\bf if} \; a \not\in B \\\\
                       &=& A \;\vec{+}\; B       &{\bf if} \; a \in B
\end{array}
$$

###### 例
请考虑以下类定义：

```scala
abstract class AbsIterator extends AnyRef { ... }
trait RichIterator extends AbsIterator { ... }
class StringIterator extends AbsIterator { ... }
class Iter extends StringIterator with RichIterator { ... }
```

那么`Iter`类的线性化就是

```scala
{ Iter, RichIterator, StringIterator, AbsIterator, AnyRef, Any }
```

注意，类的线性化改进了继承关系：如果$C$是$D$的子类，那么在$C$和$D$出现的任何线性化中$C$都在$D$的前面。[线性化](#定义:-线性化)还满足以下属性：类的线性化始终包含其直接超类的线性化作为后缀。



例如 `StringIterator` 的线性化是：

```scala
{ StringIterator, AbsIterator, AnyRef, Any }
```

这是其子类`Iter`的线性化的后缀。对于混入的线性化，情况并非如此。例如：`RichIterator`的线性化是


```scala
{ RichIterator, AbsIterator, AnyRef, Any }
```

这不是`Iter`线性化的后缀.

### 类成员

由模板`$C_1$ with $\ldots$ with $C_n$ { $\mathit{stats}$ }`定义的类$C$可以在其语句序列$\mathit{stats}$ 中定义成员，并且可以从所有父类继承成员。scala采用Java和C\# 的方法来静态重载方法。因此，类可能定义或继承具有相同名称的多个方法。为了确定类$C$的已定义成员是否覆盖父类的成员，或者两者是否作为$C$中的重载变体共存，Scala使用以下成员 *匹配* 定义：

###### 定义: 匹配
成员定义$M$ *匹配* 成员定义$M'$，如果$M$和$M'$绑定相同的名称，探后符合下面给中的一条：

1. $M$和$M'$都不是方法定义。
2. $M$ 和 $M'$ 定义具有等效参数类型的两个单态方法。
3. $M$定义无参方法，$M'$定义具有空参数列表`()`的方法，*反之亦然*。
4. $M$和$M'$定义两种参数类型相同数量的多态方法$\overline T$, $\overline T'$和相等数量的类型参数 $\overline t$, $\overline t'$，并且 $\overline T' = [\overline t'/\overline t]\overline T$.

成员定义分为两类：实体类和抽象类。类$C$的成员要么 *直接定义*(即他们出现在$C$的语句序列$\mathit{stats}$中)，或 *继承*。有两个规则确定一个类的成员即，每个类别对应一个:

类$C$的 *具体成员* 是某些类$C_i \in \mathcal{L}(C)$中的任意具体定义$M$，在前置内$C_j \in \mathcal{L}(C)$ 且 $j < i直接定义一个抽象成员$M'$匹配$M$。

类$C$的 *抽象成员* 是某些类$C_i \in \mathcal{L}(C)$中的任何抽象定义$M$,除非$C$已包含具体成员$M'$匹配$M$,或者前面的类$C_j \in \mathcal{L}(C)$ 并且 $j < i$直接定义一个抽象成员$M'$匹配$M$。

此定义还确定类$C$及其父类的匹配成员之间的[重载](#覆盖)关系。首先，具体定义总是重载抽象定义。第二，对于定义$M$和$M'$,他们都是实体或抽象的，只有$M$定义的类在($C$线性化之中)$M'$所在的类的前面出现，$M$才会被重载为$M'$.

如果木匾直接定义两个匹配成员，则会出错。如果模板包含两个具有相同名称且具有相同[擦除](03-types.html#type-erasure)类型的成员，则也会出错。最后，模板不允许包含两个具有相同名称的方法（直接定义或继承），这两个方法都定义了默认参数。

###### 例
考虑特征定义：

```scala
trait A { def f: Int }
trait B extends A { def f: Int = 1 ; def g: Int = 2 ; def h: Int = 3 }
trait C extends A { override def f: Int = 4 ; def g: Int }
trait D extends B with C { def h: Int }
```

特征`D`有一个直接定义的抽象成员`h`。它从特征`C`继承成员`f`，从特征`B`继承成员`g`。

### 覆盖

$C$类的成员$M$[与](#class-members)定义的$C$的非私有成员$M'$一致可定义为 *覆盖* 该成员。在这种情况下，覆盖成员$M$的绑定必须[包含](03-types.html#conformance)被覆盖成员$M'$的绑定。此外，对修饰符的一下限制适用于$M$和$M'$。

- $M'$ 不能标记为 `final`.
- $M$ 不能是 [`private`](#修饰符).
- 如果 $M$ 在某些封闭类或包$C$中标记为 `private[$C$]`，那么$M'$必须在类或包$C'$标记为 `private[$C'$]` ，且$C'$ 等于 $C$ 或 $C'$被包含于 $C$.
- 如果 $M$ 标记为`protected`, 那么 $M'$ 也必须标记为 `protected`.
- 如果$m'$不是抽象成员，则$M$必须标记为`override`。此外，必须保留一下两种可能性中的一种：
    - 在定义了$M'$的子类中定义$M$.
    - 或者，$M$和$M'$覆盖第三个成员$M''$，该成员在包含$M$和$M'$的两个类的基类中定义.
- 如果$M'$在$C$中[不完整](#修饰符)，则$M$必须标记为`abstract override`。
- 如果$M$和$M'$都是具体的值定义，那么他们中没有一个标记为`lazy`或者两者都标记为`lazy`。
- 稳定的成员只能被稳定的成员覆盖。 例如，这是不允许的：

```scala
class X { val stable = 1}
class Y extends X { override var stable = 1 } // error
```
另一个限制适用于抽象类型成员：具有[不稳定性类型](03-types.html#volatile-types)作为其上限的抽象类型成员可能不会覆盖不具有不稳定性上限的抽象类型成员。

一个特殊的规则涉及到无参方法。如果定义为`def $f$: $T$ = ...` 或者 `def $f$ = ...`的无参方法覆盖具有空参数列表的$()T'$类型的方法，则还假定$f$具有空参数列表。

重写方法从超类中的定义继承所有默认参数。通过在重写方法中指定默认参数，可以添加新的默认值（如果超类中的相应参数没有默认值）或覆盖超类的默认值（如果超类中的相应参数有默认值）。


###### 例

考虑定义：

```scala
trait Root { type T <: Root }
trait A extends Root { type T <: A }
trait B extends Root { type T <: B }
trait C extends A with B
```
类定义`C`的格式是错误的，因为`C`中的`T`的绑定是`type T <: B`,它无法包含类型`A`中的`T`的绑定`type T <: A`。可以通过在`C`类中添加类型`T`的覆盖定义来解决该问题。


```scala
class C extends A with B { type T <: C }
```

### 继承闭包

$C$为类类型。$C$的 *继承闭包* 就是以下类型的最小集合$\mathscr{S}$：

- $C$ 在 $\mathscr{S}$中.
- 如果 $T$ 在 $\mathscr{S}$中, 那么每个类型 $T'$在语法上形成 $T$ 的一部分也在$\mathscr{S}$中.
- 如果 $T$ 是 $\mathscr{S}$中的类型, 那么  $T$ 的所有[父类](#模板)也在 $\mathscr{S}$.

如果类类型的继承闭包由无数种类型组成，那么这是一个静态错误。（这种限制对于使子类型可判定是必要的[^kennedy])。

[^kennedy]: Kennedy, Pierce. [ [关于具有方差的名义子类型的可判定性]( http://research.microsoft.com/pubs/64041/fool2007.pdf) in FOOL 2007

### 前置定义

```ebnf
EarlyDefs         ::= ‘{’ [EarlyDef {semi EarlyDef}] ‘}’ ‘with’
EarlyDef          ::=  {Annotation} {Modifier} PatVarDef
```
模板可以从 *前置字段定义* 子句开始，该子句用于在调用超类型构造函数之前定义某些字段值。 在模板中

```scala
{ val $p_1$: $T_1$ = $e_1$
  ...
  val $p_n$: $T_n$ = $e_n$
} with $sc$ with $mt_1$ with $mt_n$ { $\mathit{stats}$ }
```

$p_1 , \ldots , p_n$ 的初始模式定义称为前置定义。它们定义了构成模板一部分的字段。 每个前置定义必须至少定义一个变量。

前置定义在模板被定义与赋类型参数以及在此之前的任意前置定义之前做类型检查与求值,参数可以是类的任意类型参数。在前置定义的右侧对`this`的任何引用都指的是模板外部的`this` 的标识符。因此，前置定义不可能引用模板创建的对象，或者一弄其他字段与方法，除了同一段落中前面的前置定义之外。再者,对前面的前置定义的引用总是引用那里定义的值,并不牵涉到覆盖的定义。

在调用模板的超类构造函数之前，按照它们的定义顺序计算前置定义。

###### 例
前置定义对于没有正常构造函数参数的特征特别有用。 例：
```scala
trait Greeting {
  val name: String
  val msg = "How are you, "+name
}
class C extends {
  val name = "Bob"
} with Greeting {
  println(msg)
}
```

在上面的代码中，字段`name`在调用`Greeting`的构造函数之前被初始化。因此，`Greeting`中的字段`msg`被适当的初始化为`"How are you, Bob"`。

如果`name`已经在`C`的普通类体中初始化，那么它将在`Greeting`的构造函数之后初始化。在这种情况下，`msg`将被初始化为`"How are you, <null>"`。

## 修饰符

```ebnf
Modifier          ::=  LocalModifier
                    |  AccessModifier
                    |  ‘override’
LocalModifier     ::=  ‘abstract’
                    |  ‘final’
                    |  ‘sealed’
                    |  ‘implicit’
                    |  ‘lazy’
AccessModifier    ::=  (‘private’ | ‘protected’) [AccessQualifier]
AccessQualifier   ::=  ‘[’ (id | ‘this’) ‘]’
```
成员定义之前可以有修饰符，这些修饰符会影响他们绑定的标识符的可访问性和用法。如果给出了几个修饰符，他们的顺序无关紧要，但相同的修饰符可能不会出现多次。重复定义之前的修饰符适用于所有组成定义。管理修饰符的有效性和含义的规则如下：

### `private`
 `private` 修饰符可以与模板中的任何定义或声明一个适用。只能从直接封闭的模板及其伴随模块或[伴随类](#对象定义)中访问此类成员。

`private`修饰符可以 *适用* 标识符$C$(例如：`private[$C$]`)进行限定，该标识符必须包含该定义的类或包。标有这种修饰符的成员只能分别从$C$包中的代码访问，或者只能从$C$类及其[伴随类](#对象定义)中的代码访问。

还有种特殊形式是`private[this]`。标有此修饰符的成员$M$被称为*受保护对象*；它只能从定义它的对象中访问。也就是说，选择$p.M$仅在前缀为`this` 和包含引用的$O$类型`$O$.this`的情况下才合法。此外，这就是没有加限定的 `private`的应用。

标记为私有而没有限定符的成员称为*私有类*,而标记为 `private[this]`的成员被称为*私有对象*.不管事私有类还是私有对象都被称为*私有成员*,但是`private[$C$]`不是，$C$是一个标识符，在后者该成员称为*限定私有*。

私有类或私有成员可能不是抽象的，也可能没有`protected` 或 `override`修饰符。他们不是由子类继承的，他们可能不会覆盖父类中的定义。

### `protected`
`protected` 修饰符适用与类成员定义。可以从内部访问类的受保护成员:
  - 定义类的模板
  - 所有将定义类作为基类的模板，
  - 任何这些类的伴随模块。

`protected`修饰符可以使用标识符$C$(例如：`protected[$C$]`)进行限定，该标识符必须表示包含该定义的类或包。标有这种修饰符的成员也可以分别从$C$包中的所有代码或类$C$及[伴随模块]($对象定义)中的所有代码访问。

只有在满足一下条件之一时，受保护的标识符$x$才可以做选择`$r$.$x$` 中的成员名称：

  - 访问权限在定义成员的模板内，或者，如果给定资格$C$,则在$C$包或在$C$类或其伴随模块中
  - $r$ 是保留字 `this` 和`super`之一
  - $r$的类型符合包含访问权的类的类型实例。

不同形式的资格是`protected[this]`。标有此修饰符的成员$M$称为*保护对象*; 它只能从定义它的对象中访问。也就是说，选择 $p.M$仅在前缀为`this`或`$O$.this`时合法，对于包含引用的某些类$O$。这也就是未加限定的`protected`应用。

### `override`

`override`修饰符适用于类成员定义或声明。成员定义或声明必须覆盖父类中的某些其他具体的成员定义。如果给出了`ocerride`修饰符，则必须至少有一个重写的成员定义或声明(具体或抽象)。

### `abstract override`
当与`abstract`修饰符组合时，`override`修饰符具有额外的意义。该修饰符组合仅允许特征的值成员使用。

我们称一个模板$M$ _不完整_ 的条件是：$M$是抽象的(未被声明和定义)，或者它被标记为`abstract` 和 `override`，并且每个被$M$覆盖的成员也是不完整的。

请注意，`abstract override`修饰符组合不会影响成员是具体还是抽象的概念。 如果仅为其提供声明，则该成员是*抽象*的; 如果给出完整的定义，则是*具体*的。

### `abstract`
`abstract`修饰符用于类定义中。它对于特征来说是多余的。对于具有不完整成员的类来说是必须的。抽象类不能通过构造器调用[初始化](06-expressions.html#instance-creation-expressions),除非后面跟覆盖了类中所有不完整成员的混入和/或修饰体。只有抽象类和特征可以由抽象术语成员。

`abstract`修饰符也可以与类成员定义的`override`一起使用。前面的讨论的就是这种情况。

### `final`
`final`修饰符适用于类.成员定义和类定义.不能在子类中重写`final`类成员定义。 `final`类不会被模板继承。 `final`对于对象定义是多余的。 标记为`final`的类或对象的成员隐含定义为`final`的，对他们来说是多余的。 但请注意，[常量值定义](04-basic-declarations-and-definitions.html#value-declarations-and-definitions)确实需要显式`final`修饰符，即使它们是在最终类或对象中定义的。 `final`被允许用于抽象类，但它不适用于特征或不完整的成员，并且它可能不会与`sealed`组合在一个修饰符列表中。

### `sealed`
`sealed`修饰符适用于类定义。 除非继承模板在与继承类相同的源文件中定义，否则不能直接继承`sealed`类。 但是，密封类的子类可以在任何地方继承。

### `lazy`
`lazy`修饰符适用于值定义.`lazy`值在第一次访问（可能永远不会发生）时初始化。尝试在初始化期间访问惰性值可能会导致循环行为。 如果在初始化期间抛出异常，则该值被视为未初始化，稍后的访问将重试以对其右侧求值。

###### 例
以下代码说明了使用限定私有：

```scala
package outerpkg.innerpkg
class Outer {
  class Inner {
    private[Outer] def f()
    private[innerpkg] def g()
    private[outerpkg] def h()
  }
}
```

在这里，对方法`f`的访问可以出现在`Outer`内的任何地方，但不能出现在它之外。 对方法`g`的访问可以出现在包`outerpkg.innerpkg`中的任何位置，就像Java中的包私有方法一样。 最后，对方法`h`的访问可以出现在包`outerpkg`中的任何位置，包括其中包含的包。

###### 例
防止类的客户端构造该类的新实例的有用习惯是声明类`abstract` 和 `sealed`:

```scala
object m {
  abstract sealed class C (x: Int) {
    def nextC = new C(x + 1) {}
  }
  val empty = new C(0) {}
}
```
例如，在上面的代码中，客户端只能通过调用现有`m.C`对象的`nextC`方法来创建类`m.C`的实例; 客户端无法直接创建类`m.C`的对象。 实际上，以下两行都是错误的：

```scala
new m.C(0)    // **** 错误：C是抽象的，因此无法实例化。
new m.C(0) {} // **** 错误：密封类的非法继承。
```
通过将主构造函数标记为`private`（[示例](#示例私有构造函数)），可以实现类似的访问限制。


## 类定义

```ebnf
TmplDef           ::=  ‘class’ ClassDef
ClassDef          ::=  id [TypeParamClause] {Annotation}
                       [AccessModifier] ClassParamClauses ClassTemplateOpt
ClassParamClauses ::=  {ClassParamClause}
                       [[nl] ‘(’ implicit ClassParams ‘)’]
ClassParamClause  ::=  [nl] ‘(’ [ClassParams] ‘)’
ClassParams       ::=  ClassParam {‘,’ ClassParam}
ClassParam        ::=  {Annotation} {Modifier} [(‘val’ | ‘var’)]
                       id [‘:’ ParamType] [‘=’ Expr]
ClassTemplateOpt  ::=  ‘extends’ ClassTemplate | [[‘extends’] TemplateBody]
```

最常见的类定义形式是

```scala
class $c$[$\mathit{tps}\,$] $as$ $m$($\mathit{ps}_1$)$\ldots$($\mathit{ps}_n$) extends $t$    $\quad(n \geq 0)$.
```

在这里面：

  - $c$ 是要定义的类的名称。
  - $\mathit{tps}$是要定义的类的类型参数的非空列表。类型参数的范围是整个类定义，包括类型参数本身部分。定义具有相同名称的两个参数类型是非法的。可以省略类型参数部分 `[$\mathit{tps}\,$]`。具有类型参数部分的类称为 _多态_，否则称为 _单态_。
  - $as$是一个可为空的[标注](11-annotations.html#user-defined-annotations)序列。如果给出任何标注，它们将应用于类的主要构造函数。
  - $m$是一个[访问修饰符](#修饰符)， 例如`private` 或 `protected`，可能具有限定条件。如果给出这样的访问修饰符，它们应用与类的主构造函数。
  - $(\mathit{ps}\_1)\ldots(\mathit{ps}\_n)$ 是类的 _主要构造函数_ 的正式值参数子句。正式值参数的范围包括所有后续参数部分和模板$t$。但是，正式值参数不会构成任何父类的类型或类型模板$t$的成员的一部分。定义具有相同名称的两个正式值参数是非法的。

    如果一个类没有隐含正式的参数部分，则会假定有一个空参数段 `()`.

    如果形式参数声明$x: T$前面有`val`或 `var`关键字，则此参数的访问器(getter)[定义](04-basic-declarations-and-definitions.html#variable-declarations-and-definitions)将会自动加入类中。

    getter引入了$C$类的成员值$x$,定义为参数别名。如果引入关键字是`var`,一个setter访问[`$x$_=`](04-basic-declarations-and-definitions.html#variable-declarations-and-definitions)会自动添加到该类中。在调用setter`$x$_=($e$)`时，将参数值更改为$e$的求值结果。

    形式参数声明可以包含修饰符，然后修饰符转移到访问者定义。当参数给出访问修饰符，但没有`val`或`var`关键字时，假定为`val`。以`val`或 `var`为前缀的形式参数可能不会同时是[名称调用参数](04-basic-declarations-and-definitions.html#by-name-parameters)。

  - $t$ 是一个[模板](#模板)，具有一下形式

    ```scala
    $sc$ with $mt_1$ with $\ldots$ with $mt_m$ { $\mathit{stats}$ } // $m \geq 0$
    ```

    它定义了类的对象基类，行为和初始状态。可以省略扩展子句  `extends $sc$ with $mt_1$ with $\ldots$ with $mt_m$`，这时候假设扩展为  `extends scala.AnyRef` 。类体`{ $\mathit{stats}$ }`也可以省略，在这种情况下假定为空`{}`。

此类定义定义了类型为`$c$[$\mathit{tps}\,$]`和一个构造函数，当该构造函数应用于符合类型$\mathit{ps}$时，构造函数通过计算模板$t$的值来初始化类型`$c$[$\mathit{tps}\,$]` 实例。

###### 示例 - `val`和`var`参数
以下示例说明了类`C`的`val`和`var`参数：

```scala
class C(x: Int, val y: String, var z: List[String])
val c = new C(1, "abc", List())
c.z = c.y :: c.z
```

###### 示例 - 私有构造函数
只能从其配套模块创建以下类。

```scala
object Sensitive {
  def makeSensitive(credentials: Certificate): Sensitive =
    if (credentials == Admin) new Sensitive()
    else throw new SecurityViolationException
}
class Sensitive private () {
  ...
}
```

### 构造函数定义

```ebnf
FunDef         ::= ‘this’ ParamClause ParamClauses
                   (‘=’ ConstrExpr | [nl] ConstrBlock)
ConstrExpr     ::= SelfInvocation
                |  ConstrBlock
ConstrBlock    ::= ‘{’ SelfInvocation {semi BlockStat} ‘}’
SelfInvocation ::= ‘this’ ArgumentExprs {ArgumentExprs}
```

除了主构造函数之外，类还有其他构造函数。这些由`def this($\mathit{ps}_1$)$\ldots$($\mathit{ps}_n$) = $e$`形式的构造函数的定义来定义。这样的定义为封闭类引入了一个额外的构造函数，其参数在形式参数列表$\mathit{ps}_1
, \ldots , \mathit{ps}_n$中给出，其评估由构造函数表达式$e$定义。每个形式参数的范围是构造函数表达式$e$.构造函数表达式是自构造函数调用`this($\mathit{args}_1$)$\ldots$($\mathit{args}_n$)`或以一个以构造器自调用开始的代码块。自构造函数调用必须构造该类的通用实例。例如，如果问题中的类具有名称$C$和类型参数`[$\mathit{tps}\,$]`，则自构造函数调用必须生成`$C$[$\mathit{tps}\,$]`的实例。不郧西实例化正式类型参数。

构造函数定义中的签名和构造函数自调用是有类型检查的，并在类内产生作用域的地方求值，可以加该类的任何类型参数以及该模板的任何[前置定义](#前置定义)。构造函数表达式的其余部分经过类型检查并作为当前类中的函数体进行求值。

如果由一个类$C$的辅助构造函数，他们与$C$的主构造函数一起形成一个重载的[构造函数](#类定义)定义。[重载解析](06-expressions.html#overloading-resolution)的通常规则适用于$C$的构造函数调用，包括构造函数表达式本身中的自构造函数调用。但是，与其他方法不同，构造函数永远不会被继承。为了房子构造函数调用的无限循环，有一个限制是每个自构造函数调用必须引用它之前的构造函数定义(即它必须引用前面的辅助构造函数或类的主构造函数)。

###### 例
考虑类定义

```scala
class LinkedList[A]() {
  var head = _
  var tail = null
  def isEmpty = tail != null
  def this(head: A) = { this(); this.head = head }
  def this(head: A, tail: List[A]) = { this(head); this.tail = tail }
}
```

这定义了一个带有三个结构的类`LinkedList`。 第二个构造函数构造一个单例列表，而第三个构造函数构造一个具有给定头尾的列表。

### case类

```ebnf
TmplDef  ::=  ‘case’ ‘class’ ClassDef
```

如果类定义以`case`为前缀，则该类被称为 _case类_。

case类需要有一个参数部分，而不是隐藏。第一个参数部分中的形式参数称之为 _元素_，并经过特殊处理。首先，可以将这样的参数的值提取为构造函数模式的字段，其次，该参数默认添加`val`前缀，除非该参数已经有了`val` 或 `var`修饰符。然后[生成](#类定义)参数的访问定义。

带有类型参数$\mathit{tps}$和值参数$\mathit{ps}$的case类定义`$c$[$\mathit{tps}\,$]($\mathit{ps}_1\,$)$\ldots$($\mathit{ps}_n$)`，会自动生成一个[扩展对象](08-pattern-matching.html#extractor-patterns)。定义如下：

```scala
object $c$ {
  def apply[$\mathit{tps}\,$]($\mathit{ps}_1\,$)$\ldots$($\mathit{ps}_n$): $c$[$\mathit{tps}\,$] = new $c$[$\mathit{Ts}\,$]($\mathit{xs}_1\,$)$\ldots$($\mathit{xs}_n$)
  def unapply[$\mathit{tps}\,$]($x$: $c$[$\mathit{tps}\,$]) =
    if (x eq null) scala.None
    else scala.Some($x.\mathit{xs}_{11}, \ldots , x.\mathit{xs}_{1k}$)
}
```
在这里，$\mathit{Ts}$代表类型参数部分 $\mathit{tps}$中定义的类型向量，每个$\mathit{xs}\_i$表示$\mathit{ps}\_i$的参数部分。$\mathit{xs}\_{11}, \ldots , \mathit{xs}\_{1k}$表示第一个参数段$\mathit{xs}\_1$中的所有参数明。如果类中缺少类型参数的部分，则`apply` 和 `unapply`方法中也会缺失。

如果已经定义了伴随对象$c$，则`apply`和`unapply`方法将会添加到现有对象中。如果对象$c$已具有[匹配](#匹配定义)的`apply` (或 `unapply`)成员，则不会添加新的定义。如果类$c$是`abstract`，则省略`apply`。

如果case类定义包含空值参数列表，则`unapply`方法返回`Boolean`而不是`Option` 类型，定义如下

```scala
def unapply[$\mathit{tps}\,$]($x$: $c$[$\mathit{tps}\,$]) = x ne null
```

如果$c$的一个参数部分$\mathit{ps}_1$以[重复参数](04-basic-declarations-and-definitions.html#repeated-parameters)结束，则`unapply`方法的名称更改为`unapplySeq`。

除非该类已具有具有该名称的成员（直接定义或继承），或者该类具有重复参数，否则将隐式地向每个案例类添加名为`copy`的方法。 该方法定义如下：


```scala
def copy[$\mathit{tps}\,$]($\mathit{ps}'_1\,$)$\ldots$($\mathit{ps}'_n$): $c$[$\mathit{tps}\,$] = new $c$[$\mathit{Ts}\,$]($\mathit{xs}_1\,$)$\ldots$($\mathit{xs}_n$)
```

同样，`$\mathit{Ts}$`代表类型参数`$\mathit{tps}$`中定义的类型向量，每个`$xs_i$`代表参数部分`$ps'_i$`的参数名称。第一个参数列表的值`$ps'_{1,j}$`的格式为`$x_{1,j}$:$T_{1,j}$=this.$x_{1,j}$`，其他参数`$ps'_{i,j}$`的`copy`方法定义为`$x_{i,j}$:$T_{i,j}$`。在所有情况下，`$x_{i,j}$` 和 `$T_{i,j}$`引用相应类型参数`$\mathit{ps}_{i,j}$`的名称和类型。

每个case类都默认覆盖类[`scala.AnyRef`](12-the-scala-standard-library.html#root-classes)的一些方法定义，除非在case类本身已经给出了相同方法的定义，或者在不同于`AnyRef`的case类的某个基类中给出了相同方法的具体定义。特别是

- 方法`equals: (Any)Boolean`是结构相等，其中如果两个实例都属于所讨论的案例类并且他们具有相等(相对于`equals`)构造函数参数（限于类的 _元素_，即第一个参数部分),则他们相等。
- 方法`hashCode: Int`计算哈希码。 如果数据结构成员的hashCode方法将相等（相对于等于）值映射为相等的哈希码，则case类中的hashCode方法也会这样做。
- 方法`toString: String`返回一个字符串形式，其中包含类及其元素的名称

###### 例
以下是lambda演算的抽象语法定义：

```scala
class Expr
case class Var   (x: String)          extends Expr
case class Apply (f: Expr, e: Expr)   extends Expr
case class Lambda(x: String, e: Expr) extends Expr
```

这定义了一个具有案例类`Var`, `Apply` 和 `Lambda`的类`Expr`。然后可以然后可以按如下方式编写lambda表达式的值调用求值程序。


```scala
type Env = String => Value
case class Value(e: Expr, env: Env)

def eval(e: Expr, env: Env): Value = e match {
  case Var (x) =>
    env(x)
  case Apply(f, g) =>
    val Value(Lambda (x, e1), env1) = eval(f, env)
    val v = eval(g, env)
    eval (e1, (y => if (y == x) v else env1(y)))
  case Lambda(_, _) =>
    Value(e, env)
}
```
例如，可以在程序的其他部分中定义扩展类型`Expr`的其他案例类

```scala
case class Number(x: Int) extends Expr
```

可以通过声明基类`Expr`标记为`sealed`来移除扩展性。在这种情况下，所有直接扩展`Expr`的类必须与`Expr`位于同一个源文件中

## 特征

```ebnf
TmplDef          ::=  ‘trait’ TraitDef
TraitDef         ::=  id [TypeParamClause] TraitTemplateOpt
TraitTemplateOpt ::=  ‘extends’ TraitTemplate | [[‘extends’] TemplateBody]
```

 _特征_ 是一个类，他可以作为混合添加到其他类中。与普通类不同，特征类不能具有构造函数参数。此外，没有构造函数参数传递给特征的超类。这是不必要的，因为在初始化超类后就初始化特征。


假设特征$D$定义了$C$类型的实例$x$的某些性质(即$D$是$C$的基类)。那么$x$中的$D$的 _实际超类_ 是由$\mathcal{L}(C)$中的所有超越$D$的基类组成的符合类型。实际的超类给出了解决特征中[`super`引用](06-expressions.html#this-and-super)的上下文。要注意到实际超类型依赖于特征所添加进的混入组合,当定义特征时是无法知道的。

如果$D$不是特征，那么它的实际超类型就是它最小合适的超类型（实际在定义时可知）

###### 例
以下特征定义了与某种类型的对象相当的属性。 它包含一个抽象方法`<`和其他比较运算符`<=`, `>`和`>=`的默认实现。

```scala
trait Comparable[T <: Comparable[T]] { self: T =>
  def < (that: T): Boolean
  def <=(that: T): Boolean = this < that || this == that
  def > (that: T): Boolean = that < this
  def >=(that: T): Boolean = that <= this
}
```

###### 例

考虑抽象类`Table`实现了由键类型`A`到值类型`B`的映射。该类有一个方法`set`来将一个新的键值对放入表中,和方法`get`来返回与给定键值匹配的可选值。最后,有和`get`方法类似的方法`apply`,除非它为给定的键未定义表时返回给定的默认值。 该类实现如下：

```scala
abstract class Table[A, B](defaultValue: B) {
  def get(key: A): Option[B]
  def set(key: A, value: B)
  def apply(key: A) = get(key) match {
    case Some(value) => value
    case None => defaultValue
  }
}
```

这是`Table`类的具体实现：

```scala
class ListTable[A, B](defaultValue: B) extends Table[A, B](defaultValue) {
  private var elems: List[(A, B)]
  def get(key: A) = elems.find(._1.==(key)).map(._2)
  def set(key: A, value: B) = { elems = (key, value) :: elems }
}
```

这是一个特性，可以防止并发访问其父类的`get`和`set`操作：

```scala
trait SynchronizedTable[A, B] extends Table[A, B] {
  abstract override def get(key: A): B =
    synchronized { super.get(key) }
  abstract override def set((key: A, value: B) =
    synchronized { super.set(key, value) }
}
```

请注意，即使`Table`使用形式参数定义，`SynchronizedTable`也不会将参数传递给其超类`Table`.另请注意，在`SynchronizedTable`的`get`和`set`方法中，`super`调用静态引用类`Table`中的抽象方法。 这是合法的，只要调用方法标记为[抽象覆盖](#修饰符)。


最后，以下混合组合创建一个同步列表，其中字符串作为键，整数作为值，默认值为`0`：

```scala
object MyTable extends ListTable[String, Int](0) with SynchronizedTable
```
对象`MyTable`从`SynchronizedTable`继承其`get`和`set`方法。 这些方法中的`super`调用被重新绑定以引用`ListTable`中的相应实现，这是`MyTable`中`SynchronizedTable`的实际超类型。


## 对象定义

```ebnf
ObjectDef       ::=  id ClassTemplate
```

_对象定义_ 定义新类型的单个对象。其最通用的形式是`object $m$ extends $t$`。这里$m$ 是要定义的对象的名称，$t$ 是一个具有一下形式的[模板](#模板)。

```scala
$sc$ with $mt_1$ with $\ldots$ with $mt_n$ { $\mathit{stats}$ }
```

它定义了$m$的基类，行为和初始状态。可以省略扩展条例`extends $sc$ with $mt_1$ with $\ldots$ with $mt_n$`，在这种情况下默认`extends scala.AnyRef` 。也可以省略`{ $\mathit{stats}$ }` ，在这种情况下默认空体`{}`.

对象定义定义符合模板$ t $的单个对象（或：_模块_）。 它大致相当于以下值的定义：

```scala
lazy val $m$ = new $sc$ with $mt_1$ with $\ldots$ with $mt_n$ { this: $m.type$ => $\mathit{stats}$ }
```

请注意，对象定义定义的值是懒惰实例化的。`new $m$\$cls`构造函数不是在对象定义进行计算，而是在程序执行期间第一次取消引用$m$(可能根本不会)时进行计算。在评估构造函数期间尝试再次取消引用$m$将倒是无限循环或运行错误。在计算构造函数是，尝试取消引用$m的其他线程会停止，知道计算完成。

对于顶级对象，上面给出的扩展不准确。 它不能是因为变量和方法定义不能出现在[包对象](09-top-level-definitions.html#package-objects)外部的顶层。 而是将顶级对象转换为静态字段。

###### 例
Scala中的类没有静态成员; 然而，通过伴随的对象定义可以实现等效的效果

```scala
abstract class Point {
  val x: Double
  val y: Double
  def isOrigin = (x == 0.0 && y == 0.0)
}
object Point {
  val origin = new Point() { val x = 0.0; val y = 0.0 }
}
```
这定义了一个`Point`类和一个包含`origin`作为成员的对象`Point`。 请注意，名称`Point`的双重使用是合法的，因为类定义在类型名称空间中定义名称`Point`，而对象定义在术语名称空间中定义名称。

在使用静态成员解释Java类时，Scala编译器将应用此技术。这样的$C$类在概练上被视为包含包含$C$的所有实例成员的一个Scala类，和一个包含$C$的所有静态成员的scala对象的对组合。

通常，类的 _伴随模块_ 是与类具有相同名称的对象，并且在同一范围和编译单元中定义。同理，该类称为模块的 _伴随类_。

非常类似于具体的类定义，对象定义可能仍然包含抽象类型成员的声明，但不包含抽象术语成员的声明。
