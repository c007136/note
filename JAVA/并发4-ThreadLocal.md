# ThreadLocal

## 对ThreadLocal的理解

ThreadLocal，很多地方叫做线程本地变量，也有些地方叫做线程本地存储。其作用：为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	ThreadLocal<Long> longLocal = new ThreadLocal<Long>();
	ThreadLocal<String> stringLocal = new ThreadLocal<String>();

	public void set() {
		longLocal.set(Thread.currentThread().getId());
		stringLocal.set(Thread.currentThread().getName());
	}

	public long getLong() {
		return longLocal.get();
	}

	public String getString() {
		return stringLocal.get();
	}

	public static void main(String[] args) throws Exception {
		Demo demo = new Demo();

		demo.set();
		System.out.println(demo.getLong());
		System.out.println(demo.getString());

		Thread thread1 = new Thread() {
			public void run() {
				demo.set();
				System.out.println(demo.getLong());
				System.out.println(demo.getString());
			}
		};
		thread1.start();
		thread1.join();

		System.out.println(demo.getLong());
		System.out.println(demo.getString());
	}
}

/*
1
main
10
Thread-0
1
main
*/
```

## ThreadLocal的应用场景

最常见的ThreadLocal使用场景为：数据库连接、Session管理等。

## 参考资料

[Java并发编程：深入剖析ThreadLocal](https://www.cnblogs.com/dolphin0520/p/3920407.html)