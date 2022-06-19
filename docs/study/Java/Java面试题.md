# Java基础

为什么写equals()方法后还要写hashCode()



说说wait()和sleep()的异同点？

相同点：

	1. 他们都会使当前线程暂停运行，把运行机会交给其他线程
 	2. 线程调用wait()和sleep()后，在等待期间被中断都会抛出InterruptedException

不同点：

	1. wait()是Object类的方法，而sleep()是线程Thread类的方法
 	2. 对锁的持有不同，wait()会释放锁，而sleep()并不会释放锁
 	3. 唤醒方法不同，wait()依靠notify，notifyAll，或者指定时间来唤醒， 而sleep()到达指定时间被唤醒
 	4. 调用wait()需要先获取对象的锁，而Thead.sleep()不用



# JUC



# JVM



# Netty

