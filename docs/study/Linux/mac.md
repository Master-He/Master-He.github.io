

> brew问题 brew 安装的软件会放在**/usr/local/Cellar/**

我卸载brew， 结果之前下载的软件全部没了，日，气死了！/usr/local/Cellar/的目录都没了

浪费一晚上的时间！百度搜了很多垃圾教程。

最后在B站找到一个不错的教程（安装brew, 源是国内的）https://gitee.com/cunkai/HomebrewCN/



> Mac 查看CPU核数，CPU线程数， CPU型号

```shell
# cpu 型号
sysctl machdep.cpu.brand_string;
# cpu 核数
sysctl -n machdep.cpu.core_count;
# cpu 线程数 或者说 逻辑数
sysctl -n machdep.cpu.thread_count;
```

