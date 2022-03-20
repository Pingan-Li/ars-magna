# GN
## 1. 概述
GN是Chromium的御用构建工具，它不直接执行构建任务，而是将Ninja作为后端。GN通过分析构建文件，生成Ninja的构建文件，然后由Ninja执行具体的构建任务。因此，GN自称是meta build system。这样做的好处非常明显。首先，Ninja只需定义一些非常简单的语法就能完成工作，这有助于Ninja的功能保持正交和稳定。
## 2. 设计哲学

