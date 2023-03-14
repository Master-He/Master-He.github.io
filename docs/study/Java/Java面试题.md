参考：

https://pdai.tech/md/interview/x-interview.html

# Java基础

封装继承多态

封装是将对象的属性和方法封装在一起，使其构成一个不可分割的实体。

封装的优点

- 减少耦合，可独立开发测试修改
- 方便分析调节性能
- 方便复用

继承应该遵从里氏替换原则，子类对象能够完全替换掉父类对象

多态分为编译型多态和运行时多态：

- 编译时多态是指方法的重载
- 运行时多态是指对象引用所指的具体类型在运行期间才确定。List在运行期间才确定是ArrayList还是LinkedList，运行多态有三个条件
  - 继承
  - 重写（覆盖）父类方法
  - 向上转型

------

a = a+b 和 a+=b的区别在于，a+=b会将类型进行强转

-----

3\*0.1不等于 0.3的原因： 计算机的浮点数不能完全精确的表示小数，3\*0.1=0.3000000004

-----


为什么写equals()方法后还要写hashCode()

因为相同的对象要有相同的hashCode，这个是规范。

参考：https://zhuanlan.zhihu.com/p/50206657

------

final,finalize,finally的区别？

final是修饰符，修饰变量，方法，类，修饰变量时就是将变量变成常量，修饰方法就是将方法变得不可覆盖，修饰类就是让类变得不可继承

finalize()是个废弃的方法，对象没有引用时，垃圾回收器会调用这个方法，具体是什么时候调用，没有保障。

finally是个异常处理相关的关键字， 无论异常发不发生都执行finally块里面的代码

---

String StringBuffer StringBuilder的区别？

String是不可变类型，每次对String操作都会生成新的String对象

StringBuffer和StringBuilder都是可变序列，对他们的操作不会生成新的对象，StringBuffer是线程安全的，而StringBuilder不是线程安全的，效率更好

---

接口与抽象类的区别？

- 一个子类只能继承一个抽象类, 但能实现多个接口
- 抽象类可以有构造方法, 接口没有构造方法
- 抽象类可以有普通成员变量, 接口没有普通成员变量
- 抽象类和接口都可有静态成员变量, 抽象类中静态成员变量访问类型任意，接口只能public static final(默认)
- 抽象类可以没有抽象方法, 抽象类可以有普通方法；接口在JDK8之前都是抽象方法，在JDK8可以有default方法，在JDK9中允许有私有普通方法
- 抽象类可以有静态方法；接口在JDK8之前不能有静态方法，在JDK8中可以有静态方法，但只能被接口类直接调用（不能被实现类的对象调用）

---

this-super-在构造方法中的区别this() & super()在构造方法中的区别？

- 调用super()必须写在子类构造方法的第一行, 否则编译不通过
- super从子类调用父类构造, this在同一类中调用其他构造均需要放在第一行
- 尽管可以用this调用一个构造器, 却不能调用2个
- this()、super()都指的对象,不可以在static环境中使用
- 本质this指向本对象的指针。super是一个关键字

---

Java移位运算符？

java中有三种移位运算符

- `<<` :左移运算符,`x << 1`,相当于x乘以2(不溢出的情况下),低位补0
- `>>` :带符号右移,`x >> 1`,相当于x除以2,正数高位补0,负数高位补1
- `>>>` :无符号右移,忽略符号位,空位都以0补齐


---

 泛型

泛型方法，泛型接口，泛型类

泛型可以约束输入的方法参数类型，约束上限和下限

- 约束泛型类的上限class Info\<T extends Number\>
- 约束不了泛型类的下限
- 约束方法参数的上限  public static void fun(Info<? extends Number> temp)
- 约束方法参数的下限  public static void fun(Info<? super String> temp)

泛型擦除， 在编译的时候，泛型会进行擦除，约定的类型就变成了Object

---

注解

注解的作用： 对代码进行说明或者进行动态处理

---

异常

Throwable

- exception
  - runtime exception
    - null pointer exception
    - index outof bounds exception
  - 非运行时异常（编译异常）
    - IOEXCEPTION
    - SQL...
    - ClassNotFound
- error
  - IOerror
  - ThreadDeath

throw和throws区别

throw是抛出异常， throws是异常的声明

---

反射

反射就是运行时动态地获取类的属性和方法，包括私有方法

---







说说wait()和sleep()的异同点？

相同点：

	1. 他们都会使当前线程暂停运行，把运行机会交给其他线程
	2. 线程调用wait()和sleep()后，在等待期间被中断都会抛出InterruptedException

不同点：

	1. wait()是Object类的方法，而sleep()是线程Thread类的方法
	2. 对锁的持有不同，wait()会释放锁，而sleep()并不会释放锁
	3. 唤醒方法不同，wait()依靠notify，notifyAll，或者指定时间来唤醒， 而sleep()到达指定时间被唤醒
	4. 调用wait()需要先获取对象的锁，而Thead.sleep()不用





# Java集合

ArrayList的底层是数组

ensureCapacity方法，扩容的时候，看源码是int newCapacity = oldCapacity + (oldCapacity >> 1);， 大概都是之前的1.5倍

ArrayList的Fail-Fast机制？  是指在遍历数组是如果对数组的元素的数量进行了修改（增加或者删除元素）就会发生并发修改异常。

---



HashMap的底层是数组+链表+红黑树

HashMap初始容量是16， 负载系数是0.75

hashmap的扩容机制 --> 每次 \* 2

---



HashSet就是特殊的HashMap



---

HashMap和HashTable的区别

```
1. 都实现了map接口， 但HashTable继承Dictionary类， HashMap继承AbstractMap类
2. HashMap不是线程安全的，而HashTable是线程安全的
3. 基于第二点，HashMap的key可以为null, HashTable的key不能为null（原因是get()方法调用的时候无法判断是键为空还是存的值为空）
4. 遍历方法不同，HashTable是通过Enumeration遍历， HashMap是通过iterator遍历
5. 初始大小和扩容方式不同， HashTable初始大小为11. 扩容方式是2n+1, HashMap初始大小是16， 扩容方式是2的指数形式
```





# Java并发

java 内存模型

1. 因为CPU直接读内存速度比较慢，就增加了CPU缓存，又因为每个线程都有各自的CPU缓存，然后CPU缓存不会实时更新到内存，所以就有了可见性的问题
2. 因为高效使用CPU，均衡CPU/IO设备的速度， 操作系统增加了进程和线程以分时复用CPU，就有了原子性问题（简单的说就是为了分时复用CPU， 就有了原子性问题）
3. 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用，就有了有序性的问题

---

java内存模型规范了JVM什么时候禁用缓存，什么时候编译优化。

---

volatile 保证了可见性和有序性，同时禁止指令重排

---

synchronized 和 lock 保证了原子性, 可见性，有序性

---

happens-before

happens-before是一个规则

如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系
这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

为了提高指令执行效率，指令会进行重排序，happens-before就是保证执行结果不改变的规则

---

as-if-serial

as-if-serial语义是指不管怎么重排序，（单线程）程序的执行结果不能被改变。

为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果

---

重排序

编译器和处理器为了提高并行度，会对指令进行重排序

# JUC

# JVM



# Netty

