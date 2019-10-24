# Callable、Future和FutureTask

## Callable+Future

```
import java.util.concurrent.*;

public class Demo {
	public static void main(String[] args) {
		ExecutorService es = Executors.newCachedThreadPool();
		MyTask task = new MyTask();
		Future<Integer> result = es.submit(task);
		es.shutdown();

		System.out.println("main thrad running");

		try {
			System.out.println("result of task is " + result.get());
		} catch (Exception e) {
			e.printStackTrace();
		}

		System.out.println("all done");
	}
}

class MyTask implements Callable<Integer> {
	public Integer call() throws Exception {
		System.out.println(Thread.currentThread().getName() + " running");
		Thread.sleep(3000);
		int sum = 0;
		for (int i = 0; i < 100; i++) {
			sum += i;
		}
		return sum;
	}
}


/*
main thrad running
pool-1-thread-1 running
result of task is 4950
all done
*/
```

## Callable+FutureTask

```
import java.util.concurrent.*;

public class Demo {
	public static void main(String[] args) {
		MyTask task = new MyTask();
		FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
		Thread thread = new Thread(futureTask);
		thread.start();

		System.out.println("main thrad running");

		try {
			System.out.println("result of task is " + futureTask.get());
		} catch (Exception e) {
			e.printStackTrace();
		}

		System.out.println("all done");
	}
}

class MyTask implements Callable<Integer> {
	public Integer call() throws Exception {
		System.out.println(Thread.currentThread().getName() + " running");
		Thread.sleep(3000);
		int sum = 0;
		for (int i = 0; i < 100; i++) {
			sum += i;
		}
		return sum;
	}
}


/*
main thrad running
Thread-0 running
result of task is 4950
all done
*/
```