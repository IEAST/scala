---
title: Top-Level Definitions
layout: default
chapter: 9
---

# 顶级定义

## 编译单元

```ebnf
CompilationUnit  ::=  {‘package’ QualId semi} TopStatSeq
TopStatSeq       ::=  TopStat {semi TopStat}
TopStat          ::=  {Annotation} {Modifier} TmplDef
                   |  Import
                   |  Packaging
                   |  PackageObject
                   |
QualId           ::=  id {‘.’ id}
```

编译单元由一系列包，导入子句，类和对象定义(前面可以是包子句)构成。


 _编译单元_

```scala
package $p_1$;
$\ldots$
package $p_n$;
$\mathit{stats}$
```

从一个或多个包子句开始等价与所述包的编译单元

```scala
package $p_1$ { $\ldots$
  package $p_n$ {
    $\mathit{stats}$
  } $\ldots$
}
```
每个编译单元按给定顺序隐式导入以下包：

 1. 包`java.lang`,
 2. 包 `scala`,
 3. 对象 [`scala.Predef`](12-the-scala-standard-library.html#the-predef-object), 除非有一个引用的 `scala.Predef`的显式顶级导入.

后引入的成员将隐藏先前引入的成员。

隐式导入`scala.Predef`的例外情况是可用于隐藏(例如，预定义的隐式转换)

## 包

```ebnf
Packaging       ::=  ‘package’ QualId [nl] ‘{’ TopStatSeq ‘}’
```

_包_ 是一个特殊的对象，他定义了一组成员类，对象和包。与其他对象不同，包不是由定义引入的。报的成员集合是由打包确定的。

包`package $p$ { $\mathit{ds}$ }` 将 $\mathit{ds}$中的所有定义作为成员放到限定名为$p$的包中。包的成员称为_顶级定义_。如果$\mathit{ds}$中某定义标记为`private`，则仅在包内成员中可见。

在包内部，$p$包的所有成员都以其简单名称可见。但是，此规则不会扩展到由$p$的路径前缀指定的封闭$p$包的成员。

```scala
package org.net.prj {
  ...
}
```
包`org.net.prj`的所有成员都以其简单名称可见，但包`org`或`org.net`的成员需要显示限定或导入。

$p$中或$p$导入的选择$p$.$m$工作方式类似对象与包的特殊对象。与其他对象不同，包不能做值。拥有与模块或类相同的完全相同的限定名称的包是非法的。

假设包外的顶级定义被注入特殊的空包中。该包无法命名，也不能被导入。然而空包的成员不用限定就对彼此可见。


## 包对象

```ebnf
PackageObject   ::=  ‘package’ ‘object’ ObjectDef
```

_包对象_`package object $p$ extends $t$`将模板$t$的成员添加道包$p$中。每个包只能有一个包对象。标准命名约束是将上面的定义放在`package.scala`文件中，该文件位于与$p$包对于的目录中。

包对象不应定义与包$p$中定义的顶级对象或类具有相同名称的成员。如果存在命名冲突，则程序的行为当前未定义。预计此限制将会在Scala的未来版本中解除。

## 包引用

```ebnf
QualId           ::=  id {‘.’ id}
```

对报的引用采用限定标识符的形式。与所有其他的引用一样，包引用是相对的。也就是说，以名称$p$开头的包引用将在定义了名为$p$的成员的最接近的封闭范围中查找。

如果包名称被镜像，则可以通过在其前面添加特殊的预定义名称`_root_`来引用其完全限定名称，改名称引用包含所有顶级包的最外层root包。

名称`_root_`仅在用作限定符的第一个元素时才具有此特殊标记，否则他就是普通的标识符。

###### 例
考虑一下程序：

```scala
package b {
  class B
}

package a {
  package b {
    class A {
      val x = new _root_.b.B
    }
    class C {
      import _root_.b._
      def y = new B
    }
  }
}

```

这里，引用`_root_.b.B`引用顶层包`b`中的`B`类。如果省略了 `_root_`前缀，该名称`b`将改为解析包为`a.b`，并且如果该包不包含类`B`，则会参数编译错误。

## 程序

_程序_ 是顶级对象，具有一个类型为`(Array[String])Unit`的成员方法 _main_。程序可以在命令行界面中执行。程序的命令行参量将以类型`Array[String]`的形式传递给`main`方法。

程序的`main`方法可以直接在对象中定义，也可以继承。scala库中定义了一个类，叫做`scala.App`，其主体是`main`方法。因此，此类继承的对象$m$是一个程序，会执行对象$m$的初始化代码。

###### 例
以下示例将通过在模块`test.HelloWorld`中定义方法`main`来创建hello world 程序。

```scala
package test
object HelloWorld {
  def main(args: Array[String]) { println("Hello World") }
}
```

该程序可以通过命令启动

```scala
scala test.HelloWorld
```

在Java环境中，该命令为

```scala
java test.HelloWorld
```

程序正常工作

通过从`App`继承，也可以在没有`main`方法的情况下定义`HelloWorld`:

```scala
package test
object HelloWorld extends App {
  println("Hello World")
}
```
