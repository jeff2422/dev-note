---
title: 《Java并发编程的艺术》之 Java并发编程基础
description: Java为多线程编程提供了良好、考究并且一致的编程模型，使开发人员能够更加专注于问题的解决，即为所遇到的问题建立合适的模型，而不是绞尽脑汁地考虑如何将其多线程化。一旦开发人员建立好了模型，稍作修改总是能够方便地映射到Java提供的duo多线程编程模型上。
categories:
 - Java并发编程
tags:
 - Java并发编程
---

> 本文为本人学习《Java并发编程的艺术》一书的笔记整理、文中代码基本上来源于该书，为方便学习形成记录。

<!-- more -->

### 线程优先级

1. 线程优先级的高低决定处理器分配到该线程的时间片的多少，决定该线程获得的时间。
2. 优先级的范围：1～10. 默认为：5 。
3. 针对频繁阻塞的线程需要设置较高的优先级，偏重计算的线程设置较低的优先级； 有些操作系统和JVM会忽略优先级。

### 线程的状态

1. NEW：初始状态，线程被构建，但是还没有调用start()方法。
2. RUNNABLE：运行状态，Java线程将操作系统中的就绪和运行两种状态笼统地称作“运行中”。
3. BLOCKED：阻塞状态，表示线程阻塞于锁。
4. WAITING：等待状态， 表示线程进入等待状态， 需要等待其他线程通知或中断。
5. TIME_WAITING: 超时等待状态，可以指定时间自行返回。
6. TERMINATED：终止状态， 表示当前线程已经执行完毕。

线程状态流图：【后续补充】

### Deamon 线程

1. 可以其他线程设置为守护线程，主要用于后台调度以及支持性工作。当Java虚拟机中不存在Deamon线程的时候Java虚拟机将会推出 Deamon线程立即退出。
2. 在构建Deamon线程时，不能依靠Finally快中的内容来确保执行官彼货清理资源的逻辑。

### 启动和终止线程

#### 1. 启动线程： 
调用start()方法启动线程，含义是高数当前线程（parent）同步告知Java虚拟机，只要线程规划器空闲，应立即启动调用start()方法的线程。「建议启动线程设置线程名称，方便调试」。

#### 2. 理解中断： 
1. 其他线程调用目标线程的方法isInterrupt()方法对其中断操作。
2. 调用静态方法Thread.interrupted() 对当前线程中断操作进行复位。
3. 流程已终结状态，即便被中断过，调用Thread.interrupted()依旧返回false。
4. Thread.sleep()等很多方法在抛出InterruptedException之前Java虚拟机会清楚中断标识位，此时调用Thread.interrupted()返回false。

#### 3. 过期的suspend() resume() stop()
由于资源不释放容易引发的死锁问题，不建议使用。

#### 4. 安全地终止线程
通过标识位或中断操作的方式能够使线程在终止时有机会去清理资源，而不是无端的将线程停止，因此这种终止线程的做法显得更加安全和优雅。

例子：

	public class Shutdown {
	    public static void main(String[] args) throws InterruptedException {
	        Runner one = new Runner();
	        Thread countThread = new Thread(one, "CountThread");
	        countThread.start();
	        TimeUnit.SECONDS.sleep(1);
	        countThread.interrupt();
	
	        Runner two = new Runner();
	        countThread = new Thread(two, "CountThread");
	        countThread.start();
	        TimeUnit.SECONDS.sleep(1);
	        two.cancel();
	    }
	
	    private static class Runner implements Runnable {
	        private long i;
	        private volatile boolean on = true;
	        @Override
	        public void run() {
	            while(on && !Thread.currentThread().isInterrupted()) {
	                i++;
	            }
	            System.out.println("Count i = " + i);
	        }
	
	        public void cancel() {
	            on = false;
	        }
	    }
	}

### 线程间通信

#### 1. volatile 关键字
1. volatile可以修饰字段（成员变量），告知访问该变量需要从共享变量中获取，改变它的同时必须同步刷新会共享变量。保证所有线程对变量访问的可见性。
2. 过多的使用volatile会降低程序的执行效率。

#### 2. synchronized 关键字
1. synchronized可以修饰方法或同步代码块的形式，保证多个线程在同一时刻只能有一个线程处于方法或同步块中，保证了线程对变量访问的可见性和排他性。
2. 当一个线程占用时，其他线程将会被阻塞在同步块和同步方法的入口处，进入BLOCKED状态，占用的线程释放锁的时候会唤醒主色在同步队列中的线程。
3. 对象、监视器、同步队列和执行线程之间的关系图例：
「」

#### 3. 等待/通知机制
Java的等待/通知机制，是指一个线程A调用了对象O的wait()方法进入等待状态，而另一个线程B调用了对象O的notify()或者notifyAll()方法，线程A收到通知后从对象O的wait()方法返回，进而执行后续操作。

1. notify()： 通知一个在对象上等待的线程，时期从wait()方法返回，而返回的前提是该线程获取到了对象的锁。
2. notifyAll(): 通知所有等待在该对象上的线程。
3. wait(): 调用该方法的线程进入WAITING状态， 只有等待另外线程的通知或被中断才会返回，需要注意，调用wait()方法后，会释放对象的锁
4. wait(long): 超时等待一段时间，未通知就超时返回，参数时间是毫秒。
5. wait(long, int): 对于超时时间更细颗粒度的控制，可以达到纳秒。 

