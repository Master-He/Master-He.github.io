Alibaba Arthas

```shell

```





# 死锁排查

```java
public class ClassLoadDeadLockTest {
 
    public static class A {
        static {
            System.out.println("Init class A...");
 
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
 
            new B();
        }
 
        static void print() {
            System.out.println("I am A.");
        }
    }
 
    public static class B {
        static {
            System.out.println("Init class B...");
            new A();
        }
 
        static void print() {
            System.out.println("I am B.");
        }
    }
 
    public static void main(String... args) {
        new Thread(A::print).start();
        new Thread(B::print).start();
    }
}
```

查看 jstack [pid] 

```shell
"Thread-1" #10 prio=5 os_prio=0 tid=0x00007fce2814b800 nid=0xfe73 in Object.wait() [0x00007fce110f6000]
   java.lang.Thread.State: RUNNABLE
	at ClassLoadDeadLockTest$B.<clinit>(ClassLoadDeadLockTest.java:24)
	at ClassLoadDeadLockTest$$Lambda$2/1044036744.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"Thread-0" #9 prio=5 os_prio=0 tid=0x00007fce28149800 nid=0xfe72 in Object.wait() [0x00007fce111f7000]
   java.lang.Thread.State: RUNNABLE
	at ClassLoadDeadLockTest$A.<clinit>(ClassLoadDeadLockTest.java:13)
	at ClassLoadDeadLockTest$$Lambda$1/1418481495.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
```

注意，这里两个线程是RUNNABLE的。。jstack看不出来是互锁。。





# 基准测试

## JMH

JMH（Java Microbenchmark Harness，Java微基准测试框架）是专门用于进行代码的微基准测试的一套工具API。何谓 Micro Benchmark 呢？ 简单地说就是在 method 层面上的 benchmark，精度可以精确到微秒级。

官网：http://openjdk.java.net/projects/code-tools/jmh/

github demo: https://github.com/openjdk/jmh/tree/master/jmh-samples/src/main/java/org/openjdk/jmh/samples

Java的基准测试需要注意的几个点：

- 测试前需要预热
- 防止无用代码进入测试方法中
- 并发测试
- 测试结果呈现

比较典型的使用场景：

- 当你已经找出了热点函数，而需要对热点函数进行进一步的优化时，就可以使用 JMH 对优化的效果进行定量的分析
- 想定量地知道某个函数需要执行多长时间，以及执行时间和输入 n 的相关性
- 一个函数有两种不同实现（例如JSON序列化/反序列化有Jackson和Gson实现），不知道哪种实现性能更好

Maven依赖

```xml
 <!-- JMH-->
 <dependency>
     <groupId>org.openjdk.jmh</groupId>
     <artifactId>jmh-core</artifactId>
     <version>${jmh.version}</version>
 </dependency>
 <dependency>
     <groupId>org.openjdk.jmh</groupId>
     <artifactId>jmh-generator-annprocess</artifactId>
     <version>${jmh.version}</version>
     <scope>provided</scope>
 </dependency>
```

### 注解介绍

#### @BencharkMode
基准测试类型。这里选择的是Throughput也就是吞吐量。根据源码点进去，每种类型后面都有对应的解释，比较好理解，吞吐量会得到单位时间内可以进行的操作数。

- Throughput: 整体吞吐量，例如“1秒内可以执行多少次调用”。
- AverageTime: 调用的平均时间，例如“每次调用平均耗时xxx毫秒”。
- SampleTime: 随机取样，最后输出取样结果的分布，例如“99%的调用在xxx毫秒以内，99.99%的调用在xxx毫秒以内”
- SingleShotTime: 以上模式都是默认一次 iteration 是 1s，唯有 SingleShotTime 是只运行一次。往往同时把 warmup 次数设为0，用于测试冷启动时的性能。
- All(“all”, “All benchmark modes”)

