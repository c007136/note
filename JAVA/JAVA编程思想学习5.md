# JAVA编程思想学习五
> 第13章 ～ 第14章

## 第13章 字符串

String对象是不可变的。每一个看起来会修改String值的方法，实际上都是创建了一个全新的String对象。

```
public class Demo {
	public static String upcase(String s) {
		return s.toUpperCase();
	}

	public static void main (String[] args) {
		String q = "howdy";
		System.out.println(q);
		String qq = upcase(q);
		System.out.println(qq);
		System.out.println(q);
	}
}
```

### 无意识的递归

```
import java.util.*;

public class Demo {
	public String toString() {
		// 会无限递归
		return "InfiniteRecursion address: " + this + "\n";
		//return "InfiniteRecursion address: " + super.toString() + "\n";
	}

	public static void main (String[] args) {
		List<Demo> v = new ArrayList<Demo>();
		for (int i = 0; i < 10; i++) {
			v.add(new Demo());
		}
		System.out.println(v);
	}
}
```

## 第14章 类型信息

**运行时类型信息使得你可以在程序运行时发现和使用类型信息**

1. RTTI：在编译时已经知道了所有的类型。
2. 发射机制：在运行时发现和使用类的信息。

RTTI的含义：在运行时，识别一个对象的类型。多态的概念。

### Class对象

Class对象就是用来创建类的所有的“常规”对象的，类是程序的一部分，每个类都有一个Class对象。换言之，每当编写并且编译了一个新类，就会产生一个Class对象（更恰当地说，是被保存在一个同名的.class文件中）。为了生成这个类的对象，运行这个程序的Java虚拟机（JVM）将使用被称为“类加载器”的子系统。

所有的类都是在对其第一次使用时，动态加载到JVM中的。当程序创建第一个对类的静态成员的引用时，就会加载这个类。这个证明构造器也是类的静态方法，即使在构造器之前并没有static关键字。因此，使用new操作符创建类的新对象也会被当做对类的静态成员的引用。

类加载器首先检查这个类的Class对象是否已经加载，如果尚未加载，默认的类加载器就会根据类查找.class文件。在这个类的字节码被加载时，它们会接受验证，以确保其没有被破坏，并且不包含不良Java代码

一旦某个类的Class对象被载入内存，它就被用来创建这个类的所有对象。

Java使用Class对象来执行其RTTI

下面的程序，证明Java Class是懒加载的。

```

class A {
	static { System.out.println("A Loading"); }
}

class B {
	static { System.out.println("B Loading"); }
}

class C {
	static { System.out.println("C Loading"); }
}

public class Demo {
	public static void main (String[] args) {
		System.out.println("inside main");
		new A();
		System.out.println("----------");
		try {
			Class.forName("B");
		} catch (ClassNotFoundException e) {
			System.out.println("Class B not found");
		}
		System.out.println("----------");
		new C();
		System.out.println("----------");
	}
}
```

### 类字面常量

```
A.class
```

这样做不仅更简单，而且更安全，因为它在编译时就会受到检查。

对于基本数据类型的包装器类，还有一个标准字段TYPE。TYPE是一个引用，指向对应的基本数据类型的Class对象，

例如：
```
boolean.class   ---   Boolean.TYPE
short.class   ---   Short.TYPE
...
```

建议使用.class的形式，以保持与普通类的一致性。

有一点很有趣，当使用".class"来创建对Class对象的引用时，不会自动初始化该Class对象。为了使用类而做的准备工作实际包含三个步骤：

1. 加载。这是由类加载器执行的。该步骤将查找字节码，并从这些字节码中创建一个Class对象。
2. 链接。在链接阶段将验证类中的字节码，为静态域分配存储空间，并且如果必需的话，将解析这个类创建的对其他类的所有引用。
3. 初始化。如果该类具有超类，则对其初始化，执行静态初始化器和静态初始化块。

```

import java.util.*;

class A {
	static final int a = 47;
	static final int a2 = Demo.rand.nextInt(1000);
	static { System.out.println("A Loading"); }
}

class B {
	static final int b = 147;
	static int b2 = 247;
	static { System.out.println("B Loading"); }
}

class C {
	static final int c = 74;
	static { System.out.println("C Loading"); }
}

public class Demo {
	public static Random rand = new Random(47);

	public static void main (String[] args) {
		Class aa = A.class;
		System.out.println("--- after A.class ---");
		System.out.println(A.a);
		System.out.println(A.a2);
		System.out.println("--- before B.b ---");
		// 编译器常量，不需要初始化
		//System.out.println(B.b);
		System.out.println(B.b2);
		System.out.println("--- after B.b ---");
		try {
			Class cc = Class.forName("C");
		} catch (ClassNotFoundException e) {
			System.out.println("Class C not found");
		}
		System.out.println("--- after forName C ---");
		System.out.println(C.c);
		System.out.println("--- after C.c ---");
	}
}
```

