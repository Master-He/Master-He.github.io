

# Python基础



## 安装



linux下安装问题















##  魔法方法总结

with 上下文管理器相关的魔法方法

```python3
#!/usr/bin/env python
# encoding: utf-8

class A(object):
    def __init__(self):
        print "初始化"

    def __enter__(self):
        print("with 进来了")
        return "我是as关键字后面的东西（f）"

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("with 出去了")


a = A()
with a as f:
    print("with 里面")
    print(f)
    
```

输出结果

```shell
with 进来了
with 里面
我是as关键字后面的东西
with 出去了
```





### `__repr__`

```python
python中，repr是一个特殊的方法，用于将类（class）的一个实例表示为字符串。repr即represent，简言之，就是对象的字符串表示。

对象通过__repr__方法转化为字符串后，该字符串又可以用来创建该对象。
在类中定义了__repr__方法后， 可以通过python内置的repr()函数来调用。
```

```python
# A simple Person class

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __repr__(self):
        rep = 'Person(' + self.name + ',' + str(self.age) + ')'
        return rep

# Let's make a Person object and print the results of repr()
person = Person("John", 20)
print(repr(person))  # 输出Person(John,20)
```

```
与__str__方法的区别
我们知道__str__方法也可以返回表示对象的字符串，对应的调用方法是python内置的str()函数。

区别在于，

python的官方文档说__repr__创建的字符串更加”official“（官方 ，更加正式的意思？）
__repr__返回的字符串可以用于创建对象。
__repr__的字符串所包含的信息更加丰富（对机友好）。
但__str__方法返回的字符创更加易读（对人友好）。
下面的例子说明两种方法返回的字符串有什么不一样：

>>> import datetime
>>> now = datetime.datetime.now()
>>> now.__str__()
'2020-12-27 22:28:00.324317' # 更易读（人）
>>> now.__repr__()
'datetime.datetime(2020, 12, 27, 22, 28, 0, 324317)' # 信息更丰富（机）
```



### `__slots__`

如果我们想要限制实例的属性怎么办？比如，只允许对Student实例添加`name`和`age`属性。

为了达到限制的目的，Python允许在定义class的时候，定义一个特殊的`__slots__`变量，来限制该class实例能添加的属性：

```
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

然后，我们试试：

```
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```




## 常见排序算法

```python
def bubble_sort(lst):
    for i in range(len(lst) - 1, 0, -1):
        exchange = False
        for j in range(i):
            if lst[j] > lst[j + 1]:
                exchange = True
                lst[j], lst[j + 1] = lst[j + 1], lst[j]
        if not exchange:
            break


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
bubble_sort(a_list)
print("bubble sort", a_list)


def select_sort(lst):
    for i in range(len(lst) - 1, 0, -1):
        max_index = 0
        for j in range(1, i + 1):
            if lst[j] > lst[max_index]:
                max_index = j
        lst[i], lst[max_index] = lst[max_index], lst[i]

# 另外想的解法
# def select_sort(lst):
#     for i in range(len(lst) - 1, 0, -1):
#         max_index = i
#         for j in range(i):
#             if lst[j] > lst[max_index]:
#                 max_index = j
#         lst[i], lst[max_index] = lst[max_index], lst[i]

a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
select_sort(a_list)
print("select sort", a_list)


def insert_sort(lst):
    for i in range(1, len(lst)):
        for j in range(i, 0, -1):
            if lst[j - 1] > lst[j]:
                lst[j - 1], lst[j] = lst[j], lst[j - 1]
            else:
                break

# 另外想的解法
# def insert_sort(lst):
#     for i in range(len(lst)-1):
#         for j in range(i + 1, len(lst)):
#             while j > 0 and lst[j] < lst[j - 1]:
#                 lst[j], lst[j-1] = lst[j-1], lst[j]
#                 j -= 1

a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
insert_sort(a_list)
print("insert sort", a_list)


def shell_sort(lst):
    n = len(lst)
    gap = n // 2
    while gap > 0:
        for i in range(gap, n):
            j = i
            while j >= gap and lst[j - gap] > lst[j]:
                lst[j - gap], lst[j] = lst[j], lst[j - gap]
                j -= gap
        gap = gap // 2


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
shell_sort(a_list)
print("shell sort", a_list)


