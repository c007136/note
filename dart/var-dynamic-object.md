# var/dynamic/Object的区别

## var

var表示写代码的地方不在乎变量的类型，但它具有类型推断功能，已经确定某种类型后，便不能再更改了，一般用于局部变量中。

```
void main() {
  var a = "123";    // 类型被自动推断为String类型
  print(a);

  a = 123;          // Error: A value of type 'int' can't be assigned to a variable of type 'String'
  print(a);
}
```

## Object
在[object.dart](https://github.com/dart-lang/sdk/blob/master/sdk/lib/core/object.dart)，第一句注释就是：

> The base class for all Dart objects.

这表明所有类型都继承了Object。


## dynamic
dynamic表示未知类型，和Object一样可以接收任意类型的参数。那它和Object有啥区别呢。

### Object和dynamic的区别
* 动态任意类型，编译阶段检查类型；
* 动态任意类型，编译阶段不检查类型。

直接来一段代码，会有一个更直观的感受：

```
void main() {
  Object a = "string";
  a.toString();             // 只能调用Object中声明的函数

  dynamic b = "string";
  b["message"] = "1234";    // 编译能通过，但在运行会报错：Object.noSuchMethod
}

```

简单说dynamic，是在告诉编译器，我们知道自己在做什么，**不用做类型检查**。但这又让人很困惑：在有了Object的情况下，dynamic是用来干啥的呢？

老规矩，还是直接来一段代码更为直观：

```
class ClassA {
  void a() {}
}

class ClassB {
  void a() {}
}

void fun1(Object object) {
  object.a();     // 会报错
}

void fun2(dynamic object) {
  object.a();     // 不会报错
}

void main() {
  
}
```

ClassA和ClassB是两个没有任何关系的类，如果用了Object则编译不通过，但是用dynamic则编译通过。虽然这样，但是官方还是不推荐，能不用dynamic则不用。比较不进行类型检查，会容易导致类型不安全。