#### @Warmup
上面我们提到了，进行基准测试前需要进行预热。一般我们前几次进行程序测试的时候都会比较慢， 所以要让程序进行几轮预热，保证测试的准确性。其中的参数iterations也就非常好理解了，就是预热轮数。

为什么需要预热？因为 JVM 的 JIT 机制的存在，如果某个函数被调用多次之后，JVM 会尝试将其编译成为机器码从而提高执行速度。所以为了让 benchmark 的结果更加接近真实情况就需要进行预热。

参数如下所示：

- iterations：预热的次数
- time：每次预热的时间
- timeUnit：时间的单位，默认秒
- batchSize：批处理大小，每次操作调用几次方法

#### @Measurement
度量，其实就是一些基本的测试参数。

- iterations 进行测试的轮次
- time 每轮进行的时长
- timeUnit 时长单位

都是一些基本的参数，可以根据具体情况调整。一般比较重的东西可以进行大量的测试，放到服务器上运行。

#### @Threads
每个进程中的测试线程，这个非常好理解，根据具体情况选择，一般为cpu乘以2。



#### @Fork
进行 fork 的次数。如果 fork 数是2的话，则 JMH 会 fork 出两个进程来进行测试。



#### @OutputTimeUnit
这个比较简单了，基准测试结果的时间类型。一般选择秒、毫秒、微秒。



#### @Benchmark
方法级注解，表示该方法是需要进行 benchmark 的对象，用法和 JUnit 的 @Test 类似。



#### @Param
属性级注解，@Param 可以用来指定某项参数的多种情况。特别适合用来测试一个函数在不同的参数输入的情况下的性能。



#### @Setup
方法级注解，这个注解的作用就是我们需要在测试之前进行一些准备工作，比如对一些数据的初始化之类的。



#### @TearDown
方法级注解，这个注解的作用就是我们需要在测试之后进行一些结束工作，比如关闭线程池，数据库连接等的，主要用于资源的回收等。



#### @State
当使用@Setup参数的时候，必须在类上加这个参数，不然会提示无法运行。

State 用于声明某个类是一个“状态”，然后接受一个 Scope 参数用来表示该状态的共享范围。 因为很多 benchmark 会需要一些表示状态的类，JMH 允许你把这些类以依赖注入的方式注入到 benchmark 函数里。Scope 主要分为三种。

- Thread:默认的state, 该状态为每个线程独享(每个测试线程分配一个实例)。
- Group: 该状态为同一个组里面所有线程共享(同一个线程在同一个 group 里共享实例)。
- Benchmark: 该状态在所有线程间共享(所有测试线程共享一个实例，测试有状态实例在多线程共享下的性能)。





# OOM问题

```shell
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

I know what an `OutOfMemoryError` is, but what does GC overhead limit mean? How can I solve this?

```shell
This message means that for some reason the garbage collector is taking an excessive amount of time (by default 98% of all CPU time of the process) and recovers very little memory in each run (by default 2% of the heap).

This effectively means that your program stops doing any progress and is busy running only the garbage collection at all time.

To prevent your application from soaking up CPU time without getting anything done, the JVM throws this Error so that you have a chance of diagnosing the problem.

The rare cases where I've seen this happen is where some code was creating tons of temporary objects and tons of weakly-referenced objects in an already very memory-constrained environment.

Check out the Java GC tuning guide, which is available for various Java versions and contains sections about this specific problem:

Java 11 tuning guide has dedicated sections on excessive GC for different garbage collectors:
for the Parallel Collector
for the Concurrent Mark Sweep (CMS) Collector
there is no mention of this specific error condition for the Garbage First (G1) collector.
Java 8 tuning guide and its Excessive GC section
Java 6 tuning guide and its Excessive GC section.
```

参考

https://stackoverflow.com/questions/1393486/error-java-lang-outofmemoryerror-gc-overhead-limit-exceeded







flink 提交任务oom问题

```shell
进到alphasecurity  pod  /opt/flink/conf/flink-conf.yaml
env.java.opts: -Xms4096m -Xmx4096m
```