Demo:

	public class WaitNotify {
	    static boolean flag = true;
	    static Object lock = new Object();
	
	    public static void main(String[] args) throws InterruptedException {
	        new Thread(new Wait(), "WaitThread").start();
	        TimeUnit.SECONDS.sleep(1);
	        new Thread(new Notify(), "NotifyThread").start();
	    }
	
	    static class Wait implements Runnable {
	        @Override
	        public void run() {
	            synchronized (lock) {
	                while(flag) {
	                    try {
	                        System.out.println(String.format(Thread.currentThread() + "flag is true. wait @ " + new SimpleDateFormat("HH:mm:ss").format(new Date())));
	                        lock.wait();
	                    } catch (InterruptedException e) {
	                        e.printStackTrace();
	                    }
	                }
	                System.out.println(String.format(Thread.currentThread() + "flag is false. running @ " + new SimpleDateFormat("HH:mm:ss").format(new Date())));
	            }
	        }
	    }
	
	    static class Notify implements  Runnable {
	        @Override
	        public void run() {
	            synchronized (lock) {
	                System.out.println(String.format(Thread.currentThread() + "hold lock. notify @ " + new SimpleDateFormat("HH:mm:ss").format(new Date())));
	                lock.notifyAll();
	                flag = false;
	                try {
	                    TimeUnit.SECONDS.sleep(5);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	            synchronized (lock) {
	                try {
	                    System.out.println(String.format(Thread.currentThread() + "hold lock again. sleep @ " + new SimpleDateFormat("HH:mm:ss").format(new Date())));
	                    TimeUnit.SECONDS.sleep(5);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	}
	
注意：

1. 使用wait(),notify(),和notifyAll()时需要先对调用对象加锁。
2. 调用wait()方法后，线程状态由RUNNING变为WAITING， 将当前线程放入到等待队列。
3. notify()和notifyAll()方法调用后，等待线程不会立即返回，需要等到调用线程的释放锁，等待线程才有机会从返回。
4. notify()方法将等待队列中的一个等待线程移到同步队列中，notifyAll()作用全部线程。被移动的线程从WAITING变为BLOCKED。
5. 从wait()方法返回的前提是获得了调用对象的锁。

【补充示意图】

#### 4. 等待/通知的经典范式（生产者、消费者）
1. 等待方（消费者）遵循如下原则：
	1. 获取对象的锁。
	2. 如果条件不满足，调用对象wait()方法，被通知后仍要检查条件。
	3. 条件满足泽执行对应的逻辑。
2. 通知方（生产者）遵循如下原则：
	1. 获取对象的锁。
	2. 改变条件。
	3. 通知所有等待在该对象上的线程。

#### 5. 管道输入/输出
管道输入/输出主要用于线程之间的数据传输，而传输的饿媒介为内存。对于Piped类型的流，必须先进行绑定（connect）；主要包含4个具体实现：PipedOutputStream，PipedInputStream,PipedReader,PipedWriter.

Demo:

	public class Piped {
	    public static void main(String[] args) throws IOException {
	        PipedWriter out = new PipedWriter();
	        PipedReader in = new PipedReader();
	        out.connect(in);
	        Thread printThread = new Thread(new Print(in), "PrintThread");
	        printThread.start();
	        int receive = 0;
	        try {
	            while((receive = System.in.read()) != -1) {
	                out.write(receive);
	            }
	        } catch (IOException e) {
	            e.printStackTrace();
	        }finally {
	            out.close();
	        }
	    }
	
	    static class Print implements Runnable {
	        private PipedReader in;
	        public Print(PipedReader in) {
	            this.in = in;
	        }
	        @Override
	        public void run() {
	            int receive = 0;
	            try {
	                while((receive = in.read()) != -1) {
	                    System.out.print((char) receive );
	                }
	            } catch (IOException e) {
	                e.printStackTrace();
	            }
	        }
	    }
	}
#### 6. Thread.join()的使用
如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才从thread.join()返回。Thread还提供了join(long millis)和join(long millis, int nanos)两个超时特性的方法。

Demo:

	public class Join {
	    public static void main(String[] args) throws InterruptedException {
	        Thread previous = Thread.currentThread();
	        for(int i = 0; i < 10; i++) {
	            Thread thread = new Thread(new Domino(previous), String.valueOf(i));
	            thread.start();
	            previous = thread;
	        }
	        TimeUnit.SECONDS.sleep(5);
	        System.out.println(Thread.currentThread().getName() + " teminate.");
	    }
	
	    static class Domino implements Runnable{
	        private Thread thread;
	        public Domino(Thread thread) {
	            this.thread = thread;
	        }
	
	        @Override
	        public void run() {
	            try {
	                thread.join();
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	            System.out.println(Thread.currentThread().getName() + " teminate.");
	        }
	    }
	}

#### 7. ThreadLocal的使用
线程变量，是一个以ThreadLocal对象为键，任意对象为值的存储结构。这个结构被附带在线程上，可以通过set(T)方法设置，Get()方法获取。

### 线程应用实例

#### 1. 等待超时模式
和前面的例子基本一致，只需要调用wait(long)方法即可实现。该模式比之前Demo的模式更加灵活，不会产生永久阻塞调用者的情况。

