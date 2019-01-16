---
题目: 类型
布局: 默认
章节: 3
---

# 类型

```ebnf
  Type              ::=  FunctionArgTypes ‘=>’ Type
                      |  InfixType [ExistentialClause]
  FunctionArgTypes  ::=  InfixType
                      |  ‘(’ [ ParamType {‘,’ ParamType } ] ‘)’
  ExistentialClause ::=  ‘forSome’ ‘{’ ExistentialDcl
                             {semi ExistentialDcl} ‘}’
  ExistentialDcl    ::=  ‘type’ TypeDcl
                      |  ‘val’ ValDcl
  InfixType         ::=  CompoundType {id [nl] CompoundType}
  CompoundType      ::=  AnnotType {‘with’ AnnotType} [Refinement]
                      |  Refinement
  AnnotType         ::=  SimpleType {Annotation}
  SimpleType        ::=  SimpleType TypeArgs
                      |  SimpleType ‘#’ id
                      |  StableId
                      |  Path ‘.’ ‘type’
                      |  Literal
                      |  ‘(’ Types ‘)’
  TypeArgs          ::=  ‘[’ Types ‘]’
  Types             ::=  Type {‘,’ Type}
```

我们区分了一阶类型和类型构造器(用类型的参数构造类型)。一阶类型的一个子集是*值类型*，表示(一阶)值的集合。 值类型可以是 *具体* 或 *抽象* 的。  

