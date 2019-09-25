# Dart编程语言学习1

> by 木愚

## 重要的概念

* 任何保存在变量中的都是一个对象，并且所有的对象都是对应一个类的实例。无论是数字，函数和null都是对象。所有对象继承`Object`类型
* 尽管Dart是强类型的，但是Dart可以推断类型。
* Dart支持泛型，如`List<int>`
* Dart支持顶级函数（全局函数）
* Dart支持顶级变量
* Dart语法中包含`表达式(expressions)`（有运行时值）和`语句(statements)`（没有运行时值）。
* Dart工具提示两种类型问题：警告和错误。

## final和const

final变量的值只能被初始化一次，const变量在编译时就已经固定（const变量是隐式final的类型）。

## 内建类型

* Number
* String
* Boolean
* List（也被称为Array）
* Map
* Set
* Rune（用于在字符串中表示Unicode字符）
* Symbol

这些类型都可以被初始化为字面量（字面量是一种编译型常量）。

### Rune

在Dart中，Rune用来表示字符串中的UTF-32编码字符。

表示Unicode编码的常用方法是，\uXXXX，这里XXXX是一个4的16进制数。例如，心形符号（♥）\u2665。对于特殊的非4个数值的情况，把编码值放到大括号中即可。例如emoji的笑脸（😀）是 \u{1f600}。

```
main() {
  var clapping = '\u{1f44f}';
  print(clapping);
  print(clapping.codeUnits);
  print(clapping.runes.toList());

  Runes input = new Runes('\u2665 \u{1f605} \u{1f60e} \u{1f47b} \u{1f596} \u{1f44d}');
  print(new String.fromCharCodes(input));
}
```

### Symbol

一个Symbol对象表示Dart程序中声明的运算符或者标识符。你也许永远都不需要使用Symbol，但要按名称引用标识符的API时，Symbol就非常有用了。因为代码压缩后会改变标识符的名称，但不会改变标识符的符号。通过字面量Symbol，也就是标识符前面添加一个`#`号，来获取标识符的Symbol。

反射也会用到。

## 函数

Dart是一门真正面向对象的语言，甚至其中的函数也是对象，并且有它的类型`Function`。这也意味着函数可以被赋值给变量或者作为参数传递给其他函数。也可以把Dart类的实例当做方法来调用，也就是`Callable classes`（可调用类）。


### 可调用类

```
class A {
  // call方法会被调用
  call(String s) {
     print('s is $s');
  }
}

main() {
  var a = A();
  a("hello world");
}
```

### 匿名函数

```
Function doSome(result) {
	return () => print(result);
}

main() {
  var f = doSome("test");
  f();
}
```

### 词法闭包

闭包即一个函数对象，即使函数对象的调用在它原始作用域之外，依然能够访问在它词法作用域内的变量。

```
Function makeAdder(num addBy) {
	return (num i) {
	    print("i = ${i}");
	    print("addBy = ${addBy}");
	    return addBy +i;
	};
}

main() {
  var add2 = makeAdder(2);
  var add4 = makeAdder(4);

  print(add2(3));
  print(add4(3));
}

/*
i = 3
addBy = 2
5
i = 3
addBy = 4
7
*/
```

函数可以封闭定义到它作用域内的变量。上面的例子中，`makeAdder()`捕获了变量`addBy`。无论在什么时候执行返回函数，函数都会使用捕获的`addBy`变量。

### 级联运算符（..）

级联运算符（..）可以实现对同一个对象进行一系列的操作。除了调用函数，还可以访问同一对象上的字段属性。这通常可以节省创建临时变量的步骤，同时编写更流畅的代码。

```
querySelector('#confirm')
  ..text = 'Confirm'
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed'));
```

等价于

