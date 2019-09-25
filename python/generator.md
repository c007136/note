# Python之Generator

在Python中有一种函数，是用关键字**yield**来返回值，这种函数叫生成器函数，函数被调用时会返回一个生成器对象，生成器对象本质上还是一个迭代器，唯一的区别在于实现方式上不一样，后者更简洁。

```
def fun(n):
    yield  n*2

print(fun)     # fun为generator函数
g = fun(10)    # g为generator对象

print(g)
```

输出

```
<function fun at 0x105904268>
<generator object fun at 0x1059ac1b0>
```

[官方](https://wiki.python.org/moin/Generators)的说法

> Python provides generator functions as a convenient shortcut to building iterators. Lets us rewrite the above iterator as a generator function

用[迭代器](./iterator.md)中的例子，利用生成器的方式生成一个偶数序列，代码上简洁不少：

```
def fun(n):
    num = 0
    while num < n:
        yield num
        num += 2

for f in fun(10):
    print(f)
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


## 参考资料
* [看完这篇，你就知道Python生成器是什么](https://foofish.net/what-is-python-generator.html)
* [Generators](https://wiki.python.org/moin/Generators)