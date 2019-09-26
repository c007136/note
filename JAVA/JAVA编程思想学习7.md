# JAVA编程思想学习7
> 第20章 ～ 第X章

## 第20章 注解

注解（也被称为元数据）为我们代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些数据。

注解在一定程度上是把元数据与源代码文件结合在一起，而不是保存在外部文档中这一大大趋势之下所催生的。


### 基本语法

```
public class Testable {
	public void execute() {
		System.out.println("Excuting..");
	}
	
	@Test void testExecute() {
		execute();
	}
}
```

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
	
}
```

除了`@`符号以外，`@Test`的定义很像一个空的接口。定义注解时，会需要一些元注解（meta-annotation），如`@Target`和`@Retention`，`@Target`用来定义你的注解将应用于什么地方（例如是一个方法或者一个域）。`@Retention`用来定义该注解在哪一个级别可用，在源代码中（SOUCE）、类文件中（CLASS）或者运行时（RUNTIME）。

在注解中，一般都会包含一些元素以表示某些值。当分析处理注解时，程序或工具可以利用这些值。注解的元素看起来就像接口的方法，唯一区别是你可以为其指定默认值。

没有元素的注解称为标记注解（marker annotation），例如上面的`@Test`。

```
import java.lang.annotation.*;
import java.util.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)

@interface UseCase {
	public int id();
	public String description() default "no description";
}

class PasswordUtils {
	@UseCase(id = 47, description = "Passwords must contain at least one numeric")
	public boolean validatePassword(String password) {
        return (password.matches("\\w*\\d\\w*"));
    }
    @UseCase(id = 48)
    public String encryptPassword(String password) {
    	return new StringBuilder(password).reverse().toString();
    }
    @UseCase(id = 49, description = "New passwords can't equal previously used ones")
    public boolean checkForNewPassword(List<String> prevPasswords, String password) {
    	return !prevPasswords.contains(password);
    }
}

public class Demo {
	public static void main (String[] args) {
	}
}
```

### 元注解

#### @Target

> 表示该注解可以用于什么地方。可能的ElementType参数包括：

* CONSTRUCTOR：构造函数
* FIELD：域声明（包括enum实例）
* LOCAL_VARIABLE：局部变量
* METHOD：方法声明
* PACKAGE：包声明
* PARAMETER：参数声明
* TYPE：类、接口（包括注解类型）或enum声明

#### @Retention

> 表示需要在什么级别保存该注解信息。可选的RetentionPolicy参数包括：

* SOURCE：源代码中，注解将被编译器丢弃
* CLASS：注解在class文件中可用，但会被VM丢弃
* RUNTIME：VM将在运行时也保留注解，因此可以通过反射机制读取注解的信息

#### @Documented

> 将此注解包含在Javadoc中

#### @Inherited

> 允许子类继承父类中的注解




































































