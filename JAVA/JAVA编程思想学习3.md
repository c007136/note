# JAVA编程思想学习三
> 第九章 ～ 第十章

## 第九章 接口

### 抽象类和抽象方法
接口和内部类为我们提供了一种将接口与实现分离的更加结构化的方法。接口本质就是抽象类。包含抽象方法的类叫做抽象类。如果一个类包含一个或多个抽象方法，该类必须被限定为抽象的。我们可能会创建一个没有任何抽象方法的抽象类，用了阻止产生这个类的任何对象。

抽象类是很有用的重构工具，因为它们使得我们可以很容易地将公共方法沿着继承层次结构向上移动。

### 接口
interface这个关键字产生一个完全抽象的类，它根本没有提供任何具体实现。如果interface前，不加public字段，表明此接口只有包访问权限。

接口也可以包含域，但是这些域隐式地是static & final的。继承接口用关键字implements。接口中的所有方法都是public的。

#### 协变与逆变

参考链接： https://www.cnblogs.com/en-heng/p/5041124.html

### 多重继承

父类只能有一个非接口的基类，其他的必须为接口。

### 嵌套接口

嵌套在class中的接口：

被定义为私有的接口只能在接口所在的类被实现。可以被实现为public的类也可以被实现为private的类。当被实现为public时，只能在被自身所在的类内部使用。只能够实现接口中的方法,在外部不能像正常类那样上传为接口类型

嵌套在接口中的接口，只能是pubilic

```
class A {
    private interface D {
        void f();
    }
    private class DImp implements D {
        public void f() {}
    }
    public class DImp2 implements D {
        public void f() {}
    }
    public D getD() { return new DImp2(); }
    private D dRef;
    public void receiveD(D d) {
        dRef = d;
        dRef.f();
    }
}
 
public class NestingInterfaces {
    public static void main(String[] args) {
        A a = new A();
        //The type A.D is not visible
        //! A.D ad = a.getD();
        //Cannot convert from A.D to A.DImp2
        //A.DImp2 di2 = a.getD();
        //The type A.D is not visible
        //! a.getD().f();        
        A a2 = new A();
        a2.receiveD(a.getD());
    }
}
```


### 接口与工厂方法

能够实现方法的声明和方法的实现分离。

### 小结

** 恰当的原则应该是优先选择类而不是接口 ** ？？？


## 第十章 内部类

内部类还拥有其外围类的所有元素的访问权。

当某个外围类的对象创建了一个内部类对象时，此内部类对象必定会秘密地捕获一个指向那个外围类对象的引用。

如果你需要生成对外部类对象的引用，可以使用外部类的名字紧跟圆点和this。

```
class Outer {
	public class Inner {
		public Outer outer() {
			return Outer.this;
		}
	}
}
```

其他对象内部，要创建某个内部类的对象，需要使用.new语法。

```
class Outer {
	public class Inner {
	}
}


public class Demo {
	public static void main(String[] args) {
		Outer outer = new Outer();
		Outer.Inner inner = outer.new Inner();
	}
}
```

#### 内部类的嵌套内部类

能透明访问所有外围类的所有成员

```
class M {
	private void f() {
		System.out.println("M.f");
	}

	class A {
		private void f() {
			System.out.println("A.f");
		}

		private void g() {
			System.out.println("A.g");
		}

		public class B {
			public void h() {
				//f();   // 找最近的
				g();
			}
		}
	}
}


public class Demo {
	public static void main(String[] args) {
		M m = new M();
		M.A ma = m.new A();
		M.A.B mab = ma.new B();
		mab.h();
	}
}
```

### 向上转型

能够方便地隐藏实现细节

```
interface AInter {
	void hello();
}

class Outer {
	private class AChild implements AInter {
		@Override
		public void hello() {
			System.out.println("hello");
		}
	}

	AInter aChild() {
		return new AChild();
	}
}


public class Demo {
	public static void main(String[] args) {
		Outer outer = new Outer();
		AInter inter = outer.aChild();
		inter.hello();
	}
}
```

### 在方法和作用域内的内部类

#### 方法中的内部类

```
interface IInter {
	public String readLabel();
}

class Outer {
	public IInter inter(String s) {
		class AInter implements IInter {
			private String label;

			private AInter(String aString) {
				label = aString;
			}

			public String readLabel() {
				return label;
			}
		}

		return new AInter(s);
	}
}


public class Demo {
	public static void main(String[] args) {
		Outer outer = new Outer();
		IInter inter = outer.inter("hello world");
		String label = inter.readLabel();
		System.out.println("label is " + label);
	}
}

```

#### 作用域中的内部类

