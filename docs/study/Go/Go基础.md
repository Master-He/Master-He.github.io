

# GO环境变量

参考 https://blog.csdn.net/qq_38151401/article/details/105729884

https://cloud.tencent.com/developer/article/1650021





GOROOT, GOPATH

Go开发相关的环境变量如下：

- GOROOT：GOROOT就是Go的安装目录，（类似于java的JDK）
- GOPATH：GOPATH是我们的工作空间,保存go项目代码和第三方依赖包

**`GOPATH`**可以设置多个，其中，第一个将会是默认的包目录，使用 go get 下载的包都会在第一个path中的src目录下，使用 go install时，在哪个**`GOPATH`**中找到了这个包，就会在哪个`GOPATH`下的bin目录生成可执行文件





GOROOT是Go的安装路径。GOROOT在绝大多数情况下都不需要修改





GOPATH是开发时的工作目录。用于：

1. 保存编译后的二进制文件。
2. `go get`和`go install`命令会下载go代码到GOPATH。
3. import包时的搜索路径





在 Go 编程语言中，有几个重要的环境变量可以设置：

1. `GOROOT`：该环境变量指定了 Go 的安装路径，即 Go 标准库和工具的位置。如果未设置该变量，则 Go 会尝试自动检测安装路径。
2. `GOPATH`：该环境变量指定了 Go 项目的工作目录，即源代码和依赖包的位置。在 GOPATH 中，通常会创建三个子目录：`src`、`bin` 和 `pkg`。其中，`src` 目录用于存放源代码，`bin` 目录用于存放编译后的可执行文件，`pkg` 目录用于存放编译后的包文件。
3. `GOBIN`：该环境变量指定了编译后的可执行文件的输出目录。如果未设置该变量，则默认为 `$GOPATH/bin`。
4. `GOOS` 和 `GOARCH`：这两个环境变量用于指定目标操作系统和处理器架构。例如，`GOOS=linux` 和 `GOARCH=amd64` 表示编译出的程序适用于 Linux 操作系统和 x86_64 处理器架构。
5. `GOENV`：该环境变量用于指定 Go 程序的运行环境，例如 `production`、`staging` 或 `development` 等。





# Go Build





# GOLAND操作

生成可执行文件： 

https://www.cnblogs.com/zz962/p/14414970.html





# 远程调试





