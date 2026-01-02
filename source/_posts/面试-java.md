title: java面试准备
author: ming
date: 2020-08-17 10:56:00
tags:
---

# java基础
### 1.继承, 封装, 多态

### 2.集合框架list，map，set
* ArrayList 底层是数组，默认10个元素，每次增加原来size的一半

* LinkedList 底层是双向列表，可以当作列表用(add/remove), 可以当做队列用(offer/poll)，也可以当作栈用(push/pop)

* HashSet 底层用HashMap实现的

* TreeSet 使用TreeMap实现

* HashMap 哈希表的实现

* Hashtable 相当于HashMap加了synchronized

* LinkedHashMap 继承了HashMap，每个Entry加了前驱和后继指针，用于保持插入顺序

* TreeMap 底层是二叉搜索树 

* Vector 与ArrayList类似，是线程安全的，一般不用

* Stack 继承了Vector，栈的实现


### 3.反射
Class.forName("xxx");

### 4.泛型
泛型是一种类模版，就比如ArrayList，如果不用泛型，数组只能定义为Object，如果这样，编译的时候，就没办法做类型检查。为了兼容老版本的java，使用了类型擦除，也就是字节码中，不保存元素的类型信息。

[参考](https://juejin.im/post/6844903650788114439#heading-5)

### 6.注解
* 分三种运行期：源码，class，运行时
    * 源码级别：会被编译器忽略
    * class级别：会被JVM忽略
    * 运行时级别: 运行期(JVM)也保留
https://blog.csdn.net/javazejian/article/details/71860633

### 7.其他


# java并发
## 1. 并发编程的挑战
* 并发是什么
    
    并发就是同时执行两段指令。

* 为什么要用并发

    因为cpu的速度远远超过存储器和其他硬件的速度，如果不用并发，大部分时间，cpu是空闲的，为了充分利用cpu，所以并发。

* 并发面对的问题

    1. 开了太多线程，线程切换会消耗一些资源，如果耗费在线程切换上资源太多，性能下降太快。 
    2. 死锁，主要的死锁场景包括: 

        * 一个线程获取多个锁 
        * 一个线程，在锁内，占用了多个资源
        

## 2. java并发的实现原理
1. volatile
* 保证了原子性和可见性。
* 加了volatile的变量，每次都会把缓存写会主内存，其他内核会嗅探到缓存改变，其他线程再读这个变量时，会从主内存读取

2. synchronized
* 保证了原子性，可见性，顺序性
* 对象方法是对当前对象加锁，类方法是对class对象加锁。代码块是对配置的对象加锁
* markword中保存锁信息，最后两位保存了锁标记
* 锁升级，无锁(对象头中保存hashcode，分代年龄，锁标记)，偏向锁(偏向线程id，偏向时间戳，分代年龄，锁标记)，轻量级锁(指向栈中锁记录的指针)，重量级锁(指向重量级锁的指针)

3. 原子操作
    1. 锁, synchronized可以保持原子性
    2. CAS, 对于单个变量可以保持原子性。存在的问题:ABA问题，循环时间长和只能处理一个变量

4. DCL单例

    ```java
    public class Singleton {
        private static volatile Singleton instance; 

        private Singleton() {}

        public static Singleton getInstance() {
            if (instance == null) {
                synchronized(Singleton.class) {
                    if (instance == null) {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
    ```

5. java并发解决的三个性质
    1. 原子性, 原子操作
    2. 可见性, 工作内存，主内存
    3. 顺序性, 指令重排序, 内存屏障


6. java线程状态机
  * New 
  * Runnable
  * Terminated
  * Blocked
  * Wait
  * Time wait
  * 怎样查看线程状态？先jps找到对应的进程id，再jstack pid
  * juc包里的Lock，阻塞时，是wait状态，因为Lock使用的LockSupport.park

7. 生产者消费者
  ```java
  public class PC {

    public static void main(final String[] args) {
        final PC pc = new PC();

        for (int i = 0; i < 2; i++) {
            new Thread(pc.new Productor(), "product_thread_" + i).start();
        }

        for (int i = 0; i < 20; i++) {
            new Thread(pc.new Customer(), "custom_thread_" + i).start();
        }
    }

    private int count = 0;
    private final Object lock = new Object();

    private class Productor implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized (lock) {
                    while (count == 10) {// 这个要用while, 防止下次被同类唤醒后，再次累加
                        try {
                            lock.wait();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                    count++;
                    lock.notifyAll();// 这里要用notifyAll, 防止唤醒的线程是同类，会导致假死
                }
            }
        }
    }

    private class Customer implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized(lock) {
                    while (count == 0) {
                        try {
                            lock.wait();
                        } catch (final Exception e) {
                            e.printStackTrace();
                        }
                    }
                    count--;
                    lock.notifyAll();
                }
            }
        }
    }
    
}
  ```

8. join() 

```java
public class TestJoin {
    public static void main(String[] args) {
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("nani");
            }
        });
        t.start();

        try {
            t.join();// join内部循环判断t线程是否isAlive, 如果还活着则调用wait
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("main thread");
    }
}
```

9. Lock
```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class TestLock {
    public static void main(String[] args) {
        TestLock ins = new TestLock();
        for (int i=0; i<5; i++) {
            new Thread(ins.new Productor(), "productor-" + i).start();
        }    
        for (int i=0; i<5; i++) {
            new Thread(ins.new Customer(), "customer-" + i).start();
        }    
    }

    private boolean on = true;
    private int count = 0;
    private Lock lock = new ReentrantLock();
    private Condition fullCond = lock.newCondition();
    private Condition emptyCond = lock.newCondition();

    private class Productor implements Runnable {
        @Override
        public void run() {
            while (on) {
                try {
                    lock.lock();
                    while (count == 10) {
                        try {
                            fullCond.await();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                    count++;
                    emptyCond.signalAll();
                } finally {
                    lock.unlock();
                }
            }
        }
    }

    private class Customer implements Runnable {
        @Override
        public void run() {
            while (on) {
                try {
                    lock.lock();
                    while (count == 0) {
                        try {
                            emptyCond.await();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                    count--;
                    fullCond.signalAll();
                } finally {
                    lock.unlock();
                }
            }
        }
    }
}
```

10. 读写锁(读读共享，读写互斥, 写写互斥)
```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class TestRWLock {
    public static void main(String[] args) {
        TestRWLock test = new TestRWLock();

        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run() {
                test.read();
            }
        }, "t1");
        t1.start();

        Thread t2 = new Thread(new Runnable(){
            @Override
            public void run() {
                test.write();
            }
        }, "t2");
        t2.start();
    }

    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    private void read() {
        try {
            lock.readLock().lock();
            System.out.println("read " + System.currentTimeMillis());
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } finally {
            lock.readLock().unlock();
        }
    }

    private void write() {
        try {
            lock.writeLock().lock();
            System.out.println("write " + System.currentTimeMillis());
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

11. 队列同步器([AQS](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html))
    * 独占状态的获取和释放: 同步器维护了一个双向链表，头节点是获取到锁的线程，如果有新线程请求锁失败，则加入到队尾，并且保持自旋。当头节点释放锁后，会唤醒后继节点，尝试获取锁。
    * 共享状态的获取和释放


12. ConcurrentHashMap

13. ConcurrentLinkedQueue

14. 阻塞队列

15. 原子操作类

16. 线程池
    * 工作原理：当添加新任务时，先判断核心线程池是否满了，如果是再判断阻塞队列是否满了，如果是再判断线程池中的线程是否全再工作，如果是则抛出拒绝任务的异常

# JVM
* java内存布局：
    1. 方法区
    2. 堆
    3. 程序计数器
    4. 虚拟机栈
    5. 本地方法栈
* GC的几个问题：
    1. 怎样找到垃圾
        通过搜索GC root，如果没有引用被GC根指向，则认为是垃圾。
        
        引用的类型分四种：
        * 强引用, 永远不回收
        * 软引用, 主要用与缓存，如果gc一次之后，还是内存不足，才会回收。
        * 弱引用, 只能生存到下一次gc之前
        * 虚引用, 相当于没有引用，只是为了回收前，收到一个通知

        GC root是什么：
        * 虚拟机栈空间中，局部变量表的引用
        * Navite栈中，变量的引用
        * 方法区中，静态变量的引用
        * 方法区中，常量的引用

    2. 怎样回收
        分代回收。

        回收算法：
        * 标记清除
        * 标记整理
        * 拷贝

        分代：
        * 新生代, 8:1:1，使用拷贝算法，每次把需要保留的对象拷贝到幸存区, minor gc 频繁
        * 老年代, 使用标记整理算法, full gc 不频繁
        * 永久代, 主要是对不用的常量和对类的卸载，很少做回收
        
    3. 何时回收
    
    4. 怎样分配对象内存
        1. 优先在eden区分配，如果eden内存不足，则gc一次
        2. 如果age超过阈值，则移动到老年代
        3. 如果是大对象，直接分配到老年代
        4. 动态分配，如果幸存区中的对象，相同age的总和超过一半，就把大于或等于这个年龄的对象移动到老年代
    
    5. 垃圾回收器：G1，CMS。。。

* 类加载
    1. 加载的是什么
    2. 怎样加载
    3. 加载之后，怎样运行，栈帧是怎样的
