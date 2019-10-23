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

