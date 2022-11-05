# Mojom IDL and Bindings Generator
## Overview
Mojom用于生成Mojo协议接口(bindings interfaces)的IDL，文件的后缀一般是`.mojom`。Mojom文件需要通过协议生成器（Bindings Generator）生成特定编程语言（目前支持C++，JavaScript和Java）的接口。
## Mojom Syntax
Mojom IDL支持定义结构体`structs`，联合体`unions`，接口`interfaces`，常量`constants`和枚举`enums`，以上都必须定义在一个模块`module`中。这些定义在项目构建的时候被用来生成的代码。Mojom文件内可以导入其他的Mojom文件，这样可以使用其他文件内的定义。
### Primitive Types
Mojom支持一些基本的数据类型，这些基本的数据类型可以放入结构体中。
| Type                         | Description                                                     |
| ---------------------------- | --------------------------------------------------------------- |
| bool                         | 布尔类型                                                        |
| int[8, 16, 32, 64]           | 有符号数                                                        |
| uint[8, 16 ,32, 64]          | 无符号数                                                        |
| float，double                | 单精度和双精度浮点数                                            |
| string                       | UTF-8编码的字符串                                               |
| array\<T\>                   | 数组类型，元素类型的为T；比如：array\<uint8\>                   |
| array\<T, N\>                | 固定长度的数组类型，元素类型为T，N必须是整数字面量，            |
| map\<S, T\>                  | map类型，S必须为字符串，枚举，或是整数                          |
| handle                       | 通用的Mojo句柄，可以是任意类型的，包括封装的native句柄          |
| handle\<message_pipe\>       | 通用message pipe句柄                                            |
| handle\<shared_buffer\>      | Shared pipe句柄                                                 |
| handle\<data_pipe_producer\> | Data pipe producer handle.                                      |
| handle\<data_pipe_consumer\> | Data pipe consumer handle.                                      |
| InterfaceType                | Any user-defined Mojom interface type.                          |
| InterfaceType&               | An interface request for any user-defined Mojom interface type. |
| associated InterfaceType     | An associated interface handle.                                 |
| associated InterfaceType&    | An associated interface request.                                |
| T?                           | An optional value. Primitive types.                             |
### Modules
每个Mojom文件就定义了一个模块。代码生成器会根据Mojom的模块将生成的符号放入相应的命名空间中。
比如：
```mojom
module business.stuff;
interface MoneyGenerator {
    GenerateMoney();
};
```
以上的mojom会生成一个MoneyGenerator位于business::stuff中。
```C++
namespace business{
namespace sutff{
class MoneyGenerator {
    void GenerateMoney() = 0;
};    
}// namespace business
}// namespace stuff
```

