# Python之class学习

## 类的定义
定义类是通过class关键字，括号里面表示父类

```
class Student(object):
    pass

s = Student()
print(s)            # 输出： <__main__.Student object at 0x10c2ecef0>
print(Student)      # 输出： <class '__main__.Student'>

s.name = "muyu"
print(s.name)       # 输出： muyu
```

这里有点跟静态语言（C++）不一样的地方，可以随时给一个对象添加属性，如`s.name="muyu"`。

当然，也可以把必须绑定的属性强制填写进去，通过`__init__`函数，例如：

```
class Student(object):
    def __init__(self, name, score):         # 构造函数
        self.name = name
        self.score = score

    def get_grade(self):
        if self.score >= 90:
            return 'A'
        elif self.score >= 70:
            return 'B'
        else:
            return 'C'


s = Student("muyu", 90)
print(s.name)
print(s.score)
print(s.get_grade())
```

从上面的`get_grade`也可以看出，定义函数，第一参数永远是实例变量self（相当于this指针），并且调用时，不用传递该参数，除此之外，类的方法和普通函数没有什么区别。

## 访问限制

如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线`__`。

```
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score


s = Student("muyu", 90)
#print(s.__name)            # AttributeError: 'Student' object has no attribute '__name'
print(s._Student__name)     # 实际上被改成这样，但不推荐这般使用，
print(s._Student__score)    # 因为不同版本的Python解释器可能会改成不同的名字
```

有些时候，你会看到以一个下划线开头的实例变量名，比如`_name`，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是：“虽然我可以被访问，但是请把我视为私有变量，不要随便访问”。
为了访问私有变量，可以为私有变量增加`setter/getter`方法，如下：

```
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def get_name(self):
        return self.__name

    def set_name(self, name):
        self.__name = name

    def get_score(self):
        return  self.__score

    def set_score(self, score):
        self.__score = score


s = Student("muyu", 90)
print(s.get_name())
print(s.get_score())

s.set_name("chen")
s.set_score(80)
print(s.get_name())
print(s.get_score())
```

### 使用@property

上面的`getter/setter`写法比较麻烦，可以采用内置的@property装饰器（decorator），比如：

```
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, name):
        self.__name = name

    @property
    def score(self):
        return self.__score

    @score.setter
    def score(self, score):
        self.__score = score


s = Student("muyu", 90)
print(s.name)
print(s.score)

s.name = "chen"
s.score = 80
print(s.name)
print(s.score)
```

当然，如果不定义setter方法，该属性就是一个只读属性。

```
    # 如果注释该方法，会输出
    # AttributeError: can't set attribute
    @score.setter
    def score(self, score):
        self.__score = score
```


## 多继承
在Python中，类也是支持多继承的。只需要在定义类时的括号里把继承的所有类名写入就可以。例如：

```
class Dog(Animal, Runnable):
    pass
```

## 使用__slots__
正常情况下，当我们定义了一个class，创建一个class的实例后，我们可以给该实例绑定任何属性和方法，这就是动态语言的灵活性。但是，如果我们想要限制实例的属性怎么办？比如，只允许对Student实例添加name和score属性，这时就可以采用`__slots__`变量。

```
class Student(object):
    __slots__ = ('name', 'score')

s = Student()
s.name = "muyu"
print(s.name)
s.score = 90
print(s.score)
s.age = 18
print(s.age)        # 输出：AttributeError: 'Student' object has no attribute 'age'
```

## dir()

要获得一个对象的所有属性和方法，可以使用dir()函数，它返回一个list。

```
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, name):
        self.__name = name

    @property
    def score(self):
        return self.__score

    @score.setter
    def score(self, score):
        self.__score = score

s = Student("chen", 90)
print(dir(Student))
print(dir(s))
```

输出：
```
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'name', 'score']

['_Student__name', '_Student__score', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'name', 'score']
```

仅仅把属性和方法列出来是不够的，配合`getattr()`、`setattr`和`hasattr()`，我们可以直接操作一个对象的状态。

```
class Student(object):
    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, name):
        self.__name = name

    @property
    def score(self):
        return self.__score

    @score.setter
    def score(self, score):
        self.__score = score

s = Student("chen", 90)
print(hasattr(s, "name"))

setattr(s, "age", 19);
print(getattr(s, "age"))
```

## 鸭子类型

```
class Animal(object):
    def run(self):
        print('Animal is running...')

class Timer(object):
    def run(self):
        print('Timer is running...')


def run(animal:Animal):
    animal.run()

a = Animal()
t = Timer()

run(a)
run(t)
```

输出

```
Animal is running...
Timer is running...
```

就上面的代码来说，对于有些语言来说，必须传入Animal类型的对象，但Python不需要，只要保证传入的对象有run方法就可以了。这就是动态语言的“鸭子类型”，它并不要求严格的继承体系，一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看作是鸭子。



## 参考资料
* [Python类的使用总结](https://blog.csdn.net/fengxinlinux/article/details/77091914)
* [Python：动态语言与鸭子类型](https://foofish.net/dynamic_type_and_duck_type.html)
