# Template

### 函数

```
#include <iostream>

template <typename T>
void swap(T &a, T &b) {
    T temp = a;
    a = b;
    b = temp;
}

int main() {
	int ix = 1, iy =2;
	swap(ix, iy);
	std::cout << ix << " " << iy << std::endl;

	float fx = 3.1, fy = 2.2;
	swap(fx, fy);
	std::cout << fx << " " << fy << std::endl;

	return 0;
}

/*
2 1
2.2 3.1
*/
```

### 类

```
#include <iostream>

template <typename T>
class Triplet {
  private:
    T a, b, c;

  public:
    Triplet(T a, T b, T c) : a(a), b(b), c(c) {}

    T getA() { return a; }
    T getB() { return b; }
    T getC() { return c; }
};

int main() {
	Triplet<int> intTriplet(1, 2, 3);
	std::cout << intTriplet.getA() << " ";
	std::cout << intTriplet.getB() << " ";
	std::cout << intTriplet.getC() << std::endl;

	Triplet<float> floatTriplet(3.141, 2.901, 10.5);
	std::cout << floatTriplet.getA() << " ";
	std::cout << floatTriplet.getB() << " ";
	std::cout << floatTriplet.getC() << std::endl;

	return 0;
}

/*
1 2 3
3.141 2.901 10.5
*/
```

### 不同类型的类

```
#include <iostream>

template <typename TA, typename TB, typename TC>
class Triplet {
  private:
    TA a;
    TB b;
    TC c;

  public:
    Triplet(TA a, TB b, TC c) : a(a), b(b), c(c) {}

    TA getA() { return a; }
    TB getB() { return b; }
    TC getC() { return c; }
};

int main() {
	Triplet<int, float, char> intTriplet(1, 2.2, 'A');
	std::cout << intTriplet.getA() << " ";
	std::cout << intTriplet.getB() << " ";
	std::cout << intTriplet.getC() << std::endl;

	return 0;
}

/*
1 2.2 A
*/
```