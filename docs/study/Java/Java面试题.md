# Java基础

## 1、为什么写equals()方法后还要写hashCode()



说说wait()和sleep()的异同点？

相同点：

	1. 他们都会使当前线程暂停运行，把运行机会交给其他线程
	2. 线程调用wait()和sleep()后，在等待期间被中断都会抛出InterruptedException

不同点：

	1. wait()是Object类的方法，而sleep()是线程Thread类的方法
	2. 对锁的持有不同，wait()会释放锁，而sleep()并不会释放锁
	3. 唤醒方法不同，wait()依靠notify，notifyAll，或者指定时间来唤醒， 而sleep()到达指定时间被唤醒
	4. 调用wait()需要先获取对象的锁，而Thead.sleep()不用



## 2、HashMap和HashTable的区别

```
1. 都实现了map接口， 但HashTable继承Dictionary类， HashMap继承AbstractMap类
2. HashMap不是线程安全的，而HashTable是线程安全的
3. 基于第二点，HashMap的key可以为null, HashTable的key不能为null（原因是get()方法调用的时候无法判断是键为空还是存的值为空）
4. 遍历方法不同，HashTable是通过Enumeration遍历， HashMap是通过iterator遍历
5. 初始大小和扩容方式不同， HashTable初始大小为11. 扩容方式是2n+1, HashMap初始大小是16， 扩容方式是2的指数形式
```





# JUC



# JVM



# Netty

