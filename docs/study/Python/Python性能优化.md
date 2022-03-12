# 速率

```
profile
	pycharm带的profile
time.time()分析方法也不错
	注意，使用这个有点影响代码执行时间
	特别适合常驻进程，特别多time.sleep干扰的时候
```



# 内存

​	

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



# CPU



# IO

linux 下的

```
iotop
iftop
```



## 网络IO

​	

## 磁盘IO



