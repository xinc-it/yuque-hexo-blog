---
title: JUC
date: '2021/12/18 20:47:34'
updated: '2023-12-06 17:15:12'
categories: Java
tags:
  - JUC
  - Java高级
---
# 1. 并发基础
## 1. 多线程实现


### 1. 实现方法分类


1. 继承Thread类，重写run方法
2. 实现Runnable接口，实现run方法



### 2.两种方法对比


+ 实现runnable接口更好
1.  Java只支持单继承，继承Thread类导致程序拓展性不好
2. 解耦，将创建线程和线程任务调度分离了
+ 两种方法本质区别对比
1. 继承Thread类是通过重写Thread类的方法
2. 实现Runnable是在Thread类中调用Runnable实现类的run方法

![](/images/3c0e42c40e4b097835d08c08c39e4f99.png)



![](/images/51d40c78ac91a43082820aef157aaf6f.png)



**思考题：同时使用两种方法实现多线程**

```java
public class BothRunnableThread {

    public static void main(String[] args) {
        new Thread(() -> {
            //实现Runnable接口的方法
            System.out.println("我来自Runnable");
        }) {
            //重写Thread类的run方法
            @Override
            public void run() {
                System.out.println("我来自Thread");
            }
        }.start();
    }
}
```



结果



我来自Thread









### 3. 总结
**实现线程创建的方法:只有一种通过Thread类来创建线程**

**实现线程执行方法：**

1. **实现Runnable接口的run方法，并把接口实例传给Thread类在其Thread类中的run方法中调用**
2. **重写Thread类的run方法**

![](/images/3c0e42c40e4b097835d08c08c39e4f99.png) 











### 错误观点
1. 线程池也是创建线程的一种方式（其本质还是通过Thread类来创建线程）

![](/images/fd093faf716b8b12e0e8c206b84d6746.png)

2. Callable也是创建线程的一种方式（本质是调用其创建一个内部线程执行run方法在run方法内部调用callable的call方法）

