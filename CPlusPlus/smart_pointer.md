# Smart Pointer

### shared pointer

```
#include <iostream>
#include <memory>

class A {
public:
	A() { std::cout << "A::A()" << std::endl; }
	virtual ~A() { std::cout << "A::~A()" << std::endl; }
};

void f() {
	std::shared_ptr<A> p1(new A());
	std::shared_ptr<A> p2 = p1;
	std::shared_ptr<A> p3 = p1;
}

int main() {
	std::cout << "before f()" << std::endl;

	f();

	std::cout << "after f()" << std::endl;

	return 0;
}

/*
before f()
A::A()
A::~A()
after f()
*/
```

### weak_ptr

```
#include <iostream>
#include <memory>

class B;

class A {
public:
	A() { std::cout << "A::A()" << std::endl; }
	virtual ~A() { std::cout << "A::~A()" << std::endl; }

	std::shared_ptr<B> spB;

	void f() {
		std::cout << "A::f()" << std::endl;
	}
};

class B {
public:
	B() { std::cout << "B::B()" << std::endl; }
	virtual ~B() { std::cout << "B::~B()" << std::endl; }

    // weak_ptr是来解决循环引用的问题
	//std::shared_ptr<A> spA;
	std::weak_ptr<A> spA;

	void f() {
		std::cout << "B::f()" << std::endl;
	}
};

int main() {
	std::shared_ptr<B> b(new B());
	std::shared_ptr<A> a(new A());

	b->spA = a;
	a->spB = b;

	a->spB->f();

	// 不能通过weak_ptr直接访问对象的方法
	// 要转成shared_ptr
	//b->spA->f();

	std::shared_ptr<A> p = b->spA.lock();
	p->f();

	return 0;
}

/*
B::B()
A::A()
B::f()
A::f()
A::~A()
B::~B()
*/
```

### unique_ptr

unique_ptr是一个独享所有权的智能指针，它提供了严格意义上的所有权，包括：
1. 拥有它指向的对象
2. 无法进行复制构造，无法进行赋值操作。即无法使两个unique_ptr指向同一个对象。但是可以移动构造和移动赋值操作
3. 保存指向某个对象的指针，当它本身被删除释放的时候，会使用给定的删除器释放它指向的对象。

```
#include <iostream>
#include <memory>

class A {
public:
	A() { std::cout << "A::A()" << std::endl; }
	virtual ~A() { std::cout << "A::~A()" << std::endl; }

	void f() {
		std::cout << "A::f()" << std::endl;
	}
};

std::unique_ptr<A> fun() {
	return std::unique_ptr<A>(new A());
}

int main() {
	std::unique_ptr<A> pA(new A());

    // 编译错误
	//std::unique_ptr<A> pOne(pA);
	//std::unique_ptr<A> pTwo = pA;

    // 这里是显式的所有权转移，把pA所指的内存转给pA1
    // 而pA不再拥有该内存
	std::unique_ptr<A> pA1 = std::move(pA);

    // 如上
	std::unique_ptr<A> pA2 = fun();

	return 0;
}

/*
A::A()
A::A()
A::~A()
A::~A()
*/
```

### 参考资料

1. [Introduction to C++ for iOS Developers: Part 2](https://www.raywenderlich.com/2483-introduction-to-c-for-ios-developers-part-2)
2. [智能指针shared_ptr的用法](https://www.cnblogs.com/jiayayao/p/6128877.html)
3. [c++ 智能指针用法详解](https://www.cnblogs.com/TenosDoIt/p/3456704.html)
4. [C++ 11 创建和使用 unique_ptr](https://www.cnblogs.com/DswCnblog/p/5628195.html)