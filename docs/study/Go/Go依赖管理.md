`go module`是Go1.11版本之后官方推出的版本管理工具，并且从Go1.13版本开始，`go module`将是Go语言默认的依赖管理工具。

### GO111MODULE

要启用`go module`支持首先要设置环境变量`GO111MODULE`，通过它可以开启或关闭模块支持，它有三个可选值：`off`、`on`、`auto`，默认值是`auto`。

1. `GO111MODULE=off`禁用模块支持，编译时会从`GOPATH`和`vendor`文件夹中查找包。
2. `GO111MODULE=on`启用模块支持，编译时会忽略`GOPATH`和`vendor`文件夹，只根据 `go.mod`下载依赖。
3. `GO111MODULE=auto`，当项目在`$GOPATH/src`外且项目根目录有`go.mod`文件时，开启模块支持。

简单来说，设置`GO111MODULE=on`之后就可以使用`go module`了，以后就没有必要在GOPATH中创建项目了，并且还能够很好的管理项目依赖的第三方包信息。

使用 go module 管理依赖后会在项目根目录下生成两个文件`go.mod`和`go.sum`。





go proxy

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```





# go glide

入门参考文档

https://www.huweihuang.com/golang-notes/introduction/package/glide-usage.html

https://glidedocs.readthedocs.io/zh/latest/getting-started/





防止被墙

glide mirror set golang.org/x/crypto github.com/golang/crypto

参考https://juejin.cn/post/6844903784242479117





https://www.cnblogs.com/laolieren/p/pull_golang_vendor_with_gogs_and_glide.html

https://www.cnblogs.com/laolieren/p/golang_import_tool_glide.html

