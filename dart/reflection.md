# Dart之反射

## 反射类（ClassMirror）
### 获取反射类对象

```
external ClassMirror reflectClass(Type key);
```

> Type，即class类的类名（var、dynamic这些关键字不记入）

使用

```
abstract class Test{}
ClassMirror classMirror = reflectClass(Test);
```

### 反射类分析

在[Dart的源码](https://github.com/dart-lang/sdk/blob/master/sdk/lib/mirrors/mirrors.dart)中，查看ClassMirror的属性和方法，主要用来判断是否是抽象类/枚举类/子类，获取接口列表/函数列表/mixin，调用构造方法等作用。这里会用到

> Symbol

Symbol是Dart中比较特殊的一种类型，一般情况下不会用到它，但如果需要获取标识符的字面量时，这个作用就无法估量了。

```
var name = 'a';
Symbol s = #name;
print(s);
```

### 调用构造函数

```
import 'dart:mirrors';

class A {
  int a;
  A(this.a){
    print(a);
  }

  void fun(int a) {
    print(a);
  }
}

main(){
  ClassMirror classMirror = reflectClass(A);   
  classMirror.newInstance(Symbol.empty ,[1]); //调用构造方法，打印1
}
```

## 反射方法

方法反射由`MethodMirror`和`DeclarationMirror`实现，

### metadata列表获取

```
import 'dart:mirrors';

@Todo('todo', 'work')
class A {
  int a;
  
  A(this.a){
    print(a);
  }
}

class Todo {
  final String who;
  final String what;
  const Todo(this.who, this.what);
}

main(){
  ClassMirror classMirror = reflectClass(A);
  // 获取 class 上的元数据
  classMirror.metadata.forEach((metadata) {
    print(metadata.reflectee.who + ' ==> ' + metadata.reflectee.what);
  });
}
```

输出

```
todo ==> work
```

## 对象反射

对象反射由`InstanceMirror`实现，示例：

```
import 'dart:mirrors';

class A {
  int a;

  A(this.a) {
    print(a);
  }
}

main(){
  ClassMirror classMirror = reflectClass(A);
  var instance = classMirror.newInstance(Symbol.empty, [1]);     // 调用构造方法，打印1
  print(instance.reflectee.a);                                   // 获取a，打印1
}
```

##其他反射
### 属性反射 VariableMirror
### 类型反射

* TypeMirror
* TypedefMirror
* TypeVariableMirror

```
import 'dart:mirrors';
typedef F<T> = int Function(T a);

class A {
 Set<int> a1;
 List<String> a2;
 Map<int, String> a3;
 F<int> a4;

 num m2;
 String m3;
}

main() {
  ClassMirror classMirror = reflectClass(A);
  classMirror.declarations.forEach((Symbol key, DeclarationMirror value) {
    if(value is VariableMirror) {
      var type = value.type;
      // 有范型变量的变量的集合，函数变量不包括
      type.typeVariables.forEach((value)=>print(value));
    }
  });

  print("----------");

  classMirror.declarations.forEach((Symbol key, DeclarationMirror value) {
    if(value is VariableMirror) {
      var type = value.type;
      // 范型变量中的类型
      type.typeArguments.forEach((value)=>print(value));
    }
  });
}
```

输出

```
TypeVariableMirror on 'E'
TypeVariableMirror on 'E'
TypeVariableMirror on 'K'
TypeVariableMirror on 'V'
----------
ClassMirror on 'int'
ClassMirror on 'String'
ClassMirror on 'int'
ClassMirror on 'String'
```

### 函数类型反射 FuctionTypeMirror

```
import 'dart:mirrors';
typedef F<T> = int Function(T a);
typedef G<T1, T2> = num Function(T1 a, T2 b);

class A<T>{
 List<int> a1;
 F<int> a2;
 G<int, String> a3;
}

main() {
  ClassMirror classMirror = reflectClass(A);
  classMirror.declarations.forEach((Symbol key, DeclarationMirror value) {
    if(value is VariableMirror){
      var type = value.type;
      if(type is FunctionTypeMirror){
        print(type.reflectedType);
        print(type.returnType);
        type.parameters.forEach((value)=>print(value));
        print('----------');
      }
    }
  });
}
```

输出

```
(int) => int
ClassMirror on 'int'
ParameterMirror on 'noname'
----------
(int, String) => num
ClassMirror on 'num'
ParameterMirror on 'noname'
ParameterMirror on 'noname'
----------
```


## 备注
**扯了辣么多，但是flutter中禁止开发者使用mirros，无法进行反射！！！**

## 参考资料

* [Dart基础4-反射](https://www.jianshu.com/p/d68278d19f79)



