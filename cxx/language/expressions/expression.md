# Expression

表达式（expression）是操作符（operators）和操作数（operands）的序列，用于定义一个计算（computation）。
表达式可能会产生一个结果（result），比如对`2 + 2`求值会产生结果`4`，可能会产生一些意料之外的副作用。
每个C++的表达式都有两个独立的属性：`type`和`value category`

## Value Cagetory

## Order of Evaluation

## Operators

## Operator Precedence

| Precedence | Operator | Description |  Associativity  |
| - | - | - | - |
| 1 | :: | Scope resolution | L2R |
| 2 | a++, a-- | 后置++，后置-- | L2R |  
| 2 | type(), type{} | 转型 | L2R |
| 2 | a() | 函数调用 | L2R |
| 2 | a[] | 下标运算符 | L2R |
| 2 | . -> | 成员访问 | L2R |
| 3 | ++a, --a | 前置++，前置-- | R2L |
| 3 | +a, -a | 一元加和一元减 | R2L |
| 3 | !, ~ | 逻辑非，和位非 | R2L |
| 3 | (type) | C-style 转型 | R2L |
| 3 | *a | 解引用 | R2L |
| 3 | &a | 取地址 | R2L |
| 3 | sizeof | 计算字节数量 | R2L |
| 3 | co_wait | await表达式 | R2L |
| 3 | new, new[] | 动态内存分配 | R2L |
| 3 | delete, delete[] | 动态内存回收 | R2L |
| 4 | .*, ->* | 指向成员的指针 | L2R|
| 5 | a*b, a/b, a%b | 乘法，除法和取余 | L2R |
| 6 | a+b, a-b | 加法和减法 | L2R |
| 7 | << >> | 左移和右移 | L2R |
| 8 | <=> | 飞船运算符 | L2R |
| 9 | < <= > >= | 关系运算符 | L2R |
| 10 | == != | 判等运算 | L2R |
| 11 | a&b | 按位与 | L2R |
| 12 | ^ | 按位异或 | L2R |
| 13 | \| | 按位或 | L2R |
| 14 | && | 逻辑与 | L2R |
| 15 | \|\| | 逻辑或 | L2R |
| 16 | a?b:c | 三元运算符 | R2L |
| 16 | throw | throw 运算符 | R2L |
| 16 | co_yield | yield-expression | R2L |
| 16 | = | 赋值运算符 | R2L |
| 16 |+=, -=, *=, %=, <<=, >>=, &=, ^=, \|= | 复合赋值运算符 | R2L |
| 17 | , | 逗号运算符 | L2R |

## Conversions

## Memory allocation

## Other

## Primary expressions

## Literals

## Full-expressions

## Potentially-evaluated expressions

## Discarded-value expressions
