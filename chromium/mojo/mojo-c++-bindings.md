# Mojo C++ bindings API

## Overview

Mojo C++绑定API利用C++系统API提供一组更自然的原语，用于通过Mojo消息管道进行通信。结合Mojom IDL和绑定生成器生成的代码，用户可以轻松地跨任意进程内和进程间边界连接接口客户端和实现。

## Getting Started

当绑定生成器处理Mojom IDL文件时，C++代码以一系列.h和.cc文件的形式发出，其名称基于输入的.Mojom文件。假如，我们有一个如下的mojom文件："//services/db/public/mojom/db.mojom"

```mojom
module db.mojom;

interface Table {
  AddRow(int32 key, string data);
};

interface Database {
  CreateTable(Table& table);
};
```

在BUILD.gn文件中添加一个新的GN Targets

```gn
import ("//mojo/public/tools/bindings/mojom.gni")

mojom("mojom") {
    sources = [
        "db.mojom"
    ]
}
```

确保需要此mojo接口的任何GN目标都依赖于此接口，例如，使用以下行：

```gn
executable("db_exe"){
  deps += [ '//services/db/public/mojom' ] 
  //...
}

```

使用ninja构建生成C++代码

```sh
ninja -C out/r servies/db/public/mojom
```

这将生成几个生成的源文件，其中一些与C++绑定相关。其中两个文件是：
out/gen/services/db/public/mojom/db.mojom.cc

out/gen/services/db/public/mojom/db.mojom.h

那么，要如何在自己的C++文件使用这些自动生成的mojo C++文件呢？
只需要使用:

```C++
#include "services/business/public/mojom/factory.mojom.h"

class TableImpl : public db::mojom::Table {
  // ...
};
```

以上就是一个最简单的例子
