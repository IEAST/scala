---
title: XML
layout: default
chapter: 10
---

# XML表达式和模式

__By Burak Emir__

本章描述了XML表达式和模式的语法结构。他尽可能的遵循XML 1.0规范，由嵌入Scala代码片段的可能性强制的更改。

## XML表达式
XML表达式是由以下方法产生的表达式，第一个元素的左尖括号`<`必须要在一个开始[XML语法模式](01-lexical-syntax.html#xml-mode)的位置。

```ebnf
XmlExpr ::= XmlContent {Element}
```
要符合XML规范的良好格式，例如开始标记和结束标记必须匹配，并且属性只可以定义一次，除非与实体解析相关的限制。

以下作品描述楼scala的可扩展标记语言，其瑟及尽可能的解决W3C可扩展标记语法标准。仅更改属性值和字符数据的产生。Scala不支持声明，CDATA段或处理指令。实体引用并不在运行时解析。


```ebnf
Element       ::=    EmptyElemTag
                |    STag Content ETag

EmptyElemTag  ::=    ‘<’ Name {S Attribute} [S] ‘/>’

STag          ::=    ‘<’ Name {S Attribute} [S] ‘>’
ETag          ::=    ‘</’ Name [S] ‘>’
Content       ::=    [CharData] {Content1 [CharData]}
Content1      ::=    XmlContent
                |    Reference
                |    ScalaExpr
XmlContent    ::=    Element
                |    CDSect
                |    PI
                |    Comment
```

如果XML表达式是单个元素，则其值是XML节点(`scala.xml.Node`的子类实例)的运行时表示形式。如果XML表达式由多个元素组成，则其值是XML节点序列的运行时表示形式(`scala.Seq[scala.xml.Node]`的子类的实例)。

如果XML表达式是实体引用，CDATA部分，处理指令或注释，则它由相应Scala运行时类的表达式表示。

默认情况下，将删除元素内容中的首尾空格，连续空白字符用单个空白字符`\u0020`替换。可通过用一个编译选项来保留所有空白字符来改变该行动。

```ebnf
Attribute  ::=    Name Eq AttValue

AttValue      ::=    ‘"’ {CharQ | CharRef} ‘"’
                |    ‘'’ {CharA | CharRef} ‘'’
                |    ScalaExpr

ScalaExpr     ::=    Block

CharData      ::=   { CharNoRef } $\textit{ without}$ {CharNoRef}‘{’CharB {CharNoRef}
                                  $\textit{ and without}$ {CharNoRef}‘]]>’{CharNoRef}
```

<!-- {% raw  %} sigh: liquid borks on the double brace below; brace yourself, liquid! -->
XML表达式可能包含Scala表达式作为属性值或在节点内。在后者他们以左大括号 `{`开始右大括号`}`结束的方式嵌入。在CharDate产生的XML文本中表示单个左大括号需加倍。因此`{{`表示XML文本`{`，并不会引入嵌入的Scala表达式。
<!-- {% endraw %} -->

```ebnf
BaseChar, Char, Comment, CombiningChar, Ideographic, NameChar, S, Reference
              ::=  $\textit{“as in W3C XML”}$

Char1         ::=  Char $\textit{ without}$ ‘<’ | ‘&’
CharQ         ::=  Char1 $\textit{ without}$ ‘"’
CharA         ::=  Char1 $\textit{ without}$ ‘'’
CharB         ::=  Char1 $\textit{ without}$ ‘{’

Name          ::=  XNameStart {NameChar}

XNameStart    ::= ‘_’ | BaseChar | Ideographic
                 $\textit{ (as in W3C XML, but without }$ ‘:’$)$
```

## XML模式

XML模式是由以下方式产生的模式，元素模式的左尖括号`<`必须在开始[XML模式](01-lexical-syntax.html#xml-mode)词法的起始位置。

```ebnf
XmlPattern  ::= ElementPattern
```

XML规范的格式具有良好格式限制。

XML模式必须是单个元素模式。他完全匹配XML树的运行时表示，这些表示具有与模式描述的结构相同的结构。XML模式可以表示[Scala模式](08-pattern-matching.html#pattern-matching-expressions)。

空格的处理方式与XNL表达式相同。

默认情况下，将删除元素内容中的首尾空格，连续空白字符用单个空白字符`\u0020`替换。可通过用一个编译选项来保留所有空白字符来改变该行动。

```ebnf
ElemPattern   ::=    EmptyElemTagP
                |    STagP ContentP ETagP

EmptyElemTagP ::=    ‘<’  Name [S] ‘/>’
STagP         ::=    ‘<’  Name [S] ‘>’
ETagP         ::=    ‘</’ Name [S] ‘>’
ContentP      ::=    [CharData] {(ElemPattern|ScalaPatterns) [CharData]}
ContentP1     ::=    ElemPattern
                |    Reference
                |    CDSect
                |    PI
                |    Comment
                |    ScalaPatterns
ScalaPatterns ::=    ‘{’ Patterns ‘}’
```
