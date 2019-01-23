---
title: Annotations
layout: default
chapter: 11
---

# 标注

```ebnf
  Annotation       ::=  ‘@’ SimpleType {ArgumentExprs}
  ConstrAnnotation ::=  ‘@’ SimpleType ArgumentExprs
```

## 定义

标注将元信息与定义相关联。一个简单的标注具有`@$c$`或`@$c(a_1 , \ldots , a_n)$`的形式。这里$c$是类$C$的构造函数，它必须符合类`scala.Annotation`。


标注可以应用在定义或声明，类型或表达式上。定义或声明的标注出现在该定义的前面，类型的标注在该类型之后。表达式$e$的标注出现在表达式$e$之后，用冒号分隔。对一个实体可应用多个标注。标注给出的顺序没有意义。

例:

```scala
@deprecated("Use D", "1.0") class C { ... } // Class 标注
@transient @volatile var m: Int             // Variable 标注
String @local                               // Type 标注
(e: @unchecked) match { ... }               // Expression 标注
```

## 预定义的标注

### Java平台标注

标注子句的含义取决于实现。在java平台上，一下标注具有标准含义：

  * `@transient`将字段标记为非持久性；这相当于java中的`transient`修饰符。  

  * `@volatile`标记一个可以在程序控制之外改变其值的字段，这相当于java中的`volatile`修饰符。

  * `@SerialVersionUID(<longlit>)`将序列版本标识符(`long`常量)附加道类。这相当于java中的以下字段

    ```java
    private final static SerialVersionUID = <longlit>
    ```

  * `@throws(<classlit>)` java编译器会通过分析一个方法或构造器是否会导致已检查异常来检查程序是否包含已检查异常的处理器。对于每个可能的已检查异常，方法或构造其的`throws`子句必须要说明该异常的类或该异常的类的一个超类。

### Java Bean标注

  * `@scala.beans.BeanProperty` 当作为定义或某个变量`X`的前缀时，该标注将使java bean 生成的getter 和setter方法`getX`和`setX`添加到该变量所在的类中。`get`和`set`后变量的第一个字母将变为大小写。当该标注加在一个不可变值的定义`X`上时则只产生getter。这些方法的构建是代码生成的一部分;因此这些方法只有在对应的class文件生成后才可见。

  * `@scala.beans.BooleanBeanProperty` 此标注等同于`scala.reflect.BeanProperty`，单生成的getter方法名为`isX`而不是`getX`。

### 弃用标注

  * `@deprecated(message: <stringlit>, since: <stringlit>)`<br/>
将定义标记为弃用。对已定义实体的访问将导致一个不赞成的警告，提示编译器将发出_消息_`<stringlit>`。_自文_ 档依赖的参数，因为该定义应被视为已弃用。  
  已弃用的警告在代码中被禁止，该代码属于标记为已弃用的定义。

  * `@deprecatedName(name: <stringlit>, since: <stringlit>)`<br/>
    将正式参数名称标记为已弃用。使用引用已弃用参数名称的命名参数语法对此实体进行调用会导致警告。

### Scala编译器标注

  * `@unchecked` 当引用于`match`表达式的选择器时，此属性会抑制所有有关不完全的匹配模式匹配的警告，这些匹配将会被忽略。比如一下方式定义不会参数警告。
    ```scala
    def f(x: Option[Int]) = (x: @unchecked) match {
    case Some(y) => y
    }
    ```
    如果没有`@unchecked`标注，Scala编译器就会指出模式匹配是不完整的，并参生一个警告(因为`Option`是一个`sealed`类)。

  *
  `@uncheckedStable`当应用与一个值声明或定义时，则运行定义的值出现在一个路径中，即使其类型是[易变的](03-types.html#volatile-types)。例如，以下定义是合法的：

    ```scala
    type A { type T }
    type B
    @uncheckedStable val x: A with B // volatile type
    val y: x.T                       // OK since `x' is still a path
    ```
    如果没有`$uncheckedStable`标记，则指示符`x`将不是路径，因为其类型`A with B`是易失性的。继而引用`x.T`是格式错误的。

    当应用于非可变类型的值声明或定义时，该标注没有作用。

  * `@specialized` 当应用于类型参数的定义时，此批注会使编译器为基本类型生成专用定义。可以给出可选的基本类型列表，在这种情况下，专业化仅考虑那些类型。例如，以下代码将会`Unit`, `Int` 和 `Double`生成专门的特征。

    ```scala
    trait Function0[@specialized(Unit, Int, Double) T] {
      def apply: T
    }
    ```

    只要表达式的静态类型与定义的特定变体匹配，编译器就会使用专用版本。有关实现的更多详细信息，请参阅[specialization sid](http://docs.scala-lang.org/sips/completed/scala-specialization.html)。


## 用户定义的标注

其他标注可以由平台或应用程序相关工具解释。类`scala.annotation.Annotation`是用户定义的标注的基类。它有两个子特征：

- `scala.annotation.StaticAnnotation`:此特征的子类的实例将存储在生成的类文件中，因此可供运行时反射和以后的编译运行访问。
- `scala.annotation.ConstantAnnotation`: 此特征的子类的实例可能只有[常量表达式](06-expressions.html#constant-expressions)的参数，并且也存储在生成的类文件中。
- 如果标注类既不继承`scala.ConstantAnnotation` 也不继承  `scala.StaticAnnotation`, 则其实例仅在分析它们的编译运行期间在本地可见。

## 主机平台标注

主机平台可以定义其自己的标注格式。 这些标注不会扩展`scala.annotation`包中的任何类，但通常可以像Scala标注一样使用。主机平台会对作为标注参数有效的表达式施加额外限制。
