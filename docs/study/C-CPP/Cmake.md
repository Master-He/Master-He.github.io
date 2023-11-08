

# CMake Hello 项目

以下是使用 CMake 构建一个简单的 "Hello World" 项目的示例：

首先，创建一个名为 "hello_world" 的文件夹，并在该文件夹下创建以下两个文件：

1. CMakeLists.txt：用于配置项目的构建规则。

```cmake
# 指定 CMake 最低版本要求
cmake_minimum_required(VERSION 3.10)

# 设置项目名称
project(HelloWorld)

# 添加可执行目标
add_executable(hello_world main.cpp)
```



2. main.cpp：包含 "Hello World" 的源文件。



```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```



接下来，打开终端，进入 "hello_world" 文件夹，并执行以下命令：



```shell
cmake .
```

这将在当前文件夹生成构建所需的 Makefile 或其他构建系统文件。

然后，执行以下命令进行构建：



```shell
make
```

这将使用生成的 Makefile 构建项目，并生成可执行文件。

最后，执行以下命令运行项目：



```shell
./hello_world
```

输出将是 "Hello, World!"。

这是一个简单的 "Hello World" 项目的 CMake 构建示例。您可以根据自己的需求和项目复杂度，进一步扩展和配置 CMakeLists.txt 文件以满足更复杂的项目构建需求。