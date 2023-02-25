# Font Matching Algorithm

## 引言

浏览器字体匹配的算法是特指CSS Fonts Module标准中定义的算法，是所有浏览器都必须实现的算法。CSS Fonts Module标准实行的Level3,
Level4标准目前还是草案，改动还是有的。所以还是以Level3的标准来看。字体匹配算法的作用就是将Fonts和Text关联起来，

## 流程

***必须使用UNICODE标准中定义的Default Caseless Matching，简单理解为大小写不敏感匹配即可。
***注意区分family-generic和generic-family。generic-family总共有serif（有衬线），sans-serif（无衬线），cursive，fantasy和monospace（等宽）。所谓的generic-family是一种概念意义上的字体族，浏览器会提供设置供用户选择。比如，用户可以将Yahei指定为monospace，那实际上monospace就是Yahei了。CSS标准建议，将generic-family放到family-name的最后，作为字体generic-fallback的候选字体族。
CSS规定，所有的实现都必须至少有一个generic-family匹配到
***composite fonts和synthetic fonts

使用last-Sort

serif：英文：Times New Roman，中文：宋体
sans-serif：英文：Helvetica，中文：黑体
cursive:
fantasy:
monospace: 主要是用作代码的

### 1. Style matching

简单来说，匹配fonts的依据就是HTML元素中的font-family，font-weight，font-style和font-stretch（注意没有font-size）。
font-weight
100 ~ 900
font-stretch
ultra-expanded ~ ultra-condensed
font-style
normal，italic，oblique
这样一组合就有243种，所幸weight和font-style可以通过浏览器合成，又因为font-stretch实际上用的非常少，基本上都是normal，
所以经常就有这种。

### 2

## Localized name matching

本地化名字匹配，

## Matching font styles

一个字体文件支持某个字符，需要满足：
(1)字体文件的字符映射表包含了这个字符。
(2)字体文件包含了这个字符的字形信息。

如何选择一个字体文件：
(1) Family name
(2) Font properties

有些遗留的字体可能满足条件(1)但是不满足条件(2),

流程：
family-name也有初始值，但不是CSS规定的，而是浏览器自行决定。
default font的概念，在一个family中，三个属性都为默认值的font。WWS模型（ weight, width or slope）
font-stretch：Normal，100 %
font-style：Normal，0
font-weight：Normal(Regular)，400

还有一种模型就是R/B/I/BI family model.

这些属性可以是动态的，由渲染的引擎去合成，
或者是静态的，以单独的字体文件表示。

从设计的角度来看，一个排版丰富的字体系列可能包含大量的样式变体，涉及多个设计变体轴。例如，一些可变字体有十几个或更多的轴。现代应用程序的设计应适应具有这种多样性的家庭。

然而，许多应用程序受到关于字体族中可能包含的设计变化的面或轴的传统假设的约束。一些应用程序使用字体系列模型，该模型仅允许将重量、宽度或斜度（斜体/倾斜/倾斜）作为设计轴（“WWS”模型）。一些应用程序使用的模型甚至更为有限，该模型仅允许在一个系列中使用常规、粗体、斜体或粗体-斜体字体（“R/B/I/BI”模型）。

在使用有限字体族模型的应用程序中，字体字体族中不适合应用程序族模型的任何样式差异都必须视为单独的族。例如，如果应用程序使用R/B/I/BI模型，则必须将Calibri Light字体视为“Calibri Light”系列中的“常规”样式。因此，需要将印刷字体族和子族名称投影到传统字体族模型中，并使用交替的族和子类名称。

给定的族将具有一组特定的属性类型，成员字体因其不同：变化轴。这些可以被实现为可变字体的动态变化；或者它们可以以离散的静态实例字体的形式实现。例如，重量是Skia可变字体中的一个可变轴，但也反映为Arial系列中的设计变化轴，静态实例包括“Arial Regular”和“Arial Bold”。

一、浏览器需要计算出HTML元素的CSS Style，并获取其中的font-family，然后根据font-stretch，font-style，font-weight，选出同一个family中的具体的font，font-family的计算优先级是从左到右。
二、如果遇到的family name是一个generic family name，那么浏览器就会去查找generic family name在设置中对应的family name。这一部分受到用户设置的影响。
三、如果是其他的family name，那么浏览器会首先在@font-face规则中进行匹配，然后再去Installed font里面进行匹配。如果@font-face中的family name与installed font中的family重名的话，@font-face中的字体规则会覆盖installed font中的字体。这种覆盖是很强力的，即便@font-face中的字体出现问题导致不可用，浏览器也不会再去installed font中匹配，直接进入下面一个family name的匹配。
四、如果family匹配成功，那么浏览器会尝试使用`font-stretch`,`font-style`,`font-weight`这个三个字体属性为依据，在整个`family`中挑出一个单独的字体文件。如果这个`family`中的`font`

Ref:

[1]<https://drafts.csswg.org/css-fonts/#font-matching-algorithm>
[2]<https://learn.microsoft.com/en-us/typography/opentype/spec/otvaroverview>
[3]<https://learn.microsoft.com/en-us/typography/opentype/spec/stat>

字体的family-name不区分大小写。

## Chromium 中的字体匹配流程

相关的类
Font
FontDescription
FontFallbackList
FontSelector
FontFaceCache
FontCache
