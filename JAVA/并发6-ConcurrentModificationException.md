# ConcurrentModificationException

## ConcurrentModificationException异常出现的原因

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	public static void main(String[] args) throws Exception {
		ArrayList<Integer> list = new ArrayList<Integer>();
		list.add(2);

		Iterator<Integer> iterator = list.iterator();
		while (iterator.hasNext()) {
			Integer integer = iterator.next();
			if (integer == 2) {
				list.remove(integer);
				//iterator.remove();
			}
		}
	}
}

/*
Exception in thread "main" java.util.ConcurrentModificationException
        at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
        at java.util.ArrayList$Itr.next(ArrayList.java:859)
        at Demo.main(Demo.java:12)
*/
```

原因为：modCount != expectedModCount，所以抛出了异常。

## 单线程解决办法

```
iterator.remove();
```

## 多线程解决办法

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class Demo {
	static ArrayList<Integer> list = new ArrayList<Integer>();

	public static void main(String[] args) throws Exception {
		list.add(1);
		list.add(2);
		list.add(3);
		list.add(4);
		list.add(5);

		Thread thread1 = new Thread() {
			public void run() {
				synchronized (list) {
					Iterator<Integer> iterator = list.iterator();
					while (iterator.hasNext()) {
						Integer integer = iterator.next();
						System.out.println("integer = " + integer);
						try {
							Thread.sleep(100);
						} catch (Exception e) {
							e.printStackTrace();
						}
					}
				}
			}
		};

		Thread thread2 = new Thread() {
			public void run() {
				synchronized (list) {
					Iterator<Integer> iterator = list.iterator();
					while (iterator.hasNext()) {
						Integer integer = iterator.next();
						if (integer == 2) {
							System.out.println("remove integer = " + integer);
							iterator.remove();
						}
					}
				}
			}
		};

		thread1.start();
		thread2.start();
	}
}

/*
 * integer = 1 Exception in thread "Thread-0"
 * java.util.ConcurrentModificationException at
 * java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909) at
 * java.util.ArrayList$Itr.next(ArrayList.java:859) at Demo$1.run(Demo.java:18)
 */
```

上面的代码不带synchronized，也会出现异常。


## 参考资料

* [Java ConcurrentModificationException异常原因和解决方法](https://www.cnblogs.com/dolphin0520/p/3933551.html)

