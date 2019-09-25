# Polymorphism

```
#include <iostream>

class A {
public:
	int value() { return 5; }
};

class B : public A {
public:
	int value() { return 10; }
};

int main() {
	B *b = new B();
	A *a = (A *)b;
	std::cout << a->value() << std::endl;

	return 0;
}

/*
5
*/
```

上面的例子，结果输出为5。这是C++静态绑定的例子，编译器负责解析要调用的函数，因此在编译后相应的函数被绑定到二进制文件中，无法在运行时更改。

如果函数前面增加关键字`virtual`，结果就变成了10。这是C++的多态。

```
#include <iostream>

class A {
public:
	virtual int value() { return 5; }
};

class B : public A {
public:
	virtual int value() { return 10; }
};

int main() {
	B *b = new B();
	A *a = (A *)b;
	std::cout << a->value() << std::endl;

	return 0;
}

/*
10
*/
```

### 析构函数

下面的例子只调用了`A::~A()`，这不是我们想要的。解决的版本是在析构函数前面加上`virtual`。

这是一个原则：永远在析构函数前面加上`virtual`，除非你绝对确定这个类不被继承。


```
#include <iostream>

class A {
public:
	~A() { std::cout << "A::~A()" << std::endl; }
};

class B : public A {
public:
	~B() { std::cout << "B::~B()" << std::endl; }
};

int main() {
	B *b = new B();
	A *a = (A *)b;
	delete a;

	return 0;
}

/*
A::~A()
*/
```

### 参考资料

[Introduction to C++ for iOS Developers: Part 2](https://www.raywenderlich.com/2483-introduction-to-c-for-ios-developers-part-2)