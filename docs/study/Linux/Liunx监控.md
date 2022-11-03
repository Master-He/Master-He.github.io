# Linux中CPU、内存、Swap使用情况查看

参考 https://blog.csdn.net/csdn_shoelace/article/details/118916436



## top

```
M 根据驻留内存大小进行排序

P 根据CPU使用百分比大小进行排序
T 根据时间/累计时间进行排序
```



孤儿进程？

僵尸进程？



怎么跟踪进程？

strace命令



服务监控，心跳监控，超时机制，服务重启？



???

iotop

```shell
iotop -oP  
# -o： Only show processes or threads actually doing I/O
# -P:  Only show processes. Normally iotop shows all threads.
```



iftop



# free

## 概念

free -mh

man free 可以查看命令介绍

```shell
DESCRIPTION
       free  displays  the  total  amount  of  free  and used physical and swap memory in the system, as well as the buffers and caches used by the kernel. The information is gathered by parsing
       /proc/meminfo. The displayed columns are:

       total  Total installed memory (MemTotal and SwapTotal in /proc/meminfo)

       used   Used memory (calculated as total - free - buffers - cache)

       free   Unused memory (MemFree and SwapFree in /proc/meminfo)

       shared Memory used (mostly) by tmpfs (Shmem in /proc/meminfo, available on kernels 2.6.32, displayed as zero if not available)

       buffers
              Memory used by kernel buffers (Buffers in /proc/meminfo)

       cache  Memory used by the page cache and slabs (Cached and SReclaimable in /proc/meminfo)

       buff/cache
              Sum of buffers and cache

       available
              Estimation of how much memory is available for starting new applications, without swapping. Unlike the data provided by the cache or free fields, this field takes into account page
              cache  and  also  that  not  all reclaimable memory slabs will be reclaimed due to items being in use (MemAvailable in /proc/meminfo, available on kernels 3.14, emulated on kernels
              2.6.27+, otherwise the same as free)

```



如何释放buffer/cache 参考文章  https://focusss.github.io/2019/02/10/Linux%E4%B8%ADbuff-cache%E5%8D%A0%E7%94%A8%E8%BF%87%E9%AB%98%E8%A7%A3%E5%86%B3%E6%89%8B%E6%AE%B5/

buff和cache的概念

> buffer(Buffer Cache) 是一种I/O缓存，用于内存和硬盘的缓冲，是IO设备的读写缓冲区，根据磁盘的读写设计的，把分散的写操作集中进行，减少磁盘碎片和硬盘的反复寻道，从而提供系统性能。

> cache(Page Cache) 是一种高速缓存，用于CPU和内存之间的缓冲，是文件系统的Cache.
> 把读取过来的数据保存起来， 重启读取时若命中（找到需要的数据）就不要去读硬盘了，若没有命中就读硬盘，其中的数据会根据读取频率进行组织，把最频繁读取的内容放在最容易找到的位置，把不再读的内容不断往后排，直至从中删除

它们都是占用内存，两者都是RAM中的数据。简单来说，buff是即将要被写入磁盘的，而cache是被从磁盘中读取出来的。

目前进程实际正在被使用的内存的计算方式为used-buff/cache， 通过释放buff/cache内存后，我们还可以使用的内存量是free+buff/cache。 通常我们在频繁存取文件后，会导致buff/cache的占用量增高。



## 处理方式

### 手动清楚

#### 手动清除

    执行以下命令即可

```
[root@izbp17wg1wphb6f95b76obz ~]# sync
[root@izbp17wg1wphb6f95b76obz ~]# echo 1 > /proc/sys/vm/drop_caches
[root@izbp17wg1wphb6f95b76obz ~]# echo 2 > /proc/sys/vm/drop_caches
[root@izbp17wg1wphb6f95b76obz ~]# echo 3 > /proc/sys/vm/drop_caches
```

- sync：将所有未写的系统缓冲区写到磁盘中，包含已修改的i-node、已延迟的块I/O和读写映射文件
- echo 1 > /proc/sys/vm/drop_caches：清除page cache
- echo 2 > /proc/sys/vm/drop_caches：清除回收slab分配器中的对象（包括目录项缓存和inode缓存）。slab分配器是内核中管理内存的一种机制，其中很多缓存数据实现都是用的pagecache。
- echo 3 > /proc/sys/vm/drop_caches：清除pagecache和slab分配器中的缓存对象。
  /proc/sys/vm/drop_caches的值,默认为0

自动清除

1、创建脚本cleanCache.sh

```shell
#!/bin/bash
#每两小时清除一次缓存
echo "开始清除缓存"
sync;sync;sync #写入硬盘，防止数据丢失
sleep 10 #延迟10秒
echo 1 > /proc/sys/vm/drop_caches
echo 2 > /proc/sys/vm/drop_caches
echo 3 > /proc/sys/vm/drop_caches
echo "清除缓存完成"
```

2、创建定时任务

```shell
crontab -e #弹出配置文件
```

3、添加定时任务执行频率

```shell
#分　 时　 日　 月　 周　 命令
0 */2 * * * ./cleanCache.sh
```

4、设置crond启动以及开机自启

```shell
systemctl start crond.service
systemctl enable crond.service
```

5、查看定时任务是否被执行

```shell
cat /var/log/cron | grep cleanCache
```

