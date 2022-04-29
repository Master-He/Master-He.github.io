Alibaba Arthas

```shell
wget https://arthas.aliyun.com/arthas-demo.jar;java -jar arthas-demo.jar
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