def quick_sort(lst, start, end):
    if start >= end:
        return

    mid = lst[start]
    low = start
    high = end

    while low < high:
        while low < high and lst[high] >= mid:
            high -= 1
        lst[low] = lst[high]

        while low < high and lst[low] <= mid:
            low += 1
        lst[high] = lst[low]

    lst[low] = mid
    quick_sort(lst, start, low - 1)
    quick_sort(lst, low + 1, end)


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
quick_sort(a_list, 0, len(a_list) - 1)
print("quick sort", a_list)


def merge_sort(lst):
    if len(lst) <= 1:
        return lst
    mid = len(lst) // 2
    left = merge_sort(lst[:mid])
    right = merge_sort(lst[mid:])

    merged = []
    while left and right:
        if left[0] < right[0]:
            merged.append(left.pop(0))
        else:
            merged.append(right.pop(0))
    merged.extend(left if left else right)
    return merged


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
print("merge sort", merge_sort(a_list))

```



## 多线程

https://www.runoob.com/python/python-multithreading.html

### 用类的方式实现多线程

pass...



### 用函数的方式实现多线程

pass...



## 字符问题

报错： UnicodeDecodeError: 'utf8' codec can't decode byte 0x82 in position 35

然后google搜索 `json dumps error ignored`

```python
json.dumps(packet, default=lambda o: '<not serializable>')
```

https://stackoverflow.com/questions/51674222/how-to-make-json-dumps-in-python-ignore-a-non-serializable-field





## 死锁问题

工作中遇到： python常驻进程CPU占用100%-->无法调试（因为不知道是哪里报错）--> 然后用strace命令发现是进程在不断获取锁卡死了， 然后修改了代码后， CPU的占用100%的问题就解决了。

另外举个例子

```python
import time
import threading

class MyThread(threading.Thread):
    def __init__(self, name, lock1, lock2):
        # type: (str, threading.Lock, threading.Lock) -> None
        super(MyThread, self).__init__()
        self.name = name
        self.lock1 = lock1
        self.lock2 = lock2

    def run(self):
        self.lock1.acquire()
        print "{} got lock: {}, and ready to get lock: {}".format(threading.current_thread().name, self.lock1, self.lock2)
        time.sleep(1)
        self.lock2.acquire()
        print "{} got lock: {}".format(threading.current_thread().name, self.lock2)
        self.lock1.release()
        self.lock2.release()


def main():
    lock1 = threading.Lock()
    lock2 = threading.Lock()
    print "there are {} and {}".format(lock1, lock2)
    MyThread("T1", lock1, lock2).start()
    MyThread("T2", lock2, lock1).start()


if __name__ == '__main__':
    main()
```

然后程序打印

```
T1 got lock1 and ready to get lock2
T2 got lock2 and ready to get lock1
```

用strace -p pid命令

![image-20220312113908818](Python基础.assets/image-20220312113908818.png)

不难发现是futex锁的问题



> pystack工具

```shell
pip install pystack-debugger
```

然后pystack 【pid】

![image-20220312115738038](Python基础.assets/image-20220312115738038.png)





### 检测安全性

RestrictedPython





### Python2的类型提示

python3.5之后才内置了typing包。python2需要自己安装

```shell
pip install typing
```



怎么指定一个类的类型提示？ How to Specify a Class Rather Than an Instance Thereof

https://adamj.eu/tech/2021/05/16/python-type-hints-return-class-not-instance/

https://mypy.readthedocs.io/en/stable/cheat_sheet.html

```python
def __init__(self, lock1, lock2):
    # type: (threading.Lock, threading.Lock) -> None   # 后来发现不import typing包就可以不用pip install typing
    self.lock1 = lock1
    self.lock2 = lock2
