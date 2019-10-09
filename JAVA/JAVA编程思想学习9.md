# JAVA编程思想学习9
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


### 编写注解处理器

如果没有用来读取注解的工具，那注解也不会比注释更有用。使用注解的过程中，很重要的一个部分就是创建与使用注解处理器。

```
import java.lang.reflect.*;
import java.util.*;
import java.lang.annotation.*;

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

class UseCaseTracker {
    public static void trackUseCases(List<Integer> useCases, Class<?> cl) {
        for (Method m : cl.getDeclaredMethods()) {
            UseCase uc = m.getAnnotation(UseCase.class);
            if (uc != null) {
                System.out.println("Found Use Case: " + uc.id() + " " + uc.description());
                useCases.remove(new Integer(uc.id()));
            }
        }
        for (int i : useCases) {
            System.out.println("Warning: Missing use case-" + i);
        }
    }
}

public class Demo {
	public static void main (String[] args) {
        List<Integer> useCases = new ArrayList<Integer>();
        Collections.addAll(useCases, 47, 48, 49, 50);
        UseCaseTracker.trackUseCases(useCases, PasswordUtils.class);
	}
}

/*
Found Use Case: 47 Passwords must contain at least one numeric
Found Use Case: 48 no description
Found Use Case: 49 New passwords can't equal previously used ones
Warning: Missing use case-50
*/
```

#### 注解元素
* 所有基本类型（int，float， boolean等）
* String
* Class
* enum
* Annotation
* 以上类型的数组

```
import java.lang.reflect.*;
import java.util.*;
import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface DBTable {
    public String name() default "";
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface Constraints {
    boolean primaryKey() default false;
    boolean allowNull() default true;
    boolean unique() default false;
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface SQLInteger {
    String name() default "";
    Constraints constraints() default @Constraints;
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface SQLString {
  int value() default 0;
  String name() default "";
  Constraints constraints() default @Constraints;
}

@DBTable(name = "MEMBER")
class Member {
    @SQLString(30)
    String firstName;
    @SQLString(50)
    String lastName;
    @SQLInteger
    Integer age;
    @SQLString(value= 30, constraints = @Constraints(primaryKey = true))
    String handle;

    static int memberCount;
    public String getHandle() { return handle; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String toString() { return handle; }
    public Integer getAge() { return age; }
}


public class Demo {
	public static void main (String[] args) throws Exception {
        if(args.length < 1) {
            System.out.println("arguments: annotated classes");
            System.exit(0);
        }

        for (String className : args) {
            Class<?> cl = Class.forName(className);
            DBTable dbTable = cl.getAnnotation(DBTable.class);
            if (dbTable == null) {
                System.out.println("No DBTable annotations in class " + className);
                continue;
            }
            String tableName = dbTable.name();
            if (tableName.length() < 1) {
                tableName = cl.getName().toUpperCase();
            }

            List<String> columnDefs = new ArrayList<String>();
            for (Field field : cl.getDeclaredFields()) {
                String columnName = null;
                Annotation[] anns = field.getDeclaredAnnotations();
                if (anns.length < 1) {
                    continue;
                }

                if (anns[0] instanceof SQLInteger) {
                    SQLInteger sInt = (SQLInteger)anns[0];
                    if (sInt.name().length() < 1) {
                        columnName = field.getName().toUpperCase();
                    } else {
                        columnName = sInt.name();
                    }
                    columnDefs.add(columnName + " INT" + getConstraints(sInt.constraints()));
                }

                if (anns[0] instanceof SQLString) {
                    SQLString sString = (SQLString)anns[0];
                    if (sString.name().length() < 1) {
                        columnName = field.getName().toUpperCase();
                    } else {
                        columnName = sString.name();
                    }
                    columnDefs.add(columnName + " VARCHAR(" + sString.value() + ")" + getConstraints(sString.constraints()));
                }

                StringBuilder createCommand = new StringBuilder("CREATE TABLE " + tableName + "(");
                for (String columnDef : columnDefs) {
                    createCommand.append("\n   " + columnDef + ",");
                }
                String tableCreate = createCommand.substring(0, createCommand.length() - 1) + ");";
                System.out.println("Table Creation SQL for " + className + " is:\n" + tableCreate);
            }
        }
	}

    private static String getConstraints(Constraints con) {
        String constraints = "";
        if (!con.allowNull()) {
            constraints += " NOT NULL";
        }
        if (con.primaryKey()) {
            constraints += " PRIMARY KEY";
        }
        if (con.unique()) {
            constraints += " UNIQUE";
        }
        return constraints;
    }
}

/*
java Demo Member:

Table Creation SQL for Member is:
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30));
Table Creation SQL for Member is:
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30),
   LASTNAME VARCHAR(50));
Table Creation SQL for Member is:
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30),
   LASTNAME VARCHAR(50),
   AGE INT);
Table Creation SQL for Member is:
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30),
   LASTNAME VARCHAR(50),
   AGE INT,
   HANDLE VARCHAR(30) PRIMARY KEY);
*/
```

