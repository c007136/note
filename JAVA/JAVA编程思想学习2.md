# JAVA编程思想学习二
> 第五章 ～ 第八章

##第五章： 初始化与清理

### 方法重载（Method overloading）
即：方法名相同而形参不同

#### 涉及基本类型的重载 

```
void f(byte x);
void f(int x);
```

最好不要有这玩意儿

### this
表示对“调用方法的那个对象”的引用，只能在方法内部使用。

### finalize方法
需要记住的3点：

1. 对象可能不被垃圾回收
2. 垃圾回收不等于“析构”
3. 垃圾回收只与内存有关

### 垃圾回收器
java语言规范没有明确地说明JVM使用哪种垃圾回收算法，但是任何一种垃圾回收算法一般要做两件基本的事情：

1. 发现无用的信息
2. 回收被无用对象占用的内存空间

**<font color=#FF0000>由于有个垃圾回收机制，JAVA中的对象不再有“作用域”的概念？？？</font>**

参考链接：
https://www.cnblogs.com/sunniest/p/4575144.html


### 初始化顺序

```
public class Counter {
    int i = 2;
    Counter() {
        i = 7;
    }
}
```

在上面的代码中，i首先会被初始化为2，然后变成7。也就是说：**自动初始化的进行，在构造函数之前**

初始化的顺序是先静态对象，而后是“非静态”对象。

### 对象的创建过程
假设有个名为Dog的类：

1. 即使没有显式地使用static关键字，构造器实际上也是静态方法。因此，当首次创建类型为Dog的对象时（构造器可以看成静态方法），或者Dog类的静态方法/静态域首次被访问时，Java解释器必须查找类路径，以定位Dog.class文件。
2. 然后载入Dog.class，有关静态初始化的所有动作都会执行。因此，静态初始化只在Class对象首次加载的时候进行一次。
3. 当用`new Dog()`创建对象的时候，首先将在堆上为Dog对象分配足够的存储空间。
4. 这块存储空间会被清零，这就自动地将Dog对象中的所有基本类型数据都设置成了默认值，而引用则被设置成了null。
5. 执行所有出现于字段定义处的初始化动作。
6. 执行构造器。


## 第六章：访问权限控制

重构即重写代码，以使其更可读，更易理解，并因此而更具有可读性。

当你编写一个Java源代码文件时，此文件通常被称为编译单元。每个编译单元都必须有一个后缀名.java，而编译单元内则只能有一个public类，该类的名称必须与文件的名称相同。但也可以没有public类，这个类就拥有了包访问权限。

JAR包，class文件集合。JAVA解释器负责这些文件的查找、装载和解释。

类既不可以是private，也不可以是protected的，但是内部类可以是private/protected


## 第七章 复用类

初始化的位置有四个地方：
1. 在定义对象的地方
2. 在构造函数中
3. 惰性初始化
4. 实例初始化  { i = 4; }

即使一个文件中含有多个类，也只有命令行所调用的那个类的main方法会被调用。

JAVA不直接支持代理，所谓的代理本质上还是组合。

确保清理：需要有一个类似dispose的方法，清理顺序与生成顺序相反，类似C++的析构函数。

名称屏蔽：JAVA的父类有某个已被重载多次的方法，在子类中重新定义该方法，并不会屏蔽其在基类中的任何版本（与C++不同）。

```
class Father {
	void f(char c) {}     // 依然能用，但会很困惑
	void f(double c) {}   // 依然能用，但会很困惑
}

class Child {
	void f(String c) {}
}
```

### final关键字

#### 数据
1. 一个永不改变的编译时常量，必须是基本类型
2. 一个在运行时被初始化的值，而你不希望它被改变
3. 既是static又是final的域只占据一段不能改变的存储空间
4. 当对象引用（不是基本类型）运用final时，final使其引用恒定不变，然而对象自身却是可以被修改的
5. final参数，表示无法在方法中更改参数引用所指向的对象
#### 方法
1. 把方法锁定，以防止任何继承类修改它的含义。
2. 类中所有的private方法都隐式地指定为final的。

** 除非很确定，否则还是不要用final指明方法，优点效率更高。 **
#### 类
不希望被继承


## 第八章 多态
多态通过分离做什么和怎么做，从另一个角度将接口和实现分离开来 ？？？也称作动态绑定、后期绑定或运行时绑定。

“封装”通过合并特征和行为来创建新的数据类型，“实现隐藏”则通过将细节“私有化”把接口和实现分离开来，而“多态”的作用则是消除类型之间的耦合关系。

### 方法调用绑定
将一个方法调用同一个方法主体关联起来被称作绑定。若在程序执行前绑定，叫做前期绑定，在程序运行时根据对象的类型进行绑定，叫做后期绑定。java中除了static方法和final方法之外，其他所有的方法都是后期绑定。后期绑定都是在对象中安置某种“类型消息”。

域（变量）也没有“多态”的功能。

**<font color=#FF0000>不要在构造调用多态函数</font>**

```
import static util.Print.*;

class Glyph {
	Glyph() {
		print("Glyph before draw");
		draw();
		print("Glyph after draw");
	}

	void draw() { 
		print("Glyph draw"); 
	}
}

class SubGlyph extends Glyph {
	private int radius = 1;

	SubGlyph(int r) {
		radius = r;
		print("SubGlyph radius = " + radius);
	}

	void draw() {
		print("SubGlyph radius = " + radius);
	}
}

public class Demo {
	public static void main(String[] args) {
		new SubGlyph(5);
	}
}

/*
Glyph before draw
SubGlyph radius = 0
Glyph after draw
SubGlyph radius = 5
*/
```

### 协变返回类型
它表示在导出类中的被覆盖方法可以返回基类方法的返回类型的某种导出类型。

```
import static util.Print.*;

class Grain {
	public String toString() { return "Grain"; }
}

class Wheat extends Grain {
	public String toString() { return "Wheat"; }
}

class Mill {
	Grain process() { return new Grain(); }
}

class WheatMill extends Mill {
	Wheat process() { return new Wheat(); }
}

public class Demo {
	public static void main(String[] args) {
		Mill m = new Mill();
		Grain g = m.process();
		print(g);

		m = new WheatMill();
		g = m.process();
		print(g);
	}
}

/*
Grain
Wheat
*/

// 这里是不是违背里氏替换原则第4层含义
// 当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格
```

### 向下转型

```
import static util.Print.*;

class A {
	public void f() { print("F"); }
	public void g() { print("G"); }
}

class B extends A {
	public void f() { print("f"); }
	public void g() { print("g"); }
	public void u() { print("u"); }
	public void v() { print("v"); }
	public void w() { print("w"); }
}

public class Demo {
	public static void main(String[] args) {
		A[] a = {new A(), new B()};
		a[0].f();
		a[1].g();
		// a[1].u();  // 编译错误，函数找不到
		((B)a[1]).u();
		((B)a[1]).v();
	}
}

/*
F
g
u
v
*/
```