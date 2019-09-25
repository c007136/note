# Class的声明与定义

### 方式一

```
#include <iostream>

class A {
public:
	void f();
};

void A::f() {
	std::cout << "A::f()\n";
}


int main() {
	A a = A();
	a.f();

	return 0;
}
```

### 方式二

```
#include <iostream>

class A {
public:
	// 内联函数
	void f() {
		std::cout << "A::f()\n";
	}
};

int main() {
	A a = A();
	a.f();

	return 0;
}
```

### 方式三

```
// A.hpp
class A {
public:
	void f();
};

// A.cpp
#include <iostream>
#include "MyClass.hpp"

void A::f() {
	std::cout << "A::f() \n";
}

// demo.cpp

#include "MyClass.hpp"

int main() {
	A a = A();
	a.f();

	return 0;
}

```

### 参考资料

[Introduction to C++ for iOS Developers: Part 1](https://www.raywenderlich.com/2484-introduction-to-c-for-ios-developers-part-1)