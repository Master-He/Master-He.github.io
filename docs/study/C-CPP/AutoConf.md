编译一个简单的 "Hello, World!" 项目可以使用 autoconf 工具链来自动生成配置脚本。下面是一个简单的示例，演示如何使用 autoconf 编译 "Hello, World!" 项目：

1. 创建源代码文件：
   创建一个名为 `hello.c` 的文件，并将以下代码复制到文件中：

   ```
   #include <stdio.h>
   int main() {
       printf("Hello, World!\n");
       return 0;
   }
   ```

2. 创建 `configure.ac` 文件：
   创建一个名为 `configure.ac` 的文件，并将以下内容复制到文件中：

   ```
   AC_INIT([hello], [1.0])
   AC_PREREQ([2.69])
   AC_CONFIG_SRCDIR([hello.c])
   AC_CONFIG_HEADERS([config.h])
   
   AM_INIT_AUTOMAKE([-Wall -Werror foreign])
   
   AC_PROG_CC
   
   AC_OUTPUT(Makefile)
   ```


3. 创建 `Makefile.am` 文件：
   创建一个名为 `Makefile.am` 的文件，并将以下内容复制到文件中：

   ```
   bin_PROGRAMS = hello
   hello_SOURCES = hello.c
   ```
   
4. 生成 `configure` 脚本：
   打开终端，导航到包含 `hello.c` 和 `configure.ac` 文件的目录，并运行以下命令：

   ```
   autoreconf -i
   
   这将根据 `configure.ac` 文件生成 `configure` 脚本和其他必要的配置文件。
   ```


5. 生成 `Makefile.in` 文件：
   运行以下命令以生成 `Makefile.in` 文件：

   ```
   automake --add-missing
   
   这将根据 `Makefile.am` 文件生成 `Makefile.in` 文件。
   ```

6. 运行 `configure`：
   运行以下命令以生成 `Makefile` 文件：

   ```
   ./configure
   
   这将根据系统配置生成 `Makefile` 文件。
   ```

7. 编译项目：
   运行以下命令以编译项目：

   ```
   make
   
   这将编译源代码并生成可执行文件。
   ```

8. 运行项目：
   运行以下命令以运行项目：

   ```
   ./hello
   
   这将输出 "Hello, World!"。
   ```

这样，您就使用 autoconf 工具链成功编译并运行了一个简单的 "Hello, World!" 项目。请注意，这是一个简化的示例，实际项目可能需要更复杂的配置和构建步骤。