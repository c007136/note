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


## 参考资料

* [Java并发编程：Lock](https://www.cnblogs.com/dolphin0520/p/3923167.html)