#### 参考链接
[Java 深度历险（二）——Java 类的加载、链接和初始化](https://www.infoq.cn/article/cf-Java-class-loader/)

### 泛化的Class引用
Class引用总是指向某个Class对象，它可以制造类的实例，并包含可作用于这些实例的所有方法代码。它还包含该类的静态成员，因此，Class引用表示的就是它所指向的对象的确切类型，而该对象便是Class类的一个对象。

Class<?>的好处是它表示你并非是碰巧或者由于疏忽，而使用了一个非具体的类引用，你就是选择了非具体的版本

```
public class Demo {
	public static void main (String[] args) {
		Class<? extends Number> cls = int.class;
		cls = double.class;
		cls = Number.class;
	}
}
```

#### newInstance

```
class A {}

class B extends A {}

public class Demo {
	public static void main (String[] args) {
		Class<B> bClass = B.class;
		try {
			B b = bClass.newInstance();
		} catch (Exception e) {
			System.out.println("B newInstance exception " + e);
		}

        Class<? super B> aClass = bClass.getSuperclass();
        // 不能编译
        // 只能像上面一样<? super B>
		//Class<A> aClass = bClass.getSuperclass();
		System.out.println(aClass);
		try {
			Object a = aClass.newInstance();
			//A a = aClass.newInstance();
		} catch (Exception e) {
			System.out.println("A newInstance exception " + e);
		}
	}
}
```

#### 转型语法 cast
```
class A {}

class B extends A {}

public class Demo {
	public static void main (String[] args) {
		A a = new B();
		Class<B> bClass = B.class;
		// cast方法接受参数对象，并将其转型为Class引用的类型
		B b = bClass.cast(a);
		// 上句代码的功效
		//b = (B)a;
	}
}
```

#### 类型转换前先做检查

```
if (x instanceof Dog) {
	((Dog)x).bark();
}
```

例子：

```
class A {}

class B extends A {}

public class Demo {
	static void test(Object x) {
		System.out.println("Testing x of type " + x.getClass());
		System.out.println("x instanceof A " + (x instanceof A));
		System.out.println("x instanceof B " + (x instanceof B));
		System.out.println("A.isInstance(x) " + A.class.isInstance(x));
		System.out.println("B.isInstance(x) " + B.class.isInstance(x));
		System.out.println("x.getClass() == A.class " + (x.getClass() == A.class));
		System.out.println("x.getClass() == B.class " + (x.getClass() == B.class));
		System.out.println("x.getClass().equals(A.class) " + (x.getClass().equals(A.class)));
		System.out.println("x.getClass().equals(B.class) " + (x.getClass().equals(B.class)));
	}


	public static void main (String[] args) {
		test(new A());
		test(new B());
	}
}

/*
Testing x of type class A
x instanceof A true
x instanceof B false
A.isInstance(x) true
B.isInstance(x) false
x.getClass() == A.class true
x.getClass() == B.class false
x.getClass().equals(A.class) true
x.getClass().equals(B.class) false
Testing x of type class B
x instanceof A true
x instanceof B true
A.isInstance(x) true
B.isInstance(x) true
x.getClass() == A.class false
x.getClass() == B.class true
x.getClass().equals(A.class) false
x.getClass().equals(B.class) true
*/
```

### 反射

RTTI的限制：对象的类型在编译时必须已知，这样才能使用RTTI识别它。

```
import java.lang.reflect.*;

public class Demo {
	public int a;
	public static final int b = 10;

	public static void main (String[] args) {
		if (args.length < 1) {
			System.out.println("worng");
			System.exit(0);
		}

		System.out.println("args : " + args[0]);

		try {
			Class<?> c = Class.forName(args[0]);

			Method[] methods = c.getMethods();
	        for (Method method : methods) {
	        	System.out.println("method : " + method);
	        }
	        Constructor[] ctors = c.getConstructors();
	        for(Constructor ctor : ctors) {
	        	System.out.println("ctor : " + ctor);
	        }
		} catch (ClassNotFoundException e) {
			System.out.println("ClassNotFoundException : " + e);
		}
	}
}

/*
java Demo Demo的结果是：

args : Demo
method : public static void Demo.main(java.lang.String[])
method : public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
method : public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
method : public final void java.lang.Object.wait() throws java.lang.InterruptedException
method : public boolean java.lang.Object.equals(java.lang.Object)
method : public java.lang.String java.lang.Object.toString()
method : public native int java.lang.Object.hashCode()
method : public final native java.lang.Class java.lang.Object.getClass()
method : public final native void java.lang.Object.notify()
method : public final native void java.lang.Object.notifyAll()
ctor : public Demo()
*/

```

#### 动态代理

利用静态方法`Proxy.newProxyInstance`可以创建动态代理。

```
import java.lang.reflect.*;

interface I {
	public void f();
}

class A implements I {
	public void f() {
		System.out.println("A f()");
	}
}

class DynamicProxy implements InvocationHandler {
	private Object p;

	public DynamicProxy() {

	}

	public DynamicProxy(Object p) {
		this.p = p;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("**** proxy: " + proxy.getClass() + ", method: " + method + ", args: " + args);
	    if(args != null) {
	    	for(Object arg : args) {
				System.out.println("  " + arg);
			}
	    }
	    return method.invoke(p, args);
	} 
}

public class Demo {
	public static void consumer(I i) {
		i.f();
	} 

	public static void main (String[] args) {
		A a = new A();
		consumer(a);

		I proxy = (I)Proxy.newProxyInstance(I.class.getClassLoader(), new Class[]{ I.class }, new DynamicProxy(a));
		consumer(proxy);
	}
}

/*
A f()
**** proxy: class $Proxy0, method: public abstract void I.f(), args: null
A f()
*/

```

### 空对象

作用？

### 接口与类型信息

接口并非对耦合性的一种无懈可击的保障。

```
interface I {
	public void f();
}

class A implements I {
	public void f() {
		System.out.println("A f()");
	}

	public void g() {
		System.out.println("B g()");
	}
}

public class Demo {
	public static void main (String[] args) {
		I a = new A();
	    a.f();
	    // 编译错误
	    //a.g();
	    System.out.println(a.getClass().getName());

	    if (a instanceof A) {
	    	A aa = (A)a;
	    	aa.g();
	    }
	}
}

/*
A f()
A
B g()
*/
```

避免上面的情况，使用包访问权限是比较好的方式。但是从下面的例子可以看出，使用反射仍然可以调用到所有的方法。

```
// typeinfo包
// TI只是个前缀，避免重名

package typeinfo;

public interface TIA {
	public void f();
	public void g();
}
```
```
// typeinfo包
// TI只是个前缀，避免重名

package typeinfo;

class TIC implements TIA {
	public void f() {
		System.out.println("C f()");
	}

	public void g() {
		System.out.println("C g()");
	}

	void u() {
	    System.out.println("C u() -- package");	
	}

	protected void v() {
		System.out.println("C v() -- protected");	
	}

	private void w() {
		System.out.println("C v() -- private");	
	}
}

public class HiddenC extends TIC {
	public static TIA makeA() { return new TIC(); }
}
```
```
import typeinfo.*;
import java.lang.reflect.*;

public class Demo {
	public static void main (String[] args) {
		TIA a = HiddenC.makeA();
		a.f();
		a.g();

        // 编译错误
        // 错误：TIC在typeinfo中不是公共的; 无法从外部程序包中对其进行访问
        // 不能访问接口A以外的函数
		// if (a instanceof TIC) {

		// }

		callHiddenMethod(a, "u");
		callHiddenMethod(a, "v");
		callHiddenMethod(a, "w");
		// 不存在的函数会有异常
		//callHiddenMethod(a, "W");
	}

	public static void callHiddenMethod(Object o, String methodName) {
		try {
			Method m = o.getClass().getDeclaredMethod(methodName);
			m.setAccessible(true);
			m.invoke(o);
		} catch (Exception e) {
			System.out.println("Exception is " + e);
		}
	}
}

/*
C f()
C g()
C u() -- package
C v() -- protected
C v() -- private
*/
```

### 域的反射

其中，final域是安全的，不会被改变。

```
import java.lang.reflect.*;

class A {
	private int i = 1;
	private final String s = "total safe";
	private String s2 = "no safe";
	public String toString() {
		return "i = " + i + ", s = " + s + ", s2 = " + s2 + ".";
	}
}

public class Demo {
	public static void main (String[] args) {
		try {
			A a = new A();
			Field f = a.getClass().getDeclaredField("i");
			f.setAccessible(true);
			System.out.println("field is " + f);
			f.setInt(a, 47);
			System.out.println("i is " + f.getInt(a));
			System.out.println(a);
			f = a.getClass().getDeclaredField("s");
			f.setAccessible(true);
			System.out.println("field is " + f);
			f.set(a, "changed");
			System.out.println("s is " + f.get(a));
			System.out.println(a);
			f = a.getClass().getDeclaredField("s2");
			f.setAccessible(true);
			System.out.println("field is " + f);
			f.set(a, "changed");
			System.out.println("s2 is " + f.get(a));
			System.out.println(a);
		} catch (Exception e) {
			System.out.println("Exception is  " + e);
		}
	}
}

/*
field is private int A.i
i is 47
i = 47, s = total safe, s2 = no safe.
field is private final java.lang.String A.s
s is changed
i = 47, s = total safe, s2 = no safe.
field is private java.lang.String A.s2
s2 is changed
i = 47, s = total safe, s2 = changed.
*/
```

















