# CSS Selector

## 1. 选择器

所谓的CSS选择器，他的作用就是从DOM树中选择能匹配某种模式的元素。
总的来说CSS选择器按照如下的模式工作:

    expression * element -> boolean
选择器的表达式会对DOM中的元素进行匹配，在挑选出元素之后，浏览器就能根据CSS规则中定义的样式，在网页上展现出对应的视觉效果。

## 2. 语法规则

选择器(Selector)是一个或多个由组合器分隔的简单选择器序列组成的链。 一个伪元素可以附加到选择器中的最后一个简单选择器序列。

简单选择器序列(sequence of simple selectors)是不被组合器分隔的简单选择器链。 它总是以类型选择器或通用选择器开头。 序列中不允许有其他类型选择器或通用选择器。

简单选择器(simple selector)可以是type选择器、universal选择器、arribute选择器、class选择器、ID选择器或伪类。

### 2.1 组合器

组合器包括:

    whitespace, "greater-than sign" (U+003E, >), "plus sign" (U+002B, +) and "tilde" (U+007E, ~). White space may appear between a combinator and the simple selectors around it. Only the characters "space" (U+0020), "tab" (U+0009), "line feed" (U+000A), "carriage return" (U+000D), and "form feed" (U+000C) can occur in whitespace.

## 3. 简单选择器(Simple Selector)

## 3.1 Type Selector

Type Selector会匹配一个给定的DOM元素类型。
比如:

```css
input
```

会匹配DOM中所有的input元素。这很好理解，但是如果和命名空间搭配起来就复杂了，总之有以下的四种情况：

```text
ns|E
elements with name E in namespace ns
*|E
elements with name E in any namespace, including those without a namespace
|E
elements with name E without a namespace
E
if no default namespace has been declared for selectors, this is equivalent to *|E. Otherwise it is equivalent to ns|E where ns is the default namespace.
```

## 3.2 Universal Selector

将会匹配DOM中所有的元素，用*表示。在某些情况下可以被省略:

```text
*[hreflang|=en] and [hreflang|=en] are equivalent,
*.warning and .warning are equivalent,
*#myid and #myid are equivalent.
```

## 3.3 Attribute Selector

选出DOM中所有具有给定Attribute的元素，具有多种形态可以使用。
| 形式|含义 |
|-|-|
|[att]| Represents an element with the att attribute, whatever the value of the attribute.|
|[att=val]|epresents an element with the att attribute whose value is exactly "val". |
|[att~=val]|Represents an element with the att attribute whose value is a whitespace-separated list of words, one of which is exactly "val". If "val" contains whitespace, it will never represent anything (since the words are separated by spaces). Also if "val" is the empty string, it will never represent anything.|
|[att\|=val]|Represents an element with the att attribute, its value either being exactly "val" or beginning with "val" immediately followed by "-" (U+002D). This is primarily intended to allow language subcode matches (e.g., the hreflang attribute on the a element in HTML) as described in BCP 47 ([BCP47]) or its successor. For lang (or xml:lang) language subcode matching, please see the :lang pseudo-class.|
|[att^=val]|Represents an element with the att attribute whose value begins with the prefix "val". If "val" is the empty string then the selector does not represent anything.|
|[att$=val]|Represents an element with the att attribute whose value ends with the suffix "val". If "val" is the empty string then the selector does not represent anything.|
|[att*=val]|Represents an element with the att attribute whose value contains at least one instance of the substring "val". If "val" is the empty string then the selector does not represent anything.|

## 3.4 Class Selector

按照元素的class属性进行匹配，可以视为是一种与语法糖

```css
div.name
```

等价于

```
div[class~=value]
```

还有下面的例子：

```text
CSS examples:

We can assign style information to all elements with class~="pastoral" as follows:

*.pastoral { color: green }  /* all elements with class~=pastoral */
or just

.pastoral { color: green }  /* all elements with class~=pastoral */
The following assigns style only to H1 elements with class~="pastoral":

H1.pastoral { color: green }  /* H1 elements with class~=pastoral */
Given these rules, the first H1 instance below would not have green text, while the second would:

<H1>Not green</H1>
<H1 class="pastoral">Very green</H1>
The following rule matches any P element whose class attribute has been assigned a list of whitespace-separated values that includes both pastoral and marine:

p.pastoral.marine { color: green }
This rule matches when class="pastoral blue aqua marine" but does not match for class="pastoral blue".
```

## 3.5 ID Selector

按照元素的id属性进行匹配

```text

Examples:

The following ID selector represents an h1 element whose ID-typed attribute has the value "chapter1":

h1#chapter1
The following ID selector represents any element whose ID-typed attribute has the value "chapter1":

#chapter1
The following selector represents any element whose ID-typed attribute has the value "z98y".

*#z98y
```

## 3.6 Pesudo-classes



## 4. 选择器组

一个以逗号分隔的选择器列表表示由列表中的每个单独选择器选择的所有元素的并集。 （逗号是 U+002C。）例如，在 CSS 中，当多个选择器共享相同的声明时，它们可能会被分组到一个逗号分隔的列表中。 空格可能出现在逗号之前和/或之后。

CSS example:

In this example, we condense three rules with identical declarations into one. Thus,

```css

h1 { font-family: sans-serif }
h2 { font-family: sans-serif }
h3 { font-family: sans-serif }
```

is equivalent to:

```css
h1, h2, h3 { font-family: sans-serif }
```

千万要注意，选择器组中有一个选择器是非法的，那么整个选择器组也是非法的。
Invalid CSS example:

```css
h1 { font-family: sans-serif }
h2..foo { font-family: sans-serif }
h3 { font-family: sans-serif }
```

is not equivalent to:

```css
h1, h2..foo, h3 { font-family: sans-serif }
```

because the above selector (h1, h2..foo, h3) is entirely invalid and the entire style rule is dropped. (When the selectors are not grouped, only the rule for h2..foo is dropped.)
