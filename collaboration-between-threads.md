# 线程间协作：wait、notify、notifyAll

在 Java 中，可以通过配合调用 Object 对象的 wait() 方法和 notify()方法或 notifyAll() 方法来实现线程间的通信。在线程中调用 wait() 方法，将阻塞等待其他线程的通知（其他线程调用 notify() 方法或 notifyAll() 方法），在线程中调用 notify() 方法或 notifyAll() 方法，将通知其他线程从 wait() 方法处返回。

Object 是所有类的超类，它有 5 个方法组成了等待/通知机制的核心：notify()、notifyAll()、wait()、wait(long)和 wait(long，int)。在 Java 中，所有的类都从 Object 继承而来，因此，所有的类都拥有这些共有方法可供使用。而且，由于他们都被声明为 final，因此在子类中不能覆写任何一个方法。

这里详细说明一下各个方法在使用中需要注意的几点。

## wait()

**public final void wait()  throws InterruptedException,IllegalMonitorStateException**

该方法用来将当前线程置入休眠状态，直到接到通知或被中断为止。在调用 wait()之前，线程必须要获得该对象的对象级别锁，即只能在同步方法或同步块中调用 wait()方法。进入 wait()方法后，当前线程释放锁。在从 wait()返回前，线程与其他线程竞争重新获得锁。如果调用 wait()时，没有持有适当的锁，则抛出 IllegalMonitorStateException，它是 RuntimeException 的一个子类，因此，不需要 try-catch 结构。

## notify()

**public final native void notify() throws IllegalMonitorStateException**

该方法也要在同步方法或同步块中调用，即在调用前，线程也必须要获得该对象的对象级别锁，的如果调用 notify()时没有持有适当的锁，也会抛出 IllegalMonitorStateException。

该方法用来通知那些可能等待该对象的对象锁的其他线程。如果有多个线程等待，则线程规划器任意挑选出其中一个 wait()状态的线程来发出通知，并使它等待获取该对象的对象锁（notify 后，当前线程不会马上释放该对象锁，wait 所在的线程并不能马上获取该对象锁，要等到程序退出 synchronized 代码块后，当前线程才会释放锁，wait所在的线程也才可以获取该对象锁），但不惊动其他同样在等待被该对象notify的线程们。当第一个获得了该对象锁的 wait 线程运行完毕以后，它会释放掉该对象锁，此时如果该对象没有再次使用 notify 语句，则即便该对象已经空闲，其他 wait 状态等待的线程由于没有得到该对象的通知，会继续阻塞在 wait 状态，直到这个对象发出一个 notify 或 notifyAll。这里需要注意：它们等待的是被 notify 或 notifyAll，而不是锁。这与下面的 notifyAll（）方法执行后的情况不同。 

## notifyAll()

**public final native void notifyAll() throws IllegalMonitorStateException**

该方法与 notify ()方法的工作方式相同，重要的一点差异是：

notifyAll 使所有原来在该对象上 wait 的线程统统退出 wait 的状态（即全部被唤醒，不再等待 notify 或 notifyAll，但由于此时还没有获取到该对象锁，因此还不能继续往下执行），变成等待获取该对象上的锁，一旦该对象锁被释放（notifyAll 线程退出调用了 notifyAll 的 synchronized 代码块的时候），他们就会去竞争。如果其中一个线程获得了该对象锁，它就会继续往下执行，在它退出 synchronized 代码块，释放锁后，其他的已经被唤醒的线程将会继续竞争获取该锁，一直进行下去，直到所有被唤醒的线程都执行完毕。

## 深入理解

如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。

当有线程调用了对象的 notifyAll()方法（唤醒所有 wait 线程）或 notify()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。

优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它还会留在锁池中，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。