```




# pip



## pip 源

https://zhuanlan.zhihu.com/p/109939711	

```shell
# 直接运行这个命令就能修改源，本质就是自动创建/root/.config/pip/pip.conf(linux下),C:\Users\User\pip\pip.ini(win下)
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
```






## pip install 问题

一开始安装pip install fastavro==0.24.0报这个错

```shell
pkg_resources.VersionConflict: (setuptools 0.9.8 (/usr/lib/python2.7/site-packages), Requirement.parse('setuptools>=18.0'))

Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-0v3VJX/pip/
```

然后升级setuptools ：pip install --upgrade setuptools， 发现还是报错，错误如下

```shell
$ /etc/yum.repos.d # pip install --upgrade setuptools
Collecting setuptools
  Downloading http://mirrors.sangfor.org/pypi/packages/ea/a3/3d3cbbb7150f90c4cf554048e1dceb7c6ab330e4b9138a40e130a4cc79e1/setuptools-62.1.0.tar.gz (2.5MB)
    100% |████████████████████████████████| 2.5MB 36.4MB/s
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "setuptools/__init__.py", line 8, in <module>
        import _distutils_hack.override  # noqa: F401
      File "_distutils_hack/override.py", line 1, in <module>
        __import__('_distutils_hack').do_override()
      File "_distutils_hack/__init__.py", line 72, in do_override
        ensure_local_distutils()
      File "_distutils_hack/__init__.py", line 55, in ensure_local_distutils
        importlib.import_module('distutils')
      File "/usr/lib64/python2.7/importlib/__init__.py", line 37, in import_module
        __import__(name)
    AttributeError: find_module
```

然后   pip install --upgrade pip 还是报错！

```shell

$ /home/hwj/data_convert/conf # python -m pip install --upgrade pip
Collecting pip
  Downloading http://mirrors.sangfor.org/pypi/packages/33/c9/e2164122d365d8f823213a53970fa3005eb16218edcfc56ca24cb6deba2b/pip-22.0.4.tar.gz (2.1MB)
    100% |████████████████████████████████| 2.1MB 33.0MB/s
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-dIISr4/pip/setup.py", line 7
        def read(rel_path: str) -> str:
                         ^
    SyntaxError: invalid syntax

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-dIISr4/pip/
```

最后，不管了pip 太旧了，想办法升级上去， 下载whl包吧！

```shell
pip install pip-20.3.4-py2.py3-none-any.whl  # 更新pip后终于能安装fastavro了！
```



错误： fatal error: Python.h: No such file or directory

https://stackoverflow.com/questions/21530577/fatal-error-python-h-no-such-file-or-directory



# 内存分析







# 类型定义 提示



# 工具类

## 时间

### 计算时间差（天，秒）

```python
import datetime
d1 = datetime.datetime(2018,10,31)   # 第一个日期
d2 = datetime.datetime(2019,2,2)   # 第二个日期
interval = d2 - d1                   # 两日期差距
interval.days                        # 具体的天数                     

d1 = datetime.datetime(2018,10,31)   # 第一个日期
d2 = datetime.datetime.now()   # 第二个日期
interval = d2 - d1                   # 两日期差距
interval.seconds  # 具体秒数， 没有具体分钟数，秒数/60即可
interval.total_seconds() # 总描述
```

### 获取当前日期时间字符串

```python
import time
time.strftime("%Y%m%d-%H%M%S", time.localtime()) # '20220609-170220'
time.strftime("%Y%m%d-%H%M%S", time.localtime(time.time())) # '20220609-170220'
```





## 文件读取

## 读json,yaml

```python
import yaml
import json
# 获取当前脚本所以在的目录
dir_path = os.path.dirname(os.path.abspath(__file__))
conf_file = os.path.join(dir_path, "conf", "conf.yaml")
desc_file = os.path.join(dir_path, "desc.json")
with open(conf_file, 'r') as f:
    conf = yaml.load(f.read())

with open(desc_file, 'r') as f:
    desc = json.load(f)
```

## glob批量获取文件

```python
import os
import glob

data_dir_path = "/data/log_data/*/*/_progress"
for data_path in glob.glob(data_dir_path):
    print "remove {}".format(data_path)
    os.remove(data_path)
```



## 执行shell命令



os.system

commands

os.popen

subprocess



## 调试问题

traceback

```python
import traceback
traceback.format_exc()
```





## 日志

...待补充



base64

```python
import base64
base64.b64decode(b"YTEyMw==") # a123
base64.b64encode(b"a123") # YTEyMw==
```



# pycharm 远程调试

https://blog.csdn.net/weixin_35848967/article/details/122242904