### 使用apt处理注解

```
import java.util.*;
import java.lang.annotation.*;
import java.io.*;
import com.sun.mirror.apt.*;
import com.sun.mirror.declaration.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
@interface ExtractInterface {
    public String value();
}

@ExtractInterface("IMultiplier")
class Multiplier {
    public int multiply(int x, int y) {
        int total = 0;
        for (int i = 0; i < x; i++) {
            total = add(total, y);
        }
        return total;
    }

    private int add(int x, int y) {
        return x + y;
    }
}

class InterfaceExtractorProcessor implements AnnotationProcessor {
    private final AnnotationProcessorEnvironment env;
    private ArrayList<MethodDeclaration> interfaceMethods = new ArrayList<MethodDeclaration>();

    public InterfaceExtractorProcessor(AnnotationProcessorEnvironment e) {
        env = e;
    }

    public void process() {
        for (TypeDeclaration d : env.getSpecifiedTypeDeclarations()) {
            ExtractInterface a = d.getAnnotation(ExtractInterface.class);
            if (a == null) {
                break;
            }

            for (MethodDeclaration m : d.getMethods()) {
                if (m.getModifiers().contains(Modifier.PUBLIC) && 
                    !(m.getModifiers().contains(Modifier.STATIC))) {
                    interfaceMethods.add(m);
                }
            }

            if (interfaceMethods.size() > 0) {
                try {
                    PrintWriter writer = env.getFiler().createSourceFile(a.value());
                    writer.println("package " + d.getPackage().getQualifiedName() + ";");
                    writer.println("public interface " + a.value() + " {");
                    for (MethodDeclaration m : interfaceMethods) {
                        writer.print("    public ");
                        writer.print(m.getReturnType() + " ");
                        writer.print(m.getSimpleName() + " (");
                        int i = 0;
                        for (ParameterDeclaration param : m.getParameters()) {
                            writer.print(param.getType() + " " + param.getSimpleName());
                            if (++i < m.getParameters().size()) {
                                writer.print(", ");
                            }
                        }
                        writer.println(");");
                    }
                    writer.println("}");
                    writer.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

public class InterfaceExtractorProcessorFactory implements AnnotationProcessorFactory {
    public AnnotationProcessor getProcessorFor(Set<AnnotationTypeDeclaration> atds, AnnotationProcessorEnvironment env) {
        return new InterfaceExtractorProcessor(env);
    }

    public Collection<String> supportedAnnotationTypes() {
        return Collections.singleton("ExtractInterface");
    }

    public Collection<String> supportedOptions() {
        return Collections.emptySet();
    }
}
```

### 基于注解的单元测试

单元测试是对类中的每个方法提供一个或多个测试的一种实践，其目的是为了有规律地测试一个类中的各个部分是否具备正确的行为。


## 并发

```
class LiftOff implements Runnable {
    protected int countDown = 10;

    private static int taskCount = 0;
    private final int id = taskCount++;

    public LiftOff() {}

    public LiftOff(int countDown) {
        this.countDown = countDown;
    }

    public String status() {
        return "#" + id + "(" + (countDown > 0 ? countDown : "Liftoff!") + "), ";
    }

    public void run() {
        while (countDown-- > 0) {
            System.out.print(status());
            Thread.yield();
        }
    }
}

public class Demo {
    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 5; i++) {
            Thread t = new Thread(new LiftOff());
            t.start();
        }
    }
}

/*
#1(9), #4(9), #3(9), #2(9), #4(8), #2(8), #0(9), #2(7), #0(8), #4(7), #3(8), #1(8), #4(6), #3(7), #4(5), #0(7), #2(6), #4(4), #3(6), #1(7), #4(3), #3(5), #4(2), #3(4), #1(6), #2(5), #3(3), #0(6), #3(2), #2(4), #1(5), #4(1), #1(4), #2(3), #0(5), #3(1), #0(4), #1(3), #2(2), #4(Liftoff!), #2(1), #1(2), #0(3), #3(Liftoff!), #0(2), #1(1), #2(Liftoff!), #1(Liftoff!), #0(1), #0(Liftoff!), 
*/t
```

