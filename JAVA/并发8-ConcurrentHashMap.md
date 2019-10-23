# ConcurrentHashMap

## 线程不安全的HashMap

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	public static void main(String[] args) throws Exception {
		final HashMap<String, String> map = new HashMap<String, String>(2);
		for (int i = 0; i < 100; i++) {
			MyThread thread = new MyThread(map, "thread_"+i);
			thread.start();
		}

	}
}

class MyThread extends Thread {
	public Map map;
	public String name;

	public MyThread(Map map, String name) {
		this.map = map;
		this.name = name;
	}

	public void run() {
		double i = Math.random() * 100000;
		map.put("key"+i, "value"+i);
		map.remove("key"+i);
		System.out.println(name + " i = " + i + " size = " + map.size());
	}
}

/*
// 部分输出，非线程安全
thread_48 i = 69212.77971762283 size = 5
thread_64 i = 24116.71307325044 size = 5
thread_63 i = 78299.57354194386 size = 5
thread_61 i = 7920.545634727616 size = 5
thread_59 i = 37548.81931385709 size = 5
 */
```

## 效率低下的HashTable

HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。

## ConcurrentHasMap的锁分段技术

## 参考资料

* [聊聊并发（四）深入分析ConcurrentHashMap](http://ifeve.com/concurrenthashmap/)
* [java 并发编程之 ConcurrentHashMap](https://juejin.im/entry/58808a221b69e60059038382)