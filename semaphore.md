# 并发新特性—信号量 Semaphore

在操作系统中，信号量是个很重要的概念，它在控制进程间的协作方面有着非常重要的作用，通过对信号量的不同操作，可以分别实现进程间的互斥与同步。当然它也可以用于多线程的控制，我们完全可以通过使用信号量来自定义实现类似 Java 中的 synchronized、wait、notify 机制。

Java 并发包中的信号量 Semaphore 实际上是一个功能完毕的计数信号量，从概念上讲，它维护了一个许可集合，对控制一定资源的消费与回收有着很重要的意义。Semaphore 可以控制某个资源被同时访问的任务数，它通过acquire（）获取一个许可，release（）释放一个许可。如果被同时访问的任务数已满，则其他 acquire 的任务进入等待状态，直到有一个任务被 release 掉，它才能得到许可。

下面给出一个采用 Semaphore 控制并发访问数量的示例程序：

```
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
import java.util.concurrent.Semaphore;  
public class SemaphoreTest{  
    public static void main(String[] args) {  
    //采用新特性来启动和管理线程——内部使用线程池  
    ExecutorService exec = Executors.newCachedThreadPool();  
    //只允许5个线程同时访问  
    final Semaphore semp = new Semaphore(5);  
    //模拟10个客户端访问  
    for (int index = 0; index < 10; index++){  
        final int num = index;  
        Runnable run = new Runnable() {  
            public void run() {  
                try {  
                    //获取许可  
                    semp.acquire();  
                    System.out.println("线程" +   
                        Thread.currentThread().getName() + "获得许可："  + num);  
                    //模拟耗时的任务  
                    for (int i = 0; i < 999999; i++) ;  
                    //释放许可  
                    semp.release();  
                    System.out.println("线程" +   
                        Thread.currentThread().getName() + "释放许可："  + num);  
                    System.out.println("当前允许进入的任务个数：" +  
                        semp.availablePermits());  
                }catch(InterruptedException e){  
                    e.printStackTrace();  
                }  
            }  
        };  
          exec.execute(run);  
    }  
    //关闭线程池  
    exec.shutdown();  
    }  
}  
```

某次执行的结果如下：

```
线程pool-1-thread-1获得许可：0
线程pool-1-thread-1释放许可：0
当前允许进入的任务个数：5
线程pool-1-thread-2获得许可：1
线程pool-1-thread-6获得许可：5
线程pool-1-thread-4获得许可：3
线程pool-1-thread-8获得许可：7
线程pool-1-thread-2释放许可：1
当前允许进入的任务个数：2
线程pool-1-thread-5获得许可：4
线程pool-1-thread-8释放许可：7
线程pool-1-thread-3获得许可：2
线程pool-1-thread-4释放许可：3
线程pool-1-thread-10获得许可：9
线程pool-1-thread-6释放许可：5
线程pool-1-thread-10释放许可：9
当前允许进入的任务个数：2
线程pool-1-thread-3释放许可：2
当前允许进入的任务个数：1
线程pool-1-thread-5释放许可：4
当前允许进入的任务个数：3
线程pool-1-thread-7获得许可：6
线程pool-1-thread-9获得许可：8
线程pool-1-thread-7释放许可：6
当前允许进入的任务个数：5
当前允许进入的任务个数：3
当前允许进入的任务个数：3
当前允许进入的任务个数：3
线程pool-1-thread-9释放许可：8
当前允许进入的任务个数：5
```

可以看出，Semaphore 允许并发访问的任务数一直为 5，当然，这里还很容易看出一点，就是 Semaphore 仅仅是对资源的并发访问的任务数进行监控，而不会保证线程安全，因此，在访问的时候，要自己控制线程的安全访问。