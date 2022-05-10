# Basis
这里会介绍一些C++的基本概念和术语。

1. 一个C++程序本质上包含声明(Declarations)的文本文件（头文件和源文件）这些文件可以通过编译器生成二进制产物，如果这些文件中定义了main函数，那么二进制产物就能在对应的运行环境(Runtime)中执行。
2. 在C++中有些单词具有特殊的含义，被称为关键词(Keywords)，用户在定义自己的标识符(identifiers)应该避开关键词。注释(comments)在编译时会被忽略。
3. C++中的实体(entities)包括：值(values)，对象(objects),引用(references)，结构化绑定(structured bindings)，函数(functions)，枚举(enumerators)，类型(types)，类成员(class members)，模版(templates)，模版特化(template specication)，命名空间(namespaces)以及参数包(parameter packs)。
4. 声明可能会引入新的实体，可以将它们与命名(name)关联起来以及定义它们的属性(properties)。如果声明包含了实体所需的所有属性，那么这个声明被视为一个定义(definitions)。也就是说，定义是一种特殊的声明。程序中所有non-inline的函数以及所有变量都需要满足ODR(one definition rule).
5. 函数的定义通常需要包含一系列的语句(statements)，其中的语句包含表达式(expression)。
6. 在C++程序中，命名可以通过命名查找(name lookup)与引它的声明关联起来。每一个命名都有它的作用域(scope)。如果命名具有(链接)linkage，那么它也可以在其他的作用域可见。
7. 每个对象，引用，函数，表达式在C++中都与某种类型(type)关联，它可以是基本类型(fundamental)，复合类型(compound)，用户定义的(user-defined)，完整的(compelete)以及不完整的(incompelete).
8. 非静态数据成员的声明对象和声明引用是变量。