```
class Outer {
	private void inernalTracking(boolean b) {
		if (b) {
			class TrackingSlip {
				private String id;
				TrackingSlip(String s) {
					id = s;
				}
				String getSlip() { return id; }
			}
			TrackingSlip ts = new TrackingSlip("slip");
			String s = ts.getSlip();
			System.out.println("s is " + s);
		}
	}

	public void track() { inernalTracking(true); }
}


public class Demo {
	public static void main(String[] args) {
		Outer outer = new Outer();
		outer.track();
	}
}
```

### 匿名类

```
new 父类构造器（参数列表）| 实现接口（）  
   {  
     //匿名内部类的类体部分  
   }
```

#### 实现了接口的匿名类

```
interface IInter {
    public int value();
}

class Outer {
	public IInter inter() {
		return new IInter() {
			private int i = 11;
			public int value() { return i; }
		};
	}
}


public class Demo {
	public static void main(String[] args) {
		Outer outer = new Outer();
		IInter inter = outer.inter();
		System.out.println("value = " + inter.value());
	}
}
```

#### 匿名类，它扩展了有非默认构造函数的类

```
class Wrapping {
	private int i;

	public Wrapping(int x) { i = x; }

	public int value() { return i; }
}

public class Demo {
  public Wrapping wrapping(int x) {
    return new Wrapping(x) {
      public int value() {
        return super.value() * 47;
      }
    };
  }
  public static void main(String[] args) {
    Demo p = new Demo();
    Wrapping w = p.wrapping(10);
    System.out.println(w.value());
  }
}
```

#### 匿名类，它执行字段初始化

```
interface IInter {
    public String value();
}

class Outer {
	public IInter inter(final String src) {
		return new IInter() {
			private String label = src;
			public String value() { return label; }
		};
	}
}


public class Demo {
	public static void main(String[] args) {
		Outer outer = new Outer();
		IInter inter = outer.inter("kkk");
		System.out.println("value = " + inter.value());
	}
}
```

通过命令行，不会报编译错误

#### 匿名类，它通过实例初始化实现构造（匿名类不可以有构造函数）

```
abstract class Base {
	public Base(int i) {
		System.out.println("Base i = " + i);
	}

	public abstract void f();
}

class Outer {
	public Base getBase(int i) {
		return new Base(i) {
			{
				System.out.println("Inside i = " + i);
			}
			public void f() {
				System.out.println("Inside f called");
			}
		};
	}
}


public class Demo {
	public static void main(String[] args) {
		Outer outer = new Outer();
		Base base = outer.getBase(10);
		base.f();
	}
}
```

** 和上面的上面有啥区别？？ **

#### 工厂设计模式

```
interface Service {
	void f1();
	void f2();
}

interface ServiceFactory {
	Service getService();
}

class S1 implements Service {
	private S1() {}

	public void f1() { 
		System.out.println("S1 f1");
	}

	public void f2() { 
		System.out.println("S1 f2");
	}

	public static ServiceFactory factory = 
	    new ServiceFactory() {
	    	public Service getService() {
	    		return new S1();
	    	}
	    };
}

class S2 implements Service {
	private S2() {}

	public void f1() { 
		System.out.println("S2 f1");
	}

	public void f2() { 
		System.out.println("S2 f2");
	}

	public static ServiceFactory factory = 
	    new ServiceFactory() {
	    	public Service getService() {
	    		return new S2();
	    	}
	    };
}

public class Demo {
	public static void serviceConsumer(ServiceFactory factory) {
		Service s = factory.getService();
		s.f1();
		s.f2();
	}

	public static void main(String[] args) {
		serviceConsumer(S1.factory);
		serviceConsumer(S2.factory);
	}
}
```


### 嵌套类

如果不需要内部类对象与其外围类对象之间有联系，那么可以将内部类声明为static，这通常称为嵌套类。这意味着：

1. 要创建嵌套类的对象，并不需要其外围类的对象。
2. 不能从嵌套类的对象中访问非静态的外围类对象。
3. 普通内部类的字段和方法，只能放在类的外部层次上，所以普通的内部类不能有static数据和字段，也不能包含嵌套类，但嵌套类可以。内部类必须依赖外部类的实例建立实例，在实例化的时候才能分配内存，静态的东西需要在编译时分配内存，所以不能有static数据和字段

```
interface IInter {
    public String value();
}

class Outer {
	private static int i = 7;
	private int j = 8;

	public static class AInner {
		public static int x = 10;

		public static void f() {
			// 可以访问
			System.out.println("i = " + i);
			// 不能访问
			// System.out.println("j = " + j);
		}

		public void g() {
			// 可以访问
			System.out.println("i = " + i);
			// 不能访问
			// System.out.println("j = " + j);
		}
	}

	public static AInner ainner() {
		return new AInner();
	}
}


public class Demo {
	public static void main(String[] args) {
		Outer.AInner ainner = new Outer.AInner();
		System.out.println("value = " + ainner.x);
		ainner.f();
		ainner.g();
	}
}
```