### Imports
如果需要导入外部的mojom文件，使用
```mojom
import "services/widget/public/interfaces/frobinator.mojom";
```
导入的路径永远是top-level目录的相对路径，注意不支持循环引用。
### Structs
结构体使用`struct`关键词，结构体主要用于将若干变量集中到一起。
```mojom
struct StringPair {
    string first;
    string second;
};
```
结构体支持默认值
```mojom
struct Request {
    int32 id = -1;
    string details;
};
```
下面是一个使用支持的字段类型的相当全面的示例:
```mojom
struct StringPair {
  string first;
  string second;
};

enum AnEnum {
  YES,
  NO
};

interface SampleInterface {
  DoStuff();
};

struct AllTheThings {
  // Note that these types can never be marked nullable!
  bool boolean_value;
  int8 signed_8bit_value = 42;
  uint8 unsigned_8bit_value;
  int16 signed_16bit_value;
  uint16 unsigned_16bit_value;
  int32 signed_32bit_value;
  uint32 unsigned_32bit_value;
  int64 signed_64bit_value;
  uint64 unsigned_64bit_value;
  float float_value_32bit;
  double float_value_64bit;
  AnEnum enum_value = AnEnum.YES;

  // Strings may be nullable.
  string? maybe_a_string_maybe_not;

  // Structs may contain other structs. These may also be nullable.
  StringPair some_strings;
  StringPair? maybe_some_more_strings;

  // In fact structs can also be nested, though in practice you must always make
  // such fields nullable -- otherwise messages would need to be infinitely long
  // in order to pass validation!
  AllTheThings? more_things;

  // Arrays may be templated over any Mojom type, and are always nullable:
  array<int32> numbers;
  array<int32>? maybe_more_numbers;

  // Arrays of arrays of arrays... are fine.
  array<array<array<AnEnum>>> this_works_but_really_plz_stop;

  // The element type may be nullable if it's a type which is allowed to be
  // nullable.
  array<AllTheThings?> more_maybe_things;

  // Fixed-size arrays get some extra validation on the receiving end to ensure
  // that the correct number of elements is always received.
  array<uint64, 2> uuid;

  // Maps follow many of the same rules as arrays. Key types may be any
  // non-handle, non-collection type, and value types may be any supported
  // struct field type. Maps may also be nullable.
  map<string, int32> one_map;
  map<AnEnum, string>? maybe_another_map;
  map<StringPair, AllTheThings?>? maybe_a_pretty_weird_but_valid_map;
  map<StringPair, map<int32, array<map<string, string>?>?>?> ridiculous;

  // And finally, all handle types are valid as struct fields and may be
  // nullable. Note that interfaces and interface requests (the "Foo" and
  // "Foo&" type syntax respectively) are just strongly-typed message pipe
  // handles.
  handle generic_handle;
  handle<data_pipe_consumer> reader;
  handle<data_pipe_producer>? maybe_writer;
  handle<shared_buffer> dumping_ground;
  handle<message_pipe> raw_message_pipe;
  SampleInterface? maybe_a_sample_interface_client_pipe;
  SampleInterface& non_nullable_sample_interface_request;
  SampleInterface&? nullable_sample_interface_request;
  associated SampleInterface associated_interface_client;
  associated SampleInterface& associated_interface_request;
  associated SampleInterface&? maybe_another_associated_request;
};
```
### Unions
Mojom支持使用union关键字标记的联合。联合是一组字段，一次可以取其中任何一个字段的值。因此，它们提供了一种表示变量值类型的方法，同时最小化了存储需求。例如：
```mojom
union ExampleUnion {
  string str;
  StringPair pair;
  int64 id;
  array<uint64, 2> guid;
  SampleInterface iface;
};
```
### Enumeration Types
枚举类型可以直接在模块中使用enum关键字定义，也可以嵌套在某些结构体或接口的命名空间中
```mojom
module business.mojom;

enum Department {
  SALES = 0,
  DEV,
};

struct Employee {
  enum Type {
    FULL_TIME,
    PART_TIME,
  };

  Type type;
  // ...
};
```
与C样式枚举类似，单个值可以在枚举定义中显式赋值。默认情况下，值以0为基础，并按顺序递增1。嵌套定义对生成的绑定的影响因目标语言而异。
### Constants
常量可以使用const关键字直接定义在模块中，也可以嵌套在某个结构或接口的命名空间中
```mojom
module business.mojom;

const string kServiceName = "business";

struct Employee {
  const uint64 kInvalidId = 0;

  enum Type {
    FULL_TIME,
    PART_TIME,
  };

  uint64 id = kInvalidId;
  Type type;
};
```
嵌套定义对生成的绑定的影响因目标语言而异。
### Interfaces
接口是参数化请求消息的逻辑束。每个请求消息可以可选地定义参数化响应消息。下面是定义具有各种请求的接口Foo的示例：
```mojom
interface Foo {
  // A request which takes no arguments and expects no response.
  MyMessage();

  // A request which has some arguments and expects no response.
  MyOtherMessage(string name, array<uint8> bytes);

  // A request which expects a single-argument response.
  MyMessageWithResponse(string command) => (bool success);

  // A request which expects a response with multiple arguments.
  MyMessageWithMoarResponse(string a, string b) => (int8 c, int8 d);
};
```
任何有效的结构字段类型（请参见Structs）也是有效的请求或响应参数类型。两者的类型符号相同。
### Attributes
Mojom定义的含义可能会被Attributes改变，这些属性使用类似于Java或C#属性的语法指定。

[Sync] 可以为任何需要响应的接口方法指定Sync属性。这使得方法的调用方可以同步等待响应。请参见C++绑定文档中的同步调用。请注意，其他目标语言当前不支持同步调用。

[Extensible] 可以为任何枚举定义指定可扩展属性。这实际上是在消息中接收枚举类型的值时禁用内置范围验证，从而允许旧绑定容忍来自较新版本的枚举的未识别值。

[Native] 可以为空结构声明指定Native属性，以在Mojo IPC和传统IPC:：ParamTraits或IPC_struct_TRAITS*宏之间提供名义桥。有关详细信息，请参见使用传统IPC特性。请注意，此属性的支持仅限于C++

[MinVersion=N] MinVersion属性用于指定引入给定字段、枚举值、接口方法或方法参数的版本。有关详细信息，请参见版本控制。

[EnableIf=value] EnableIf属性用于在解析mojom时有条件地启用定义。如果GN文件中的mojom目标不包括enabled_features列表中的匹配值，则定义将被禁用。这对于仅在一个平台上有意义的mojom定义很有用。请注意，每个定义只能设置EnableIf属性一次。

