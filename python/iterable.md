# Python之Iterable

## 迭代的定义

在[维基百科](https://zh.wikipedia.org/wiki/%E8%BF%AD%E4%BB%A3)对迭代的定义是这样：

> 迭代是重复反馈过程的活动，其目的通常是为了接近并到达所需的目标或结果。每一次对过程的重复称为一次“迭代”，而每一次得到的结果会被用来作为下一次迭代的初始值。

## Iterable

翻译为可迭代对象，特点就是序列的大小长度已经确定了（list，tuple，dict， string等），它遵循可迭代的协议：

* 含`__iter__()`方法，返回的是一个对应的迭代器。

## Iterator

翻译为迭代器，特点就是它不知道要执行多少次，所以理解不知道有多少个元素，每调用一次`__next__()`方法，就会往下走一步，只能往下走，不会回退，且惰性的。它遵循迭代器协议：

* 含`__iter__()`方法，返回的是Iterator对象本身
* 含`__next__()`方法，每当`__next__()`被调用，第一步：更新iterator状态，令其指向后一项，以便下一次调用；第二步：返回当前结果。

```
list = [1, 2, 3, 4]
list_iterator = iter(list)

print(type(list))                       # 输出：<class 'list'>
print(type(list_iterator))              # 输出：<list_iterator object at 0x107410e48>

print(list_iterator)                    # 输出：<list_iterator object at 0x107410e48>
print(list_iterator.__next__())         # 输出：1
print(list_iterator.__next__())         # 输出：2
print(list_iterator.__next__())         # 输出：3
print(list_iterator.__next__())         # 输出：4
#print(list_iterator.__next__())         # 抛出StopIteration异常
```

## 自定义一个迭代器

```
class EvenIterators(object):
    def __init__(self, n):
        self.stop = n
        self.value = -2;

    def __iter__(self):
        return self;

    def __next__(self):
        if self.value + 2 > self.stop:
            raise StopIteration
        self.value += 2
        return self.value

for e in EvenIterators(10):
    print(e)
```

输出：

```
0
2
4
6
8
10
```

上面的代码实现了一个偶数迭代器。在代码中我们实现了迭代器协议，即`__iter__()`和`__next__()`方法，也就实现了iterator。


## 参考资料

* [Python之Iterable与Iterator](https://zhuanlan.zhihu.com/p/32162554)
* [Iterator](https://wiki.python.org/moin/Iterator)