![](/images/8e697545451fc408736ba8cb347b35e6.png)[FutureTask与Callable的本质](https://blog.csdn.net/xzongyuan/article/details/71378769)





### 面试问题


#### 1. 多少种线程实现方法
参考总结

#### 2. Runnable和Thread类那种方法实现多线程更好
Runnable好

1. 职责分离：<font style="color:rgb(0, 0, 0);"> Runnable 定义了执行内容，Thread类用于创建线程权责分明</font>
2. 提高性能：<font style="color:rgb(18, 18, 18);">每次执行一次任务，都需要新建一个独立的线程，如果还想执行这个任务，就必须再新建一个继承了 Thread 类的类，整个线程从开始创建到执行完毕被销毁，这一系列的操作比 run() 方法打印文字本身带来的开销要大得多，相当于捡了芝麻丢了西瓜，得不偿失。如果我们使用实现 Runnable 接口的方式，就可以把任务直接传入线程池，使用一些固定的线程来完成任务，不需要每次新建销毁线程，大大降低了性能开销。</font>
3. Java只支持单继承













## 2. 线程的启动


### 1. start和run方法比较
#### 1. start方法


##### 作用
1.  启动新线程
2. 新线程的准备工作



##### 源码解读
**<font style="color:#F5222D;">执行流程</font>**

1. **<font style="color:#F5222D;">判断线程状态</font>**
2. **<font style="color:#F5222D;">将线程加入线程组</font>**
3. **<font style="color:#F5222D;">调用start0方法</font>**

```java
 /**
     * Causes this thread to begin execution; the Java Virtual Machine
     * calls the <code>run</code> method of this thread.
     * <p>
     * The result is that two threads are running concurrently: the
     * current thread (which returns from the call to the
     * <code>start</code> method) and the other thread (which executes its
     * <code>run</code> method).
     * <p>
     * It is never legal to start a thread more than once.
     * In particular, a thread may not be restarted once it has completed
     * execution.
     *
     * @exception  IllegalThreadStateException  if the thread was already
     *               started.
     * @see        #run()
     * @see        #stop()
     */
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    private native void start0();
```







#### 2. run方法


##### 作用
1. 执行该线程任务





##### 源码解读
```java
   /**
     * If this thread was constructed using a separate
     * <code>Runnable</code> run object, then that
     * <code>Runnable</code> object's <code>run</code> method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of <code>Thread</code> should override this method.
     *
     * @see     #start()
     * @see     #stop()
     * @see     #Thread(ThreadGroup, Runnable, String)
     */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```



#### 3. 面试题


1. 两次调用start方法会出现什么情况？

参考start方法源码解读

2. 为什么不能直接调用run方法

start方法用于创建新的线程，如果直接调用run方法则是有main线程调用run方法



```java
public class StartAndRunMethod {

    public static void main(String[] args) {
        Runnable runnable = () -> {
            System.out.println(Thread.currentThread().getName());
        };
        runnable.run();
        new Thread(runnable).start();
    }
}

//结果 
main
Thread-0

```





## 3. 停止线程


**interupt停止线程原理：****<font style="color:#F5222D;">通知线程停止</font>****，但是****<font style="color:#F5222D;">不强制停止线程</font>****。由线程的run方法决定是否停止。**

### 1. 正确停止线程




方法1 

```java
public class RightWayStopThreadInProd implements Runnable {

    @Override
    public void run() {
        while (true) {
            if (Thread.currentThread().isInterrupted()) {
                System.out.println("线程中断程序结束");
                break;
            }
            System.out.println("go");
            throwInMethod();

        }
    }

    private void throwInMethod() {
        try {
            Thread.sleep(1000);
            //sleep()、wait()等会抛中断异常的方法在抛出异常之前会清除线程的中断标识
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadInProd());
        thread.start();
        Thread.sleep(3000);
        thread.interrupt();
    }
}
```



方法2

```java
public class RightWayStopThreadInProd2 implements Runnable {

    @Override
    public void run() {
        while (true) {
            System.out.println("go");
            try {
                throwInMethod();
            } catch (InterruptedException e) {
                e.printStackTrace();
                break;
            }
        }
    }

    private void throwInMethod() throws InterruptedException {

        Thread.sleep(1000);

    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadInProd2());
        thread.start();
        Thread.sleep(3000);
        thread.interrupt();
    }
}

```



### 2. 错误的停止方法
1. 调用stop方法
2. 用volatile设置boolean标记位

```java
public class WrongWayVolatile {

    public static void main(String[] args) throws InterruptedException {
        BlockingQueue blockingQueue = new ArrayBlockingQueue(30);
        Producer producer = new Producer(blockingQueue);
        Thread thread = new Thread(producer);
        Consumer consumer = new Consumer(blockingQueue);
        thread.start();
        while (true) {
            if (!consumer.needMoreNums()) {

                producer.canceled = true;

            }
            Object take = blockingQueue.take();
            System.out.println("消费者开始消费" + take);
            Thread.sleep(100);
        }
    }
}


class Producer implements Runnable {
    public volatile boolean canceled = false;


    private BlockingQueue blockingQueue;

    public Producer(BlockingQueue blockingQueue) {
        this.blockingQueue = blockingQueue;
    }

    @Override
    public void run() {
        int num = 0;
        try {
            while (num <= 10000 && !canceled) {//线程中断的地方
                if (num % 100 == 0) {
						
                    blockingQueue.put(num);//导致线程不能中断的地方是由于put一直处于阻塞状态
                    System.out.println("生产者生产完成" + num);
                    Thread.sleep(10);
                }
                num++;
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("生产结束");
    }
}


class Consumer {
    /**
     * 概率约为95%的随机消费
     *
     * @return
     */
    public boolean needMoreNums() {
        double random = Math.random();
        if (random > 0.96) {
            return false;
        }
        return true;
    }

    private BlockingQueue blockingQueue;


    public Consumer(BlockingQueue blockingQueue) {
        this.blockingQueue = blockingQueue;
    }
}
```





## 4. 线程的生命周期




### 1. 线程的6种状态


+ New  创建了线程但是没有执行start方法
+ Runnable  调用了start方法后变成Runnable状态
+ Blocked  线程进入**<font style="color:#F5222D;">Synchronized修饰的方法或者代码块</font>**但是锁被其他线程拿走
+ Waiting   wait() 、Thread.join() 、LockSupport.park() 只能等待唤醒信号唤醒
+ Timed Waiting wait(time) sleep(time) join(time) parkNanos(time) parkUntiil(time) 等待信号唤醒或超时
+ Terminated



![](/images/e9ce8a3aa1f523c21d93c13cba394768.png)

### 2. 阻塞状态
Blocked、Waiting、TimedWaiting





## 5. Thread类和Object 类线程方法详解


![](/images/e721e514dff2cb8707c4e4dbcc7f5059.png)



### 1. wait、notify、notifyAll方法详解


1. 阻塞阶段

执行wait方法进入阻塞阶段



唤醒方法

+ 其他线程调用notify方法刚好唤醒阻塞线程
+ 调用notifyAll唤醒所有阻塞线程
+ 过了超时时间，自动唤醒
+ 线程自身调用interupt方法



2. 唤醒阶段

通过调用notify或者notifyAll方法



3. 遇到中断



### 2. wait、notify、notifyAll特点和性质
+ 调用wait方法之前必须要拥有monitor
+ notify只能唤醒其中一个
+ 都属于object类



### 3. wait原理
![](/images/f34173e64150a9f4ff4667bd73a53a29.png)





1. 线程进入线程节点集
2. 线程节点集中的节点尝试获取锁
3. 调用wait方法释放锁进入等待集
4. 等待集中的线程等待其他线程调用notify、notifyAll方法将线程唤醒（线程由等待状态转为Blocked）
5. 重新唤醒的线程尝试重新获取锁
6. 释放锁并退出 





### 4. join方法
1. 作用

因为新的线程要加入我们，所以我们等他执行完再执行

2. 用法

主线程等待需要加入的线程执行完毕





## 6. 线程未被捕获异常 


### 1. 异常处理器解决方法
1. 在run方法中捕获异常并处理
2. 实现UncaughtExceptionHandler接口

![](/images/1b78b679ebe6399a2cf9eaaa95ec577c.png)







源码解析



```java
 public void uncaughtException(Thread t, Throwable e) {
     //判断线程是否存在父线程的异常处理器
        if (parent != null) {
            parent.uncaughtException(t, e);
        } else {
            //获取线程默认异常处理器
            Thread.UncaughtExceptionHandler ueh =
                Thread.getDefaultUncaughtExceptionHandler();
            if (ueh != null) {
               //调用线程默认异常处理器
                ueh.uncaughtException(t, e);
            } else if (!(e instanceof ThreadDeath)) {
                //没有直接打印异常
                System.err.print("Exception in thread \""
                                 + t.getName() + "\" ");
                e.printStackTrace(System.err);
            }
        }
    }
```



### 2. 自定义异常处理器


1. 实现Thread.UncaughtExceptionHandler的uncaught方法

```java
public class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler{
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        Logger logger=Logger.getAnonymousLogger();
        logger.log(Level.WARNING,"自定义异常处理器"+t.getName(),e);
    }
}
```



2. 设置线程异常处理器

```java
public class UseOwnUncaughtExceptionHandler implements Runnable {

    @Override
    public void run() {
        throw new RuntimeException();
    }


    public static void main(String[] args) {
        Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
        Thread thread = new Thread(new UseOwnUncaughtExceptionHandler());
        Thread thread1 = new Thread(new UseOwnUncaughtExceptionHandler());
        Thread thread2 = new Thread(new UseOwnUncaughtExceptionHandler());
        thread.start();
        thread1.start();
        thread2.start();
    }
}

```



## 7. 线程安全


### 1. 什么是线程安全
**<font style="color:#F5222D;">多个线程访问某个对象或方法时，在编写方法或对象的业务逻辑时，不需要做额外的处理（可以像单线程编程一样），程序可以正常运行（不会因为多线程而出错）</font>**

****

### 2. 线程安全问题




1. 运行结果错误：a++多线程问题
2. 活跃性问题：死锁、活锁、饥饿
3. 对象发布和初始化安全问题

****

## 8. 死锁
### 1. 死锁概念
特点

+ 发生在并发中
+ 互不相让





![](/images/891a7379a9651789b92eedc8bf527109.png)



### 2. 死锁事例
1. 两个用户转账

```java


/**
 * @author: xin
 * @date: 2021/10/9
 * 转账时遇到死锁情况
 */

public class TransferMoney implements Runnable {

    private int flag;

    static Account from = new Account(300);
    static Account to = new Account(300);


    public static void main(String[] args) throws InterruptedException {
        TransferMoney user1 = new TransferMoney();
        TransferMoney user2 = new TransferMoney();
        user1.flag = 0;
        user2.flag = 1;
        Thread thread = new Thread(user1);
        Thread thread1 = new Thread(user2);
        thread.start();
        thread1.start();
        thread.join();
        thread1.join();
        System.out.println(from.balance);
        System.out.println(to.balance);
    }

    @Override
    public void run() {
        if (flag == 0) {
            transferMoney(from, to, 200);
        }
        if (flag == 1) {
            transferMoney(to, from, 200);
        }
    }

    static void transferMoney(Account from, Account to, int account) {
        synchronized (from) {
            synchronized (to) {
                if (from.balance - account < 0) {
                    System.out.println(from + "转账用户余额不足");
                    return;
                }
                from.balance -= account;
                to.balance += account;
                System.out.println(from + "转账成功" + account + "元");
            }
        }
    }

    static class Account {
        public int balance;

        public Account(int balance) {
            this.balance = balance;
        }
    }
}

```

2. 多个用户转账

```java


import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author: xin
 * @date: 2021/10/9
 */

public class MultiTransferMoney {


    /**
     * 账户数
     */
    public static final int NUMS_ACCOUNT = 500;
    /**
     * 账户初始化金额数
     */
    public static final int NUMS_MONEY = 10000;
    /**
     * 转账次数
     */
    private static final int NUMS_TRANSFERS = 1000000;
    private static final int NUM_THREDS = 20;

    static Random rdm = new Random();
    private static TransferMoney.Account[] accounts = new TransferMoney.Account[NUMS_ACCOUNT];

    public static void main(String[] args) {
        for (int i = 0; i < NUMS_ACCOUNT; i++) {
            accounts[i] = new TransferMoney.Account(NUMS_MONEY);
        }
        ExecutorService threadPool = Executors.newFixedThreadPool(20);
        for (int i = 0; i < NUM_THREDS; i++) {
            threadPool.execute(new TransferMoneyThread());
        }
    }

    static class TransferMoneyThread implements Runnable {


        @Override
        public void run() {
            for (int i = 0; i < NUMS_TRANSFERS; i++) {
                int accountFromIndex;
                int accountToIndex;
                do {
                    accountFromIndex = rdm.nextInt(NUMS_ACCOUNT);
                    accountToIndex = rdm.nextInt(NUMS_ACCOUNT);
                } while (accountFromIndex == accountToIndex);
                int money = rdm.nextInt(NUMS_MONEY);
                TransferMoney.transferMoney(accounts[accountFromIndex], accounts[accountToIndex], money);
            }
            System.out.println("转账结束");
        }
    }
}

```





### 3. 死锁条件
1. **互斥条件 **
2. **请求与保持条件**
3. **不剥夺条件**
4. **循环等待条件**





### 4. 定位死锁
#### 1. Jstack 
1. 通过jps 获取Java运行程序的pid
2. 通过jstack  pid 查看对应程序的死锁情况

```shell
J:\JavaBase\cocurrency_tools_practice>jps
16672 TransferMoney
17784
19592 Jps
4236 Launcher

J:\JavaBase\cocurrency_tools_practice>jstack 16672

```



### 5. 修复死锁
### 1. 避免策略




# 2. 并发高级
## 1. 线程池


### 1. 线程池的停止
1. shutdown** 关闭线程池，但是线程池需要正在执行的任务和队列中的任务执行完之后关闭**
2. shutdownNow  立刻关闭线程池，并返回任务队列中的任务。同时中断正在执行的任务
3. isShutdown 判断线程是否关闭
4. isTerminated 判断线程池是否终止运行
5. awaitTermination 判断线程池在判断延后的时间范围内线程池是否终止



### 2. 拒绝策略
+ AbortPolicy  直接抛出异常
+ DiscardPolicy 直接抛弃任务
+ DiscardOldestPolicy 直接抛弃执行时间最长的任务
+ CallerRunsPolicy 直接返回给调用者





### 3. 实现原理
#### 1. 组成部分
+ 线程池管理器
+ 工作线程
+ 任务队列
+ 任务接口



![](/images/e59b2bc7c0c1e2bfd6377d4f4015f8bc.png)



**线程池架构图**

![](/images/acc02ec119e05ad87133ffd1c6d4d8e7.png)



+ Executor 线程池顶级接口
+ ExecutorService  线程池业务接口



**线程池任务复用原理**

```java
  public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
      	//获取线程池状态
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
      //判断线程池状态以及任务队列是否满了
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
      //
        else if (!addWorker(command, false))
            reject(command);
    }

 //调用addWorker创建新的工作线程
   boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;



//然后调用runWorker执行当前线程的任务


 final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) {
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
```

**线程池状态**

+ Running 接受新任务并排队处理任务
+ Shutdown 不接受新任务，但处理正在运行任务
+ Stop 不接受新任务 ，也不处理队列任务，并中断正在运行任务
+ Tidying  所有任务都终止，所有工作线程为0并接下来执行terminated（）钩子方法
+ Terminated 运行完成





## 2. ThreadLocal


### 1. 使用场景


1. 每个线程需要一个独享的对象（工具类对象SimpleDateFormat 和Random）

```java
package threadlocal;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashSet;
import java.util.Set;
import java.util.concurrent.*;
import java.util.logging.SimpleFormatter;

/**
 * @author: xin
 * @date: 2021/10/4
 * ThreadLocal 存放工日期格式化具类对象
 * 每个线程都需要存放一个独享的对象
 */

public class TreadLocalNormalUsage {
    public static Set<String > dateSet = new HashSet<>();

    public static void main(String[] args) {
        ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(100, 100, 10, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(10));
        for (int i = 0; i < 100; i++) {
            int finalI = i;
            poolExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    dateParse(finalI);
                }

                public void dateParse(int i) {
                    Date date = new Date(1000 * i);
                    SimpleDateFormat dateFormat = ThreadLocalDateFormatter.dateFormatThreadLocal.get();
                    dateSet.add(dateFormat.format(date));
                    System.out.println(dateFormat.format(date));
                }
            });

        }

        System.out.println(dateSet.size());
    }
}


class ThreadLocalDateFormatter {
    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal =
            ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
}
```

2. 每个线程内需要保存全局变量，可以不同方法直接使用，避免传递参数的麻烦



### 2. 作用
1. 多线程，对象数据数据隔离
2. 多线程下，轻松获取获取全局变量



### 3. 好处
+ 线程安全 
+ 不需要加锁，提高效率
+ 线程池的情况下，节省内存开销
+ 免去多线程情况下传参麻烦





### 3. 原理 、源码分析
**主要方法**



1. **initialValue()**
+ 返回当前线程对应的初始值，延迟加载只有第一次调用get时候才会触发。
+ 当线程先前调用了set()，则不会调用initialValue()方法 
+ 线程最多只能调用一次该方法，除非调用remove()删除了对应的数据，在调用get
+ 不重写默认返回null

```java
   public T get() {
        Thread t = Thread.currentThread();
       //获取当前线程的ThreadLocalMap
        ThreadLocalMap map = getMap(t);
       //如果对应的map不为空
        if (map != null) {
            //通过map获取对应的值
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
       //反之则调用初始化方法
        return setInitialValue();
    }
```

****

****

2. **set()**

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

```





**ThreadLocalMap类**

****

**键：ThreadLocal对象**

**值：存储对应的值**

****

**ThreadLocal哈希冲突解决办法：线性探测法（发生冲突判断下一个位置是否为空）**

****

****

****

### 4. 注意点


+ **内存泄漏问题**（对象不在使用，但却无法回收）









## 3. AQS




### 1. 原理解析


#### 1. State状态
在不同的并发类中有不同的含义

Semaphore 剩余许可证的数量

CountDownLatch 需要倒数的数量

ReentrantLock 表示可重入锁的次数 当state 表示锁不被任何线程持有



#### 2.FIFO队列 
+ 存放等待线程的双向队列



![](/images/ffda6652d43b8ea94ed89e17a497eb55.png)

#### 3. 工具类实现的获取/释放方法


1. 获取方法（**<font style="color:#F5222D;">导致线程阻塞</font>**）
+ ReentrantLock判断state是否为0，如果不为为0则会阻塞
+ Semaphore通过acquire方法获取到state,判断state是否为正数，是则state-1可以获取一个许可证
+ CountDownLatch 通过await获取state，判断是否等于0，如果为0则唤醒，反之阻塞





2. 释放方法
+ Semaphore 通过使用release方法使state+1
+ CountDownLatch countDown方法







#### 4. 源码分析


##### 1. CountDownLatch源码分析
![](/images/274faaa2100ff7eec49c65d1ae45fe31.png)

![](/images/8fab9ccdeaf25bd6ebc07116ee2c57b2.png)



```java

	//构造函数
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
  		Sync(int count) {
            setState(count);
        }

    protected final void setState(int newState) {
        state = newState;
    }

		
   int getCount() {
            return getState();
        }


	//线程等待直到倒数结束	
  public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
	//
  public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
      //判断是否倒数结束
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }

  private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
      //将线程包装成一个node节点然后放入队列然后中断该线程
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

