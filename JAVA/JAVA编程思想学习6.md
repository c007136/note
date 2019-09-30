# JAVA编程思想学习六
> 第15章

## 第15章 泛型 Generics

在面向对象编程语言中，多态算是一种泛型机制。

泛型这个术语的意思：适用于许许多多的类型。其最初的目的是希望类或方法能够具备最广泛的表达能力。如果做到这一点呢，正是通过解藕类或方法与所使用的类型之间的约束。

泛型的主要目的之一就是用来指定容器要持有什么类型的对象，而且由编译器来保证类型的正确性。

Java泛型的核心概念：告诉编译器想使用什么类型，然后编译器帮你处理一切细节。

### 泛型容器

泛型最引人注目的一个原因，就是创造容器类。

#### 元组 tuple

它是将一组对象直接打包存储于其中的一个单一对象。这个容器允许读取其中的元素，但是不允许向其中存放新的对象。（这个概念也称为数据传送对象，或信使）。通常，元组可以具有任意长度，同时，元组中的对象可以是任意不同的类型，且我们希望能够为每一个对象指明其类型，并且从容器中读取出来时，能够得到正确的类型。要处理不同长度的问题，我们需要创建多个不同的元组。

### 泛型接口

泛型也可以应用于接口。例如生成器（generator），这是一种专门负责创建对象的类。实际上，这是工厂方法设计模式的一种应用。不过，当使用生成器创建新的对象时，它不需要任何参数，而工厂方法一般需要参数。也就是说，生成器无需额外的信息就知道如何创建对象。

```
public interface Generator<T> { T next(); }
```

```
import java.util.*;

class Letter {
	private static long counter = 0;
	private final long id = counter++;
	public String toString() {
		return getClass().getSimpleName() + " " + id;
	}
}

class A extends Letter {}

class B extends Letter {}

class C extends Letter {}

class D extends Letter {}

class E extends Letter {}

class F extends Letter {}

class G extends Letter {}

class H extends Letter {}

class I extends Letter {}

class J extends Letter {}

interface Generator<T> { T next(); }

class LetterGenerator implements Generator<Letter>, Iterable<Letter> {
	private Class[] types = {A.class, B.class, C.class, D.class, E.class, F.class, G.class, H.class, I.class, J.class};

	private static Random rand = new Random(26);

	private int size = 0;

	public LetterGenerator(int sz) { size = sz; }

	public Letter next() {
		try {
			int index = rand.nextInt(types.length);
			return (Letter)types[index].newInstance();
		} catch (Exception e) {
			System.out.println("Exception is " + e);
			return null;
		}
	}

	class LetterIterator implements Iterator<Letter> {
		int count = size;

		public boolean hasNext() { return count > 0; }

		public Letter next() {
			count--;
			return LetterGenerator.this.next();
		}

		public void remove() {
			throw new UnsupportedOperationException();
		}
	}

	public Iterator<Letter> iterator() {
		return new LetterIterator();
	}

}

public class Demo {
	public static void main (String[] args) {
		for (Letter l : new LetterGenerator(10)) {
			System.out.println(l);
		}
	}
}

/*
其中一种结果：

E 0
E 1
J 2
D 3
C 4
H 5
H 6
H 7
D 8
E 9
*/
```

### 泛型方法

基本原则：无论何时，只要你能做到，你就应该尽量使用泛型方法。也就是说，如果你使用泛型方法可以取代将整个类泛型化，那么就应该只使用泛型方法，因为它可以使事情更清楚明白。

对于一个static的方法而言，无法访问泛型类的类型参数，所以，如果static方法需要使用泛型能力，就必须使其成为泛型方法。 

要定义泛型方法，只需将泛型参数列表置于返回值之前。例如：

```
class GenericMethod {
	public <T> void f(T x) {
		System.out.println(x.getClass().getSimpleName());
	}
}

public class Demo {
	public static void main (String[] args) {
		GenericMethod gm = new GenericMethod();
		gm.f("");
		gm.f(1);
		gm.f(1.0);
		gm.f(1.0F);
		gm.f('c');
		gm.f(gm);
	}
}

/*
String
Integer
Double
Float
Character
GenericMethod
*/
```

注意，当使用泛型类时，必须在创建对象的时候指定类型参数的值，而使用泛型方法的时候，通常不必指明参数类型，因为编译器会为我们找出具体的类型参数。这成为**类型参数判断（type argument inference）**。因此我们可以像调用普通方法一样调用f(),而且就好像f()被无限次地重载过。

类型参数判断只对赋值操作有效，其他时候并不起作用。

### 显式的类型说明

要显式地指明类型，必须在点操作符与方法名之间插入尖括号，然后把类型置于尖括号内。如果是在定义该方法的类的内部，必须在点操作符之前使用this关键字，如果是使用static的方法，必须在点操作符之前加上类名。

```
public class Demo {
	public static void main (String[] args) {
		f(New.<Person, List<Pet>>map())
	}
}
```

### 可变参数与泛型方法

```
import java.util.*;

public class Demo {
	public static <T> List<T> makeList(T... args) {
		List<T> result = new ArrayList<T> ();
		for (T item : args) {
			result.add(item);
		}
		return result;
	}


	public static void main (String[] args) {
		List<String> ls = makeList("A");
		System.out.println(ls);
		ls = makeList("A", "B", "C");
		System.out.println(ls);
	}
}
```

### 一个通用的Generator

```
interface Generator<T> { T next(); }

class BasicGenerator<T> implements Generator<T> {
	private Class<T> type;

	public BasicGenerator(Class<T> type) {
		this.type = type;
	}

	public T next() {
		try {
			return type.newInstance();
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}

	public static <T> Generator<T> create(Class<T> type) {
		return new BasicGenerator<T>(type);
	}
}

class CountedObject {
	private static long counter = 0;
	private final long id = counter++;
	public String toString() {
		return getClass().getSimpleName() + " " + id;
	}
}

public class Demo {
	public static void main (String[] args) {
		Generator<CountedObject> gen = BasicGenerator.create(CountedObject.class);
		for (int i = 0; i < 5; i++) {
			System.out.println(gen.next());
		}
	}
}
```

### 擦除的神秘之处

```
import java.util.*;

public class Demo {
	public static void main (String[] args) {
		Class c1 = new ArrayList<String>().getClass();
		Class c2 = new ArrayList<String>().getClass();
		System.out.println(c1 == c2);
	}
}
/*
true
*/
```

```
import java.util.*;

public class Demo {
	public static void main (String[] args) {
		List<String> list = new ArrayList<String>();
		System.out.println(Arrays.toString(list.getClass().getTypeParameters()));
	}
}

/*
[E]
*/
```

** 在泛型代码内部，无法获得任何有关泛型参数类型的信息**

Java泛型是使用擦除来实现的，这意味着当你在使用泛型时，任何具体的类型信息都被擦除了，你唯一知道的就是你在使用一个对象。因此List<String>和List<Integer>在运行时事实上是相同的类型。这两种形式都被擦除成它们的“原生”类型，即List。

### C++的方式

```
#include <iostream>
using namespace std;

template<class T> class Manipulator {
  T obj;
public:
  Manipulator(T x) { obj = x; }
  void manipulate() { obj.f(); }
};

class HasF {
public:
  void f() { cout << "HasF::f()" << endl; }
};

int main() {
  HasF hf;
  Manipulator<HasF> manipulator(hf);
  manipulator.manipulate();
}
```

在`manipulate`方法中，C++怎么能知道`f()`方法是为类型参数T而存在的呢？当你实例化这个模版时，C++编译器将进行检查，因此在`Manipulator<HasF>`被实例化的这一刻，它看到`HasF`拥有一个方法`f()`。如果情况并非如此，就会得到一个编译期错误，这样类型安全就得到了保障。（这是不是理解C++是静态语言的切入口）

