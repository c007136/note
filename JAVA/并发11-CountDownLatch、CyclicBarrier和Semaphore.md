# CountDownLatch、CyclicBarrier和Semaphore

## CountDownLatch用法

比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	public static void main(String[] args) throws Exception {
		final CountDownLatch latch = new CountDownLatch(2);

		new Thread() {
			public void run() {
				try {
					System.out.println(Thread.currentThread().getName() + " running");
					Thread.sleep(1000);
					latch.countDown();
					System.out.println(Thread.currentThread().getName() + " finish");
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}.start();

		new Thread() {
			public void run() {
				try {
					System.out.println(Thread.currentThread().getName() + " running");
					Thread.sleep(1000);
					latch.countDown();
					System.out.println(Thread.currentThread().getName() + " finish");
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}.start();

		try {
			System.out.println("wait 2 thread finish");
			latch.await();
			System.out.println("2 thread finished");
			System.out.println("main thread continue run");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

/*
Thread-0 running
wait 2 thread finish
Thread-1 running
Thread-0 finish
Thread-1 finish
2 thread finished
main thread continue run
 */
```

## CyclicBarrier用法

CyclicBarrier字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CycliBrrier可以被重用。

```
import java.util.concurrent.*;

public class Demo {
	static class Writer extends Thread {
		private CyclicBarrier cyclicBarrier;

		public Writer(CyclicBarrier cyclicBarrier) {
			this.cyclicBarrier = cyclicBarrier;
		}

		public void run() {
			System.out.println(Thread.currentThread().getName() + " is writing data");
			try {
				Thread.sleep(1000);
				System.out.println(Thread.currentThread().getName() + " write data finished");
				cyclicBarrier.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (BrokenBarrierException e) {
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName() + " all thread finished");
		}
	}

	public static void main(String[] args) throws Exception {
		int N = 4;
		CyclicBarrier barrier = new CyclicBarrier(N, new Runnable(){
		
			// 会从4个线程中选择一个线程执行
			@Override
			public void run() {
				System.out.println(Thread.currentThread().getName() + " in CyclicBarrier Runnable");
			}
		});

		for (int i = 0; i < N; i++) {
			new Writer(barrier).start();
		}
	}
}


/*
Thread-1 is writing data
Thread-2 is writing data
Thread-0 is writing data
Thread-3 is writing data
Thread-3 write data finished
Thread-0 write data finished
Thread-1 write data finished
Thread-2 write data finished
Thread-2 all thread finished
Thread-1 all thread finished
Thread-3 all thread finished
Thread-0 all thread finished
*/
```

await()，用来挂起当前线程，直到所有线程都到达barrier状态再同时执行后续任务。

## Semaphore用法

Semaphore字面意思为**信号量**，Semaphore可以控制同时访问的线程个数，通过acquire()获取一个许可，如果没有就等待，而release()释放一个许可。

```
import java.util.concurrent.*;

public class Demo {
	static class Worker extends Thread {
		private int num;
		private Semaphore semaphore;

		public Worker(int num, Semaphore semaphore) {
			this.num = num;
			this.semaphore = semaphore;
		}

		public void run() {
			try {
				System.out.println("worker " + this.num + " waiting");
				semaphore.acquire();
				System.out.println("worker " + this.num + " use a machine");
				Thread.sleep(2000);
				System.out.println("worker " + this.num + " release a machine");
				semaphore.release();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args) {
		int N = 8;
		Semaphore semaphore = new Semaphore(5);
		for (int i = 0; i < N; i++) {
			new Worker(i, semaphore).start();
		}
	}
}


/*
worker 0 waiting
worker 3 waiting
worker 0 use a machine
worker 2 waiting
worker 1 waiting
worker 2 use a machine
worker 4 waiting
worker 3 use a machine
worker 6 waiting
worker 4 use a machine
worker 5 waiting
worker 1 use a machine
worker 7 waiting
worker 0 release a machine
worker 3 release a machine
worker 2 release a machine
worker 5 use a machine
worker 7 use a machine
worker 6 use a machine
worker 4 release a machine
worker 1 release a machine
worker 5 release a machine
worker 7 release a machine
worker 6 release a machine
*/
```

## 总结

* CountDownLatch和CyclicBarrier都能够实现线程的等待，只不过侧重点不同：
   * CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行
   * CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行
* Semephore其实和锁有点类似，它一般用于控制对某组资源的访问权限 

## 参考资料

* [Java并发编程：CountDownLatch、CyclicBarrier和Semaphore](https://www.cnblogs.com/dolphin0520/p/3920397.html)