# 工具

> py-spy

https://blog.csdn.net/liming89/article/details/109663846?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&utm_relevant_index=2

```shell
# 打印当前调用栈 
py-spy dump --pid 115930
```





https://stackoverflow.com/questions/3378953/is-there-a-visual-profiler-for-python

```py
  python -m cProfile -o profile.dat my_program.py
  gprof2dot.py -f pstats profile.dat | dot -Tpng -o profile.png
```









# 速率

```
profile
	pycharm带的profile
time.time()分析方法也不错
	注意，使用这个有点影响代码执行时间
	特别适合常驻进程，特别多time.sleep干扰的时候
```



# 内存

Memory Profiling in Python - Checking Code Memory Usage (2021)

​	https://www.youtube.com/watch?v=tIKo2uex8x4





> memory_profiler

在函数上面加装饰器@profile
注意，使用这个有点影响代码执行时间



> 查看内存占用

```python
import os
import psutil
print("current memory：{}MB".format(psutil.Process(os.getpid()).memory_info().rss / 1024 / 1024))
```



> linux 查看进程内存

```shell
ps aux | grep 进程
watch -n 0.1 cat /proc/[进程pid]/status
top -c -d 0.5 | grep dnsdetect
```



## 大内存对象分析

https://blog.csdn.net/qq_16681169/article/details/113804000





# CPU





```shell
[root@localhost111 ~]# py-spy -help
py-spy 0.3.11
Sampling profiler for Python programs

USAGE:
    py-spy <SUBCOMMAND>

OPTIONS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    record    Records stack trace information to a flamegraph, speedscope or raw file
    top       Displays a top like view of functions consuming CPU
    dump      Dumps stack traces for a target program to stdout
    help      Prints this message or the help of the given subcommand(s)

[root@localhost111 ~]# py-spy top --pid 115930
```







# IO

linux 下的

```
iotop
iftop
```



## 网络IO

​	

## 磁盘IO