```





##### 2. Semaphore源码分析


```java
	//获取资源
public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }


 private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    //如果state大于0则将线程放入阻塞队列
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

	//公平锁的方式获取资源state
   protected int tryAcquireShared(int acquires) {
            for (;;) {
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }


//非公平锁的方式获取资源state
 final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }

```







##### 3. ReentrantLock






**释放锁**

```java
public void unlock() {
        sync.release(1);
    }

		
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

	//尝试释放当前线程的锁
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
    //判断当前线程是否持有锁
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
    		//判断state是否为0
            if (c == 0) {
                free = true;
                //释放锁
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```



流程：

判断当前锁对应的线程是不是该线程如果是直接state-1,不是抛出异常，减到0就返回true。并且释放当前线程的锁



**加锁方法**

****

```java
    public void lock() {
        sync.lock();
    }

//非公平加锁的实现

   final void lock() {
       //通过cas判断其他线程是否持有锁
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                //尝试获取锁
                acquire(1);
        }


  public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }


     final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
         //判断锁是否被持有
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
         //判断锁的线程和当前线程是否是同一线程
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

	
	//公平锁加锁
   final void lock() {
            acquire(1);
        }


 protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```



### 2. 实现简化版ConuntDownLatch


步骤：

1. 创建一个类，实现获取/释放的方法
2. 写一个内部类Sync继承AbstractQueuedSynchronizer
3. 根据锁是否独占来重写tryAcquire/tryRelease或tryAcquireShared和tryReleaseShared等方法



```java
package aqs;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;

/**
 * @author: xin
 * @date: 2021/10/7
 * 自定义一次性门闩
 */

public class OneShortLatch {
    private final Sync sync = new Sync();

    public void signal() {
        sync.releaseShared(0);
    }

    public void await() {
        sync.acquireShared(0);
    }

    private class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected int tryAcquireShared(int arg) {
            return getState() == 1 ? 1 : -1;
        }

        @Override
        protected boolean tryReleaseShared(int arg) {
            setState(1);
            return true;
        }
    }


    public static void main(String[] args) throws InterruptedException {
        OneShortLatch oneShortLatch = new OneShortLatch();
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "尝试获取latch");
                oneShortLatch.await();
                System.out.println("门闩调用了放行方法");
            }).start();
        }
        Thread.sleep(1000);
        oneShortLatch.signal();
    }
}

```