`Thread.yield()`的调用是对线程调度器（Java线程机制的一部分，可以将CPU从一个线程转移到另一个线程）的一种建议，它在声明：“我已经执行完生命周期中最重要的部分，此刻正是切换给其他任务执行一段时间的大好时机。”


### 使用Executor

```
import java.util.concurrent.*;

class LiftOff implements Runnable {
    protected int countDown = 5;

    private static int taskCount = 0;
    private final int id = taskCount++;

    public LiftOff() {}

    public LiftOff(int countDown) {
        this.countDown = countDown;
    }

    public String status() {
        String threadName = Thread.currentThread().getName();
        return threadName + "- #" + id + "(" + (countDown > 0 ? countDown : "Liftoff!") + "), ";
    }

    public void run() {
        while (countDown-- > 0) {
            System.out.println(status());
            Thread.yield();
        }
    }
}

public class Demo {
    public static void main(String[] args) throws Exception {
        ExecutorService es = Executors.newFixedThreadPool(2);
        for (int i = 0; i < 5; i++) {
            es.execute(new LiftOff());
        }
        es.shutdown();
    }
}

/*
pool-1-thread-2- #1(4), 
pool-1-thread-1- #0(4), 
pool-1-thread-2- #1(3), 
pool-1-thread-1- #0(3), 
pool-1-thread-2- #1(2), 
pool-1-thread-1- #0(2), 
pool-1-thread-2- #1(1), 
pool-1-thread-2- #1(Liftoff!), 
pool-1-thread-1- #0(1), 
pool-1-thread-1- #0(Liftoff!), 
pool-1-thread-2- #2(4), 
pool-1-thread-2- #2(3), 
pool-1-thread-1- #3(4), 
pool-1-thread-1- #3(3), 
pool-1-thread-2- #2(2), 
pool-1-thread-1- #3(2), 
pool-1-thread-1- #3(1), 
pool-1-thread-2- #2(1), 
pool-1-thread-1- #3(Liftoff!), 
pool-1-thread-2- #2(Liftoff!), 
pool-1-thread-1- #4(4), 
pool-1-thread-1- #4(3), 
pool-1-thread-1- #4(2), 
pool-1-thread-1- #4(1), 
pool-1-thread-1- #4(Liftoff!), 
*/
```

#### 从任务中产生返回值

Callable的类型参数表示的是从方法`call()`中返回的值，并且必须使用`ExecutorService.submit()`方法调用它。

```
import java.util.concurrent.*;
import java.util.*;

class TaskWithResult implements Callable<String> {
    private int id;

    public TaskWithResult(int id) {
        this.id = id;
    }

    public String call() {
        String threadName = Thread.currentThread().getName();
        return "result of TaskWithResult " + id + " thread name is " + threadName;
    }
}

public class Demo {
    public static void main(String[] args) throws Exception {
        ExecutorService es = Executors.newCachedThreadPool();
        ArrayList<Future<String>> results = new ArrayList<Future<String>>();
        for (int i = 0; i < 10; i++) {
            Future<String> fs = es.submit(new TaskWithResult(i));
            results.add(fs);
        }
        for (Future<String> fs : results) {
            try {
                System.out.println(fs.get());
            } catch (InterruptedException e) {
                System.out.println(e);
                return;
            } catch (ExecutionException e) {
                System.out.println(e);
                return;
            } finally {
                es.shutdown();
            }
        }
    }
}

/*
result of TaskWithResult 0 thread name is pool-1-thread-1
result of TaskWithResult 1 thread name is pool-1-thread-2
result of TaskWithResult 2 thread name is pool-1-thread-3
result of TaskWithResult 3 thread name is pool-1-thread-4
result of TaskWithResult 4 thread name is pool-1-thread-5
result of TaskWithResult 5 thread name is pool-1-thread-6
result of TaskWithResult 6 thread name is pool-1-thread-2
result of TaskWithResult 7 thread name is pool-1-thread-6
result of TaskWithResult 8 thread name is pool-1-thread-5
result of TaskWithResult 9 thread name is pool-1-thread-3
*/
```

上面的`Future`表示异步计算的结果。

#### 优先级

