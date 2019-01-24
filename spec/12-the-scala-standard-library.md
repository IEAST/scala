---
title: Standard Library
layout: default
chapter: 12
---

# Scala标准库

Scala标注库包括包`scala`和一些类与模块。下面描述了其中一些类


![Class hierarchy of Scala](public/images/classhierarchy.png)

## 跟类

此层次结构的跟是由`Any`类创建。Scala执行环境中的每个类都直接或间接地从此类继承。`Any`类有两个直接子类：`AnyRef` 和 `AnyVal`

子类`AnyRef`表示在底层主机系统中表示为对象的所有值。用其他语言编写的类继承自`scala.AnyRef`。

`AnyVal`类的预定义子类描述了在底层主机系统中没有作为对象实现的值。

未确定从`AnyVal`继承的用户定义的Scala类直接或间接地从`AnyRef`继承。它不能同时继承`AnyRef`和`AnyVal`。

类`AnyRef`和`AnyVal`只需要提供在类`Any`中声明的成员，但实现可能会向这些类添加特定与主机的方法(例如，实现可以使自己的对象root类来标识`AnyRef`)。

这些根类的签名由以下定义描述:


```scala
package scala
/** 通用根类 */
abstract class Any {

  /** 定义相等，在这里是抽象的 */
  def equals(that: Any): Boolean

  /** 值的语意相等 */
  final def == (that: Any): Boolean  =
    if (null eq this) null eq that else this equals that

  /** 值的语义不等 */
  final def != (that: Any): Boolean  =  !(this == that)

  /** Hash code; 在这里是抽象的 */
  def hashCode: Int = $\ldots$

  /** 文本表示，在这里是抽象的 */
  def toString: String = $\ldots$

  /** 类型测试，需要按照下面的方式内联 */
  def isInstanceOf[a]: Boolean

  /** 类型转换，需要按照下面的方式内联 */ */
  def asInstanceOf[A]: A = this match {
    case x: A => x
    case _ => if (this eq null) this
              else throw new ClassCastException()
  }
}

/** T所有值类型的根类 */
final class AnyVal extends Any

/** 所有引用类型的根类 */
class AnyRef extends Any {
  def equals(that: Any): Boolean      = this eq that
  final def eq(that: AnyRef): Boolean = $\ldots$ // 引用相等
  final def ne(that: AnyRef): Boolean = !(this eq that)

  def hashCode: Int = $\ldots$     // 由内存地址计算得来
  def toString: String  = $\ldots$ // 由hashcode和类名得来

  def synchronized[T](body: => T): T // 在锁定`this`时执行`body`
}
```

类型测试 `$x$.isInstanceOf[$T$]` 等同于类型化模式匹配

```scala
$x$ match {
  case _: $T'$ => true
  case _ => false
} //'
```

其中$T'$的类型和$T$相同，除非$T$的格式为$D$或者$D[\mathit{tps}]$，其中$D$是某些外类$C$的类型成员。在这种情况下，$T'$分别是`$C$#$D$`(或`$C$#$D[tps]$`)，而$T$本身将扩展为`$C$.this.$D[tps]$`。换句话说， `isInstanceOf`测试不会检查类是是否具有相同的封闭实例。

