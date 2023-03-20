# 工具

> py-spy

https://blog.csdn.net/liming89/article/details/109663846?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&utm_relevant_index=2

```shell
# 打印当前调用栈 
py-spy dump --pid 115930
```





https://stackoverflow.com/questions/3378953/is-there-a-visual-profiler-for-python

```py
# 安装gprof2dot pip install gprof2dot
# 生成dat
python -m cProfile -o profile.dat my_program.py
# 生成svg图
python -m gprof2dot -f pstats profile.dat | dot -Tsvg -o profile.svg
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



python进程内存分析

```shell
# 13831是pid
pmap -x 13831
# 确定内存具体的开始和结束的地址
cat /proc/13831/maps | grep 0x7ff00627d000
# pmap可以看出是哪块内存大，然后用gdb打印出来
gdb python 13831
dump binary memory /home/hwj/my2.bin 0x7ff00627d000 0x7ff009b7e000

```

https://wiki.python.org/moin/DebuggingWithGdb
https://drmingdrmer.github.io/tech/programming/2017/05/06/python-mem.html
http://ponder.work/2020/12/29/debug-python-with-gdb/
https://blog.csdn.net/lanyang123456/article/details/100860904



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



# psutils

```python
import os
import psutil
res = psutil.Process(os.getpid()).memory_info().rss
with open('/tmp/mail_memory_info.txt', 'w') as f:
    f.write("res: {}B\n".format(res))
    
    
import os
import psutil
res = psutil.Process(os.getpid()).memory_info().rss
dbg.warn("before read file_info mem: {}B".format(res))

res = psutil.Process(os.getpid()).memory_info().rss
dbg.warn("after read file_info mem: {}B".format(res))
```

# memory_profiler

python2的需要用0.57.0的才兼容

```shell
pip install memory-profiler==0.57.0
```

```python
from memory_profiler import profile
import time

@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
    while True:
        time.sleep(1)
```

```shell
import time
from memory_profiler import profile

class Dog:
    def __init__(self):
        self.friends = list()

    @profile
    def bark(self):
        a = [1] * (10 ** 6)
        self.friends.append(a)


if __name__ == '__main__':
    dog = Dog()
    while True:
        dog.bark()
        time.sleep(1)
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