```
import java.util.concurrent.*;

public class Demo implements Runnable {
    private int countDown = 5;
    private volatile double d;
    private int priority;

    public Demo(int priority) {
        this.priority = priority;
    }

    public String toString() {
        String threadName = Thread.currentThread().getName();
        return Thread.currentThread() + " countDown: " + countDown + " priority: " + priority;
    }

    public void run() {
        Thread.currentThread().setPriority(priority);

        while (true) {
            for (int i = 1; i < 100000; i++) {
                d += (Math.PI + Math.E) / (double)i;
                if (i % 1000 == 0) {
                    Thread.yield();
                }
            }
            System.out.println(this);
            if (--countDown == 0) {
                return;
            }
        }
    }

    public static void main(String[] args) throws Exception {
        ExecutorService es = Executors.newCachedThreadPool();
        es.execute(new Demo(Thread.MAX_PRIORITY));
        for (int i = 0; i < 5; i++) {
            es.execute(new Demo(Thread.MIN_PRIORITY));
        }
        es.shutdown();
    }
}

/*
Thread[pool-1-thread-6,1,main] countDown: 5 priority: 1
Thread[pool-1-thread-3,1,main] countDown: 5 priority: 1
Thread[pool-1-thread-1,10,main] countDown: 5 priority: 10
Thread[pool-1-thread-2,1,main] countDown: 5 priority: 1
Thread[pool-1-thread-5,1,main] countDown: 5 priority: 1
Thread[pool-1-thread-4,1,main] countDown: 5 priority: 1
Thread[pool-1-thread-3,1,main] countDown: 4 priority: 1
Thread[pool-1-thread-1,10,main] countDown: 4 priority: 10
Thread[pool-1-thread-6,1,main] countDown: 4 priority: 1
Thread[pool-1-thread-5,1,main] countDown: 4 priority: 1
Thread[pool-1-thread-2,1,main] countDown: 4 priority: 1
Thread[pool-1-thread-4,1,main] countDown: 4 priority: 1
Thread[pool-1-thread-3,1,main] countDown: 3 priority: 1
Thread[pool-1-thread-6,1,main] countDown: 3 priority: 1
Thread[pool-1-thread-1,10,main] countDown: 3 priority: 10
Thread[pool-1-thread-5,1,main] countDown: 3 priority: 1
Thread[pool-1-thread-2,1,main] countDown: 3 priority: 1
Thread[pool-1-thread-4,1,main] countDown: 3 priority: 1
Thread[pool-1-thread-3,1,main] countDown: 2 priority: 1
Thread[pool-1-thread-6,1,main] countDown: 2 priority: 1
Thread[pool-1-thread-1,10,main] countDown: 2 priority: 10
Thread[pool-1-thread-5,1,main] countDown: 2 priority: 1
Thread[pool-1-thread-2,1,main] countDown: 2 priority: 1
Thread[pool-1-thread-4,1,main] countDown: 2 priority: 1
Thread[pool-1-thread-3,1,main] countDown: 1 priority: 1
Thread[pool-1-thread-6,1,main] countDown: 1 priority: 1
Thread[pool-1-thread-1,10,main] countDown: 1 priority: 10
Thread[pool-1-thread-5,1,main] countDown: 1 priority: 1
Thread[pool-1-thread-2,1,main] countDown: 1 priority: 1
Thread[pool-1-thread-4,1,main] countDown: 1 priority: 1
*/
```

从结果可以看出，线程的优先级仍然无法保障线程的执行次序。只不过，优先级高的线程获取CPU资源的概率较大，优先级低的并非没机会执行。

#### 后台线程

后台线程，是指在程序运行的时候在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分。

```
import java.util.concurrent.*;

public class Demo implements Runnable {
    public void run() {
        try {
            while (true) {
                TimeUnit.MILLISECONDS.sleep(10);
                System.out.println(Thread.currentThread() + " " + this);
            }
        } catch (InterruptedException e) {
            System.out.println("sleep() interrupted");
        }
    }

    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 10; i++) {
            Thread deamon = new Thread(new Demo());
            deamon.setDaemon(true);
            deamon.start();
        }
        System.out.println("all deamons started");
        TimeUnit.MILLISECONDS.sleep(15);
        System.out.println("main thread ended " + Thread.currentThread());
    }
}

/*
all deamons started
Thread[Thread-5,5,main] Demo@17141277
Thread[Thread-8,5,main] Demo@6577e002
Thread[Thread-0,5,main] Demo@29fa0068
Thread[Thread-2,5,main] Demo@c804073
Thread[Thread-6,5,main] Demo@142339a3
Thread[Thread-1,5,main] Demo@4c6f1c80
Thread[Thread-9,5,main] Demo@3786deeb
Thread[Thread-3,5,main] Demo@7f2ac5a7
Thread[Thread-4,5,main] Demo@5a67a4e
Thread[Thread-7,5,main] Demo@7502521d
main thread ended Thread[main,5,main]
*/
```




































