把上面的代码翻译成Java，就会出现编译错误。为了调用`f()`，我们必须协助泛型类，给定泛型类的边界，以此告知编译器只能接受遵循这个边界的类型。这里重用了`extends`关键字。

```
class HasF {
	public void f() { System.out.println("HasF.f()"); }
}

// 如果没有extends HasF
// 将会编译错误
class Manipulator<T extends HasF> {
	private T obj;

	public Manipulator(T x) { obj = x; }

	public void manipulate() { obj.f(); }
}

public class Demo {
	public static void main (String[] args) {
		HasF hf = new HasF();
		Manipulator<HasF> manipulator = new Manipulator<HasF>(hf);
		manipulator.manipulate();
	}
}
```

擦除的核心动机是它使得泛化的client可以用非泛化的类库来使用，反之亦然，这经常成为“迁移兼容性”。？？？

擦除的代价是显著的，泛型不能用于显式地引用运行时类型的操作之中，例如转型、`instanceof`操作和`new`表达式。无论何时，当你在编写泛型代码时，必须时刻提醒自己，你只是看起来拥有有关参数的类型信息而已，即提醒自己“不，它只是一个Object”。

注意，对于在泛型中创建数组，使用`Array.newInstance()`是推荐的方式。 ？？？

正是因为有了擦除，泛型最令人困惑的方面源自这样一个事实，即可以表示没有任何意义的事物。

```
import java.util.*;
import java.lang.reflect.*;

// 即使kind被存储为Class<T>，擦除也意味着它实际将被存储为Class
class ArrayMaker<T> {
	private Class<T> kind;
	public ArrayMaker(Class<T> kind) { this.kind = kind; }
	T[] create(int size) {
		return (T[])Array.newInstance(kind, size);
	}
}

public class Demo {
	public static void main (String[] args) {
		ArrayMaker<String> stringMaker = new ArrayMaker<String>(String.class);
		String[] a = stringMaker.create(3);
		System.out.println(Arrays.toString(a));

		Array.set(a, 0, "a");
		Array.set(a, 1, "b");
		Array.set(a, 2, "c");
		// 如果a改为Object []，执行的时候还是会报错
		// 感觉没有被擦除掉啊？？？
		// Array.set(a, 2, 3);
		System.out.println(Arrays.toString(a));

		System.out.println("----------");

        // 
		String[] b = (String[])Array.newInstance(String.class, 3);
		System.out.println(Arrays.toString(b));

		Array.set(b, 0, "aa");
		Array.set(b, 1, "bb");
		Array.set(b, 2, "cc");
		System.out.println(Arrays.toString(b));

		System.out.println("----------");
	}
}

/*
[null, null, null]
[a, b, c]
----------
[null, null, null]
[aa, bb, cc]
----------
*/
```

```
import java.util.*;
import java.lang.reflect.*;

class ListMaker<T> {
	List<T> create() { return new ArrayList<T>(); }
}

public class Demo {
	public static void main (String[] args) {
		ListMaker<String> b = new ListMaker<String>();
		List<String> stringList = b.create();
	}
}
```

即使编译器无法知道有关`create()`中的T的任何信息，但是它仍旧可以在编译期确保你放置到result中的对象具有T类型，使其适合`ArrayList<T>`。因此，即使擦除在方法或类内部移除了有关实际类型的信息，编译器仍旧可以确保在方法或类中使用的类型的内部一致性。如下面的例子：

```
import java.util.*;
import java.lang.reflect.*;

class FilledListMaker<T> {
	List<T> create(T t, int n) {
		List<T> result = new ArrayList<T>();
		for (int i = 0; i < n; i++) {
			result.add(t);
		}
		return result;
	}
}

public class Demo {
	public static void main (String[] args) {
		FilledListMaker<String> stringMaker = new FilledListMaker<String>();
		List<String> list = stringMaker.create("hello", 4);
		System.out.println(list);
	}
}

/*
[hello, hello, hello, hello]
*/
```

### 边界处的动作

边界： 对象进入和离开方法的地点。

#### 非泛型代码

```
import java.util.*;
import java.lang.reflect.*;

class SimpleHolder {
	private Object obj;
	public void set(Object obj) { this.obj = obj; }
	public Object get() { return obj; }
}

public class Demo {
	public static void main (String[] args) {
		SimpleHolder holder = new SimpleHolder();
		holder.set("Item");
		String s = (String)holder.get();
	}
}

/*
public class Demo {
  public Demo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class SimpleHolder
       3: dup
       4: invokespecial #3                  // Method SimpleHolder."<init>":()V
       7: astore_1
       8: aload_1
       9: ldc           #4                  // String Item
      11: invokevirtual #5                  // Method SimpleHolder.set:(Ljava/lang/Object;)V
      14: aload_1
      15: invokevirtual #6                  // Method SimpleHolder.get:()Ljava/lang/Object;
      18: checkcast     #7                  // class java/lang/String
      21: astore_2
      22: return
}
*/
```


#### 泛型的代码

```
import java.util.*;
import java.lang.reflect.*;

class SimpleHolder<T> {
	private T obj;
	public void set(T obj) { this.obj = obj; }
	public T get() { return obj; }
}

public class Demo {
	public static void main (String[] args) {
		SimpleHolder<String> holder = new SimpleHolder<String>();
		holder.set("Item");
		String s = holder.get();
	}
}

/*
javap -c Demo

public class Demo {
  public Demo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class SimpleHolder
       3: dup
       4: invokespecial #3                  // Method SimpleHolder."<init>":()V
       7: astore_1
       8: aload_1
       9: ldc           #4                  // String Item
      11: invokevirtual #5                  // Method SimpleHolder.set:(Ljava/lang/Object;)V
      14: aload_1
      15: invokevirtual #6                  // Method SimpleHolder.get:()Ljava/lang/Object;
      18: checkcast     #7                  // class java/lang/String
      21: astore_2
      22: return
}
*/
```

两者产生的字节码是相同的，所以在泛型中的所有动作都发生在边界处--对传递进来的值进行额外的编译期检查，并插入对传递出去的值的转型。记住，“边界就是发生动作的地方。”

### 擦除的补偿

任何在运行时需要知道确切类型信息的操作都将无法工作。类似这样会编译失败：

```
// 编译失败
if (arg instanceof T)
```

补偿的方式，采用方法`isInstance`方法


```
import java.util.*;
import java.lang.reflect.*;

class A {}

class B extends A {}

class TypeCapture<T> {
	Class<T> kind;

	public TypeCapture(Class<T> kind) {
		this.kind = kind;
	}

	public boolean f(Object arg) {
		return kind.isInstance(arg);
	}
}

public class Demo {
	public static void main (String[] args) {
		TypeCapture<A> t1 = new TypeCapture<A>(A.class);
		System.out.println(t1.f(new A()));
		System.out.println(t1.f(new B()));

		TypeCapture<B> t2 = new TypeCapture<B>(B.class);
		System.out.println(t2.f(new A()));
		System.out.println(t2.f(new B()));
	}
}

/*
true
true
false
true
*/
```

### 创建类型实例

在Java中对创建一个`new T()`的尝试将无法实现，部分原因是因为擦除，而另一部分原因是因为编译器不能验证T具有默认构造器。但是在C++中，这种操作很自然，很直观，并且很安全。

Java中的解决方案是传递一个工厂对象，并使用它来创建新的实例。最便利的工厂对象就是Class对象：

