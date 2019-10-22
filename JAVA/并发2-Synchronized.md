# Synchronized

## 线程安全问题

这个就是线程安全问题，即多个线程同时访问一个资源时，会导致程序运行结果并不是想看到的结果。

这里面，这个资源被称为：临界资源（也有称为共享资源）。

也就是说，当多个线程同时访问临界资源（一个对象，对象中的属性，一个文件，一个数据库等）时，就可能会产生线程安全问题。

不过，当多个线程执行一个方法，方法内部的局部变量并不是临界资源，因为方法是在栈上执行的，而Java栈是线程私有的，因此不会产生线程安全问题。

## 如何解决线程安全问题

基本上所有的并发模式在解决线程安全问题时，都采用“序列化访问临界资源”的方案，即在同一时刻，只能有一个线程访问临界资源，也称作同步互斥访问。

通常来说，是在访问临界资源的代码前面加上一个锁，当访问完临界资源后释放锁，让其他线程继续访问。

在Java中，提供了两种方式来实现同步互斥访问：`synchronized`和`Lock`。

## 简介

在了解`synchronized`关键字的使用方法之前，我们先来看一个概念：互斥锁，故名思义：能达到互斥访问目的的锁。

举个简单的例子：如果对临界资源加上互斥锁，当一个线程在访问该临界资源时，其他线程便只能等待。

在Java中，每一个对象都拥有一个锁标记（monitor），也称为监视器，多线程同时访问某个对象时，线程只有获取了该对象的锁才能访问。

在Java中，可以使用`synchronized`关键字来标记一个方法或者代码块，当某个线程调用该对象的`synchronized`方法或者访问`synchronized`代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。

它修饰的对象有以下几种：

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的范围，作用的对象是调用这个代码块的对象。
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象。
3. 修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象。
4. 修饰一个类，其作用的范围是`synchronized`后面括号括起来的部分，作用的对象是这个类的所有对象。

## synchronized使用

### synchronized修饰代码块

#### 多个线程访问同一对象的同步代码块

```
import java.util.concurrent.*;
import java.util.*;

class ThreadSyn implements Runnable {
	private volatile static int count = 0;

	public void run() {
		synchronized(this) {
			for (int i = 0; i < 5; i++) {
				try {
					System.out.println(Thread.currentThread().getName() + " : " + (count++));
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		ThreadSyn ts = new ThreadSyn();
		Thread t1 = new Thread(ts, "thread1");
		Thread t2 = new Thread(ts, "thread2");
		t1.start();
		t2.start();
	}
}


/*
thread1 : 0
thread1 : 1
thread1 : 2
thread1 : 3
thread1 : 4
thread2 : 5
thread2 : 6
thread2 : 7
thread2 : 8
thread2 : 9
*/
```

结论：当并发线程访问同一个对象中的`synchronized`代码块时，在同一时刻只能有一个线程得到执行，另一个线程受阻塞。一个对象就是一个锁。

#### 多个线程访问不同对象的同步代码块

```
import java.util.concurrent.*;
import java.util.*;

class ThreadSyn implements Runnable {
	private volatile static int count = 0;

	public void run() {
		synchronized(this) {
			for (int i = 0; i < 5; i++) {
				try {
					System.out.println(Thread.currentThread().getName() + " : " + (count++));
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		ThreadSyn ts1 = new ThreadSyn();
		ThreadSyn ts2 = new ThreadSyn();
		Thread t1 = new Thread(ts1, "thread1");
		Thread t2 = new Thread(ts2, "thread2");
		t1.start();
		t2.start();
	}
}


/*
thread1 : 0
thread2 : 1
thread2 : 2
thread1 : 2
thread2 : 3
thread1 : 4
thread2 : 5
thread1 : 6
thread2 : 7
thread1 : 8
*/
```

结论：多个线程访问不同对象的同步代码块，线程不会阻塞，互不干扰。

#### 指定要给某个对象加锁

当有一个明确的对象作为锁时，就可以用类似下面这样的方式写程序

```
public void method3(SomeObject obj)
{
   //obj 锁定的对象
   synchronized(obj)
   {
      // todo
   }
}
```

当没有明确的对象作为锁，只是想让一段代码同步时，可以创建一个特殊的对象来充当锁：

```
class Test implements Runnable
{
   private byte[] lock = new byte[0];  // 特殊的instance变量
   public void method()
   {
      synchronized(lock) {
         // todo 同步代码块
      }
   }

   public void run() {

   }
}
```

说明：零长度的byte数组对象创建起来将比任何对象都经济——查看零长度的byte[]对象只需要3条操作码，而`Object lock = new Object()`则需要7行操作。

### synchronized修饰一个方法

写法一：

```
public synchronized void method()
{
   // todo
}
```

写法二：

```
public void method()
{
   synchronized(this) {
      // todo
   }
}
```

注意事项：

1. synchronized不能继承
2. 在定义接口方法时不能使用synchronized
3. 构造方法不能使用synchronized关键字，但可以使用synchronized代码块来同步

### synchronized修饰一个静态的方法

```
public synchronized static void method() {
   // todo
}
```

```
import java.util.concurrent.*;
import java.util.*;

class ThreadSyn implements Runnable {
	private volatile static int count = 0;

	public void run() {
		method();
	}

	public synchronized static void method() {
		for (int i = 0; i < 5; i++) {
			try {
				System.out.println(Thread.currentThread().getName() + " : " + (count++));
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		ThreadSyn ts1 = new ThreadSyn();
		ThreadSyn ts2 = new ThreadSyn();
		Thread t1 = new Thread(ts1, "thread1");
		Thread t2 = new Thread(ts2, "thread2");
		t1.start();
		t2.start();
	}
}


/*
thread1 : 0
thread1 : 1
thread1 : 2
thread1 : 3
thread1 : 4
thread2 : 5
thread2 : 6
thread2 : 7
thread2 : 8
thread2 : 9
*/
```

上面两个线程保持了线程同步。synchronized作用的对象是一个静态方法，则它取得的是类的锁，该类所有的对象同一把锁，所有会阻塞。

### synchronized作用于一个类

用法如下：

```
class ClassName {
   public void method() {
      synchronized(ClassName.class) {
         // todo
      }
   }
}
```

```
import java.util.concurrent.*;
import java.util.*;

class ThreadSyn implements Runnable {
	private volatile static int count = 0;

	public void run() {
		method();
	}

	public static void method() {
		synchronized (ThreadSyn.class) {
			for (int i = 0; i < 5; i++) {
				try {
					System.out.println(Thread.currentThread().getName() + " : " + (count++));
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		ThreadSyn ts1 = new ThreadSyn();
		ThreadSyn ts2 = new ThreadSyn();
		Thread t1 = new Thread(ts1, "thread1");
		Thread t2 = new Thread(ts2, "thread2");
		t1.start();
		t2.start();
	}
}


/*
thread1 : 0
thread1 : 1
thread1 : 2
thread1 : 3
thread1 : 4
thread2 : 5
thread2 : 6
thread2 : 7
thread2 : 8
thread2 : 9
*/
```

结论：synchronized作用于一个类时，是给这个类加锁，这个类的所有对象用的是同一把锁。

## 总结

1. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是类对象，该类所有的对象同一把锁。
2. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。
3. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。


## 参考资料

* [Java多线程——synchronized使用详解](https://blog.csdn.net/zhangqiluGrubby/article/details/80500505)
* [Java并发编程：synchronized](https://www.cnblogs.com/dolphin0520/p/3923737.html)