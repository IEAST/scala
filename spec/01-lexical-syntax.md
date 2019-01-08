---
标题: Lexical Syntax
布局: 默认
章节: 1
---

# 词汇和语法

Scala程序使用 Unicode Basic Multilingual Plane（_BMP_）字符集编写; 目前不支持 Unicode supplementary 字符。本篇定义了 Scala的词法的两种模式，即Scala模式和 _XML 模式_ 。如果没有另外提及, Scala 符号的以下描述参考 _Scala 模式_, 文字字符‘c’引用ASCII片段`\u0000` – `\u007F`.

在 Scala 模式, _Unicode escapes_ 被相应的替换为具有给定十六进制代码的Unicode字符。


```ebnf
UnicodeEscape ::= ‘\’ ‘u’ {‘u’} hexDigit hexDigit hexDigit hexDigit
hexDigit      ::= ‘0’ | … | ‘9’ | ‘A’ | … | ‘F’ | ‘a’ | … | ‘f’
```

<!--
TODO scala/bug#4583: UnicodeEscape used to allow additional backslashes,
and there is something in the code `evenSlashPrefix` that alludes to it,
but I can't make it work nor can I imagine how this would make sense,
so I removed it for now.
-->

为了构建字符, 根据以下类别区分字符 (在括弧中给出 Unicode 的一般类别):
1. 空白字符. [`\u0020 `](https://www.fileformat.info/info/unicode/char/0020/index.htm)|
 [`\u0009`](https://www.fileformat.info/info/unicode/char/0009/index.htm) |
  [`\u000D `](https://www.fileformat.info/info/unicode/char/000d/index.htm)|
  [` \u000A`](https://www.fileformat.info/info/unicode/char/000a/index.htm).
1. 字母, 包括
小写字母 ([`Ll`](https://www.fileformat.info/info/unicode/category/Ll/index.htm)),
大写字母([`Lu`](https://www.fileformat.info/info/unicode/category/Lu/index.htm)),
单词开头大小写字母 ([`Lt`](http://www.fileformat.info/info/unicode/category/Lt/index.htm)),
    其他字符
   ([`Lo`](http://www.fileformat.info/info/unicode/category/Lo/index.htm)),
    数字
   ([`Nl`](http://www.fileformat.info/info/unicode/category/Nl/index.htm))
    和另外两个字符
     [`\u0024 ‘$’`](https://www.fileformat.info/info/unicode/char/0024/index.htm) 和
     [`\u005F ‘_’`](http://www.fileformat.info/info/unicode/char/5f/index.htm).
1. 数字 [`‘0’ | … | ‘9’`](https://www.fileformat.info/info/unicode/category/Nd/list.htm).
<!--
\U0030---\U0039
-->
1. 括弧
[`‘(’ `](https://www.fileformat.info/info/unicode/char/0028/index.htm)|
[`‘)’`](https://www.fileformat.info/info/unicode/char/0029/index.htm) |
[`‘[’`](http://www.fileformat.info/info/unicode/char/ff3b/index.htm) |
[`‘]’`](http://www.fileformat.info/info/unicode/char/5d/index.htm)|
[`‘{’`](www.fileformat.info/info/unicode/char/007b/index.htm) |
[`‘}’`](www.fileformat.info/info/unicode/char/007D/index.htm).
1. 分隔符
[``‘`’``](https://www.fileformat.info/info/unicode/char/055d/index.htm)|
 [`‘'’`](https://www.fileformat.info/info/unicode/char/0027/index.htm)|
 [`‘"’`](https://www.fileformat.info/info/unicode/char/0022/index.htm)|[` ‘.’`](https://www.fileformat.info/info/unicode/char/002e/index.htm)|[ `‘;’`](https://www.fileformat.info/info/unicode/char/003b/index.htm)|[ `‘,’ `](https://www.fileformat.info/info/unicode/char/002c/index.htm).
1. 操作符字符. 它们由所有可打印的ASCII字符组成
  ([`\u0020` - `\u007E`](http://www.fileformat.info/info/unicode/font/dejavu_sans_mono_oblique/blockview.htm?block=basic_latin))，数学符号 ([`Sm`](https://www.fileformat.info/info/unicode/category/Sm/list.htm)) 和其他符号 ([`So`](https://www.fileformat.info/info/unicode/category/So/list.htm))不包含在内.

## 标识符

```ebnf
op       ::=  opchar {opchar}
varid    ::=  lower idrestg
boundvarid ::=  varid
             | ‘`’ varid ‘`’
plainid  ::=  upper idrest
           |  varid
           |  op
id       ::=  plainid
           |  ‘`’ { charNoBackQuoteOrNewline | UnicodeEscape | charEscapeSeq } ‘`’
idrest   ::=  {letter | digit} [‘_’ op]
```

有三种方法可以构成标识符。第一种，标识符以字母开头，后面可以是任意字母和数字序列. 接下来可以是`‘_‘`字符以及其他的字母和数字以及操作符字符组成的字符串。
第二种, 标识符可以一操作符字符开头，后面接任意操作符字符序列。
前两中形式称之为 *普通* 标识符.
最后一种,标识符也可以由后引号之间的任意字符串形成（主机系统可以对哪些字符串能合法构成标识符增加一些限制）。然后，标识符由除反引号本身之外的所有字符组成。


像以往一样，适用最长匹配规则。 例如，字符串
```scala
big_bob++=`def`
```

可以分解为三个标识符`big_bob`, `++=`, 和`def`.


模式匹配的规则进一步区分 *变量标识符* 和 *常量标识符*，*变量标识* 符以小写字母开头，而 *常量标识符* 则不是。为此，下划线`'_'`作为小写，'\$'字符作为大写。

'$'字符被保留给编译器合成的标识符。 用户程序不应定义包含'$'字符的标识符。

以下名称是保留字（词），而不是词法标识符的语法类`id`的成员。


```scala
abstract    case        catch       class       def
do          else        extends     false       final
finally     for         forSome     if          implicit
import      lazy        macro       match       new
null        object      override    package     private
protected   return      sealed      super       this
throw       trait       try         true        type
val         var         while       with        yield
_    :    =    =>    <-    <:    <%     >:    #    @
```

Unicode运算符 `\u21D2` ‘$\右箭头$’ and `\u2190` ‘$\左箭头$’, 也就是等同于ASCII中的 `=>` 和 `<-`, 也是保留的.

>以下是标识符的示例:
> ```scala
>     x         Object        maxIndex   p2p      empty_?
>     +         `yield`       αρετη     _y       dot_product_*
>     __system  _MAX_LEN_
> ```

<!-- -->

> 当需要访问Scala中保留的java标识符时，请使用引号括起的字符串。
> 例如, 这个声明 `Thread.yield()` 是不合法的, 因为 `yield` 是Scala中的保留词.
> 但是, 这是一个解决方法: `` Thread.`yield`() ``

## 换行符

```ebnf
semi ::= ‘;’ |  nl {nl}
```

Scala是一种面向行的语言，语句可以用分号或换行符终止。 如果满足以下三个条件，则Scala源文本中的换行符将被视为特殊标记“nl”：

1. 紧接在换行符之前的标识终止语句。
1. 紧接在换行符之后的标识开始声明。
1. 标识出现在启用换行符的区域内。

可以终止语句的标识是:字符，标识符和以下分隔符和保留字:

```scala
this    null    true    false    return    type    <xml-start>
_       )       ]       }
```

可以用于开始语句的标记都是Scala标识，但以下分隔符和保留字*除外*：

```scala
catch    else    extends    finally    forSome    match
with    yield    ,    .    ;    :    =    =>    <-    <:    <%
>:    #    [    )    ]    }
```

只有在后面接`class`和`object`标识时，`case`标记才能开始语句。  

在以下位置启动换行：

1. 所有Scala源文件，除了禁用换行的嵌套区域。
1. 匹配`{`和`}`大括号标记之间的间隔，但禁用换行符的嵌套区域除外。

以下位置禁止换行:

1. 匹配`(`和`)`括号标记之间的间隔，启用了换行符的嵌套区域除外。
1. 匹配`[`和`]`括号标记之间的间隔，启用了换行符的嵌套区域除外
1. 匹配`case`标记和`=>` 标记之间的间隔, 但启用了换行符的嵌套区域除外
1. 在 [XML mode](#XML模式)模式下分析的任何区域.

请注意，`{...}`中的括号字符以XML格式转义，字符串文字不是标记，因此不会包含启用换行符的区域。  

通常，只有一个`nl`标记插入在不同行上的两个连续非换行标记之间，即使两个标记之间存在多条线。 但是，如果两个标记被至少一个完全空行（即不包含可打印字符的行）分开，则插入两个`nl`标记。  

Scala语法包含可选`nl`标记但不接受分号的产品。 这导致其中一个位置的换行不终止表达式或语句。 这些位置可归纳如下：

在以下位置接受多个换行符令（请注意，在每种情况下，用分号代替换行符都是非法的）：

- 在
  [条件表达式](06-expressions.html#conditional-expressions)
  或者 [while 循环](06-expressions.html#while-loop-expressions) 的条件与下一个表达式之间
- 在
  [for-comprehension](06-expressions.html#for-comprehensions-and-for-loops)枚举数和下一个表达式之间, and
- 在 `type`
  [类型定义或声明](04-basic-declarations-and-definitions.html#type-declarations-and-type-aliases)中的初始类型关键字之后.

接受一个新的行标记

- 在开头括号'{'前面，如果该括号是当前陈述或表达的合法延续 ,
- 在[中缀表达式](06-expressions.html#prefix,-infix,-and-postfix-operations)之后，如果下一行的第一个标记可以启动表达式 ,
- 在 [参数子句](04-basic-declarations-and-definitions.html#function-declarations-and-definitions)之前, and
- 在 [注释](11-annotations.html#user-defined-annotations)之后.

> 两行之间的换行符不被视为语句分隔符。
> ```scala
> if (x > 0)
>   x = x - 1
>
> while (x > 0)
>   x = x / 2
>
> for (x <- 1 to 10)
>   println(x)
>
> type
>   IntList = List[Int]
> ```

<!-- -->

> ```scala
> new Iterator[Int]
> {
>   private var x = 0
>   def hasNext = true
>   def next = { x += 1; x }
> }
> ```
>
> 使用额外的换行标识符时, 类似的代码被解释为同一对象创建继承至当前区块
>
> ```scala
> new Iterator[Int]
>
> {
>   private var x = 0
>   def hasNext = true
>   def next = { x += 1; x }
> }
> ```
<!-- -->

> ```scala
>   x < 0 ||
>   x > 10
> ```
>
> 使用额外的换行符，相同的代码将被解释为两个表达式：
>
> ```scala
>   x < 0 ||
>
>   x > 10
> ```

<!-- -->

> ```scala
> def func(x: Int)
>         (y: Int) = x + y
> ```
>
> 使用额外的换行符，相同的代码被解释为抽象函数定义和语法上的非法语句:
>
> ```scala
> def func(x: Int)
>
>         (y: Int) = x + y
> ```

<!-- -->

> ```scala
> @serializable
> protected class Data { ... }
> ```
>
> 使用额外的换行符时，相同的代码将被解释为属性和单独的语句（在语法上是非法的）。
>
> ```scala
> @serializable
>
> protected class Data { ... }
> ```

## 常量

有整数型，浮点型，字符，布尔型，符号，字符串的文字。 这些文字的语法在任何情况下都与Java一样。

<!-- TODO
  say that we take values from Java, give examples of some lits in
  particular float and double.
-->

```ebnf
Literal  ::=  [‘-’] integerLiteral
           |  [‘-’] floatingPointLiteral
           |  booleanLiteral
           |  characterLiteral
           |  stringLiteral
           |  symbolLiteral
           |  ‘null’
```

### 整型常量

```ebnf
integerLiteral  ::=  (decimalNumeral | hexNumeral)
                       [‘L’ | ‘l’]
decimalNumeral  ::=  ‘0’ | nonZeroDigit {digit}
hexNumeral      ::=  ‘0’ (‘x’ | ‘X’) hexDigit {hexDigit}
digit           ::=  ‘0’ | nonZeroDigit
nonZeroDigit    ::=  ‘1’ | … | ‘9’
```

整数文字通常是`Int`类型，或者类型为`Long`，后跟`L`或`l`后缀。 `int`类型的值是$ -2 ^ {31} $和$ 2 ^ {31} -1$之间的整数.  `Long`类型的值都是$ -2 ^ {63} $和$ 2 ^ {63} -1 $之间的整数。 如果整数是超出这些范围的数字，则会发生编译时错误。.

但是，如果表达式中文字的预期类型[_pt_](06-expressions.html#expression-typing) 是 `Byte`, `Short`, 或者 `Char`并且整数适合该类型定义的数值范围，则该数字将转换为 _pt_ 类型，常量类型为 _pt_。这些类型给出的数值范围是:

|                |                          |
|----------------|--------------------------|
|`Byte`          | $-2\^7$ to $2\^7-1$      |
|`Short`         | $-2\^{15}$ to $2\^{15}-1$|
|`Char`          | $0$ to $2\^{16}-1$       |

> ```scala
> 0          21          0xFFFFFFFF       -42L
> ```

### 浮点常量

```ebnf
floatingPointLiteral  ::=  digit {digit} ‘.’ digit {digit} [exponentPart] [floatType]
                        |  ‘.’ digit {digit} [exponentPart] [floatType]
                        |  digit {digit} exponentPart [floatType]
                        |  digit {digit} [exponentPart] floatType
exponentPart          ::=  (‘E’ | ‘e’) [‘+’ | ‘-’] digit {digit}
floatType             ::=  ‘F’ | ‘f’ | ‘D’ | ‘d’
```

浮点常量的类型为 `Float` 后跟浮点类型后缀 `F` 或 `f`, 则为`Double` 类型. 。 `Float`类型包含所有IEEE 754 32位单精度二进制浮点值，而`Double`类型包含所有IEEE 754 64位双精度二进制浮点值。

如果程序中的浮点文字后跟一个以字母开头的标记，则两个标记之间必须至少有一个空白字符。

> ```scala
> 0.0        1e30f      3.14159f      1.0e-100      .1
> ```

<!-- -->

> 短语 `1.toString` 解析为三个不同的标记:
> 整数 `1` , `.` , 和标识符 `toString`.

<!-- -->

> `1.` 不是有效的浮点字面值，因为后面的强制位`.`丢失。

### 布尔常量

```ebnf
booleanLiteral  ::=  ‘true’ | ‘false’
```

布尔常量 `true` 和 `false`是 `Boolean`类型的成员.

### 字符常量

```ebnf
characterLiteral  ::=  ‘'’ (charNoQuoteOrNewline | UnicodeEscape | charEscapeSeq) ‘'’
```

字符文字是用引号括起来的单个字符。字符可以是除单引号分隔符或`\u000A`(LF)
或`\u000D` (CR)之外的任何Unicode字符。或由[Unicode转义符](01-lexical-syntax.html)或[转义序列](#转义字符)表示的何Unicode字符。

> ```scala
> 'a'    '\u0041'    '\n'    '\t'
> ```

注意，unicode转换是在解析过程的早期完成的。因此Unicode字符通常等同于源文本中的转义扩展，但文字解析接受任意Unicode转义，包括字符文字`'\u000A'`，也可以使用转义序列`'\n'`。

### 字符串文字

```ebnf
stringLiteral  ::=  ‘"’ {stringElement} ‘"’
stringElement  ::=  charNoDoubleQuoteOrNewline | UnicodeEscape | charEscapeSeq
```

字符串文字是双引号中的字符序列。字符串可以是除单引号分隔符或`\u000A`(LF)
或`\u000D` (CR)之外的任何Unicode字符。或由[Unicode转义符](01-lexical-syntax.html)或[转义序列](#转义字符)表示的何Unicode字符。

如果字符串文字包含双引号字符，则必须使用`"\""`进行转义.

字符串文字的值是类`String`的实例。

> ```scala
> "Hello, world!\n"
> "\"Hello,\" replied the world."
> ```

#### 多行字符串文字

```ebnf
stringLiteral   ::=  ‘"""’ multiLineChars ‘"""’
multiLineChars  ::=  {[‘"’] [‘"’] charNoDoubleQuote} {‘"’}
```
多行字符串文字是用三引号`""" ... """`括起来的字符序列。字符的顺序是任意的，但它可能只在最后包含三个或更多的连续引号字符。 字符不一定是可打印的; 也允许换行或其他控制字符。Unicode转义符可以像其他地方一样工作，但没有解释[转义序列](##转义字符)。

> ```scala
>   """the present string
>      spans three
>      lines."""
> ```
>
> 这会产生字符串：
>
> ```scala
> the present string
>      spans three
>      lines.
> ```
>
> Scala库包含一个实用程序方法 `stripMargin`可用于从多行字符串中去除后面几行首位的空格。 表示为
>
> ```scala
>  """the present string
>    |spans three
>    |lines.""".stripMargin
> ```
>
> 执行后结果
>
> ```scala
> the present string
> spans three
> lines.
> ```
>
> 方法`stripMargin`在类
> [scala.collection.immutable.StringLike](http://www.scala-lang.org/api/current/#scala.collection.immutable.StringLike)中定义.
> 因为从 `String` 到
> `StringLike`存在预先确定的
> [implicit conversion](06-expressions.html#implicit-conversions) , 所以该方法适用于所有的字符串。

### 转义字符

在字符和字符串文字中可识别以下转义序列。

| charEscapeSeq | unicode  | name      | char   |
|---------------|----------|-----------|--------|
| `‘\‘ ‘b‘`     | `\u0008` | 退格       |  `BS`  |
| `‘\‘ ‘t‘`     | `\u0009` | 水平制表   |  `HT`  |
| `‘\‘ ‘n‘`     | `\u000a` | 换行       |  `LF`  |
| `‘\‘ ‘f‘`     | `\u000c` | 换页       |  `FF`  |
| `‘\‘ ‘r‘`     | `\u000d` | 回车       |  `CR`  |
| `‘\‘ ‘"‘`     | `\u0022` | 双引号     |  `"`   |
| `‘\‘ ‘'‘`     | `\u0027` | 单引号     |  `'`   |
| `‘\‘ ‘\‘`     | `\u005c` | 反斜线     |  `\`   |

Unicode在0到255之间的字符也可以用八进制转义表示，即反斜杠 `'\'` 后跟最多三个八进制字符的序列。

如果字符或字符串文字中的反斜杠字符未启动有效的转义序列，则会编译时报错。

### 符号文字

```ebnf
symbolLiteral  ::=  ‘'’ plainid
```

符号文字`'x` 是表达式
`scala.Symbol("x")`的简写.`Symbol`是一个 [case class](05-classes-and-objects.html#case-classes),
定义如下。

```scala
package scala
final case class Symbol private (name: String) {
  override def toString: String = "'" + name
}
```

`Symbol`的伴随对象的`apply`方法封装了对`Symbol`的弱引用，从而确保相同的符号文字相对于引用是等价的。

## 空格和注释

标记可以用空格字符和/注释。注释的方式有两种：
单行注释是一系列以`//`开头并延伸到行尾的字符。

多行注释是`/*`和`*/`之间的字符序列。 多行注释可以嵌套，但需要正确嵌套。 因此，像`/ * / * * /`这样的评论将被拒绝，因为它有未终止的注释。


## 多行表达式中的尾随逗号
如果立即跟随逗号（`，`），忽略空格，换行符和右括号(`)`)，括号（`]`）或大括号（`}`），则逗号将被视为“尾随逗号”并被忽略。 例如：
```scala
foo(
  23,
  "bar",
  true,
)
```

## XML模式

为了允许文本包含XML片段，在下列情况下遇到开放角括号“<”时，词法分析从Scala模式切换到XML模式：'<'必须在空格，左括号或开头之前 大括号后面紧跟一个开始XML名称的字符。

```ebnf
 ( whitespace | ‘(’ | ‘{’ ) ‘<’ (XNameStart | ‘!’ | ‘?’)

  XNameStart ::= ‘_’ | BaseChar | Ideographic // as in W3C XML, but without ‘:’
```

如果有下列的话，扫描程序从XML模式切换到Scala模式

- 由初始“<”启动的XML表达式或XML模式已成功解析，或者
- 解析器遇到嵌入的Scala表达式，并迫使扫描程序返回正常模式，直到成功解析Scala表达式。在这种情况下，由于代码和XML片段可以嵌套，因此解析器必须保持一个堆栈反映XML的嵌套和Scala充分表达。

请注意，没有Scala符号是在XML模式下构造的，而且注释会被解释为文本。

> 以下定义使用带有两个嵌入式Scala表达式的XML文本：
>
> ```scala
> val b = <book>
>           <title>The Scala Language Specification</title>
>           <version>{scalaBook.version}</version>
>           <authors>{scalaBook.authors.mkList("", ", ", "")}</authors>
>         </book>
> ```