```
import java.util.*;
import java.lang.reflect.*;

class Factory<T> {
	T t;

	public Factory(Class<T> kind) {
		try {
			t = kind.newInstance();
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
}

class A {}

public class Demo {
	public static void main (String[] args) {
		Factory<A> fa = new Factory<A>(A.class);
		System.out.println("create A success");
		
		try {
			Factory<Integer> fi = new Factory<Integer>(Integer.class);
		} catch (Exception e) {
			System.out.println("create Integer success");
		}
	}
}

/*
create A success
create Integer success
*/
```

因为Integer没有任何默认的构造器，所以失败了。因为这个错误不是在编译期捕获的，Sun公司建议使用显式的工厂，并将限制其类型，使得只能接受实现了这个工厂的类：

```
import java.util.*;
import java.lang.reflect.*;

interface FactoryI<T> {
	T create();
}

class A<T> {
	private T t;

	public <F extends FactoryI<T>> A(F factory) {
		t = factory.create();
	}
}

class Widget {
	Widget() {
		System.out.println("Widget()");
	}

	public static class Factory implements FactoryI<Widget> {
		public Widget create() {
			return new Widget();
		}
	}
}

public class Demo {
	public static void main (String[] args) {
		A a = new A<Widget>(new Widget.Factory());
	}
}

/*
Widget()
*/
```

另一种方式是模板方法设计模式。

```
import java.util.*;
import java.lang.reflect.*;

abstract class GenericWithCreate<T> {
	final T element;
	GenericWithCreate() { element = create(); }

	abstract T create();
}

class X {}

class Creator extends GenericWithCreate<X> {
	X create() { return new X(); }

	void f() {
		System.out.println( element.getClass().getSimpleName() );
	}
}

public class Demo {
	public static void main (String[] args) {
		Creator c = new Creator();
		c.f();
	}
}

/*
X
*/
```

### 泛型数组

不能创建泛型数组。

```
class Erased<T> {
	private final int SIZE = 100;

	T[] array = new T[SIZE];         // Error
	T[] array = new Object[SIZE];    // Unchecked warning
}
```

一般的解决方法是在任何想要创建泛型数组的地方都使用`ArrayList`。

```
class ListOfGenerics<T> {
	private List<T> array = new ArrayList<T>();
	public void add(T item) { array.add(item); }
	public T get(int index) { return array.get(index); }
}
```

```
import java.util.*;
import java.lang.reflect.*;

class Generic<T> {}

public class Demo {
	public static void main (String[] args) {
		//Generic<Integer>[] gia = (Generic<Integer> [])new Object[10];

		//Generic<Integer>[] gia = (Generic<Integer> [])new Generic[10];
		//System.out.println(gia.getClass().getSimplename());
	}
}
```

上面的例子虽然都能编译通过，但是运行时都会抛异常。问题在于数组将跟踪它们的实际类型，而这个类型是在数组被创建时确定的，因此，即使gia已经被转型为`Generic<Integer> []`，但是这个信息只存在于编译期，在运行时，它仍旧是Object数组。成功创建泛型数组的唯一方式就是创建一个被擦除类型的新数组，然后对其转型。

```
import java.util.*;
import java.lang.reflect.*;

class GenericArray<T> {
	// 内部使用的就是Object数组，而不是T[] array
	private Object[] array;

	public GenericArray(int sz) {
		array = new Object[sz];
	}

	public void put(int index, T item) {
		array[index] = item;
	}

	@SuppressWarnings("unchecked")
	public T get(int index) { return (T)array[index]; }

    // 这是不行的，内部只能是Object[]
    // @SuppressWarnings("unchecked")
	// public T[] rep() {
	// 	return (T())array;
	// }

}

public class Demo {
	public static void main (String[] args) {
		GenericArray<Integer> g = new GenericArray<Integer>(10);
		for (int i = 0; i < 10; i++) {
			g.put(i, i);
		}
		for (int i = 0; i < 10; i++) {
			System.out.print(g.get(i) + " ");
		}
		System.out.println();
	}
}

/*
0 1 2 3 4 5 6 7 8 9
*/
```

### 边界 extends

extends关键字： 限制类型参数为某个类型子集，于是你就可以调用用这些类型子集的方法

```
import java.util.*;
import java.lang.reflect.*;

interface HasColor { java.awt.Color getColor(); }

class Colored<T extends HasColor> {
	T item;
	Colored(T item) { this.item = item; }
	T getItem() { return item; }
	java.awt.Color color() { return item.getColor(); }
}

class Dimension {
	public int x;
	public int y;
	public int z;
}

// class must be first, then interfaces
// class ColoredDimension<T extends HasColor & Dimension> {
class ColoredDimension<T extends Dimension & HasColor> {
	T item;
	ColoredDimension(T item) { this.item = item; }
	T getItem() { return item; }
	java.awt.Color color() { return item.getColor(); }
	int getX() { return item.x; }
	int getY() { return item.y; }
	int getZ() { return item.z; }
}

interface Weight {
	int weight();
}

class Solid<T extends Dimension & HasColor & Weight> {
	T item;
	Solid(T item) { this.item = item; }
	T getItem() { return item; }
	java.awt.Color color() { return item.getColor(); }
	int getX() { return item.x; }
	int getY() { return item.y; }
	int getZ() { return item.z; }
	int weight() { return item.weight(); }
}

class Bounded extends Dimension implements HasColor, Weight {
	Bounded() {
		x = 11;
		y = 12;
		z = 13;
	}

	public java.awt.Color getColor() {
		System.out.println("Bounded:getColor");
		return null;
	}

	public int weight() { return 20; }
}

public class Demo {
	public static void main (String[] args) {
		Solid<Bounded> solid = new Solid<Bounded>(new Bounded());
		solid.color();
		System.out.println("x = " + solid.getX());
		System.out.println("y = " + solid.getY());
		System.out.println("z = " + solid.getZ());
		System.out.println("weight = " + solid.weight());
	}
}

/*
Bounded:getColor
x = 11
y = 12
z = 13
weight = 20
*/
```

### 通配符

协变与逆变用来描述类型转换后的继承关系，其定义：如果A、B表示类型，f(⋅)表示类型转换，≤表示继承关系

f(⋅)是逆变（contravariant）的，当A≤B时有f(B)≤f(A)成立；

f(⋅)是协变（covariant）的，当A≤B时有f(A)≤f(B)成立；

f(⋅)是不变（invariant）的，当A≤B时上述两个式子均不成立，即f(A)与f(B)相互之间没有继承关系。



#### 数组是协变的

```
import java.util.*;
import java.lang.reflect.*;

class Letter {}

class A extends Letter {}
class B extends Letter {}

class AA extends A {}

public class Demo {
	public static void main (String[] args) {
		Letter[] l = new A[10];
		l[0] = new A();
		l[1] = new AA();

        // 数组实际上A[]，虽然编译器允许了
        // 但运行时会抛出异常
		try {
			l[0] = new Letter();
		} catch (Exception e) {
			System.out.println(e);
		}

		try {
			l[0] = new B();
		} catch (Exception e) {
			System.out.println(e);
		}
	}
}

/*
java.lang.ArrayStoreException: Letter
java.lang.ArrayStoreException: B
*/
```

#### 泛型是不变的

```
import java.util.*;
import java.lang.reflect.*;

class Letter {}

class A extends Letter {}
class B extends Letter {}

class AA extends A {}

public class Demo {
	public static void main (String[] args) {
		// 编译错误
		List<Letter> l = new ArrayList<A>();
	}
}
```

泛型的主要目标之一是将这种错误检测移入到编译期。

通配符引入协变、逆变

#### 协变

`<? extends T>`称为子类型通配符。这里，可以声明通配符是由某个特定类的任何子类来界定的。

