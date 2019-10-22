# wait

wait()、notify()、notifyAll()是Object类中的方法

```
public final void notifyAll()

public final void notify()

public final void wait() throws InterruptedException

```

1. 调用某个对象的wait方法能让当前线程阻塞，并且当前线程必须拥有此对象的monitor
2. 调用某个对象的notify方法能够唤醒一个正在等待此对象的monitor的线程，如果有多个线程都在等待这个对象的monitor，则只能唤醒其中一个线程
3. 调用notifyAll方法能够唤醒所有正在等待这个对象的monitor的线程

有朋友可能会有疑问：为何这三个不是Thread类声明中的方法，而是Object类中声明的方法（当然由于Thread类继承了Object类，所以Thread也可以调用者三个方法）？其实这个问题很简单，由于每个对象都拥有monitor（即锁），所以让当前线程等待某个对象的锁，当然应该通过这个对象来操作了。而不是用当前线程来操作，因为当前线程可能会等待多个线程的锁，如果通过线程来操作，就非常复杂了。

上面已经提到，如果调用某个对象的wait方法，当前线程必须拥有这个对象的monitor，因此调用wait方法必须在synchronized同步块或者同步方法中进行。

调用某个对象的wait方法，相当于让当前线程交出此对象的monitor，然后进入等待状态，等待后续再次获得此对象的锁。sleep方法使当前线程暂停执行一段时间，从而让其他线程有机会继续执行，但它不释放对象锁。

同样地，调用某个对象的notify方法，当前线程也必须拥有这个对象的monitor。

注意：notify和notifyAll方法只是唤醒等待该对象的monitor的线程，并不决定哪个线程能够获取到monitor。

举个简单的例子：假如有三个线程Thread1、Thread2和Thread3都在等待对象objectA的monitor，此时Thread4拥有对象objectA的monitor，当在Thread4中调用objectA.notify()方法之后，Thread1、Thread2和Thread3只有一个能被唤醒。注意，被唤醒不等于立刻就获取了objectA的monitor。假若在Thread4中调用objectA.notifyAll()方法，则Thread1、Thread2和Thread3三个线程都会被唤醒，至于哪个线程接下来能够获取到objectA的monitor就具体依赖于操作系统的调度了。

尤其要注意，一个线程被唤醒不代表立即获取了对象的monitor，只有等待调用notify或者notifyAll并退出synchronized同步块（方法），释放对象锁后，其余线程才可获得锁。

```
import java.util.concurrent.*;

public class Demo {
	public static Object object = new Object();


	public static void main(String[] args) throws Exception {
		Thread1 thread1 = new Thread1();
		Thread2 thread2 = new Thread2();

		thread1.start();

		try {
			Thread.sleep(200);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

		thread2.start();
	}

	static class Thread1 extends Thread {
		public void run() {
			synchronized (object) {
				try {
					System.out.println(Thread.currentThread().getName() + " waiting");
					object.wait();
					System.out.println(Thread.currentThread().getName() + " get lock");
				} catch (InterruptedException e) {

				}
			}
			System.out.println(Thread.currentThread().getName() + " exit");
		}
	}

	static class Thread2 extends Thread {
		public void run() {
			synchronized (object) {
				try {
					object.notify();
					System.out.println(Thread.currentThread().getName() + " notified");
					Thread.sleep(1000);
					System.out.println(Thread.currentThread().getName() + " synchronized finished");
				} catch (InterruptedException e) {

				}
			}
			System.out.println(Thread.currentThread().getName() + " exit");
		}
	}
}


/*
Thread-0 waiting
Thread-1 notified
Thread-1 synchronized finished
Thread-1 exit
Thread-0 get lock
Thread-0 exit
*/
```

## Codition

Condition是同来替代传统的await()、notify()实现线程间的协作。

* Condition是个接口，基本的方法就是await和signal方法
* Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition()
* 调用Condition的await()和signal方法，都必须在lock保护之内。

## 生产者和消费者

### wait/notify方案

```
import java.util.concurrent.*;
import java.util.*;

public class Demo {
	public int queueSize = 10;
	private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);


	public static void main(String[] args) throws Exception {
		Demo demo = new Demo();
		Producer producer = demo.new Producer();
		Consumer consumer = demo.new Consumer();

		producer.start();
		consumer.start();

		Thread.sleep(10);
		System.exit(0);
	}

	class Consumer extends Thread {
		public void run() {
			consume();
		}

		private void consume() {
			while (true) {
				synchronized (queue) {
					while (queue.size() == 0) {
						try {
							System.out.println("queue is empty, waiting...");
							queue.wait();
						} catch (InterruptedException e) {
							e.printStackTrace();
							queue.notify();
						}
					}
					queue.poll();
					queue.notify();
					System.out.println("fetch a object from queue, size is " + queue.size());
				}
			}
		}
	}

	class Producer extends Thread {
		public void run() {
			produce();
		}

		private void produce() {
			while (true) {
				synchronized (queue) {
					while (queue.size() == queueSize) {
						try {
							System.out.println("queue full, waiting...");
							queue.wait();
						} catch (InterruptedException e) {
							e.printStackTrace();
							queue.notify();
						}
					}

					queue.offer(1);
					queue.notify();
					System.out.println("add a object to queue, size is " + queue.size());
				}
			}
		}
	}
}
```

## lock/condition方案

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	public int queueSize = 10;
	private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);
	private Lock lock = new ReentrantLock();
	private Condition notFull = lock.newCondition();
	private Condition notEmpty = lock.newCondition();


	public static void main(String[] args) throws Exception {
		Demo demo = new Demo();
		Producer producer = demo.new Producer();
		Consumer consumer = demo.new Consumer();

		producer.start();
		consumer.start();

		Thread.sleep(1);
		System.exit(0);
	}

	class Consumer extends Thread {
		public void run() {
			consume();
		}

		private void consume() {
			while (true) {
				lock.lock();
				try {
					while (queue.size() == 0) {
						try {
							System.out.println("queue is empty, waiting...");
							notEmpty.await();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
					queue.poll();
					notFull.signal();
					System.out.println("fetch a object from queue, size is " + queue.size());
				} finally {
					lock.unlock();
				}
			}
		}
	}

	class Producer extends Thread {
		public void run() {
			produce();
		}

		private void produce() {
			while (true) {
				lock.lock();
				try {
					while (queue.size() == queueSize) {
						try {
							System.out.println("queue is full, waiting...");
							notFull.await();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}

					queue.offer(1);
					notEmpty.signal();
					System.out.println("add a object to queue, size is " + queue.size());
				} finally {
					lock.unlock();
				}
			}
		}
	}
}
```

## 参考资料

* [Java并发编程：线程间协作的两种方式：wait、notify、notifyAll和Condition](https://www.cnblogs.com/dolphin0520/p/3920385.html)