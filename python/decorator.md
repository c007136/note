# 理解Python装饰器（decorator）

谈decorator前，要先明白一个事，Python中的函数和JAVA、C++不太一样，它可以像普通变量一样当做参数传递给另外一个函数，例如：

```
def foo():
    print('foo')

def bar(func):
    func()

bar(foo);
```

装饰器本质上是一个Python函数或类，它可以让其他函数或类在不需要做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象。它经常用于切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等场景。

## 简单装饰器

```
def use_logging(func):
    def wrapper():
        print("%s is running" %(func.__name__))
        return func()    # 注意这儿返回func()
    return wrapper

def foo():
    print("foo")

foo = use_logging(foo)
foo()
```

use_logging就是一个装饰器，它是一个普通的函数，它把真正执行业务逻辑的函数func包裹其中，看起来像foo被use_logging装饰了一样，use_logging返回的也是一个函数，它这个函数的名字叫wrapper。在这个例子中，函数进入和退出时，被称为一个横切面，这种编程方式被称为面向切面的编程。

## @语法糖

@符号就是装饰器的语法糖，它放在函数开始定义的地方，可以省略一步赋值的操作。

```
def use_logging(func):
    def wrapper():
        print("%s is running" %(func.__name__))
        return func()    # 注意这儿返回func()
    return wrapper

@use_logging
def foo():
    print("foo")

foo()
```

## 传递参数

如果foo需要传递参数，则要在定义wrapper函数的时候指定参数

```
def use_logging(func):
    def wrapper(name):
        print("%s is running" %(func.__name__))
        return func(name)
    return wrapper

@use_logging
def foo(name):
    print("hi, my name is %s" %(name))

foo("muyu")
```

如果要传递多个参数，则使用关键字`*args`

```
def use_logging(func):
    def wrapper(*args):
        print("%s is running" %(func.__name__))
        return func(*args)
    return wrapper

@use_logging
def foo(name, card_id, phone):
    print("hi, my name is %s, card_id is %s, phone is %s" %(name, card_id, phone))

foo("muyu", "1234", "1301111")
```

如果参数带了默认值，则使用关键字`**kwargs`

```
def use_logging(func):
    def wrapper(*args, **kwargs):
        print("%s is running" %(func.__name__))
        return func(*args, **kwargs)
    return wrapper

@use_logging
def foo(name, card_id="1234", phone="1301111"):
    print("hi, my name is %s, card_id is %s, phone is %s" %(name, card_id, phone))

foo("muyu")
```

## 带参数的装饰器

```
def use_logging(level):
    def decorator(func):
        def wrapper(name):
            if level == "warn":
                print("## warning ## %s is running" % (func.__name__))
            else:
                print("## info ## \n %s is running" % (func.__name__))
            return func(name)
        return wrapper
    return decorator



@use_logging("warn")
def foo(name):
    print("hi, my name is %s" %(name))

foo("muyu")
```

## 类装饰器

```
class Foo(object):
    def __init__(self, func):
        self._func = func

    def __call__(self):
        print("class decorator running")
        self._func()
        print("class decorator ending")


@Foo
def bar():
    print("bar")

bar()
```


## functools.wraps
测试结果还存在


## 装饰器顺序

一个函数可以定义多个装饰器，比如：

```
@a
@b
@c
def f ():
    pass
```

它的执行顺序是从里到外，最先调用里层的装饰器，最后调用最外层的装饰器，它等效于：

```
f = a(b(c(f)))
```

## 参考文案
[理解 Python 装饰器看这一篇就够了](https://foofish.net/python-decorator.html)