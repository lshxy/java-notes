# Java线程

[TOC]



## 线程的实现

### 实现多线程的三种方式

- 继承 Thread 类
- 实现 Runnable 接口
- 实现 Callable 接口



### 线程的描述

- 线程是比进程更轻量级的调度执行单位，CPU 调度的基本单位就是线程。
- 线程的引入，将一个进程的资源分配和执行调度分开。
- 各个线程既可以共享进程资源（内存地址、文件 I/O 等），又可独立调度。
- **Java 的 Thread 类：** 所有关键方法都是 Native 的，说明这些方法无法使用平台无关的手段实现。



### 实现线程的 3 种方式

- 使用内核线程实现
- 使用用户线程实现
- 使用用户线程加轻量级进程




## 线程的调度

- **协同式线程调度：** 线程的执行时间由线程本身来控制，线程执行完自己的任务之后，主动通知系统切换到另一个线程。
  - 优点：实现简单，没有线程同步的问题。
  - 缺点：线程执行时间不可控，如果一个线程编写有问题一直无法结束，程序会一直阻塞在那里。
- **抢占式线程调度：** 每个线程由系统分配执行时间，系统决定切不切换线程。



## 线程的状态转换

![1559095894035](./pic/线程状态.png)



![线程状态转换示意图.png](./pic/线程状态转换示意图.png)

## 线程间的通信

- synchronized 和 volatile 关键字

  - 这两个关键字可以保障线程对变量访问的可见性

- 等待/通知机制

  - 详见 `Ch3-Java并发高级主题/00-Java中的锁.md`

- `Thread#join()`
  - 如果一个线程 A 执行了 `threadA.join()`，那么只有当线程 A 执行完之后，`threadA.join()` 之后的语句才会继续执行，类似于创建 A 的线程要等待 A 执行完后才继续执行；
  - 使用 join 方法中线程被中断的效果 == 使用 wait 方法中线程被中断的效果，即会抛出 InterruptedException。因为 join 方法内部就是用 wait 方法实现的；
  - join 还有一个带参数的方法：`join(long)`，这个方法是等待传入的参数的毫秒数，如果计时过程中等待的方法执行完了，就接着往下执行，如果计时结束等待的方法还没有执行完，就不再继续等待，而是往下执行。
    - **`join(long)` 和 `sleep(long)` 的区别**
      - 如果等待的方法提前结束，`join(long)` 不会再计时了，而是往下执行，而 `sleep(long)` 一定要等待够足够的毫秒数；
      - `join(long)` 会释放锁，`sleep(long)` 不会释放锁，原因是 `join(long)` 方法内部是用 `wait(long)` 方法实现的。

- 管道流：`PipedInputStream` & `PipedOutputStream`

	```java
	public class PipedStreamDemo {
	    public static PipedInputStream in = new PipedInputStream();
	    public static PipedOutputStream out = new PipedOutputStream();
	
	    public static void send() {
	        new Thread() {
	            @Override
	            public void run() {
	                byte[] bytes = new byte[2000];
	                while (true) {
	                    try {
	                        out.write(bytes, 0, 2000);
	                        System.out.println("Send Success");
	                    } catch (IOException e) {
	                        System.out.println("Send Failed");
	                        e.printStackTrace();
	                    }
	                }
	            }
	        }.start();
	    }
	
	    public static void receive() {
	        new Thread() {
	            @Override
	            public void run() {
	                byte[] bytes = new byte[100];
	                int len = 0;
	                while (true) {
	                    try {
	                        len = in.read(bytes, 0, 100);
	                        System.out.println("len = " + len);
	                    } catch (IOException e) {
	                        System.out.println("Receive Failed");
	                        e.printStackTrace();
	                    }
	                }
	            }
	        }.start();
	    }
	
	    public static void main(String[] args) {
	        try {
	            in.connect(out);
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        receive();
	        send();
	    }
	}
	```

- ThreadLocal

  - 详见 `Ch1-保证线程安全的两个角度/02-对象的安全共享.md`