如果$T$是[数值类型](#值类)，则特别处理测试`$x$.asInstanceOf[$T$]`。在这种情况下，广播将被翻译成一种x.to$T$的[转换方法](#数字值类型)应用。


## 值类

值类是其实例为由底层主机系统表示为对象的值。所有的值类都继承自`AnyVal`类。Scala实现需要提供值类`Unit`, `Boolean`, `Double`, `Float`,`Long`, `Int`, `Char`, `Short`, 和 `Byte`(但也可以自由提供其他类)。
这些类的签名定义如下：

### 数字值类型
类`Double`, `Float`,`Long`, `Int`, `Char`, `Short`, 和 `Byte`一起被称为_数字值类型_。类`Byte`,`Short`, 或 `Char`被称为_子范围类型_，而 `Int` 和 `Long`被称为_整数类型_，而`Float` 和`Double`被称为_浮点类型_。

数值类型的级别为以下排序：

```scala
Byte - Short
             \
               Int - Long - Float - Double
             /
        Char
```

`Byte`和`Short`是此顺序中级别最低的类型，而`Double`是级别最高的类型。级别并_不是_指[一致性](03-types.html#conformance)关系。例如，`Int`不是`Long`的子类型。但是，对象[`Predef`](#Predef对象)定义类从每个数字值类型到其他高级数字值类型的[视图](07-implicits.html#views)。因此，如果在[上下文](07-implicits.html#views)中有需求，低级的类型可以隐式的转换为高级的类型。

给定连个数值类型$S$和$T$，$S$和$T$的_操作类型_定义如下:如果$S$和$T$都是子范围类型，那么$S$和$T$的操作类型是`Int`。否则$S$和$T$的操作类型是相对排名中较大的一个。给定两个数字值类型$v$和$w$,他们的操作类型就是他们运行时的类型。

任何数值类型$T$都支持以下方法

  * 比较方法，比如全等(`==`),不等(`!=`),小于(`<`),大于(`>`),小于等于(`<=`)，大于等于(`>=`),每个方法豆存在7个重载的可选项。每个可选项有一个数字值类型的参数。结果类型为`Boolean`。操作形式为接受者和参量变为其操作类型，并在此类型上做比较。
  * 算数运算加(`+`),减(`-`),乘(`*`)，除(`/`)和求余(`%`),每个方法都存在7个重载的可选项。每个可选项有一个类型为$U$的数字值类型的参数。结果类型是$T$和$U$的操作类型。操作形式为将接收者和参量变为其操作类型，并在此类型上做算数操作。
  * 无参数算术方法正(`+`)和负(`-`)的结果类型为$T$.第一个返回其自身，第二个返回其负值。
  * 转换方法`toByte`, `toShort`, `toChar`,    `toInt`, `toLong`, `toFloat`, `toDouble`将对应的对象变为目标类型，规则是java数值类型转换操作。转换可能截取数字值(比如从`Long`到`Int`或者从`Int`到`Byte`)或丢失精度(比如从`Double` 到 `Float`或者`Long`和`Float`之间的转换)。

整形数字值也支持以下的操作：

  * 位操作方法按位与(`&`)，按位或{`|`}，和按位异(`^`)，每个方法斗存在5鸽重载的可选项。每个可选项都有一个整数值类型的参数。结果类型是$T$和$U$的操作类型。操作形式为将接收者和参量变为其操作类型，并在此类型上做对应的位运算操作。
  * 无参方法按位取反(`~`)。结果类型是接受者的类型$T$或`Int`中的较大者。操作方式是将接受者变为结果类型并按位取反。
  * 移位操作。包括左移(`<<`)，算数右移(`>>`)和无符号右移(`>>>`)。每个方法有两个重载的可选项。并有一个类型为`Int`或`Long`的参数$n$.操作的结果类型就是接受者的同类型$T$或`Int`中的较大者。操作形式为将接受者变为结果类型并移动$n$位。

数值类型还实现`Any`类中的操作`equals`,`hashCode`, 和 `toString`。

`equals`方法测试参数是否为数值类型。如果为`true`的话则执行对应类型的`==`操作。也就是说，可以认为数值类型的`equals`方法定义如下：

```scala
def equals(other: Any): Boolean = other match {
  case that: Byte   => this == that
  case that: Short  => this == that
  case that: Char   => this == that
  case that: Int    => this == that
  case that: Long   => this == that
  case that: Float  => this == that
  case that: Double => this == that
  case _ => false
}
```

`hashCode`方法返回一个整数哈希码，它将相等的数值映射到相等的结果。它保证是`Int`类型和所有子范围类型的标识符。
接受者的`toString`方法将显示为一个整数或浮点数。

###### 例

这是数值类型`Int`的签名：

```scala
package scala
abstract sealed class Int extends AnyVal {
  def == (that: Double): Boolean  // double 相等
  def == (that: Float): Boolean   // float 相等
  def == (that: Long): Boolean    // long 相等
  def == (that: Int): Boolean     // int 相等
  def == (that: Short): Boolean   // int 相等
  def == (that: Byte): Boolean    // int 相等
  def == (that: Char): Boolean    // int 相等
  /* analogous for !=, <, >, <=, >= */

  def + (that: Double): Double    // double 加
  def + (that: Float): Double     // float 加
  def + (that: Long): Long        // long 加
  def + (that: Int): Int          // int 加
  def + (that: Short): Int        // int 加
  def + (that: Byte): Int         // int 加
  def + (that: Char): Int         // int 加
  /* analogous for -, *, /, % */

  def & (that: Long): Long        // long 按位与
  def & (that: Int): Int          // int 按位与
  def & (that: Short): Int        // int 按位与
  def & (that: Byte): Int         // int 按位与
  def & (that: Char): Int         // int 按位与
  /* analogous for |, ^ */

  def << (cnt: Int): Int          // int 左移
  def << (cnt: Long): Int         // long 右移
  /* analogous for >>, >>> */

  def unary_+ : Int               // int 正
  def unary_- : Int               // int 负
  def unary_~ : Int               // int 按位取反

  def toByte: Byte                // 变为 Byte
  def toShort: Short              // 变为 Short
  def toChar: Char                // 变为 Char
  def toInt: Int                  // 变为 Int
  def toLong: Long                // 变为 Long
  def toFloat: Float              // 变为 Float
  def toDouble: Double            // 变为 Double
}
```

###  `Boolean`类

`Boolean`类只有两个值： `true` 和
`false`. 它实现了以下类定义中给出的操作.

```scala
package scala
abstract sealed class Boolean extends AnyVal {
  def && (p: => Boolean): Boolean = // boolean and
    if (this) p else false
  def || (p: => Boolean): Boolean = // boolean or
    if (this) true else p
  def &  (x: Boolean): Boolean =    // boolean strict and
    if (this) x else false
  def |  (x: Boolean): Boolean =    // boolean strict or
    if (this) true else x
  def == (x: Boolean): Boolean =    // boolean equality
    if (this) x else x.unary_!
  def != (x: Boolean): Boolean =    // boolean inequality
    if (this) x.unary_! else x
  def unary_!: Boolean =            // boolean negation
    if (this) false else true
}
```
该类椰实现了`Any`类中的 `equals`, `hashCode`,和 `toString`操作。

如果参数与接收者的布尔值相同，则`equals`方法返回`true`,否则就返回`false`。`hashcode`方法在调用`true`时返回一个固定的，特定于实现的哈希代码，并在`false`时调用另一个固定的，特定于实现的哈希代码。`toString`方法将接受者变为一个字符串，例如`"true"` 或 `"false"`

### `Unit`类

`Unit`类只有一个值`()`，它只实现了`Any`类中的三个方法`equals`, `hashCode`, 和 `toString`。

如果参量是值`()`则`equal`方法返回`true`，否则返回`false`。`hashCode`方法返回一个固定的与实现有关的hash-code值。`toString`方法返回`"()"`.


## 标准引用类

本节介绍了一些标准的Scala引用类，他们由Scala编译器以特殊方式处理-或者Scala为他们提供语法糖，或者Scala编译器为他们的操作生成特殊代码。Scala标准库中的其他类的文档在Scala库文档的HTML页中。

### `String`类

Scala的`String`类通常派生自底层地同的String类(可能有所不同)。对于Scala客户的来说该类在每种情况喜爱斗支持一种方法

```scala
def + (that: Any): String
```

它将其左操作数与其右操作数的文本表示连接起来。

### `Tuple`类

Scala 为$n = 2 , \ldots , 22$定义了元组类 `Tuple$n$`。这些定义如下

```scala
package scala
case class Tuple$n$[+T_1, ..., +T_n](_1: T_1, ..., _$n$: T_$n$) {
  def toString = "(" ++ _1 ++ "," ++ $\ldots$ ++ "," ++ _$n$ ++ ")"
}
```

### `Function`类

Scala 定义了 `Function$n$` 类函数 $n = 1 , \ldots , 22$.这些定义如下

```scala
package scala
trait Function$n$[-T_1, ..., -T_$n$, +R] {
  def apply(x_1: T_1, ..., x_$n$: T_$n$): R
  def toString = "<function>"
}
```

`Function1`的`PartialFunction`子类表示(间接)指其域的函数。使用`isDefined`方法查询是否为给定输入定义了部分函数。(即，输入是否是函数域的一部分)


```scala
class PartialFunction[-A, +B] extends Function1[A, B] {
  def isDefinedAt(x: A): Boolean
}
```
隐式导入的[`predef`]((#Predef对象))对象将名称`Function`定义为`Function1`的别名。

### `Array`类

阵列上所有操作都涉及底层平台的相应操作。因此，以下类定义仅供参考：


```scala
final class Array[T](_length: Int)
extends java.io.Serializable with java.lang.Cloneable {
  def length: Int = $\ldots$
  def apply(i: Int): T = $\ldots$
  def update(i: Int, x: T): Unit = $\ldots$
  override def clone(): Array[T] = $\ldots$
}
```

如果$T$不是类型参数或抽象类型，则类型`Array[T]`表示底层主机系统中的原生数组类型`|T|[]`，其中`|T|`是`T`的擦除。如果$T$不是类型参数或抽象类型，则可以使用不同的表示(相当于java平台的`Object`)。

#### 操作

`length`返回数组的长度，`apply`表示下标，`update`表示元素更新。

由于`apply`和`update`操作的语法糖，我们在Scala和Java代码之间具有以下对于关系，用于对数组`xs`进行操作:

|_Scala_           |_Java_      |
|------------------|------------|
|`xs.length`       |`xs.length` |
|`xs(i)`           |`xs[i]`     |
|`xs(i) = e`       |`xs[i] = e` |

`Predef`中存在两个经常引用于数组的隐式转换：转化为`scala.collection.mutable.ArrayOps`和转换为`scala.collection.mutable.ArraySeq`(`scala.collection.Seq`的子类型)

这两种类型都可以使用scala集合API中的多标准操作。转换为`ArrayOps`是临时的，因为在`ArrayOps`上定义的所有操作都返回`Array`类型的值。而转移为`ArraySeq`是永久性的，因为所有操作都返回`ArraySeq`类型的值。转换为`ArrayOps`优先与转换为`ArraySeq`。

由于Scala中的参数化类型和宿主语言中的数组的特殊实现之间的紧张关系，在处理数组时需要考虑一些微妙的问题。下面将对此进行解释。

#### 变性

与java中的数组不同，Scala中的数组_不是_共变体。也就是说，$S <: T$并不意味着Scala中的数组`Array[$S$] $<:$ Array[$T$]` 。但是，如果在主机环境中允许这样的强制转换，则可以将$S$数组转换为$T$数组。

例如，`Array[String]`不符合`Array[Object]`,即使`String `符合`Object`。但是，可以将`Array[String]`类型的表达式转换为`Array[Object]`，并且此转换将成功，而且不会引发`ClassCastException`。例如：

```scala
val xs = new Array[String](2)
// val ys: Array[Object] = xs   // **** error: incompatible types
val ys: Array[Object] = xs.asInstanceOf[Array[Object]] // OK
```

具有多态元素类型$T$的数组的实例化在运行时需要有关类型$T$的信息。通过将`scala.reflect.ClassTag`的[上下文绑定](07-implicits.html#context-bounds-and-view-bounds)添加到$T$类型来合成此信息。例如方法`mkArray`的以下实现，它创建一个任意类型$T$的数组，给定一个定义其元素的$T$序列：


```scala
import reflect.ClassTag
def mkArray[T : ClassTag](elems: Seq[T]): Array[T] = {
  val result = new Array[T](elems.length)
  var i = 0
  for (elem <- elems) {
    result(i) = elem
    i += 1
  }
  result
}
```
如果类型$T$是主机平台提供专用数组表示的类型，则使用此方法。


###### 例
在java虚拟机中，调用`mkArray(List(1,2,3))`将返回一个原始的`int`组，在java中写为`int[]`。

#### 伴随对象

`Array`的伴随对象为单维和多维数组的实例化提供类各种工厂方法，一个提取器方法是[`unapplySeq`](08-pattern-matching.html#extractor-patterns)，他支持数组上的模式匹配和其他使用的工具方法：
```scala
package scala
object Array {
  /** 从 `src`复制元素到`dest`. */
  def copy(src: AnyRef, srcPos: Int,
           dest: AnyRef, destPos: Int, length: Int): Unit = $\ldots$

  /** 返回长度为0的数组 */
  def empty[T: ClassTag]: Array[T] =

  /** 用给定元素创建一个数组. */
  def apply[T: ClassTag](xs: T*): Array[T] = $\ldots$

  /** 创建具有给定大小的数组 */
  def ofDim[T: ClassTag](n1: Int): Array[T] = $\ldots$
  /** 创建一个二维数组 */
  def ofDim[T: ClassTag](n1: Int, n2: Int): Array[Array[T]] = $\ldots$
  $\ldots$

  /** 将参数中所有数组合并一个新的数组. */
  def concat[T: ClassTag](xss: Array[T]*): Array[T] = $\ldots$

  /** 如果一个数组，其中包含多次元素计算的结果. */
  def fill[T: ClassTag](n: Int)(elem: => T): Array[T] = $\ldots$
  /** 返回一个二维数组，其中包含多次元素计算的结果. */
  def fill[T: ClassTag](n1: Int, n2: Int)(elem: => T): Array[Array[T]] = $\ldots$
  $\ldots$

  /** 返回一个数组，该数组包含从0开始的整数值范围内给定函数的值. */
  def tabulate[T: ClassTag](n: Int)(f: Int => T): Array[T] = $\ldots$
  /** 返回一个二维数组，其中包含从`0`开始的整数值范围内给定函数的值. */
  def tabulate[T: ClassTag](n1: Int, n2: Int)(f: (Int, Int) => T): Array[Array[T]] = $\ldots$
  $\ldots$

  /** 返回一个数组，其中包含一个范围内递增整数的序列。. */
  def range(start: Int, end: Int): Array[Int] = $\ldots$
  /** 返回一个包含某个整数间隔内等间距值的数组 */
  def range(start: Int, end: Int, step: Int): Array[Int] = $\ldots$

  /** 返回一个包含函数重复应用于起始值的数组 */
  def iterate[T: ClassTag](start: T, len: Int)(f: T => T): Array[T] = $\ldots$

  /** 启用数组上的模式匹配 */
  def unapplySeq[A](x: Array[A]): Option[IndexedSeq[A]] = Some(x)
}
```

## Node类型

```scala
package scala.xml

trait Node {

  /** 此节点的标签 */
  def label: String

  /** 属性轴 */
  def attribute: Map[String, String]

  /** 子节点轴（此节点的所有子节点） */
  def child: Seq[Node]

  /** 下属节点轴(此节点的所有下属节点) */
  def descendant: Seq[Node] = child.toList.flatMap {
    x => x::x.descendant.asInstanceOf[List[Node]]
  }

  /** 下属节点轴(此节点的所有下属节点) */
  def descendant_or_self: Seq[Node] = this::child.toList.flatMap {
    x => x::x.descendant.asInstanceOf[List[Node]]
  }

  override def equals(x: Any): Boolean = x match {
    case that:Node =>
      that.label == this.label &&
        that.attribute.sameElements(this.attribute) &&
          that.child.sameElements(this.child)
    case _ => false
  }

 /** XPath形式的投影函数，返回该节点所有标签为`that`的子节点。保留他们在文档中的顺序
  */
    def \(that: Symbol): NodeSeq = {
      new NodeSeq({
        that.name match {
          case "_" => child.toList
          case _ =>
            var res:List[Node] = Nil
            for (x <- child.elements if x.label == that.name) {
              res = x::res
            }
            res.reverse
        }
      })
    }

 /** XPath形式的投影函数，返回该节点的`descendant_or_self`中所有标签为`that`的子节点。保留他们在文档中的顺序
  */
  def \\(that: Symbol): NodeSeq = {
    new NodeSeq(
      that.name match {
        case "_" => this.descendant_or_self
        case _ => this.descendant_or_self.asInstanceOf[List[Node]].
        filter(x => x.label == that.name)
      })
  }

  /** 该XML节点的hashcode  */
  override def hashCode =
    Utility.hashCode(label, attribute.toList.hashCode, child)

  /** 该节点的字符串表现形式 */
  override def toString = Utility.toXML(this)

}
```

## `Predef`对象

`Predef`对象为Scala程序定义类标准函数和类型别名。 它是隐式导入的，如[关于名称绑定的章节](02-identifiers-names-and-scopes.html)所述，因此所有定义的成员都可以不受限制地使用。他在JVM环境中的定义与一下签名一致：

```scala
package scala
object Predef {

  // classOf ---------------------------------------------------------

  /** 返回累类类型的运行时表示. */
  def classOf[T]: Class[T] = null
   // 这是一个虚拟，classOf由编译器处理。

  // valueOf -----------------------------------------------------------

  /** 检索具有唯一居民的类型的单个值. */
  @inline def valueOf[T](implicit vt: ValueOf[T]): T {} = vt.value
   // 值类型的实例由编译器提供。

  // Standard type aliases ---------------------------------------------

  type String    = java.lang.String
  type Class[T]  = java.lang.Class[T]

  // Miscellaneous -----------------------------------------------------

  type Function[-A, +B] = Function1[A, B]

  type Map[A, +B] = collection.immutable.Map[A, B]
  type Set[A] = collection.immutable.Set[A]

  val Map = collection.immutable.Map
  val Set = collection.immutable.Set

  // Manifest types, companions, and incantations for summoning ---------

  type ClassManifest[T] = scala.reflect.ClassManifest[T]
  type Manifest[T]      = scala.reflect.Manifest[T]
  type OptManifest[T]   = scala.reflect.OptManifest[T]
  val ClassManifest     = scala.reflect.ClassManifest
  val Manifest          = scala.reflect.Manifest
  val NoManifest        = scala.reflect.NoManifest

  def manifest[T](implicit m: Manifest[T])           = m
  def classManifest[T](implicit m: ClassManifest[T]) = m
  def optManifest[T](implicit m: OptManifest[T])     = m

  // Minor variations on identity functions -----------------------------
  def identity[A](x: A): A         = x
  def implicitly[T](implicit e: T) = e    // for summoning implicit values from the nether world
  @inline def locally[T](x: T): T  = x    // to communicate intent and avoid unmoored statements

  // Asserts, Preconditions, Postconditions -----------------------------

  def assert(assertion: Boolean) {
    if (!assertion)
      throw new java.lang.AssertionError("assertion failed")
  }

  def assert(assertion: Boolean, message: => Any) {
    if (!assertion)
      throw new java.lang.AssertionError("assertion failed: " + message)
  }

  def assume(assumption: Boolean) {
    if (!assumption)
      throw new IllegalArgumentException("assumption failed")
  }

  def assume(assumption: Boolean, message: => Any) {
    if (!assumption)
      throw new IllegalArgumentException(message.toString)
  }

  def require(requirement: Boolean) {
    if (!requirement)
      throw new IllegalArgumentException("requirement failed")
  }

  def require(requirement: Boolean, message: => Any) {
    if (!requirement)
      throw new IllegalArgumentException("requirement failed: "+ message)
  }
```

```scala
  // Printing and reading -----------------------------------------------

  def print(x: Any) = Console.print(x)
  def println() = Console.println()
  def println(x: Any) = Console.println(x)
  def printf(text: String, xs: Any*) = Console.printf(text.format(xs: _*))

  // Implicit conversions ------------------------------------------------

  ...
}
```

### 预定义的隐式定义

`Predef`对象还包含许多隐式定义，这是默认提供的(因为隐式导入了`Predef`)。隐式定义有两个重点。高优先级的implicits是在`Predef`类本身中定义的，而低优先级的implicits是在`Predef`继承的类中定义的。静态[重载解析](06-expressions.html#overloading-resolution)规则规定，在其他条件相同的情况下，隐式解析更喜欢高优先级的影响，而不是低优先级的影响。

可用的低优先级影响包括以下类别的定义。

1.  对于每个基本类型，一个包装器，他将该类型的值带到`runtime.Rich*`类的实例。例如，`Int`类型的值可以隐式转换为类`runtime.RichInt`的值。
1.  对于既有基本类型元素的每个数组类型，将该类型的数组作为`ArraySeq`类的实例的包装器。例如，`Array[Float]`类型的值可以隐式转换为`ArraySeq[Float]`类的实例。还有通过数组包装器，它将`Array[T]`类型的元素用于任意`T`到`ArraySeq`.
1.  从`String`到`WrappedString`的隐式转换。

可用的高优先级含义包括属于以下类别的定义。


  * 一个隐式包装器，它使用以下重载变量添加`Any`的方法以键入`ensuring`。

    ```scala
    def ensuring(cond: Boolean): A = { assert(cond); x }
    def ensuring(cond: Boolean, msg: Any): A = { assert(cond, msg); x }
    def ensuring(cond: A => Boolean): A = { assert(cond(x)); x }
    def ensuring(cond: A => Boolean, msg: Any): A = { assert(cond(x), msg); x }
    ```

  * 一个隐式包装器，它使用以下实现添加`->`方法来键入`Any`。

    ```scala
    def -> [B](y: B): (A, B) = (x, y)
    ```

  * 对于具有基本类型元素的每个数组类型，一个包装器，将该类型的数组转换为`runtime.ArrayOps`类的实例。例如，`Array[Float]`类型的值可以隐式转换为类`runtime.ArrayOps[Float]`的实例。还有通用数组包装器，它将 `Array[T]` 类型的元素用于任意`T`到`ArrayOps`。

  * 一个隐式包装器，它使用以下实现添加`+`和`format`方法以键入`Any`。

    ```scala
    def +(other: String) = String.valueOf(self) + other
    def formatted(fmtstr: String): String = fmtstr format self
    ```

  * 实现以下映射的传递闭包的数字基元转换：

    ```
    Byte  -> Short
    Short -> Int
    Char  -> Int
    Int   -> Long
    Long  -> Float
    Float -> Double
    ```

  * 原始类型与其盒装版本之间的装箱和拆箱转换：

    ```
    Byte    <-> java.lang.Byte
    Short   <-> java.lang.Short
    Char    <-> java.lang.Character
    Int     <-> java.lang.Integer
    Long    <-> java.lang.Long
    Float   <-> java.lang.Float
    Double  <-> java.lang.Double
    Boolean <-> java.lang.Boolean
    ```

  * 一个隐式定义，为任何类型`T`生成类型为`T <:< T`的实例。这里，`<:<`是一个定义如下的类。


    ```scala
    sealed abstract class <:<[-From, +To] extends (From => To)
    ```

     `<:<` 类型的隐式参数通常用于实现类型约束。