## Generated Code For Target Languages
当绑定生成器成功处理输入Mojom文件时，它会为每个支持的目标语言发出相应的代码:
1. C++
2. JavaScrit
3. Java
## Message Validation
无论目标语言如何，在将所有接口消息发送到接口的接收实现之前，都会在反序列化期间进行验证。这有助于确保跨接口的一致性验证，而不会在每次添加新消息时都给开发人员和安全审查人员留下负担。
如果消息未通过验证，则永远不会发送该消息。而是在绑定对象上引发连接错误（有关详细信息，请参见C++连接错误、Java连接错误或JavaScript连接错误）
对于原始Mojom类型，一些基线级别的验证是自动完成的
### Non-Nullable Objects
Mojom字段或参数值（例如，结构、接口、数组等）可以在Mojom定义中标记为null（请参见基本类型）。如果字段或参数未标记为nullable，但收到的消息中包含null值，则该消息将无法验证。
### Enums
Mojom中声明的枚举将根据合法值范围自动验证。例如，如果Mojom声明枚举：
```mojom
enum AdvancedBoolean {
  TRUE = 0,
  FALSE = 1,
  FILE_NOT_FOUND = 2,
};
```

## Grammar Reference
```text
Below is the (BNF-ish) context-free grammar of the Mojom language:

MojomFile = StatementList
StatementList = Statement StatementList | Statement
Statement = ModuleStatement | ImportStatement | Definition

ModuleStatement = AttributeSection "module" Identifier ";"
ImportStatement = "import" StringLiteral ";"
Definition = Struct Union Interface Enum Const

AttributeSection = "[" AttributeList "]"
AttributeList = <empty> | NonEmptyAttributeList
NonEmptyAttributeList = Attribute
                      | Attribute "," NonEmptyAttributeList
Attribute = Name
          | Name "=" Name
          | Name "=" Literal

Struct = AttributeSection "struct" Name "{" StructBody "}" ";"
       | AttributeSection "struct" Name ";"
StructBody = <empty>
           | StructBody Const
           | StructBody Enum
           | StructBody StructField
StructField = AttributeSection TypeSpec Name Orginal Default ";"

Union = AttributeSection "union" Name "{" UnionBody "}" ";"
UnionBody = <empty> | UnionBody UnionField
UnionField = AttributeSection TypeSpec Name Ordinal ";"

Interface = AttributeSection "interface" Name "{" InterfaceBody "}" ";"
InterfaceBody = <empty>
              | InterfaceBody Const
              | InterfaceBody Enum
              | InterfaceBody Method
Method = AttributeSection Name Ordinal "(" ParamterList ")" Response ";"
ParameterList = <empty> | NonEmptyParameterList
NonEmptyParameterList = Parameter
                      | Parameter "," NonEmptyParameterList
Parameter = AttributeSection TypeSpec Name Ordinal
Response = <empty> | "=>" "(" ParameterList ")"

TypeSpec = TypeName "?" | TypeName
TypeName = BasicTypeName
         | Array
         | FixedArray
         | Map
         | InterfaceRequest
BasicTypeName = Identifier | "associated" Identifier | HandleType | NumericType
NumericType = "bool" | "int8" | "uint8" | "int16" | "uint16" | "int32"
            | "uint32" | "int64" | "uint64" | "float" | "double"
HandleType = "handle" | "handle" "<" SpecificHandleType ">"
SpecificHandleType = "message_pipe"
                   | "shared_buffer"
                   | "data_pipe_consumer"
                   | "data_pipe_producer"
Array = "array" "<" TypeSpec ">"
FixedArray = "array" "<" TypeSpec "," IntConstDec ">"
Map = "map" "<" Identifier "," TypeSpec ">"
InterfaceRequest = Identifier "&" | "associated" Identifier "&"

Ordinal = <empty> | OrdinalValue

Default = <empty> | "=" Constant

Enum = AttributeSection "enum" Name "{" NonEmptyEnumValueList "}" ";"
     | AttributeSection "enum" Name "{" NonEmptyEnumValueList "," "}" ";"
NonEmptyEnumValueList = EnumValue | NonEmptyEnumValueList "," EnumValue
EnumValue = AttributeSection Name
          | AttributeSection Name "=" Integer
          | AttributeSection Name "=" Identifier

Const = "const" TypeSpec Name "=" Constant ";"

Constant = Literal | Identifier ";"

Identifier = Name | Name "." Identifier

Literal = Integer | Float | "true" | "false" | "default" | StringLiteral

Integer = IntConst | "+" IntConst | "-" IntConst
IntConst = IntConstDec | IntConstHex

Float = FloatConst | "+" FloatConst | "-" FloatConst

; The rules below are for tokens matched strictly according to the given regexes

Identifier = /[a-zA-Z_][0-9a-zA-Z_]*/
IntConstDec = /0|(1-9[0-9]*)/
IntConstHex = /0[xX][0-9a-fA-F]+/
OrdinalValue = /@(0|(1-9[0-9]*))/
FloatConst = ... # Imagine it's close enough to C-style float syntax.
StringLiteral = ... # Imagine it's close enough to C-style string literals, including escapes.
```