# Array declaration

"Declaraes an object of array type."

## Syntax

`noptr-declarator [ expr (optional)] attr (optional)`

1. noptr-declarator :任何有效的表达式，如果以`*`, `&`和`&&`开始，那么必须用括号包围。
2. expr - 一个整数类型的常量表达式，可以转换为std:size_t.
3. attr - 属性列表.

数组可以从

1. `void`之外的基本类型。
2. 指针以及指向成员的指针。
3. 类，结构体和枚举。
4. 界限确定的数组。

## new[] 操作符

```C++
int *p = new int[0] // accessing p[0]或*p是未定义的， 注意malloc(0)的行为也是取决于实现，要么是空指针，要么是错误。
delete[] p; // 依然需要清理。
```

## 赋值

数组不能够在整体上被修改，即便他们是左值，数组不能够出现在赋值元算符的左边。

```C++
int a[3] = {1, 2, 3}, b[3] = {4, 5, 6};
int (*p)[3] = &a; // okay: address of a can be taken
a = b;            // error: a is an array
 
struct { int c[3]; } s1, s2 = {3, 4, 5};
s1 = s2; // okay: implicitly-defined copy assignment operator
         // can assign data members of array type
```

## Array-to-pointer decay

从数组类型的左值和右值到指针类型的右值有一个隐式转换：它构造一个指向数组第一个元素的指针。每当数组出现在不需要数组但指针的上下文中时，就会使用此转换：

```C++

```