每个具体的值类型都可以用一个 *类类型* 来表示，即引用[类或特征](05-classes-and-objects.html#class-definitions)的[类型指示符](#类型指示符)[^ 1]，或者表示类型交集的[复合类型](#复合类型)，可能还有进一步约束其成员类型的[细化](#复合类型)。

抽象值类型由[类型参数](04-basic-declarations-and-definitions.html#type-parameters)和
[抽象类型绑定](04-basic-declarations-and-definitions.html#type-declarations-and-type-aliases)引入。类型中的括号可用于分组。


[^1]: 我们假设对象和包也隐式定义了一个类（与对象或包同名，但用户程序无法访问）。

非值类型描述了那些[非值](#非值)标识符的属性。例如，[类型构造函数](#类型构造函数)不直接指定值的类型。然而，当类型构造函数应用于正确的类型参数时，它会产生一个可能是值类型的一阶类型。

非值类型在Scala中是间接表达的。例如，方法类型是通过写下方法签名来描述的，方法签名本身并不是真正的类型，尽管它可以的到对应的[方法类型](#方法类型)。类型构造函数是另一个例子，可以编写`type Swap[m[_, _], a,b] = m[b, a]`，但是没有定义直接给出对应匿名类型函数的语法。

## 路径

```ebnf
Path            ::=  StableId
                  |  [id ‘.’] this
StableId        ::=  id
                  |  Path ‘.’ id
                  |  [id ‘.’] ‘super’ [ClassQualifier] ‘.’ id
ClassQualifier  ::= ‘[’ id ‘]’
```

路径不是类型本身，但它们可以是命名类型的一部分，并且该功能是Scala的类型系统中形成核心角色。

路径是以下定义之一

- 空路径 ε（不能在程序中显示地写出来）。
- $C.$`this`, 其中 $C$ 是一个类的引用。路径`this`被视为 $C.$`this`的简写，其中 $C$ 是直接包含引用的类的名称。
- $p.x$ 其中$p$是路径，$x$是$p$的稳定成员。*稳定成员* 是由对象定义或[稳定类型](#易失性类型)的值定义引入的包或成员。
- $C.$`super`$.x$ 或 $C.$`super`$[M].x$其中$C$是一个类的引用，$x$是$C$的超类或制定的父类$M$的稳定成员的引用。前缀`super`被视为$C.$`super`的简写，$C$是直接包含引用返回的的类的名称。

一个*稳定标识符* 是以标识符结尾的路径。

## 值类型

Scala中的每个值都具有以下形式之一的类型。

### 单例类型

```ebnf
SimpleType  ::=  Path ‘.’ ‘type’
```

*单例类型* 的形式为$p.$`type`。其中$p$是指向符合`scala.AnyRef`的值的路径，该类型表示由`null`组成的值的集合以及由$p$表示的值（例如，值$v$是`v eq p`中的某一个）。

稳定类型是单例类型或特征为`scala.Singleton`的子类型。


### 文字类型

```ebnf
SimpleType  ::=  Literal
```

文字类型`lit`是一种特殊的单例类型，它表示单个文字值`lit`。因此，类型归属`1:1`为文字值1提供了最精确的类型:文字类型`1`。

在运行时，如果`e == lit`，则表达式`e`被认为具有文字类型`lit`。 具体地说，`e.isInstanceOf [lit]`和`e match {case _：lit =>}`的结果是通过评估`e == lit`来确定的。

文字类型适用于除`Unit`之外的专用语法的所有类型。 这包括数字类型（除了`Byte`和`Short`，当前没有语法），`Boolean`，`Char`，`String`和`Symbol`。

### 稳定类型
稳定类型是 *单例类型*，文字类型或声明为特征`scala.Singleton`的子类型的类型。

### 类型映射

```ebnf
SimpleType  ::=  SimpleType ‘#’ id
```

*类型映射* $T$#$x$ 引用类型 $T$ 的名为 $x$ 的类型成员。


### 类型指示

```ebnf
SimpleType  ::=  StableId
```
*类型指示* 是指命名的值类型。它可以是简单的或限定的。 所有这些类型指示符都是类型映射的缩写。


具体来说，在对象或包 $C$ 中，非限定类型名称 $t$作为 $C.$`this.type #`$t$的简写，除非$t$是类型模式的一部分。如果 $t$ 未绑定在类，对象或包中，那么 $t$ 将作为ε`.type＃` $t$的简写。

限定类型指示符的形式为`p.t`，其中`p`是[路径](#路径)，*t* 是类型名称。这种类型的指示符等同于类型映射`p.type #t`。

###### 例

下面列出了一些类型指示符及其扩展。 我们假设一个本地类型参数 $t$，一个值`maintable`具有一个类型成员`Node`，以及一个标准类`scala.Int`。

| 标志符               | 展开                      |
|-------------------- | --------------------------|
|t                    | ε.type#t                  |
|Int                  | scala.type#Int            |
|scala.Int            | scala.type#Int            |
|data.maintable.Node  | data.maintable.type#Node  |

### 参数化类型

```ebnf
SimpleType      ::=  SimpleType TypeArgs
TypeArgs        ::=  ‘[’ Types ‘]’
```

*参数化类型* $T[ T_1 , \ldots , T_n ]$包括类型指示符 $T$ 以及类型参数 $T_1，\ldots , T_n$，
其中 $T$必须指向一个具有参数类型$a_1 , \ldots , a_n$ 的类类型构造方法。

假设类型参数的下限为$L_1 , \ldots , L_n$ ，上限为$U_1,\ldots, U_n$。参数化类型必须保证每个参数化类型都是 *符合其边* 的：$\sigma L_i <: T_i <: \sigma U_i$ 其中 $\sigma$ 表示$[ a_1 := T_1 , \ldots , a_n := T_n ]$.   


###### 示例参数化类型

以下是部分类型定义：

```scala
class TreeMap[A <: Comparable[A], B] { … }
class List[A] { … }
class I extends Comparable[I] { … }

class F[M[_], X] { … }
class S[K <: String] { … }
class G[M[ Z <: I ], I] { … }
```

以下参数化类型的格式正确：

```scala
TreeMap[I, String]
List[I]
List[List[Boolean]]

F[List, Int]
G[S, String]
```

###### 例子

鉴于[上面的类型定义](＃示例参数化类型)，
以下类型格式错误：


```scala
TreeMap[I]            // 非法:参数数量错误
TreeMap[List[I], Int] // 非法:类型参数不在绑定范围内

F[Int, Boolean]       // 非法:Int不是类型构造函数
F[TreeMap, Int]       // 非法:TreeMap有两个参数
                      //   F期望构造函数只取一个
G[S, Int]             // 非法：S约束其参数为String，
                      // G期望类型构造函数具有符合Int的参数
```

### 元组类型

```ebnf
SimpleType    ::=   ‘(’ Types ‘)’
```

_元组类型_ $(T_1 , \ldots , T_n)$ 是类 `scala.Tuple$n$[$T_1$, … , $T_n$]`的别形式，此类型可以在结尾有个额外的逗号。例如 $n \geq 2$.

元组类是case类，其字段可以使用选择器 `_1` , … , `_n`访问。在对应的`Product`特征中由他们的抽象函数,_n_-ary元组类和产品特性在标准Scala库中至少定义如下（它们也可能添加其他方法并实现其他特征）。

```scala
case class Tuple$n$[+$T_1$, … , +$T_n$](_1: $T_1$, … , _n: $T_n$)
extends Product_n[$T_1$, … , $T_n$]

trait Product_n[+$T_1$, … , +$T_n$] {
  override def productArity = $n$
  def _1: $T_1$
  …
  def _n: $T_n$
}
```

### 标注类型

```ebnf
AnnotType  ::=  SimpleType {Annotation}
```

*标注类型*$T$ $a_1, \ldots, a_n$将[注释](11-annotations.html#user-defined-annotations)$a_1 , \ldots , a_n$附加到类型$T$上。

###### 例

以下类型将`@ suspendable`注释添加到`String`类型：

```scala
String @suspendable
```

### 复合类型

```ebnf
CompoundType    ::=  AnnotType {‘with’ AnnotType} [Refinement]
                  |  Refinement
Refinement      ::=  [nl] ‘{’ RefineStat {semi RefineStat} ‘}’
RefineStat      ::=  Dcl
                  |  ‘type’ TypeDef
                  |
```

_复合类型_ $T_1$ `with` … `with` $T_n \\{ R \\}$表示一个拥有
$T_1 , \ldots , T_n$ 和细化$\\{ R \\}$. 细化$\\{ R \\}$ 类型成员以及修饰${R}$的对象。
如果对象中有声明或定义覆盖了成分类型$T_1 , \ldots , T_n$中的声明或定义，就会应用通常的
[覆盖](05-classes-and-objects.html#overriding)规则; 否则声明或定义被称为“结构化的”[^2]。

[^2]：对结构定义的成员的引用（方法调用或对值或变量的访问）可以生成比非结构成员的等效代码慢得多的二进制代码。

在结构化修饰的方法声明中，任何值参数的类型只能是指修饰内部包含的类型参数或抽象类型。也就是说，它必须引用方法本身的类型参数，或者修饰内部的类型定义。 此限制不适用于方法的返回类型。

如果没有给出修饰，则默认的会添加空修饰,即 $T_1$ `with` … `with` $T_n$是  $T_1$ `with` … `with` $T_n \\{\\}$的简写.

复合类型也可以只包含$\\{ R \\}$ 的修饰，没有前面的组件类型。这种类型相当于 `AnyRef` $\\{ R \\}$.

###### 例

以下示例说明如何声明和使用具有包含具有结构声明的细化的参数类型的方法。

```scala
case class Bird (val name: String) extends Object {
        def fly(height: Int) = …
…
}
case class Plane (val callsign: String) extends Object {
        def fly(height: Int) = …
…
}
def takeoff(
            runway: Int,
      r: { val callsign: String; def fly(height: Int) }) = {
  tower.print(r.callsign + " requests take-off on runway " + runway)
  tower.read(r.callsign + " is clear for take-off")
  r.fly(1000)
}
val bird = new Bird("Polly the parrot"){ val callsign = name }
val a380 = new Plane("TZ-987")
takeoff(42, bird)
takeoff(89, a380)
```

虽然`Bird`和`Plane`没有除了`Object`之外的任何父类，但是方法`takeoff`的参数*r*是通过结构声明的细化来定义的，它接受任何声明了值`callsign`以及函数`fly`对象。

### 中缀类型

```ebnf
InfixType     ::=  CompoundType {id [nl] CompoundType}
```
_中缀类型_ $T_1$ `op` $T_2$ 由一个中缀运算符`op`组成，它应用于两个类型的操作数 $T_1$ 和 $T_2$ 。该类型相当于应用类型`op`$[T_1, T_2]$。中缀运算符“op”可以是任意标识符。

所有类型中缀运算符具有相同的优先级;必须使用括号进行排序。类型运算符的 [结合性](06-expressions.html#prefix,-infix,-and-postfix-operations)
是针对术语运算符确定的，以冒号‘:’结尾的类型运算符是右结合的;所有其他运算符都是左结合的。

在连续类型中缀操作$t_0 \, \mathit{op} \, t_1 \, \mathit{op_2} \, \ldots \, \mathit{op_n} \, t_n$里， 所有运算符$\mathit{op}\_1 , \ldots , \mathit{op}\_n$必须具有相同的结合性。 如果它们都是左结合的，则序列被解释为 $(\ldots (t_0 \mathit{op_1} t_1) \mathit{op_2} \ldots) \mathit{op_n} t_n$, 否则它被解释为$t_0 \mathit{op_1} (t_1 \mathit{op_2} ( \ldots \mathit{op_n} t_n) \ldots)$.

### 函数类型

```ebnf
Type              ::=  FunctionArgs ‘=>’ Type
FunctionArgs      ::=  InfixType
                    |  ‘(’ [ ParamType {‘,’ ParamType } ] ‘)’
```

类型 $(T_1 , \ldots , T_n) \Rightarrow U$ 表示那谢谢参数类型为$T1 , \ldots , Tn$ ，并产生一个类型为 $U$的结果函数。 如果只有一个参数类型则
$T \Rightarrow U$ 是 $(T) \Rightarrow U$的简写。$\Rightarrow T$形式的参数类型表示 $T$类型的[按名称调用参数](04-basic-declarations-and-definitions.html#by-name-parameters)参数.

函数类型与右侧相结合的，例如$S \Rightarrow T \Rightarrow U$与$S \Rightarrow (T \Rightarrow U)$相同。

函数类型是定义`apply`函数的类类型的简写。比如，$n$函数类型$(T_1 , \ldots , T_n) \Rightarrow U$是类类型`Function$_n$[T1 , … , $T_n$, U]`的简写。 这些类类型在Scala库中定义为0到22之间的 $n$ ，如下所示。


```scala
package scala
trait Function_n[-T1 , … , -T$_n$, +R] {
  def apply(x1: T1 , … , x$_n$: T$_n$): R
  override def toString = "<function>"
}
```

因此，函数类型在其结果类型中是[协变](04-basic-declarations-and-definitions.html#variance-annotations) 的，与参数类型是逆变的。

### 既存类型

```ebnf
Type               ::= InfixType ExistentialClauses
ExistentialClauses ::= ‘forSome’ ‘{’ ExistentialDcl
                       {semi ExistentialDcl} ‘}’
ExistentialDcl     ::= ‘type’ TypeDcl
                    |  ‘val’ ValDcl
```

既存类型的形式为`$T$ forSome { $Q$ }`
其中 $Q$ 是[类型声明](04-basic-declarations-and-definitions.html#type-declarations-and-type-aliases)的序列。

设$t_1[\mathit{tps}\_1] >: L_1 <: U_1 , \ldots , t_n[\mathit{tps}\_n] >: L_n <: U_n$
是 $Q$ 中声明的类型（任何类型参数部分`[ $\mathit{tps}_i$ ]`都可以缺失）。每种类型$t_i$的范围包括类型 $T$和既存子句 $Q$。类型变量 $t_i$ 被称为绑定在
`$T$ forSome { $Q$ }`类型中。在$T$类型中出现但未绑定在$T$中的类型变量在$T$中被认为是自由的。

`$T$ forSome { $Q$ }`的*类型实例*是$\sigma T$类型，其中$\sigma $是$t_1 , \ldots , t_n$的迭代，对于每个$i$，$\sigma L_i <: \sigma t_i <: \sigma U_i$。既存类型`$T$ forSome {$\,Q\,$}`表示的值集是其所有类型实例的值集的合集。

`$T$ forSome { $Q$ }` 的 _斯科伦化_ 是类型实例 $\sigma T$,其中 $\sigma$ 是 $[t_1'/t_1 , \ldots , t_n'/t_n]$的迭代， 每个$t_i'$介于 $\sigma L_i$ 和$\sigma U_i$之间的新的抽象类型.

#### 简化规则

既存类型遵守以下四个等价形式:

1. 可以合并既存类型中的多个for子句。
 例如,
`$T$ forSome { $Q$ } forSome { $Q'$ }`
相当于
`$T$ forSome { $Q$ ; $Q'$}`。
1. 未使用的限定可以被去掉。
例如,
`$T$ forSome { $Q$ ; $Q'$}`
如果$Q'$中定义的类型没有一个被 $T$ 或 $Q$ 引用，那么它们等价于
`$T$ forSome {$ Q $}`。
1. 空量化可以删除。 例如,
`$T$ forSome { }` 相当于 $T$.
1. 既存类型 `$T$ forSome { $Q$ }` 其中 $Q$ 包含一个子句 `type $t[\mathit{tps}] >: L <: U$` 相当于类型`$T'$ forSome { $Q$ }` ,$T'$是将$T$中所有$t$的[协变量](04-basic-declarations-and-definitions.html#variance-annotations)替换为$U$并且将$T$中所有的$t$的逆变量替换为$L$的结果。

#### 在值上的既存量化

为了语法上的方便，在既存类型上的绑定子句可以包括值声明`val $x$: $T$`。 既存类型`$T$ forSome { $Q$; val $x$: $S\,$;$\,Q'$ }`被视为类型
`$T'$ forSome { $Q$; type $t$ <: $S$ with Singleton; $Q'$ }`的简写，其中$t$ 是一个新类型名称，$T'$通过用$t$替换$T$中的每个`$x$.type`得到的。

#### 既存类型的占位符语法

```ebnf
WildcardType   ::=  ‘_’ TypeBounds
```
Scala支持既存类型的占位符语法。 通配符类型的形式为
`_$\;$>:$\,L\,$<:$\,U$`。 两个边界都可以省略。如果缺少下限子句`>:$\,L$`，
则假定为`>:$\,$scala.Nothing`。如果缺少上限子句`<:$\,U$`，则假定为`<:$\,$scala.Any`。 通配符类型是既存量化类型变量的简写，其中既存量化是内涵的。

通配符类型只能作为为参数化类型的类型参数出现。设 $T = p.c[\mathit{targs},T,\mathit{targs}']$ 是一个参数化类型，其中
$\mathit{targs}, \mathit{targs}'$ 可以为空
$T$ 是一个 通配符类型为 `_$\;$>:$\,L\,$<:$\,U$`. 那么 $T$ 相当于以下既存类型

```scala
$p.c[\mathit{targs},t,\mathit{targs}']$ forSome { type $t$ >: $L$ <: $U$ }
```

其中 $t$ 是一个新的类型变量。配符类型也可能显示为 [中缀类型](#中缀类型)
, [函数类型](#函数类型)
或[元组类型](#元组类型)的一部分。它们的扩展也就是等效参数化类型的扩展。

###### 例

假设类定义

```scala
class Ref[T]
abstract class Outer { type T }
```

以下是既存类型的一些示例：

```scala
Ref[T] forSome { type T <: java.lang.Number }
Ref[x.T] forSome { val x: Outer }
Ref[x_type # T] forSome { type x_type <: Outer with Singleton }
```

此列表中的最后两个类型是等效的。使用通配符语法的上述第一种类型的替代公式是：

```scala
Ref[_ <: java.lang.Number]
```

###### 例

 `List[List[_]]` 类型等同于type t

```scala
List[List[t] forSome { type t }]
```

###### 例

假定协变类型

```scala
class List[+T]
```

类型

```scala
List[T] forSome { type T <: java.lang.Number }
```

等同于（通过上面的简化规则四）

```scala
List[java.lang.Number] forSome { type T <: java.lang.Number }
```

这相当于（通过上面的简化规则二和三）`List[java.lang.Number]`.

## 非值类型

下面解释的类型不表示值集，也不在程序中明确显示。 它们在本报告中作为已定义标识符的内部类型引入。

### 方法类型

*方法类型* 在内部表示为$(\mathit{Ps})U$，其中$(\mathit{Ps})$是一个类型序列$(p_1:T_1 , \ldots , p_n:T_n)$ $n \geq 0$，$U$是（值或方法）类型，此类型表示接受名为$p_1 , \ldots , p_n$ ，类型为$T_1 , \ldots , T_n$ 的参数命名方法，并返回类型为$u$的结果。

方法类型是右侧关联的： $(\mathit{Ps}\_1)(\mathit{Ps}\_2)U$ 被视为$(\mathit{Ps}\_1)((\mathit{Ps}\_2)U)$.

不带任何参数的方法类型是一种特殊情况。可以写为`=> T`形式。无参数方法名称表达式将会在每次名称被引用时求值。

方法类型不作为值的类型存在。如果方法名称以值的方式被引用，则其类型将[自动转换](06-expressions.html#implicit-conversions)为相应的函数类型。

###### 例  

声明

```scala
def a: Int
def b (x: Int): Boolean
def c (x: Int) (y: String, z: String): String
```

产生类型

```scala
a: => Int
b: (Int) Boolean
c: (Int) (String, String) String
```

### 多态方法类型

多态方法类型在内部表示为`[$\mathit{tps}\,$]$T$`，其中`[$\mathit{tps}\,$]`是类型参数部分`[$a_1$ >: $L_1$ <: $U_1 , \ldots , a_n$ >: $L_n$ <: $U_n$]`，$n \geq 0$，$T$是一个（值或方法）类型。这种类型表示[依据](#参数化类型) 类型参数 `$S_1 , \ldots , S_n$`蚕食类型为$T$的结果，类型参数 `$S_1 , \ldots , S_n$`的下边界`$L_1 , \ldots , L_n$ `，上边界`U_1 , \ldots , U_n$`。

###### 例

声明

```scala
def empty[A]: List[A]
def union[A <: Comparable[A]] (x: Set[A], xs: Set[A]): Set[A]
```

产生类型

```scala
empty : [A >: Nothing <: Any] List[A]
union : [A >: Nothing <: Comparable[A]] (x: Set[A], xs: Set[A]) Set[A]
```

### 类型构造函数

类型构造函数在内部表示非常类似于多态方法类型。`[$\pm$ $a_1$ >: $L_1$ <: $U_1 , \ldots , \pm a_n$ >: $L_n$ <: $U_n$] $T$`表示一个期望是[类型构造函数参数](04-basic-declarations-and-definitions.html#type-parameters)或与相应类型参数子句绑定的[抽象类型构造函数](04-basic-declarations-and-definitions.html#type-declarations-and-type-aliases)的类型。

###### 例

思考一下`Iterable [+ X]`类的这个片段：

```scala
trait Iterable[+X] {
  def flatMap[newType[+X] <: Iterable[X], S](f: X => newType[S]): newType[S]
}
```

从概念上讲，类型构造函数`Iterable`是匿名类型`[+ X] Iterable [X]`的名称，它可以在`flatMap`中传递给`newType`类型构造函数参数。


## 基本类型和成员定义

类成员的类型取决于成员的引用方式。 这里主要的是三个概念，即：
1. $T$类型的基本类型集的概念。
1. 从前缀类型$S$看到的某些类$C$中的类型$T$  
1. 类型$T$的成员绑定集合

这些概念相互递归定义如下：
1. 类型的基类型集是一组类类型，如下所示。

  - 具有父类$T_1 , \ldots , T_n$的类型 $C$ 的基本类型是 $C$本身，以及复合类型`$T_1$ with … with $T_n$ { $R$ }`的基本类型。

  - 类型别名的基类型是其别名的类型的基类型。

  - 抽象类型的基类型是其上限的基类型。

  - 参数化类型`$C$[$T_1 , \ldots , T_n$]`的基类型是 $C$ 类型的基类型，其中 $C$ 类型的每一个类型参数 $a_i$ 都被相应的参数类型 $T_i$ 替换。

  - 单例类型`$p$.type`的基类型是$p$基本类型。

  - 复合类型`$T_1$ with $\ldots$ with $T_n$ { $R$ }`的基类型是所有$T_i$'的基类的 *缩减合并*。这意味着：让多集$\mathscr{S}$ 成为所有$T_i$的基类型的集合。如果$\mathscr{S}$包含同一类的几个类型实例，比如说`$S^i$#$C$[$T^i_1 , \ldots , T^i_n$]`$(i \in I)$，则所有这些实例都被其他一致的实例替换。 如果不既存这样的实例，则会出错。 由此可见，简化联合（如果既存）会产生一组类类型，其中不同类型是不同类的实例。

  - 选择类型`$S$#$T$`的基类型确定如下：如果 $T$是别名或抽象类型，则前面的子句就会被应用。否则$T$必须是(合理参数化)类类型，它在某个类$B$中定义。那么`$S$#$T$`的基类型是从前缀类型$S$看到的$B$中的$T$的基本类型。

  - 既存类型`$T$ forSome { $Q$ }`的基类型是所有的`$S$ forSome { $Q$ }`类型，其中$S$是$T$的基本类型。

1. *只有前缀类型$S$具有类$C$的类型实例作为基类型* ,比如`$S'$#$C$[$T_1 , \ldots , T_n$]`，从某些前缀类型$S$看到的类$C$中的$T$概念才有意义。然后我们定义如下:
    - 如果 `$S$ = $\epsilon$.type`, 那么从$S$看到的$C$中的$T$是$T$本身。
    - 否者，如果$S$是既存类型`$S'$ forSome { $Q$ }`，从$S'$看到的$C$中的$T$将会是$T'$。那么从$S$中看到的$T$中的$C$是`$T'$ forSome {$\,Q\,$}`.
    - 或者, 如果 $T$ 是某个类 $D$ 的第$i$个类型参数, 那么
        - 如果 $S$ 有一个基本类型 `$D$[$U_1 , \ldots , U_n$]`, 对于`[$U_1 , \ldots , U_n$]`类型参数 , 那么$S$中看到的$C$中的$T$是$U_i$
        - 或者，如果$C$定义在$C'$类中，那么$S$中看到的$C$的$T$与在$S'$中看到的$C'$的$T$相同。
        - 或者，如果$C$没有在其他类中定义，那么从$S$中看到的$C$中的$T$就是$T$本身。
    - 或者，如果$T$是某个类$D$的单例类型` $D$.this.type`，那么
        - 如果$D$是$C$的子类，$S$的基类型中有$D$的类型实例，那么从$S$中看到$C$中的$T$就是$S$。
        - 或者，如果$C$定义在$C'$类中，那么从$S$中到的$C$的$T$与从$S'$中看到的$C'$的$T$相同。
        - 或者,如果$C$没有在另一个类中定义，那么从$S$中看到的$C$中的$T$就是$T$本身。
    - 如果$T$是其他类型，则对其所有类型组件执行所描述的映射。

    如果$T$是一个可能参数化的类类型，其中$T$的类在某些其他类$D$中定义，而$S$是某种前缀类型，那么我们使用"$T$ seen from $S$"作为  "$T$ in $D$ seen from $S$"的简写。

1.  $T$ 类型的*成员绑定*是
   1. 所有绑定$d$使得在$T$的基类型中既存某个类$C$的类型实例，并且在C中既存定义或声明d',使得$d’$通过从$T’$中看到的$C’$以$C’$取代每个类型$T’$而产生$d’$
   2. 类型[修饰](#复合类型)的绑定，如果有的话。

   类型映射`S#T`的定义是`S`中的类型`T`的成员绑定$d_T$。在这种情况下，我们还说`S#T`是由$d_T$定义的.

## 类型之间的关系

我们定义类型之间的以下关系

| 名称         | 象征           | 解释                            |
|-------------|----------------|--------------------------------|
| 等价         | $T \equiv U$   | $T$和$U$在所有情况下都可以互换。   |
| 一致性       | $T <: U$       | 类型$T$符合(是的子类型)类型$U$     |
| 弱一致性      | $T <:_w U$    | 增加原始数字类型的一致性。         |
| 兼容性       |                | 类型$T$在转换后符合类型$U$。      |

### 等价

类型之间的等价$（\ equiv）$是最小的一致性[^一致性]，具有一下特点：

- 如果 $t$ 由类型别名 `type $t$ = $T$`定义, 那么 $t$ 等价于$T$.
- 如果路径 $p$ 有一个单例类型 `$q$.type`, 那么 `$p$.type $\equiv q$.type`.
- 如果 $O$ 是一个对象的定义,而且 $p$ 是一个只路径或仅包括包或对象的选择器并以$O$结尾, 那么 `$O$.this.type $\equiv p$.type`.
- 两个[复合类型](#复合类型)等价的条件是，它们的组件的序列是相等的，并且以相同的顺序出现，并且它们的修饰也相等的。如果两个修饰绑定了同样的命名，并且每个声明的实体的修饰符，类型和边界也相等，则两个修饰是相等的。
- 两个[方法类型](#方法类型)是等价的条件是:
    - 都是含蓄的,或两者都不是[^含蓄的];
    - 它们具有等效的结果类型;
    - 它们有相同数量的参数;
    - 相应的参数具有等价的类型。请注意，参数的名称对于方法类型的等价性无关紧要。
- 如果两个[多态方法类型](＃多态方法类型)具有相同数量的类型参数，并且在将一组类型参数重命名为另一组之后，结果类型以及相应的下限和上限，则它们是相等的。
- 如果两个[既存类型](#既存类型)具有相同数量的量词，并且在将一个类型量词列表重命名为另一个之后，量化类型以及相应量词的下限和上限是等价的，则他们是相等的。
- 两个[类型构造函数](类型构造函数)是相同的，如果它们具有相同数量的类型参数，并且在将一个类型参数列表重命名为另一个之后，结果类型以及变化，相应类型的下限和上限是相等的。

[^一致性]：一致性是一种等价关系下封闭环境的形成。
[^含蓄的]：如果定义它的参数部分以`implicit`关键字开头，则方法类型是隐含的。

### 一致性

一致性关系$(<:)$是满足以下条件的最小传递关系。

- 一致性包括等价。如果$T = U$，那么$T <: U$。
- 对于任意值类型$T$，有`scala.Nothing <: $T$ <: scala.Any`.
- 对于任意的类型构造函数 $T$ (具有任意数量的类型参数), 由`scala.Nothing <: $T$ <: scala.Any`.
- 对于每个类型 $T$ ,使得 `$T$ <: scala.AnyRef` 具有 `scala.Null <: $T$`.
- 类型变量或抽象类型 $t$ 与其上限一致，其下限与 $t$一致.
- 类类型或参数化类型与其任何基类型一致。
- 单例类型 `$p$.type` 和 $p$路径的类型一致。
- 单例类型 `$p$.type` 和 `scala.Singleton`类型一致.
- 如果$T$和$U$一致，则类型映射`$T$ T$`和`$U$ T$`一致。
- 参数化类型`$T$[$T_1$，…，$T_n$]`与`$T$[$U_1$，…，$U_n$]`一致的条件是$i \in { 1 , \ldots , n }$:
 1. 如果$T$的第$i$个类型参数声明为协变，则 $T_i <: U_i$.
 1. 如果$T$的第$i$个类型参数声明为逆变的，那么  $U_i <: T_i$.
 1. 如果$T$的第$i$个类型参数既不是协变的，也不是逆变的，那么 $U_i \equiv T_i$.
- 复合类型 `$T_1$ with $\ldots$ with $T_n$ {$R\,$}` 与其每个组件类型 $T_i$一致.
- 如果`$U_1$ with $\ldots$ with $U_n$ {$R\,$}` 中的每一个类型或值$x$的绑定$d$都既存一个包含$d$的$T$中$x$成员绑定，则$T$与复合类型 `$U_1$ with $\ldots$ with $U_n$ {$R\,$}`一致.
- 如果既存类型`$T$ forSome {$\,Q\,$}`的[斯柯伦化](既存类型)和 $U$一致，既存类型`$T$ forSome {$\,Q\,$}`和$U$一致.
- 如果$T$与`$U$ forSome {$\,Q\,$}`的[类型实例](既存类型)之一一致，则$T$类型与既存类型`$U$ forSome {$\,Q\,$}`一致。

- 如果 $T_i \equiv T_i'$$i \in { 1 , \ldots , n}$，$U$和$U'$一致，那么方法类型 $(p_1:T_1 , \ldots , p_n:T_n) U$和方法类型$(p_1':T_1' , \ldots , p_n':T_n') U'$一致。

- 多态类型$[a_1 >: L_1 <: U_1 , \ldots , a_n >: L_n <: U_n] T$与复合多态类型$[a_1 >: L_1' <: U_1' , \ldots , a_n >: L_n' <: U_n'] T'$一致的条件是，假设$L_1' <: a_1 <: U_1' , \ldots , L_n' <: a_n <: U_n'$，$T <: T'$。$L_i <: L_i'$,$U_i' <: U_i$，$i \in \{ 1 , \ldots , n \}$.

- 类型构造函数$T$ 和 $T'$ 遵循类似的规律。我们通过类型参数子句$[a_1 , \ldots , a_n]$和  $[a_1' , \ldots , a_n']$来区分$T$和$T'$，其中 $a_i$或$a_i'$可以包含差异注释，高阶类型参数子句和边界。那么，$T$和$T'$一致的条件是，如果任意列表$[t_1 , \ldots , t_n]$ --具有声明的差异，边界和高阶类型参数子句-- $T'$的有效类型参数是$T$和$T[t_1 , \ldots , t_n] <: T'[t_1 , \ldots , t_n]$的类型参数的有效列表。请注意，这需要
    - $a_i$的边界必须弱于$a'_i$的相应边界。
    - $a_i$的差异必须与$a'_i$的差异匹配，其中协变匹配协变，逆变匹配逆变，任何差异与无差异一致。
    - 这些限制递归地应用于相应的高阶类型参数子句$a_i$和$a'_i$。_

在以下某个条件下,类类型$C$的复合类型的声明或定义将包括另外一个类类型 $C'$的符合类型的同名声明：

- 如果$T <: T'$,一个值声明或定义定义了一个类型为 $T$的命名$x$,包括一个值或者方法声明定义了类型为 $T'$的$x$。
- 如果$T <: T'$，,一个方法声明或定义定义了一个类型为$T$的命名$x$,包括一个方法声明定义了类型为$T'$的$x$。

- 如果$T \equiv T'$，则类型别名`type $t$[$T_1$ , … , $T_n$] = $T$`包含类型别名`type $t$[$T_1$ , … , $T_n$] = $T'$`。
- 如果$L' <: L$并且 $U <: U'$，类型声明`type $t$[$T_1$ , … , $T_n$] >: $L$ <: $U$`包含类型声明`type $t$[$T_1$ , … , $T_n$] >: $L'$ <: $U'$`。
- 只有$L <: t <: U$，绑定类型名称$t$的类型或类定义包含抽象类型声明`type t[$T_1$ , … , $T_n$] >: L <: U`。


## 基本类型和成员定义

类成员的类型取决于成员的引用方式。 这里主要的是三个概念，即：
1. $T$类型的基本类型集的概念。
1. 从前缀类型$S$看到的某些类$C$中的$T$类型的概念。  
1. 某些类型 $T$的成员绑定集的概念.

这些概念相互递归定义如下：
1. 类型的基类型集是一组类类型，如下所示。

  - 具有父类$T_1 , \ldots , T_n$的类型 $C$ 的基本类型是 $C$本身，以及复合类型`$T_1$ with … with $T_n$ { $R$ }`的基本类型。

  - 混别类型的基类型是其别名的基类型。

  - 抽象类型的基类型是其上限的基类型。

  - 参数化类型`$C$[$T_1 , \ldots , T_n$]`的基类型是 $C$ 类型的基类型，其中 $C$ 类型参数 $a_i$ 的每次出现都被相应的参数类型 $T_i$ 替换。

  - 单例类型`$p$.type`的基类型是$p$类型。

  - 复合类型`$T_1$ with $\ldots$ with $T_n$ { $R$ }`的基类型是所有$T_i$'的基类的 *简化联合*。这意味着：让多集$\mathscr{S}$ 成为所有$T_i$的基类型的多集合。如果$\mathscr{S}$包含同一类的几个类型实例，比如说`$S^i$#$C$[$T^i_1 , \ldots , T^i_n$]`$(i \in I)$，则所有这些实例都被其中一个实例替换，这些实例符合所有其他实例。 如果不既存这样的实例，则会出错。 由此可见，简化联合（如果既存）会产生一组类类型，其中不同类型是不同类的实例。

  - 类型选择`$S$#$T$`的基类型确定如下。如果 $T$是别名或抽象类型，则前面的条款适用。或者，$T$必须是(合理参数化)类类型，它在某个类$B$中定义。然后`$S$#$T$`的基类型是从前缀类型$S$看到的$B$中的$T$的基本类型。

  - 既存类型`$T$ forSome { $Q$ }`的基类型都是`$S$ forSome { $Q$ }`类型，其中$S$是$T$的基本类型。

1. *只有前缀类型$S$具有类$C$的类型实例作为基类型* ,比如`$S'$#$C$[$T_1 , \ldots , T_n$]`，从某些前缀类型$S$看到的类$C$中的$T$概念才有意义。然后我们定义如下:
    - 如果 `$S$ = $\epsilon$.type`, 则从 $T$看到的$C$中的$T$是$T$本身。
    - 或者，如果$S$是既存类型`$S'$ forSome { $Q$ }`，并且从 $S'$ 看到的$C$中的$T$是$T'$。那么从$S$中看到的$C$中的$T$是`$T'$ forSome {$\,Q\,$}`.
    - 或者, i如果 $T$ 是某个类 $D$ 的$i$'类型参数, 那么
        - 如果 $S$ 有一个基本类型 `$D$[$U_1 , \ldots , U_n$]`, 对于某些类型参数 `[$U_1 , \ldots , U_n$]`, 那么$S$中$C$中的$T$是$U_i$
        - 或者，如果$C$定义在$C'$类中，那么$S$中$C$的$T$与$S$中$C$的$T$相同。
        - 或者，如果$C$没有在另一个类中定义，那么$S$中$C$中的$T$从$S$中看到的$T$就是$T$本身。
    - 或者，如果$T$是某个类$D$的单例类型` $D$.this.type`，那么
        - 如果$D$是$C$的子类，$S$的基类型中有$D$的类型实例，那么$S$中$C$中的$T$就是$S$。
        - 或者，如果$C$定义在$C'$类中，那么$S$中$C$的$T$与$S$中$C$的$T$相同。
        - 或者,如果$C$没有在另一个类中定义，那么$S$中$C$中的$T$从$S$中看到的$T$就是$T$本身。
    - 如果$T$是其他类型，则对其所有类型组件执行所描述的映射。

    如果$T$是可能参数化的类类型，其中$T$'的类在某些其他类 $D$中定义，而$S$是某种前缀类型，那么我们使用"$T$ seen from $S$"作为  "$T$ in $D$ seen from $S$"的简写。

1.  $T$ 类型的*成员绑定*是
   1. 所有绑定$d$使得在$T$的基类型中既存某个类$C$的类型实例，并且在C中既存定义或声明d',使得$d’$通过从$T’$中看到的$C’$以$C’$取代每个类型$T’$而产生$d’$
   2. 类型[细化](#复合类型)的所有绑定，如果有的话。

   类型映射`S#T`的定义是`S`中的类型`T`的成员绑定$d_T$。在这种情况下，我们还说`S#T`是由$d_T$定义的.

## 类型之间的关系

我们定义类型之间的以下关系

| 名称         | 象征           | 解释                            |
|-------------|----------------|--------------------------------|
| 等价         | $T \equiv U$   | $T$和$U$在所有情况下都可以互换。   |
| 一致性       | $T <: U$       | 类型$T$符合(是的子类型)类型$U$     |
| 弱一致性      | $T <:_w U$    | 增加原始数字类型的一致性。         |
| 兼容性       |                | 类型$T$在转换后符合类型$U$。      |

### 等价

类型之间的等价$（\ equiv）$是最小的一致[^一致]，以下使得成立：

- 如果 $t$ 由类型别名 `type $t$ = $T$`定义, 那么 $t$ 等价于$T$.
- 如果路径 $p$ 有一个单例类型 `$q$.type`, 那么 `$p$.type $\equiv q$.type`.
- 如果 $O$ 是由对象定义的,而且 $p$ 是一个只包含包或对象爱嗯选择器并以 $O$结尾的路径, 那么 `$O$.this.type $\equiv p$.type`.
- 两个[复合类型](#复合类型)是等价的，如果它们的组件的序列是成对等价的，并且以相同的顺序出现，并且它们的细化是等价的。如果它们绑定相同的名称，则两个细化是等效的，并且每个声明的实体的修饰符，类型和边界在两个细化中都是等效的。
- 两个[方法类型](#方法类型)是等价的，如果:
    - 都是隐式的,或者两者都是[^隐式];
    - 它们具有等效的结果类型;
    - 它们有相同数量的参数; and
    - 相应的参数具有等价的类型。请注意，参数的名称对于方法类型的等价性无关紧要。
- 如果两个[多态方法类型](＃多态方法类型)具有相同数量的类型参数，并且在将一组类型参数重命名为另一组之后，结果类型以及相应的下限和上限，则它们是等效的 类型参数是等价的。
- 如果两个[既存类型](#既存类型)具有相同数量的量词，则它们是等价的，并且在将一个类型量词列表重命名为另一个之后，量化类型以及相应量词的下限和上限是等价的。
- 两个[类型构造函数](类型构造函数)是相同的，如果它们具有相同数量的类型参数，并且在将一个类型参数列表重命名为另一个之后，结果类型以及方差，相应类型的下限和上限 参数是等价的。

[^congruence]：一致性是一种等价关系下封闭环境的形成。
[^ implicit]：如果定义它的参数部分以`implicit`关键字开头，则方法类型是隐式的。

### 一致性

一致性关系$(<:)$是满足以下条件的最小传递关系。

- 一致性包括等价。如果$T = U$，那么$T <: U$。
- 对于每一个值类型$T$，`scala.Nothing <: $T$ <: scala.Any`.
- 对于每个类型构造函数 $T$ (具有任意数量的类型参数), `scala.Nothing <: $T$ <: scala.Any`.
- 对于每个类型 $T$ ,使得 `$T$ <: scala.AnyRef` 具有 `scala.Null <: $T$`.
- 类型变量或抽象类型 $t$ 符合其上限，其下限符合 $t$.
-类类型或参数化类型符合其任何基类型。
- 单例类型 `$p$.type` 符合 $p$路径的类型。.
- 单例类型 `$p$.type` 符合 `scala.Singleton`类型.
- 如果$T$符合$U$，则类型映射`$T$ T$`符合`$U$ T$`。
- 参数化类型`$T$[$T_1$，…，$T_n$]`符合`$T$[$U_1$，…，$U_n$]`，如果以下三个条件包含于$i \in { 1 , \ldots , n }$:
 1. 如果$T$的$i$'th类型参数声明为协变，则 $T_i <: U_i$.
 1. 如果$T$的$i$'th类型参数声明为逆变的，那么  $U_i <: T_i$.
 1. 如果$T$的$i$'th类型参数既不是协变的，也不是逆变的，那么 $U_i \equiv T_i$.
- 复合类型 `$T_1$ with $\ldots$ with $T_n$ {$R\,$}` `符合每个组件类型 $T_i$.
- 如果$i \in { 1 , \ldots , n }$ 为$T <: U_i$，$R$ 中的每一个类型或值$x$的绑定$d$斗既存一个包含$d$的$T$中 $x$成员绑定，则$T$符合复合类型 `$U_1$ with $\ldots$ with $U_n$ {$R\,$}`.
- 如果[斯柯伦化](既存类型)符合 $U$，既存类型`$T$ forSome {$\,Q\,$}`符合$U$.
- 如果$T$符合`$U$ forSome {$\,Q\,$}`的[类型实例](既存类型)之一，则$T$类型符合既存类型`$U$ forSome {$\,Q\,$}`。

- 如果$i \in { 1 , \ldots , n}$和$U$的$T_i \equiv T_i'$符合 $(p_1:T_1 , \ldots , p_n:T_n) U$ ，则方法类型$(p_1:T_1 , \ldots , p_n:T_n) U$符合$(p_1':T_1' , \ldots , p_n':T_n') U'$。

- 多态类型$[a_1 >: L_1 <: U_1 , \ldots , a_n >: L_n <: U_n] T$复合多态类型$[a_1 >: L_1' <: U_1' , \ldots , a_n >: L_n' <: U_n'] T'$一致，前提是$L_1' <: a_1 <: U_1' , \ldots , L_n' <: a_n <: U_n'$有$T <: T'$。$L_i <: L_i'和$U_i' <: U_i$用于$i \in \{ 1 , \ldots , n \}$.

- 类型构造函数$T$ 和 $T'$ 遵循类似的规律。我们通过类型参数子句$[a_1 , \ldots , a_n]$和  $[a_1' , \ldots , a_n']$来表征$T$和$T'$，其中 $a_i$或$a_i'$可以包含方差注释，告诫类型参数子句和边界。然后$T$符合$T'$，如果任意列表$[t_1 , \ldots , t_n]$-- 具有声明的差异，边界和高阶类型参数子句--$T'$的有效类型参数是$T$和$T[t_1 , \ldots , t_n] <: T'[t_1 , \ldots , t_n]$的类型参数的有效列表。请注意，这需要
    - $a_i$的边界必须弱于$a'_i$声明的相应边界。
    - $a_i$的方差必须与$a'_i$的方差匹配，其中协方差匹配协方差，逆变匹配逆变，任何方差匹配不变性。
    - 递归地，这些限制适用于相应的高阶类型参数子句$a_i$和$a'_i$。

在某些复合类型的类类型$C$中的声明或定义*包含*在某些复合类型或classtype $C'$中的同名的另一个声明，如果下列情况之一则成立。

- 定义名称$x$且类型为$T$的值声明或定义包含一个值的或方法声明，它定义$x$，类型为$T'$，提供$T <: T'$.
- 定义名称$x$且类型为$T$的方法声明或定义包含一个方法声明，该声明定义$x$类型$T'$，提供$T <: T'$.

- 如果$T \equiv T'$，则类型别名`type $t$[$T_1$ , … , $T_n$] = $T$`包含类型别名`type $t$[$T_1$ , … , $T_n$] = $T'$`。
- 只有$L' <: L$ and $U <: U'$，类型声明`type $t$[$T_1$ , … , $T_n$] >: $L$ <: $U$`包含类型声明`type $t$[$T_1$ , … , $T_n$] >: $L'$ <: $U'$`。
- 只有$L <: t <: U$，绑定类型名称$t$的类型或类定义包含抽象类型声明`type t[$T_1$ , … , $T_n$] >: L <: U`。


#### 最小上限和最大下线


#### 最小上限和最大下线

$(<:)$ 关系在类型之间形成预订,即它具有传递性和反射性。这允许我们根据该顺序定义一组类型的*最小上限*和*最大下限*。一组类型的最小上限或最大下限并不总是既存。例如，考虑类定义：

```scala
class A[+T] {}
class B extends A[B]
class C extends A[C]
```

然后类型`A[Any], A[A[Any]], A[A[A[Any]]], ...`形成`B`和`C`的上界的递减序列。 最小上限将是该序列的无限极限，它不是Scala类型。 由于这样的情况通常无法检测，因此Scala编译器可以自由地拒绝具有指定为最小上限或最大下限的类型的术语，并且该约束将比某些编译器设置限制更复杂[^ 4]。

最小上限或最大下限也可能不是唯一的。 例如，`A with B`和`B with A`都是`A`和`B`的最大下界。如果既存多个最小上限或最大下限，则Scala编译器可以自由选择其中任何一个。

[^4]：当前的Scala编译器将这种边界中参数化的嵌套级别限制为最多比操作数类型的最大嵌套级别深两级

### 弱一致性

在某些情况下，Scala使用更一般的一致性关系。类型$S$*弱一致性*符合$T$类型，写入$S <:_w T$.如果$S <: T$或者$S$和$T$都是原始的数字类型，并且在下面的顺序中$S$ 在 $T$之前。

```scala
Byte  $<:_w$ Short
Short $<:_w$ Int
Char  $<:_w$ Int
Int   $<:_w$ Long
Long  $<:_w$ Float
Float $<:_w$ Double
```

*弱上限*是弱一致性的最小上限。

### 兼容性
如果$T$(或其相应的函数类型)在应用[eta扩展](06-expressions.html#eta-expansion)后[弱符合](#弱一致)$U$,则$T$类型与$U$类型兼容。如果$T$是方法类型，则将其转换为相应的函数类型。如果类型不符合要求，则按顺序检查一下备选方法：
  - [查看应用程序](07-implicits.html#views)：从$T$到$U$的隐含视图;
  - 删除按名称修饰符:如果$U$的形式为`$=> U'$`($T$不是)，则`$T <:_w U'$`。
  - SAM转换:如果$T$对应一个函数，$U$声明一个抽象方法，其类型[符合](06-expressions.html#sam-conversion)函数类型$U'$，`$T <:_w U'$`。

<!--- TODO: include other implicit conversions in addition to view application?

  trait Proc { def go(x: Any): Unit }

  def foo(x: Any => Unit): Unit = ???
  def foo(x: Proc): Unit = ???

  foo((x: Any) => 1) // works when you drop either foo overload since value discarding is applied

-->

#### 例

##### 通过SAM转换实现功能兼容

鉴于定义

```scala
def foo(x: Int => String): Unit
def foo(x: ToString): Unit

trait ToString { def convert(x: Int): String }
```

应用程序`foo((x: Int) => x.toString)`[确定](06-expressions.html#overloading-resolution)到第一个从在，因为他更具体
  - `Int => String` 与 `ToString` 兼容 -- 当期望类型为 `ToString`的值时, 你可以将函数文字从`Int`传递给 `String`, 因为它将被SAM转换为所述函数;
  - `ToString` 与 `Int => String`不兼容， -- 当期望从 `Int` 到 `String`的函数时, 你可能不会传递 `ToString`.

## 不稳定的类型

类型波动率近似于类型参数或类型的抽象类型实例不具有任何非空值的可能性。值类型的值成员不能出现在[路径](路径)中。

如果类型属于以下四种类别之一，则为*值类型*：

如果以下两个条件之一成立，则复合类型`$T_1$ with … with $T_n$ {$R\,$}`是不稳定的。

1. $T_2 , \ldots , T_n$ 的其中之一是类型参数或抽象类型。
1. $T_1$是一个抽象类型，并且细化 $R$或$T_j$类型$j > 1$为复合类型提供抽象成员。
1. $T_1 , \ldots , T_n$ 中的一个是单例类型。


这里面，类型$S$将抽象成员贡献给$T$类型，如果$S$包含一个也是$T$成员的抽象成员。如果$R$包含一个抽象声明，他也是$T$的成员，则精简$R$将抽象成员贡献给$T$类型。

如果类型指示符是不稳定性类型的别名，或者它指定了类型参数或具有不稳定性类型作为其上限的抽象类型，则它是不稳定性类型。

如果路径$p$的基础类型是不稳定性的，则单例类型`$p$.type`易失性的。

如果$T$是不稳定性的，则既存类型`$T$ forSome {$\,Q\,$}`是易变的。

## 类型消除

如果类型包含类型参数或类型变量，则称为*泛型*。*类型擦除*是从
(可能是通用的)类型到非泛型类型的映射。我们写$|T|$来删除$T$类型。擦除映射定义如下：

- 别名类型的擦除是其右侧的擦除。.
- 抽象类型的擦除是其上限的擦除。
- 参数化类型`scala.Array$[T_1]$`的擦除是`scala.Array$[|T_1|]$`。
- 每个其他参数化类型 $T[T_1 , \ldots , T_n]$ 的擦除是 $|T|$.
- 单例类型`$p$.type` 的擦除是$p$类型.
- 类型映射 `$T$#$x$` 的删除是 `|$T$|#$x$`.
- 一个复合型的擦除`$T_1$ with $\ldots$ with $T_n$ {$R\,$}`是$T_1 , \ldots , T_n$.的交点支配的擦除。
- 既存类型`$T$ forSome {$\,Q\,$}`的擦除是 $|T|$.

下面计算类型$T_1 , \ldots , T_n$列表的交集占领者。
设$T_{i_1} , \ldots , T_{i_m}$是$T_i$类型的子序列，他们不是某些其他类型$T_j$的超类型。如果这个子序列包含一个类型指示符$T_c$ ，它指得是一个不是特征的类，则交集支配者是$T_c$。否则，交集支配者是子序列的第一个元素$T_{i_1}$.
