# Lock

## synchronized的缺陷

采用`synchronized`的方法，获取线程锁的线程释放锁只会有两种情况：

1. 获取锁的线程执行完了该代码块。
2. 线程执行发生异常，此时JVM会让线程自动释放锁。

那么如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程只能干巴巴地等待，试想一下，这多么影响程序执行效率。

因此就需要一种机制可以不让等待的线程一直无期地等待下去（比如只等待一定的时间或者能够响应中断），通过Lock就可以办到。

另外，通过Lock可以知道线程有没有成功获取到锁。这个是`synchronized`无法办到的。

但是要注意一下几点：

1. Lock不是Java语言内置的，`synchronized`是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步互斥访问。
2. Lock和`synchronized`有一点非常大的不同，采用`synchronized`不需要用户去手动释放锁，而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。

## java.util.concurrent.locks包下常用的类

### Lock

查看源码可以知道Lock是一个接口

```
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

### ReentrantLock

lock用法：

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	private ArrayList<Integer> arrayList = new ArrayList<Integer>();
	private Lock lock = new ReentrantLock();

	public void insert(Thread thread) {
		lock.lock();
		try {
			System.out.println(thread.getName() + " get lock");
			for (int i = 0; i < 5; i++) {
				arrayList.add(i);
			}
		} catch (Exception e) {

		} finally {
			System.out.println(thread.getName() + " release lock");
			lock.unlock();
		}
	}


	public static void main(String[] args) throws Exception {
		Demo demo = new Demo();
		
		new Thread() {
			public void run() {
				demo.insert(Thread.currentThread());
			}
		}.start();

		new Thread() {
			public void run() {
				demo.insert(Thread.currentThread());
			}
		}.start();
	}
}


/*
Thread-0 get lock
Thread-0 release lock
Thread-1 get lock
Thread-1 release lock
*/
```

lockInterruptibly用法

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	private ArrayList<Integer> arrayList = new ArrayList<Integer>();
	private Lock lock = new ReentrantLock();

	public void insert(Thread thread) throws InterruptedException {
		lock.lockInterruptibly();    // 注意，如果需要正确中断等待锁的线程，必须将获取锁放在外面，然后将InterruptedException抛出
		try {
			System.out.println(thread.getName() + " get lock");
			long startTime = System.currentTimeMillis();
			while (true) {
				if(System.currentTimeMillis() - startTime >= Integer.MAX_VALUE) {
					break;
				}
			}
		} finally {
			System.out.println(thread.getName() + " release lock");
			lock.unlock();
		}
	}


	public static void main(String[] args) throws Exception {
		Demo demo = new Demo();
		MyThread thread1 = new MyThread(demo);
		MyThread thread2 = new MyThread(demo);
		thread1.start();
		thread2.start();

		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			thread2.interrupt();
			System.exit(0);
		}
	}
}

class MyThread extends Thread {
	private Demo demo = null;

	public MyThread(Demo demo) {
		this.demo = demo;
	}

	public void run() {
		try {
			demo.insert(Thread.currentThread());
		} catch (InterruptedException e) {
			System.out.println(Thread.currentThread().getName() + " is interrupted");
		}
	}
}


/*
Thread-0 get lock
Thread-1 is interrupted
*/
```

## 锁的相关概念

### 可重入锁

广义上的可重入锁指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁（前提得是同一个对象或者class），这样的锁就叫做可重入锁。ReentrantLock和synchronized都是可重入锁。

```
import java.util.concurrent.*;

public class Demo implements Runnable {
	public synchronized void get() {
		System.out.println(Thread.currentThread().getName());
		set();
	}

	public synchronized void set() {
		System.out.println(Thread.currentThread().getName());
	}


	public void run() {
		get();
	}


	public static void main(String[] args) throws Exception {
		Demo demo = new Demo();
		while (true) {
			new Thread(demo).start();
		}
	}
}


/*
*/
```

### 可中断锁

可中断锁：顾名思义，就是可以相应中断的锁。

在Java中，`synchronized`就不是可中断锁，而Lock是可中断锁。

### 公平锁

公平锁即尽量以请求锁的顺序来获取锁。比如同是多个线程在等待锁，等待时间最久的锁会获得该锁，这种就是公平锁。

`synchronized`就是非公平锁，而对于`ReentrantLock`和`ReentrantReadWriteLock`，它默认情况下是非公平锁，但是可以设置为公平锁。



## 参考资料

* [Java并发编程：Lock](https://www.cnblogs.com/dolphin0520/p/3923167.html)