```
var button = querySelector('#confirm');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

可以嵌套

```
final addressBook = (AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();
```

## 异常

Dart代码可以抛出和捕获异常。异常表示一些未知的错误情况。如果异常没有被捕获，则异常会抛出，导致抛出异常的代码终止执行。

和Java有所不同，Dart中的所有异常是非检查异常。方法不声明它们可能哪些异常，也不要求捕获任何异常。

Dart提供了`Exception`和`Error`类型，以及一些子类型。当然也可以定义自己的异常类型。但是，此外Dart程序可以抛出任何非null对象，不仅限`Exception`和`Error`对象。

下面是关于抛出或引发异常的示例：

```
throw FormatException('exception');
```

也可以抛出任意的对象：

```
throw 'Out of llamas';
```

> 提示：高质量的生产代码通常会实现`Exception`或`Error`类型的异常抛出。

因为抛出异常是一个表达式，所以可以在`=>`语句中使用，也可以在其他使用表达式的地方抛出异常：

```
void distanceTo(Point other) => throw Exception()
```

捕获语句中可以同时使用 on 和 catch ，也可以单独分开使用。 使用 on 来指定异常类型， 使用 catch 来 捕获异常对象。catch() 函数可以指定1到2个参数， 第一个参数为抛出的异常对象， 第二个为堆栈信息 ( 一个 StackTrace 对象 )。

```
try {
  // ···
} on Exception catch (e) {
  print('Exception details:\n $e');
} catch (e, s) {
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```

## 类

Dart是一种基于类和mixin继承机制的面向对象的语言。基于**mixin**意味着每个类（除Object外）都只有一个超类。

使用`?.`来代替`.`，可以避免因为左边对象可能为null，导致的异常

```
p?.y = 4;
```

### 构造函数不被继承

子类不会继承父类的构造函数。子类不声明构造函数，那么它就只有默认构造函数。

### 命名构造函数

使用命名构造函数可以一个类实现多个构造函数，也可以使用命名构造函数来更清晰的表明函数意图。

```
class Point {
  num x, y;

  Point(this.x, this.y);

  // 命名构造函数
  Point.origin() {
    x = 0;
    y = 0;
  }
}
```

### 初始化列表

在开发期间，可以使用`assert`来验证输入的初始化列表。

```
class Point {
	num x, y;

    // assert
	Point(this.x, this.y) : assert(x >= 0) {
	    print("x = $x, y = $y");
	}

    // 初始化列表
	Point.fromJson(Map<String, num> json)
	    : x = json['x'],
	      y = json['y'] {
	    print("x = $x, y = $y");
    }
}

main() {
  Point pt = Point(10, 11);
}
```

### 调用父类非默认构造函数

默认情况下，子类的构造函数会自动调用父类的默认构造函数（匿名，无参数）。

如果父类中没有匿名无参的构造函数， 则需要手工调用父类的其他构造函数。 在当前构造函数冒号 (:) 之后，函数体之前，声明调用父类构造函数。

1. 初始化参数列表
2. 父类的构造函数（一次且只有一次）
3. 子类的构造函数

```
class A {
  A() {
    print("in A()");
  }

  A.fromJson(Map data) {
	print("in A.fromJson");
  }
}

class B extends A {
    B() {
      print("in B()");
    }

	B.fromJson(Map data) {
	  print("in B.fromJson");
	}

	B.fromJson1(Map data) : super.fromJson(data) {
	  print("in B.fromJson1");
	}
}

main() {
    B b1 = B();

    print('----------');

    B b2 = B.fromJson({});

    print('----------');

    B b3 = B.fromJson1({});
}

/*
in A()
in B()
----------
in A()
in B.fromJson
----------
in A.fromJson
in B.fromJson1
*/
```

### 重定向构造函数

重定向构造函数的函数体为空，构造函数的调用在冒号之后。

```
class Point {
	num x, y;

    // assert
	Point(this.x, this.y) : assert(x >= 0), assert(y >= 0) {
	    print("x = $x, y = $y");
	}

    // 重定向构造函数
    Point.f(num x, num y) : this(x, y);
}

main() {
  Point pt = Point.f(10, 11);
}
```

### 工厂构造函数

当执行构造函数并不总是创建这个类的一个新实例时，则使用`factory`关键字。

```
class Logger {
  final String name;

  // 从命名的 _ 可以知，
  // _cache 是私有属性。
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    if (_cache.containsKey(name)) {
      print("from cache name = $name");
      return _cache[name];
    } else {
      print("from new name = $name");
      final logger = Logger._internal(name);
      _cache[name] = logger;
      return logger;
    }
  }

  // 私有构造函数
  Logger._internal(this.name);
}

main() {
  Logger l1 = Logger("A");
  Logger l2 = Logger("B");
  Logger l3 = Logger("A");
}
```

> 工厂构造函数无法访问this

### 抽象类

抽象类不能实例化。

### 隐式接口

每个类都隐式的定义了一个接口，接口包含了该类所有的实例成员及其实现的接口。如果要创建一个A类，A类要支持B类的API，但是不要继承B的实现，那么可以通过A实现B的接口。

```
class A {
    // 默认有了get方法
    final _name;

    // 不包含在接口里
	A(this._name);

	String f(String s) => 'Hello $s. I am $_name in A'; 
}

class B implements A {
    get _name => "bb";

    // 会报错
    //B(this._name);

	String f(String s) => 'Hello $s. I am $_name in B'; 
}

main() {
    A a = A('aa');
    print(a.f("AA"));

    B b = B("bbb");
    print(b.f("BB"));

    if (b is A) {
        print("b is A");
    }
}
```

### 为类添加功能：mixin

通过`with`后面跟一个或多个混入的名称，来使用`mixin`。`mixin`关键字声明的类，表示不希望作为常规类被使用，故不能被实例化或扩展。

```
//  这个方法被抛弃了
//abstract class A
mixin A
{
    num a1 = 10;
    String a2 = "11";
	void f() { print("A:f()"); }
}

class AA with A
{
	
}

void main() 
{
  AA aa = AA();
  print(aa.a1);
  print(aa.a2);
  aa.f();
}
```

## 泛型

Dart中泛型类型是固化的，也就是说它们在运行时是携带着类型信息的。

### 限制泛型类型

可以使用`extends`实现参数类型的限制。

```
class Foo<T extends SomeBaseClass> {
  // Implementation goes here...
  String toString() => "Instance of 'Foo<$T>'";
}

class Extender extends SomeBaseClass {...}
```

可以使用 SomeBaseClass 或其任意子类作为通用参数

```
var someBaseClassFoo = Foo<SomeBaseClass>();
var extenderFoo = Foo<Extender>();
```

也可以不指定泛型参数：

```
var foo = Foo();
print(foo); // Instance of 'Foo<SomeBaseClass>'
```

### 泛型函数

```
T f<T extends num>(T t) {
	return t*2;
}

void main() 
{
  print( f(2) );
  print( f(2.5) );
}
```

## 库

### 指定库前缀

```
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

A a1 = A();
lib2.A a2 = lib2.A();
```

### 导入库的一部分

```
// import only foo
import 'package:lib1/lib1.dart' show foo;

// import all names except foo
import 'pacage:lib2/lib2.dart' hide foo;
```

### 延迟加载库

使用场景：

* 减少APP的启动时间
* 执行A/B测试，例如尝试各种算法的不同实现
* 加载很少使用的功能

先使用关键字deferred as

```
import 'pacage:lib1/lib1.dart' deferred as lib1;
```

当需要使用的时候，使用库标识符调用`loadLibrary()`函数加载库

```
Future greet() async {
  await lib1.loadLibrary();
  lib1.printGreeting();
}
```

* 延迟加载库的常量在导入的时候是不可用的。 只有当库加载完毕的时候，库中常量才可以使用。 **测试的结果不是这样的？？**
* 在导入文件的时候无法使用延迟库中的类型（types）。如果你需要使用类型，则考虑把接口类型移动到另外一个库中，让两个库分别导入这个接口库。
* Dart隐含的把`loadLibrary()`函数导入到使用deferred as的命名空间中。`loadLibrary()`方法返回一个`Future`。

```
// deferred_lib.dart
final aNumber = 1234;
```
```
// demo.dart
// 结果不符合上面第一点
import 'deferred_lib.dart' deferred as deferred_lib;

void main() async
{
  var a = deferred_lib.aNumber;
  print(a);
  await deferred_lib.loadLibrary();
  print(deferred_lib.aNumber);
}

/*
1234
1234
*/
```

### 实现库 ???

[Create Library Packages](https://www.dartcn.com/guides/libraries/create-library-packages)

* 如何组织库的源文件
* 如何使用`export`命令
* 何时使用`part`命令
* 何时使用`library`命令