```
import java.util.*;
import java.lang.reflect.*;

class Letter {}

class A extends Letter {}
class B extends Letter {}

class AA extends A {}

public class Demo {
	public static void main (String[] args) {
		List<? extends Letter> l = new ArrayList<A>();
		// 下面四种全都编译错误
		//l.add(new A());
		//l.add(new B());
		//l.add(new Letter());
		//l.add(new Object());
	}
}
```

上面的情况不能往l放入任何东西，原因在于，`List<? extends Letter>`也可以合法的指向一个`List<B>`，显然往里面放`A`、`Letter`、`Object`都是非法的。编译器不知道`List<? extends Letter>`所持有的具体类型是什么，所以一旦执行这种向上转型，你就将丢失向其中传递任何对象的能力。

#### 逆变

`<? super T>`称为超类型通配符。这里，可以声明通配符是由某个特定类的任何基类来界定的。

```
import java.util.*;
import java.lang.reflect.*;

class Letter {}

class A extends Letter {}
class B extends Letter {}

class AA extends A {}

public class Demo {
	static void writeTo(List<? super A> aList) {
		aList.add(new A());
		aList.add(new AA());
		// 编译错误
		//aList.add(new Letter());
	}

	public static void main (String[] args) {
		
	}
}
```

所以逆变用来写入，协变用来读取

```
class Letter {}

class A extends Letter {}
class B extends Letter {}

class AA extends A {}

public class Demo {
	static void readFrom(List<? extends A> aList) {
		A a = aList.get(0);
		// 编译错误
		//AA aa = aList.get(0);
		Letter l = aList.get(0);
	}

	public static void main (String[] args) {
		
	}
}
```

#### 编译器有多聪明

```
// 参数类型是Object，因此不涉及通配符
aList.contains(new Apple());
aList.indexOf(new Apple());
```

这意味着由泛型类的设计者来决定哪些调用是“安全的”，并使用Object类型作为其参数类型。

#### 参考资料

