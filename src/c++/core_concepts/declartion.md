# Declartion
在C++中，Declaration可以向C++程序中引入(或者重新引入)命名。每一种实体都有自己的声明方法。如果C++中的声明提供了足够多的属性，以至于被声明的实体可以被实际使用，那么这个声明就被称为定义。也就是，定义本质上是一种特殊的声明。

在C++中，声明是以下之一：
1. Function definition
2. Template declartion(including Partial tempalte specification)
3. Explicit template instantiation
4. Explicit tempalte sepcialization
5. Namespace definition
6. Linkage specification
7. Attribute declaration
8. Empty declaration
9. A function declration without a decl-specifier-seq
10. block declartion(a declaration that can appear inside a block)
    1.  asm declaration
    2.  type alias declaration
    3.  namespace alias defnition
    4.  using-declaration
    5.  using directive
    6.  using-enum-declartion
    7.  static_assert declartion
    8.  simple declaration

## 1. Simple declaration
所谓的简单声明(simple declaration)，是一种可以引入(introduces)，创造(creates)或初始化(initialize)一个或者多个标识符(identifier)。典型的就是变量的声明。简单声明具有以下的两种语法:
1. decl-specifier-seq init-declarator-list(optional);
2. attr decl-specifier-seq init-declarator-list;

其中：attr表示attributes序列(C++11之后)；decl-specifier-seq表示delcartion specifier 序列；init-declarator-list表示通过逗号分割的declarator(可以附带initializer)的列表。当声明一个具名的class/struct/union和enumeration时，init-declarator-list是可选的。
## 2. Specifiers
所谓的Declartion specifiers序列(decl-specifier-seq)就是以下列出specifier的组合。这些specifier之间以空格区分，并且没有顺序的要求：
1. `typdef` specifier
2. function specifier
3. `friend` specifier
4. `constexpr` specifier
5. `consteval` specifier
6. `constinit` specifier
7. storage class specifier
8. type specifier
   1. `class` specifier
   2. `enum` specifier
   3. simple type specifier
      1. fundmental type(char, int...)
      2. `auto`
      3. `decltype` specifier
      4. previously declared class name(optionally qualified)
      5. previously declared enum name(optionally qualified)
      6. previously declared typedef-name or type alias(optionally qualified)
      7. template name with template arguments(optionally qualified，optionlly using template disambiguator)
   4. elaborated type specifier
      1. the keyword `class`, `struct`, or `union`, followed by the identifier (optionally qualified), previously defined as the name of a class, struct, or union.
      2. the keyword `class`, `struct`, or `union`, followed by template name with template arguments (optionally qualified, optionally using template disambiguator), previously defined as the name of a class template.
      3. the keyword `enum` followed by the identifier (optionally qualified), previously declared as the name of an enumeration.
   5. `typename` specifier
   6. cv qualifer
在decl-specifer-seq中，只能允许一个type specifier，除以下几个情况之外：
1.  const可以与除自己之外的其他typde specifier合并
2.  volatile可以与除自己之外的其他typde specifier合并
3.  signed可以和char，int，long，short，合并。
4.  short或long可以与int合并。
5.  long可以和double合并。
我们可以发现在以上的specifier中，最为复杂也最为重要的就是type specifer。它是整个声明decl-specifier-seq的灵魂，因为他确定了整个声明的"基础"类型。后续可以通过与declarator(*, & ,&& , [])的结合，来确定整个声明的最终类型。
## 3. Declarators
init-declarator-list是一组以逗号分割的init-declarators, 具有以下两种形式的语法：
1. declarator initializer(optional)
2. declarator requries-clause(since C++20)

上面的语法分为三个部分：declarator, initializer, 和requreis-clause。我们首先来看declarator，它必须是以下的9种类型之一：
1. unqualified-id attr(optional)
2. qualified-id attr(optional)
3. ... identifier attr(optional)
4. * attr(optional) cv(optional) declarator
5. nested-name-specifier * attr(optional) cv(optional) declarator
6. & attr(optional) declarator
7. && attr(optional) declarator
8. noptr-declarator [constexpr(optional)] attr(optional)
9. noptr-declarator (parameter-list) cv(optional) ref(optional) except(optional) attr(optional)

以下是对上面的逐一解释：
1. 声明的没有作用域修饰的命名
2. 声明具有作用域修饰的命名，可以namespace或者是class作为作用域的限定
3. 参数包，只能出现在参数声明(paramnter declaration)中
4. 指针声明符，例如`S * D`声明一个指向S(decl-specifier-seq)所确定的类型的指针。
5. 成员指针声明符，类型由S(decl-specifier-seq)所确定
6. 左值饮用声明符，类型由S(decl-specifier-seq)所确定
7. 右值引用声明符，类型由S(decl-specifier-seq)所确定
8. 数组声明符，类型由S(decl-specifier-seq)所确定，如果以*, &, &&开始，那么需要使用括号包围
9. 函数指针声明符，类型由S(decl-specifier-seq)所确定，如果以*, &, &&开始，那么需要使用括号包围


Const和Volatile的两重性：
const和volatile既能出现在specifier中，又能出现在declarator中，因此：
1. If cv appears before * in the pointer declaration, it is part of `decl-specifier-seq` and applies to the `pointed-to object`.
2. If cv appears after * in the pointer declaration, it is part of `declarator` and applies to the `pointer` that's being declared.


我们可以列举出5中可能的形式：

|                |                                     |
| -------------- | ----------------------------------- |
| Syntax         | meaning                             |
| const T*       | pointer to constant object          |
| T const*       | pointer to constant object          |
| T* const       | constant pointer to object          |
| const T* const | constant pointer to constant object |
| T const* const | constant pointer to constant object |