#### 接口内部的类

```
public interface Demo {
	void f();

    // 默认就是public和static
	class Test implements Demo {
		public void f() {
			System.out.println("ff");
		}
	}

	public static void main(String[] args) {
		new Test().f();
	}
}
```

如果想要创建某些公共代码，使得他们可以被某个接口的所有不同实现所共用，那么接口内部的嵌套类会显得比较方便

可以在嵌套类中放置测试代码

```
public class Demo {
	public void f() {
		System.out.println("ff");
	}

    // 如果直接写main方法，demo.class中会带着额外的编译信息
	public static class Tester {
		public static void main(String[] args) {
			Demo d = new Demo();
			d.f();
		}
	}
}
```

#### 练习题21

创建一个包含嵌套类的接口，该嵌套类中有一个static方法，它将调用接口中的方法并显示结果。实现这个接口，并将这个实现的一个实例传递给这个方法

```
interface A {
	void af();

	class M {
		public static void mf(A a) {
			a.af();
			System.out.println("mf");
		}
	}
}

class B implements A {
	public void af() {
		System.out.println("af in B");
	}
}


public class Demo {
	public static void main(String[] args) {
		B b = new B();
		//b.af();

		B.M m = new B.M();
		m.mf(b);
	}
}
```

### 为什么需要内部类

如果只是需要一个对接口的引用，为什么不通过外围类实现那个接口呢?

每个内部类都能独立地继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

内部类有效地实现了“多重继承”，也就是说，内部类允许继承多个非接口类型（类或者抽象类）

内部类的优点：

1. 内部类可以有多个实例，每个实例都有自己的状态信息，并且与其外围类对象的信息相互独立
2. 在单个外围类中，可以让多个内部类以不同方式实现同一个接口，或继承同一个类
3. 创建内部类对象的时刻并不依赖于外围类对象的创建
4. 内部类并没有令人迷惑的“is-a”关系，它就是一个独立的实体


### 闭包与回调

闭包是一个可调用的对象，它记录了一些信息，这些信息来自于创建它的作用域。通过这个定义，可以看出内部类是面向对象的闭包。

```
interface Incrementable {
	void increment();
}

class Callee1 implements Incrementable {
	private int i = 0;
	public void increment() {
		i++;
		System.out.println(i);
	}
}

class MyIncrement {
	public void increment() {
		System.out.println("other method");
	}

	static void f(MyIncrement mi) {
		mi.increment();
	}
}

class Callee2 extends MyIncrement {
	private int i = 0;
	public void increment() {
		super.increment();
		i++;
		System.out.println(i);
	}

	private class Closure implements Incrementable {
		public void increment() {
			Callee2.this.increment();
		}
	}

	Incrementable getCallbackReference() {
		return new Closure();
	}
}

class Caller {
	private Incrementable callbackReference;

	Caller(Incrementable cbh) {
		callbackReference = cbh;
	}

	void go() {
		callbackReference.increment();
	}
}

public class Demo {
	public static void main(String[] args) {
		Callee1 c1 = new Callee1();
		Callee2 c2 = new Callee2();
		MyIncrement.f(c2);
		System.out.println("-------");

		Caller caller1 = new Caller(c1);
		Caller caller2 = new Caller(c2.getCallbackReference());
		caller1.go();
		caller1.go();
		System.out.println("-------");
		caller2.go();
		caller2.go();
	}
}
```

#### 回调
```
interface I {
	void execute();
}

class Callback {
	public void run(int a, I i) {
		System.out.println("do somthing");
		i.execute();
	}
}

public class Demo {
	public static void main(String[] args) {
		Callback c = new Callback();
		c.run(100, new I() {
			public void execute() {
				System.out.println("callback execute");
			}
		});
	}
}
```

### 内部类与控制框架

控制框架是一类特殊的应用程序框架，它用来解决响应事件的需求。主要用来响应事件的系统称作“事件驱动系统”。

### 内部类的继承

```
class WithInner {
	class Inner {
		Inner() {
			System.out.println("Inner");
		}
	}
}

class InheritInner extends WithInner.Inner {
	// 必须要有外围类的引用
	// 必须使用语法：enclosingClassReference.super();
	InheritInner(WithInner wi) {
		wi.super();
		System.out.println("InheritInner");
	}
}

public class Demo {
	public static void main(String[] args) {
		WithInner wi = new WithInner();
		InheritInner ii = new InheritInner(wi);
	}
}
```


#### 参考资料
1. [搞懂JAVA内部类](https://juejin.im/post/5a903ef96fb9a063435ef0c8)
2. [Java内部类的使用小结](http://android.blog.51cto.com/268543/384844)
3. [一篇文章让你彻底了解Java内部类](https://juejin.im/post/5b7e4a04e51d4538b063eee3)