[Java泛型（二） 协变与逆变](https://www.jianshu.com/p/2bf15c5265c5)


#### 无界通配符

```
import java.util.*;

public class Demo {

	static List list1;
	static List<?> list2;
	static List<? extends Object> list3;

	static void assign1(List list) {
		list1 = list;
		list2 = list;
		// warning: unchecked conversion
		//list3 = list;
	}

	static void assign2(List<?> list) {
		list1 = list;
		list2 = list;
		list3 = list;
	}

	static void assign3(List<? extends Object> list) {
		list1 = list;
		list2 = list;
		list3 = list;
	}

	public static void main (String[] args) {
		assign1(new ArrayList());
		assign2(new ArrayList());
		// warning: unchecked conversion
		//assign3(new ArrayList());

		assign1(new ArrayList<String>());
		assign2(new ArrayList<String>());
		assign3(new ArrayList<String>());

		List<?> wildList = new ArrayList();
		List<?> wildList1 = new ArrayList<String>();

		assign1(wildList);
		assign2(wildList);
		assign3(wildList);

		assign1(wildList1);
		assign2(wildList1);
		assign3(wildList1);
	}
}
```

<?>可以被认为是一种装饰，但是它仍旧是很有价值的，因为，实际上，它是在声明：“我说想用Java的泛型来编写这段代码，我在这里并不是要用原生类型，但是在当前这种情况下，泛型参数可以持有任何类型。”

另外一个重要作用：当你在处理多个泛型参数时，有时允许一个参数可以是任何类型，同时为其他参数确定某种特定类型的这种能力会显得很重要。

```
Map<String, ?> map
```

下面的例子展示了无界通配符能够做什么不能做什么的限制。

```
import java.util.*;

public class Demo {
	// holder被表示成一个原生类型，向set传递Object，编译器
	// 只知道不安全，并不会提示错误
	static void rawArgs(Holder holder, Object arg) {
		// warning
		//holder.set(arg);

        // warning
		//holder.set(new Demo());

		Object obj = holder.get();
		System.out.println("rawArgs obj is " + obj);
	}

    // Holder<?>将持有具有某种具体类型的同构集合，但不知道哪种具体类型
    // 所以不允许向set传递数据
	static void unboundedArg(Holder<?> holder, Object arg) {
		// error
		//holder.set(arg);

		// error
		//holder.set(new Demo());


		Object obj = holder.get();
		System.out.println("unboundedArg obj is " + obj);
	}

	static <T> T exact1(Holder<T> holder) {
		T t = holder.get();
		System.out.println("exact1 t is " + t);
		return t;
	}

	static <T> T exact2(Holder<T> holder, T arg) {
		holder.set(arg);
		T t = holder.get();
		System.out.println("exact2 t is " + t);
		return t;
	}

    // PECS： producer-extends，consumer-super
    // extends是限制数据来源的
	static <T> T wildSubtype(Holder<? extends T> holder, T arg) {
		// error
		//holder.set(arg);

		T t = holder.get();
		System.out.println("wildSubtype t is " + t);
		return t;
	}

    // super是限制数据流入的
	static <T> void wildSupertype(Holder<? super T> holder, T arg) {
		holder.set(arg);
		// error
		//T t = holder.get();

		Object obj = holder.get();
		System.out.println("wildSupertype obj is " + obj);
	}

	public static void main (String[] args) {
		Long lng = 1L;
		Holder raw = new Holder<Long>();
		// or:
		//Holder raw = new Holder();
		Holder<Long> qualified = new Holder<Long>();
		Holder<?> unbounded = new Holder<Long>();
		Holder<? extends Long> bounded = new Holder<Long>();

		rawArgs(raw, lng);
		rawArgs(qualified, lng);
		rawArgs(unbounded, lng);
		rawArgs(bounded, lng);

		unboundedArg(raw, lng);
		unboundedArg(qualified, lng);
		unboundedArg(unbounded, lng);
		unboundedArg(bounded, lng);

		// warning
		//Object r1 = exact1(raw);
		Long r2 = exact1(qualified);
		Object r3 = exact1(unbounded);  // must return Object
		Long r4 = exact1(bounded);

		// warning
		//Long r5 = exact2(raw, lng);
		Long r6 = exact2(qualified, lng);
		// error
		//Long r7 = exact2(unbounded, lng);
		// error
		//Long r8 = exact2(bounded, lng);

		// warning
		//Long r9 = wildSubtype(raw, lng);
		Long r10 = wildSubtype(qualified, lng);
		Object r11 = wildSubtype(unbounded, lng);  // must return Object
		Long r12 = wildSubtype(bounded, lng);

		// warning
		//wildSupertype(raw, lng);
		wildSupertype(qualified, lng);
		// error
		//wildSupertype(unbounded, lng);
		// error
		//wildSupertype(bounded, lng);
	}
}

/*
rawArgs obj is null
rawArgs obj is null
rawArgs obj is null
rawArgs obj is null
unboundedArg obj is null
unboundedArg obj is null
unboundedArg obj is null
unboundedArg obj is null
exact1 t is null
exact1 t is null
exact1 t is null
exact2 t is 1
wildSubtype t is 1
wildSubtype t is null
wildSubtype t is null
wildSupertype obj is 1
*/
```

### 捕获转换

有一种情况特别需要使用<?>而不是原生类型。如果向一个使用<?>的方法传递原生类型，那么对编译器来说，可能会推断出实际的类型参数，使得这个方法可以回转并调用另一个使用这个确切类型的方法。

```
public class Demo {
	static <T> void f1(Holder<T> holder) {
		T t = holder.get();
		System.out.println(t.getClass().getSimpleName());
	}

	static void f2(Holder<?> holder) {
		f1(holder);    // 捕获转换
	}

    //@SuppressWarnings("unchecked")
	public static void main (String[] args) {
		Holder raw = new Holder<Integer>(1);
		//f1(raw);
		f2(raw);

		Holder rawBasic = new Holder();
		rawBasic.set(new Object());    // warning
		f2(rawBasic);                  // no warning

		Holder<?> wildcarded = new Holder<Double>(1.0);
		f2(wildcarded);
	}
}

/*
Integer
Object
Double
*/
```

捕获转换只有在这样的情况下可以工作：即在方法内部，你需要使用确切的类型。

*** ？？？？ ***


### 局限性

#### 基本类型都不能作为类型参数

[自动装箱/拆箱机制](https://blog.csdn.net/Caster_Saber/article/details/50950466)

#### 实现参数化接口
#### 转型和警告

使用带有泛型参数的转型或`instanceof`不会有任何效果。下面的容器在内部将各个值存储为`Object`，并在获取这些值时，再将它们转型回T。

```
import java.util.*;

class FixedSizeStack<T> {
	private int index = 0;
	private Object[] storage;

	public FixedSizeStack(int size) {
		storage = new Object[size];
	}

	public void push(T item) {
		storage[index++] = item;
	}

	@SuppressWarnings("unchecked")
	public T pop() {
		return (T)storage[--index];
	}
}

public class Demo {
	public static void main (String[] args) {
		FixedSizeStack<String> strings = new FixedSizeStack<String>(10);
		for (String s : "A B C D E F G H I J".split(" ")) {
			strings.push(s);
		}
		for (int i = 0; i < 10; i++) {
			String s = strings.pop();
			System.out.print(s + " ");
		}
		System.out.println("");
	}
}
```

#### 重载

由于擦除的原因，重载方法将产生相同的类型签名

```
class UseList<W, T> {
	void f(List<T> v) {}
	void f(List<W> u) {}
}
```

#### 基类劫持了接口

```
abstract class Animal implements Comparable<Animal> {}

class Dog extends Animal implements Comparable<Dog> {
		/** 无论CompareTo参数是Dog还是Animal，都不行 */
    @Override
    public int compareTo(Dog o) {
        return 0;
    }
}
```

### 自限定的类型

```
class SelfBounded<T extends SelfBounded<T>> {}
```

SelfBounded类接受泛型参数T，而T由一个边界类限定，这个边界就是拥有T作为其参数的SelfBounded。这儿强调的是当extends关键字用于边界与用来创建子类明显是不同的

```
class BasicHolder<T> {
	T element;
	void set(T arg) { element = arg; }
	T get() { return element; }
	void f() {
		System.out.println(element.getClass().getSimpleName());
	}
}

// CGR: 古怪的循环泛型
// 其本质：基类用导出类替代其参数
// 这意味着泛型基类变成了一种所有导出类的公共功能的模版
// 但是这些功能对于其所有参数和返回值，将使用导出类型。
class Subtype extends BasicHolder<Subtype> {}

public class Demo {
	public static void main (String[] args) {
		Subtype st1 = new Subtype();
		Subtype st2 = new Subtype();
		st1.set(st2);

		Subtype st3 = st1.get();
		st1.f();
	}
}
```

#### 自限定

```
class SelfBounded<T extends SelfBounded<T>> {
	T element;
	SelfBounded<T> set(T arg) {
		element = arg;
		return this;
	}
	T get() { return element; }
}

class A extends SelfBounded<A> {}
class B extends SelfBounded<A> {} // also ok

class C extends SelfBounded<C> {
	C setAndGet(C arg) {
		set(arg);
		return get();
	}
}

class F extends SelfBounded {}

public class Demo {
	public static void main (String[] args) {
		A a = new A();
		a.set(new A());
		a = a.set(new A()).get();
		a = a.get();

		C c = new C();
		c = c.setAndGet(new C());

		B b = new B();
		// error
		//b.set(new B());
		// error
		//b = b.get();

		F f = new F();
		// warning
		//f.set(new F());
		// error
		//f = f.get();
	}
}
```

自限定保证了类型参数必须与正在被定义的类相同

也可以用于方法

```
class SelfBounded<T extends SelfBounded<T>> {
	T element;
	SelfBounded<T> set(T arg) {
		element = arg;
		return this;
	}
	T get() { return element; }
}

class A extends SelfBounded<A> {}

public class Demo {
	static <T extends SelfBounded<T>> T f(T arg) {
		return arg.set(arg).get();
	}


	public static void main (String[] args) {
		A a = f(new A());
	}
}
```

#### 参数协变

自限定类型的价值在于它们可以产生协变参数类型--方法参数类型会随子类而变化。

？？？？


### 动态类型安全

还需要吗？

### 异常

由于擦除的原因，将泛型应用于异常是非常受限的。catch语句不能捕获泛型类型的异常，因为在编译期和运行时都必须知道异常的确切类型。？？？

但是，类型参数可能会在一个方法的throws子句中用到。这使得你可以编写随检查型异常的类型而发生变化的泛型代码。

```
import java.util.*;

interface Processor<T, E extends Exception> {
	void process(List<T> resultCollector) throws E;
}

class ProccessRunner<T, E extends Exception> extends ArrayList<Processor<T, E>> {
	List<T> processAll() throws E {
		List<T> resultCollector = new ArrayList<T>();
		for (Processor<T, E> processor : this) {
			processor.process(resultCollector);
		}
		return resultCollector;
	}
}

class Failure1 extends Exception {}

class Processor1 implements Processor<String, Failure1> {
	static int count = 3;

	public void process(List<String> resultCollector) throws Failure1 {
		if (count-- > 1) {
			resultCollector.add("Help");
		} else {
			resultCollector.add("Ho");
		}
		if (count < 0) {
			throw new Failure1();
		}
	}
}

class Failure2 extends Exception {}

class Processor2 implements Processor<Integer, Failure2> {
	static int count = 2;

	public void process(List<Integer> resultCollector) throws Failure2 {
		if (count-- > 1) {
			resultCollector.add(11);
		} else {
			resultCollector.add(22);
		}
		if (count < 0) {
			throw new Failure2();
		}
	}
}

public class Demo {
	public static void main (String[] args) {
		ProccessRunner<String, Failure1> runner = new ProccessRunner<String, Failure1>();
		for (int i = 0; i < 3; i++) {
			runner.add(new Processor1());
		}
		try {
			System.out.println(runner.processAll());
		} catch (Failure1 e) {
			System.out.println(e);
		}

		ProccessRunner<Integer, Failure2> runner2 = new ProccessRunner<Integer, Failure2>();
		for (int i = 0; i < 3; i++) {
			runner2.add(new Processor2());
		}
		try {
			System.out.println(runner2.processAll());
		} catch (Failure2 e) {
			System.out.println(e);
		}

	}
}
```

### 混型

混型：混合多个类的能力，以产生一个可以表示混型中所有类型的类。这往往是你最后的手段，它将使组装多个类变得简单易行。

混型的价值之一是它们可以将特性和行为一致地应用于多个类之上。如果想在混型类中修改某些东西，作为一种意外的好处，这些修改将会应用于混型所应用的所有类型之上。正由于此，混型有一点面向切面编程的味道。

#### C++的混型

```
#include <string>
#include <ctime>
#include <iostream>

using namespace std;

template<class T> class TimeStamped : public T
{
public:
	TimeStamped() { timeStamp = time(0); }
	~TimeStamped() {}

	long getStamp() { return timeStamp; }

private:
	long timeStamp;
};

template<class T> class SerialNumbered : public T
{
public:
	SerialNumbered() { serialNumber = counter++; }
	~SerialNumbered() {}

	long getSerialNumber() { return serialNumber; }

private:
	long serialNumber;
	static long counter;
};

template<class T> long SerialNumbered<T>::counter = 1;

class Basic
{
public:
	Basic() {}
	~Basic() {}

	void set(string val) { value = val; }
	string get() { return value; }

private:
	string value;
};

int main() {
	TimeStamped< SerialNumbered<Basic> > mixin1;
	TimeStamped< SerialNumbered<Basic> > mixin2;

	mixin1.set("test string 1");
	mixin2.set("test string 2");

	cout << mixin1.get() << " " << mixin1.getStamp() << " " << mixin1.getSerialNumber() << endl;
	cout << mixin2.get() << " " << mixin2.getStamp() << " " << mixin2.getSerialNumber() << endl;

	return 0;
}

/*
test string 1 1568679047 1
test string 2 1568679047 2
*/
```

*** 怎么就和AOP有关啦？？？？ ***

#### JAVA的混型

通过接口和组合的方式实现

```
import java.util.*;

interface TimeStamped {
	long getStamp();
}

class TimeStampedImp implements TimeStamped {
	private final long timeStamp;

	public TimeStampedImp() {
		timeStamp = new Date().getTime();
	}

	public long getStamp() { return timeStamp; }
}

interface SerialNumbered {
	long getSerialNumber();
}

class SerialNumberedImp implements SerialNumbered {
	private static long counter = 1;
	private final long serialNumber = counter++;
	public long getSerialNumber() { return serialNumber; }
}

interface Basic {
	void set(String val);
	String get();
}

class BasicImp implements Basic {
	private String value;

	public void set(String val) { value = val; }
	public String get() { return value; }
}

class Mixin extends BasicImp implements TimeStamped, SerialNumbered {
	private TimeStamped timeStamp = new TimeStampedImp();
	private SerialNumbered serialNumber = new SerialNumberedImp();

	public long getStamp() { return timeStamp.getStamp(); }
	public long getSerialNumber() { return serialNumber.getSerialNumber(); }
}


public class Demo {
	public static void main (String[] args) {
		Mixin mixin1 = new Mixin();
		Mixin mixin2 = new Mixin();
		mixin1.set("test string 1");
    	mixin2.set("test string 2");
		System.out.println(mixin1.get() + " " + mixin1.getStamp() +  " " + mixin1.getSerialNumber());
    	System.out.println(mixin2.get() + " " + mixin2.getStamp() +  " " + mixin2.getSerialNumber());
	}
}


/*
test string 1 1568680035099 1
test string 2 1568680035099 2
*/
```

#### 装饰器模式的混型

装饰器指定包装在最初的对象的周围的所有对象都具有相同的基本接口。某些事物是可装饰的，可以通过将其他类包装在这个可装饰对象的四周，来将功能分层。

```
import java.util.*;

class Basic {
	private String value;

	public void set(String val) { value = val; }
	public String get() { return value; }
}

class Decorator extends Basic {
	protected Basic basic;

	public Decorator(Basic basic) { this.basic = basic; }

	public void set(String val) { basic.set(val); }
	public String get() { return basic.get(); }
}

class TimeStamped extends Decorator {
	private final long timeStamp;

	public TimeStamped(Basic basic) {
		super(basic);
		timeStamp = new Date().getTime();
	}

	public long getStamp() { return timeStamp; }
}

class SerialNumbered extends Decorator {
	private static long counter = 1;
	private final long serialNumber = counter++;

	public SerialNumbered(Basic basic) { super(basic); }
	public long getSerialNumber() { return serialNumber; }
}


public class Demo {
	public static void main (String[] args) {
		TimeStamped t1 = new TimeStamped(new Basic());
		TimeStamped t2 = new TimeStamped(new SerialNumbered(new Basic()));

		t1.set("test string t1");
		t2.set("test string t2");

		System.out.println(t1.get() + " " + t1.getStamp());
		System.out.println(t2.get() + " " + t2.getStamp());

		// not available
		// t2.getSerialNumber()


		System.out.println("---------");

		SerialNumbered s1 = new SerialNumbered(new Basic());
		SerialNumbered s2 = new SerialNumbered(new TimeStamped(new Basic()));

		s1.set("test string s1");
		s2.set("test string s2");

		System.out.println(s1.get() + " " + s1.getSerialNumber());
		System.out.println(s2.get() + " " + s2.getSerialNumber());

		// not available
		// s2.getStamp()
	}
}


/*
test string t1 1568851872970
test string t2 1568851872970
---------
test string s1 2
test string s2 3
*/
```

### 动态代理


```
public class TwoTuple<A, B> {
	public final A first;
	public final B second;

	public TwoTuple(A a, B b) {
		first = a;
		second = b;
	}

	public String toString() {
		return "(" + first + "," + second + ")";
	}
}
```

```
public class Tuple {
	public static <A, B> TwoTuple<A, B> tuple(A a, B b) {
		return new TwoTuple<A, B>(a, b);
	}
}
```

```
import java.util.*;
import java.lang.reflect.*;

interface TimeStamped {
	long getStamp();
}

class TimeStampedImp implements TimeStamped {
	private final long timeStamp;

	public TimeStampedImp() {
		timeStamp = new Date().getTime();
	}

	public long getStamp() { return timeStamp; }
}

interface SerialNumbered {
	long getSerialNumber();
}

class SerialNumberedImp implements SerialNumbered {
	private static long counter = 1;
	private final long serialNumber = counter++;
	public long getSerialNumber() { return serialNumber; }
}

interface Basic {
	void set(String val);
	String get();
}

class BasicImp implements Basic {
	private String value;

	public void set(String val) { value = val; }
	public String get() { return value; }
}

@SuppressWarnings("unchecked")
class MixinProxy implements InvocationHandler {
	Map<String, Object> delegatesByMethod;

	public MixinProxy(TwoTuple<Object, Class<?>>... pairs) {
		delegatesByMethod = new HashMap<String, Object>();
		for(TwoTuple<Object, Class<?>> pair : pairs) {
			for(Method method : pair.second.getMethods()) {
				String methodName = method.getName();
				if (!delegatesByMethod.containsKey(methodName)) {
					delegatesByMethod.put(methodName, pair.first);
				}
			}
		}
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		String methodName = method.getName();
		Object delegate = delegatesByMethod.get(methodName);
		return method.invoke(delegate, args);
	}

	public static Object newInstance(TwoTuple... pairs) {
		Class[] interfaces = new Class[pairs.length];
		for(int i = 0; i < pairs.length; i++) {
			interfaces[i] = (Class)pairs[i].second;
		}
		ClassLoader cl = pairs[0].first.getClass().getClassLoader();
		return Proxy.newProxyInstance(cl, interfaces, new MixinProxy(pairs));
	}
}



public class Demo {
	public static void main (String[] args) {
		Object mixin = MixinProxy.newInstance(
			Tuple.tuple(new BasicImp(), Basic.class),
			Tuple.tuple(new TimeStampedImp(), TimeStamped.class),
			Tuple.tuple(new SerialNumberedImp(),SerialNumbered.class)
		);

		Basic b = (Basic)mixin;
    	TimeStamped t = (TimeStamped)mixin;
    	SerialNumbered s = (SerialNumbered)mixin;
    	b.set("Hello");
    	System.out.println(b.get());
    	System.out.println(t.getStamp());
    	System.out.println(s.getSerialNumber());
	}
}


/*
Hello
1568853861647
1
*/
```

### 潜在类型机制

**因为擦除要求指定可能会用到的泛型类型的边界，以安全地调用代码中的泛型对象上的具体方法。这是对“泛化”概念的一种明显的限制，因为必须限制你的泛型类型，使它们继续自特定的类，或者实现特定的接口。在某些情况下，你最终可能会使用普通类或普通接口，因为限定边界的泛型可能会和指定类或接口没有任何区别。** (怎么理解？？)

某些编程语言提供的一种解决方案称为潜在类型机制或结构化类型机制，而更古怪的术语称为鸭子类型机制，即“如果它走起来像鸭子，并且叫起来也像鸭子，那么你就可以将它当作鸭子。”

潜在类型机制是一种代码组织和复用机制。

#### python代码

```
class Dog:
	def speak(self):
		print "Dog speak"
	def sit(self):
		print "Dog Sit"
	def reproduce(self):
		pass


class Robot:
	def speak(self):
		print "Robot Speak"
	def sit(self):
	    print "Robot Sit!"
	def oilChange(self):
		pass

def perform(anything):
	anything.speak()
	anything.sit()

a = Dog();
b = Robot();
perform(a);
perform(b);

/*
Dog speak
Dog Sit
Robot Speak
Robot Sit
*/
```

#### C++代码

```
#include <iostream>

using namespace std;

class Dog {
public:
	void speak() { cout << "Dog Speak" << endl; }
	void sit() { cout << "Dog Sit" << endl; }
	void reproduce() {}
};

class Robot {
public:
	void speak() { cout << "Robot Speak" << endl; }
	void sit() { cout << "Robot Sit" << endl; }
	void oilChange() {}
};

template<class T> void perform(T anything) {
	anything.speak();
	anything.sit();
}

int main() {
	Dog d;
	Robot r;
	perform(d);
	perform(r);
}
```

#### Java泛型实现

```
interface Perform {
	void speak();
	void sit();
}

class Dog implements Perform {
	public void speak() {
		System.out.println("Dog Speak");
	}

	public void sit() {
		System.out.println("Dog Sit");
	}
}

class Robot implements Perform {
	public void speak() {
		System.out.println("Robot Speak");
	}

	public void sit() {
		System.out.println("Robot Sit");
	}
}


public class Demo {
	static <T extends Perform> void f(T t) {
		t.speak();
		t.sit();
	}


	public static void main (String[] args) {
		Dog dog = new Dog();
		f(dog);
		Robot robot = new Robot();
		f(robot);
	}
}

/*
Dog Speak
Dog Sit
Robot Speak
Robot Sit
*/
```

### 对缺乏潜在类型机制的补偿

#### Java反射实现

```
import java.lang.reflect.*;

interface Perform {
	void speak();
	void sit();
}

class Dog implements Perform {
	public void speak() {
		System.out.println("Dog Speak");
	}

	public void sit() {
		System.out.println("Dog Sit");
	}
}

class Robot implements Perform {
	public void speak() {
		System.out.println("Robot Speak");
	}

	public void sit() {
		System.out.println("Robot Sit");
	}
}

class A {
	void f() {};
	void g() {};
}


public class Demo {
	static void f(Object object) {
		Class<?> o = object.getClass();
		// 为什么要最外面包一层try呢？
		try {
			try {
				Method speak = o.getMethod("speak");
				speak.invoke(object);
			} catch (NoSuchMethodException e) {
				System.out.println("no speak method");
			}
			try {
				Method sit = o.getMethod("sit");
				sit.invoke(object);
			} catch (NoSuchMethodException e) {
				System.out.println("no sit method");
			}
		} catch (Exception e) {
			throw new RuntimeException(o.toString(), e);
		}
	}


	public static void main (String[] args) {
		Dog dog = new Dog();
		f(dog);
		Robot robot = new Robot();
		f(robot);

		A a = new A();
		f(a);
	}
}

/*
Dog Speak
Dog Sit
Robot Speak
Robot Sit
no speak method
no sit method
*/
```

#### 将一个方法应用于序列

```
import java.util.*;

public class SimpleQueue<T> implements Iterable<T> {
  private LinkedList<T> storage = new LinkedList<T>();
  public void add(T t) { storage.offer(t); }
  public T get() { return storage.poll(); }
  public Iterator<T> iterator() {
    return storage.iterator();
  }
}
```

```
import java.lang.reflect.*;
import java.util.*;

class Apply {
	public static <T, S extends Iterable<? extends T>> void apply(S seq, Method f, Object... args) {
		try {
			for (T t : seq) {
				f.invoke(t, args);
			} 
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
}

class Shape {
	public void rotate() {
		System.out.println(this + " rotate");
	}
	public void resize(int newSize) {
		System.out.println(this + " resize " + newSize);
	}
}

class Square extends Shape {}

class FilledList<T> extends ArrayList<T> {
	public FilledList(Class<? extends T> type, int size) {
		try {
			for(int i = 0; i < size; i++)
				add(type.newInstance());
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
}

public class Demo {
	public static void main (String[] args) throws Exception {
		List<Shape> shapes = new ArrayList<Shape>();
		for (int i = 0; i < 2; i++) {
			shapes.add(new Shape());
		}
		Apply.apply(shapes, Shape.class.getMethod("rotate"));
		Apply.apply(shapes, Shape.class.getMethod("resize", int.class), 5);

		System.out.println("----------");

		List<Square> squares = new ArrayList<Square>();
		for (int i = 0; i < 2; i++) {
			squares.add(new Square());
		}
		Apply.apply(squares, Shape.class.getMethod("rotate"));
		Apply.apply(squares, Shape.class.getMethod("resize", int.class), 5);

		System.out.println("----------");

		Apply.apply(new FilledList<Shape>(Shape.class, 2), Shape.class.getMethod("rotate"));
		Apply.apply(new FilledList<Shape>(Square.class, 2), Shape.class.getMethod("rotate"));

		System.out.println("----------");

		SimpleQueue<Shape> shapeQ = new SimpleQueue<Shape>();
		for (int i = 0; i < 2; i++) {
			shapeQ.add(new Shape());
			shapeQ.add(new Square());
		}
		Apply.apply(shapeQ, Shape.class.getMethod("rotate"));
	}
}

/*
Shape@4e25154f rotate
Shape@70dea4e rotate
Shape@4e25154f resize 5
Shape@70dea4e resize 5
----------
Square@5c647e05 rotate
Square@33909752 rotate
Square@5c647e05 resize 5
Square@33909752 resize 5
----------
Shape@55f96302 rotate
Shape@3d4eac69 rotate
Square@42a57993 rotate
Square@75b84c92 rotate
----------
Shape@6bc7c054 rotate
Square@232204a1 rotate
Shape@4aa298b7 rotate
Square@7d4991ad rotate
*/
```

#### 当你并未碰巧拥有正确的接口时

```
import java.util.*;

class Fill {
	public static <T> void fill(Collection<T> collection, Class<? extends T> classToken, int size) {
		for (int i = 0; i < size; i++) {
			try {
				collection.add(classToken.newInstance());
			} catch (Exception e) {
				throw new RuntimeException(e);
			}
		}
	}
}

class Contract {
	private static long counter = 0;
	private final long id = counter++;
	public String toString() {
		return getClass().getName() + " " + id;
	}
}

class TitleTransfer extends Contract {}

public class Demo {
	public static void main (String[] args) throws Exception {
		List<Contract> contracts = new ArrayList<Contract>();
		Fill.fill(contracts, Contract.class, 2);
		Fill.fill(contracts, TitleTransfer.class, 2);

		for(Contract c : contracts) {
			System.out.println(c);
		}

		SimpleQueue<Contract> contractQueue = new SimpleQueue<Contract>();
		// 参数不匹配; SimpleQueue<Contract>无法转换为Collection<T>
		//Fill.fill(contractQueue, Contract.class, 3);
	}
}

/*
Contract 0
Contract 1
TitleTransfer 2
TitleTransfer 3
*/
```

#### 用适配器仿真潜在类型机制

潜在类型机制机制在这里实现什么？它意味着你可以编写代码声明：“我不关心我在这里使用的类型，只要它具有这些方法即可。”实际上，潜在类型机制创建了一个包含所需方法的隐式接口。

从我们拥有的接口中编写代码来产生我们需要的接口，这是适配器设计模式的一个典型示例。我们可以使用适配器来适配已有的接口，以产生想要的接口。

```
import java.util.*;

interface Addable<T> {
	void add(T t);
}

class Contract {
	private static long counter = 0;
	private final long id = counter++;
	public String toString() {
		return getClass().getName() + " " + id;
	}
}

class TitleTransfer extends Contract {}

class Fill {
	public static <T> void fill(Addable<T> addable, Class<? extends T> classToken, int size) {
		for (int i = 0; i < size; i++) {
			try {
				addable.add(classToken.newInstance());
			} catch (Exception e) {
				throw new RuntimeException(e);
			}
		}
	}
}

class AddableCollectionAdapter<T> implements Addable<T> {
	private Collection<T> c;
	public AddableCollectionAdapter(Collection<T> c) {
		this.c = c;
	}

	public void add(T item) {
		c.add(item);
	}
}

class Adapter {
	public static <T> Addable<T> collectionAdapter(Collection<T> c) {
		return new AddableCollectionAdapter<T>(c);
	}
}

class AddableSimpleQueue<T> extends SimpleQueue<T> implements Addable<T> {
	public void add(T item) {
		super.add(item);
	}
}

public class Demo {
	public static void main (String[] args) throws Exception {
		List<Contract> contracts = new ArrayList<Contract>();
		Fill.fill(Adapter.collectionAdapter(contracts), Contract.class, 2);
		Fill.fill(Adapter.collectionAdapter(contracts), TitleTransfer.class, 2);
		for(Contract c : contracts) {
			System.out.println(c);
		}

		System.out.println("----------");

		AddableSimpleQueue<Contract> queue = new AddableSimpleQueue<Contract>();
		Fill.fill(queue, Contract.class, 2);
		Fill.fill(queue, TitleTransfer.class, 2);
		for(Contract c : queue) {
			System.out.println(c);
		}
	}
}

/*
Contract 0
Contract 1
TitleTransfer 2
TitleTransfer 3
----------
Contract 4
Contract 5
TitleTransfer 6
TitleTransfer 7
*/
```

### 将函数对象用作策略

函数对象就是在某种程度上行为像函数的对象 —— 一般地，会有一个相关的方法。


```
import java.math.*;
import java.util.concurrent.atomic.*;
import java.util.*;

interface Combiner<T> {
	T combine(T x, T y);
}

interface UnaryFunction<R, T> {
	R function(T x);
}

interface Collector<T> extends UnaryFunction<T, T> {
	T result();
}

interface UnaryPredicate<T> {
	boolean test(T x);
}

public class Demo {
	public static <T> T reduce(Iterable<T> seq, Combiner<T> combiner) {
		Iterator<T> it = seq.iterator();
		if (it.hasNext()) {
			T result = it.next();
			while (it.hasNext()) {
				result = combiner.combine(result, it.next());
			}
			return result;
		}
		return null;
	}

	public static <T> Collector<T> forEach(Iterable<T> seq, Collector<T> func) {
		for (T t : seq) {
			func.function(t);
		}
		return func;
	}

	public static <R, T> List<R> transform(Iterable<T> seq, UnaryFunction<R, T> func) {
		List<R> result = new ArrayList<R>();
		for (T t : seq) {
			result.add(func.function(t));
		}
		return result;
	}

	public static <T> List<T> filter(Iterable<T> seq, UnaryPredicate<T> pred) {
		List<T> result = new ArrayList<T>();
		for (T t : seq) {
			if (pred.test(t)) {
				result.add(t);
			}
		}
		return result;
	}

	static class IntegerAdder implements Combiner<Integer> {
		public Integer combine(Integer x, Integer y) {
			return x + y;
		}
	}

	static class IntegerSubtracter implements Combiner<Integer> {
		public Integer combine(Integer x, Integer y) {
			return x - y;
		}
	}

	static class BigDecimalAdder implements Combiner<BigDecimal> {
		public BigDecimal combine(BigDecimal x, BigDecimal y) {
			return x.add(y);
		}
	}

	static class BigIntegerAdder implements Combiner<BigInteger> {
		public BigInteger combine(BigInteger x, BigInteger y) {
			return x.add(y);
		}
	}

	static class AtomicLongAdder implements Combiner<AtomicLong> {
		public AtomicLong combine(AtomicLong x, AtomicLong y) {
			return new AtomicLong(x.addAndGet(y.get()));
		}
	}

	static class BigDecimalUlp implements UnaryFunction<BigDecimal, BigDecimal> {
		public BigDecimal function(BigDecimal x) {
			return x.ulp();
		}
	}

	static class GreaterThan<T extends Comparable<T>> implements UnaryPredicate<T> {
		private T bound;
		public GreaterThan(T bound) {
			this.bound = bound;
		}

		public boolean test(T x) {
			return x.compareTo(bound) > 0;
		}
	}

	static class MultiplyingIntegerCollector implements Collector<Integer> {
		private Integer val = 1;
		public Integer function(Integer x) {
			val *= x;
			return val;
		}
		public Integer result() {
			return val;
		}
	}

	public static void main (String[] args) throws Exception {
		List<Integer> li = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
		Integer result = reduce(li, new IntegerAdder());
		System.out.println("adder result = " + result);

		result = reduce(li, new IntegerSubtracter());
		System.out.println("subtracter result = " + result);

		List<Integer> lFilter = filter(li, new GreaterThan<Integer>(4));
		System.out.println("lFilter = " + lFilter);

		Integer i1 = forEach(li, new MultiplyingIntegerCollector()).result();
		System.out.println("i1 = " + i1);

		Integer i2 = forEach(lFilter, new MultiplyingIntegerCollector()).result();
		System.out.println("i2 = " + i2);

		MathContext mc = new MathContext(7);
		List<BigDecimal> lbd = Arrays.asList(
			new BigDecimal(1.1, mc),
			new BigDecimal(2.2, mc),
			new BigDecimal(3.3, mc),
			new BigDecimal(4.4, mc)
		);
		BigDecimal rbd = reduce(lbd, new BigDecimalAdder());
		System.out.println("rbd = " + rbd);

		List<BigDecimal> bFilter = filter(lbd, new GreaterThan<BigDecimal>(new BigDecimal(3)));
		System.out.println("bFilter = " + bFilter);

		List<BigInteger> lbi = new ArrayList<BigInteger>();
		BigInteger bi = BigInteger.valueOf(11);
		for (int i = 0; i < 11; i++) {
			lbi.add(bi);
			bi = bi.nextProbablePrime();
		}
		System.out.println("lbi = " + lbi);

		BigInteger rbi = reduce(lbi, new BigIntegerAdder());
		System.out.println("rbi = " + rbi);
		System.out.println(rbi.isProbablePrime(5));

		List<AtomicLong> lal = Arrays.asList(
			new AtomicLong(11),
			new AtomicLong(47),
			new AtomicLong(74),
			new AtomicLong(133)
		);
		AtomicLong ral = reduce(lal, new AtomicLongAdder());
		System.out.println("ral = " + ral);

		List lTransform = transform(lbd, new BigDecimalUlp());
		System.out.println("lTransform = " + lTransform);
	}
}

/*
adder result = 28
subtracter result = -26
lFilter =[5, 6, 7]
i1 =5040
i2 = 210
rbd = 11.000000
bFilter = [3.300000, 4.400000]
lbi = [11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
rbi = 311
true
ral = 265
lTransform = [0.000001, 0.000001, 0.000001, 0.000001]
*/
```

















































