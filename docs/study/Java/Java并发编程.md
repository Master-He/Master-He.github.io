

下列哪些操作会使线程释放锁资源？

```java
wait和join  // join()底层就是调用wait()方法的，wait()释放锁资源，故join也释放锁资源
```



**1.sleep()方法**

在指定时间内让当前正在执行的线程暂停执行，但不会释放“锁标志”。不推荐使用。

sleep()使当前线程进入阻塞状态，在指定时间内不会执行。

**2.wait()方法**

在其他线程调用对象的notify或notifyAll方法前，导致当前线程等待。线程会释放掉它所占有的“锁标志”，从而使别的线程有机会抢占该锁。

当前线程必须拥有当前对象锁。如果当前线程不是此锁的拥有者，会抛出IllegalMonitorStateException异常。

唤醒当前对象锁的等待线程使用notify或notifyAll方法，也必须拥有相同的对象锁，否则也会抛出IllegalMonitorStateException异常。

waite()和notify()必须在synchronized函数或synchronized　block中进行调用。如果在non-synchronized函数或non-synchronized　block中进行调用，虽然能编译通过，但在运行时会发生IllegalMonitorStateException的异常。

**3.yield方法** 

暂停当前正在执行的线程对象。

yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。

yield()只能使同优先级或更高优先级的线程有执行的机会。 

**4.join方法**

等待该线程终止。

等待调用join方法的线程结束，再继续执行。如：t.join();//主要用于等待t线程运行结束，若无此句，main则会执行完毕，导致结果不可预测。



基本的Java多线程

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
import java.util.concurrent.TimeUnit;

public class ThreadHello{

    public static void main(String[] args) {
        // Runnable
        R r = new R();
        new Thread(r).start();
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        r.stop();

        // Callable
        FutureTask<String> target = new FutureTask<>(new C());
        new Thread(target).start();
        try {
            System.out.println(target.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class R implements Runnable {
    private boolean isStop = false;


    @Override
    public void run() {
        while (!isStop) {
            try {
                TimeUnit.SECONDS.sleep(1);
                System.out.println("hello");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void stop() {
        isStop = true;
    }
}

class C implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("callable hello");
        return "success";
    }
}
```

