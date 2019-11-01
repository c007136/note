# JAVA编程思想学习9
> 第20章 ～ 第21章

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

后台线程，是指在程序运行的时候在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分。因此，当所有的非后台线程结束时，程序也就终止了，同时会杀死进程中的所有后台线程。

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

使用ThreadFactory

```
import java.util.concurrent.*;

class DaemonThreadFactory implements ThreadFactory {
	public Thread newThread(Runnable r) {
		Thread t = new Thread(r);
		t.setDaemon(true);
		return t;
	}
}


public class Demo implements Runnable {
	public void run() {
		try {
			while (true) {
				TimeUnit.MILLISECONDS.sleep(10);
				System.out.println(Thread.currentThread() + " " + this);
			}
		} catch (InterruptedException e) {
			System.out.println("Interrupted");
		}
	}

	public static void main(String[] args) throws Exception {
		ExecutorService es = Executors.newCachedThreadPool(new DaemonThreadFactory());
		for (int i = 0; i < 10; i++) {
			es.execute(new Demo());
		}
		System.out.println("all deamons started");
        TimeUnit.MILLISECONDS.sleep(15);
        System.out.println("main thread ended " + Thread.currentThread());
	}
}


/*
all deamons started
Thread[Thread-1,5,main] Demo@76f1ebbf
Thread[Thread-7,5,main] Demo@7506f74a
Thread[Thread-9,5,main] Demo@4045ca95
Thread[Thread-3,5,main] Demo@14f67d7d
Thread[Thread-6,5,main] Demo@16d50866
Thread[Thread-5,5,main] Demo@fafdfea
Thread[Thread-0,5,main] Demo@a8f72d1
Thread[Thread-4,5,main] Demo@37ed46b2
Thread[Thread-2,5,main] Demo@2bc2d139
Thread[Thread-8,5,main] Demo@40e785a3
main thread ended Thread[main,5,main]
*/
```

### join线程
一个线程可以在其他线程之上调用`join()`方法，其效果是等待一段时间直到第二个线程结束才继续执行。

`join()`方法的调用可以被中断，调用`interrupt()`方法

参考`concurrency/Joining.java`

```
import java.util.concurrent.*;

public class Demo {
	public static void main(String[] args) {
		Thread previousThread = Thread.currentThread();
		for (int i = 0; i < 10; i++) {
			Thread currentThread = new JoinThread(previousThread);
			currentThread.start();
			previousThread = currentThread;
		}
	}

	private static class JoinThread extends Thread {
		private Thread thread;

		public JoinThread(Thread thread) {
			this.thread = thread;
		}

		@Override
		public void run() {
			try {
				thread.join();
				System.out.println(thread.getName() + " terminated.");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

/*
main terminated.
Thread-0 terminated.
Thread-1 terminated.
Thread-2 terminated.
Thread-3 terminated.
Thread-4 terminated.
Thread-5 terminated.
Thread-6 terminated.
Thread-7 terminated.
Thread-8 terminated.
*/
```


### 捕获异常

由于线程的本质特性，使得你不能捕获从线程逃逸的异常。一旦异常逃出任务的`run()`方法，它就会向外传播到控制台，哪怕在`main`主体中加入`try-catch`也是没有用的。

```
import java.util.concurrent.*;

public class Demo implements Runnable {
	public void run() {
		throw new RuntimeException();
	}

	public static void main(String[] args) throws Exception {
		try {
			ExecutorService es = Executors.newCachedThreadPool();
			es.execute(new Demo());
		} catch (RuntimeException e) {
			System.out.println("exception is being handled");
		}
	}
}
```

采用`setUncaughtExceptionHandler`，在程序中添加额外的跟踪机制

```
import java.util.concurrent.*;

class ExceptionThread implements Runnable {
  public void run() {
    Thread t = Thread.currentThread();
    System.out.println("run() by " + t);
    System.out.println("eh = " + t.getUncaughtExceptionHandler());
    throw new RuntimeException();
  }
}

class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
  public void uncaughtException(Thread t, Throwable e) {
    System.out.println("caught " + e);
  }
}

class HandlerThreadFactory implements ThreadFactory {
  public Thread newThread(Runnable r) {
    System.out.println(this + " creating new Thread");
    Thread t = new Thread(r);
    System.out.println("created " + t);
    t.setUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
    System.out.println(
      "eh = " + t.getUncaughtExceptionHandler());
    return t;
  }
}

public class Demo{
	public static void main(String[] args) throws Exception {
		ExecutorService exec = Executors.newCachedThreadPool(new HandlerThreadFactory());
		exec.execute(new ExceptionThread());
	}
}


/*
HandlerThreadFactory@5c647e05 creating new Thread
created Thread[Thread-0,5,main]
eh = MyUncaughtExceptionHandler@33909752
run() by Thread[Thread-0,5,main]
eh = MyUncaughtExceptionHandler@33909752
HandlerThreadFactory@5c647e05 creating new Thread
created Thread[Thread-1,5,main]
eh = MyUncaughtExceptionHandler@fafdfea
caught java.lang.RuntimeException
*/
```

### 共享受限资源

```
public abstract class IntGenerator {
	private volatile boolean canceled = false;

	public abstract int next();

	public void cancel() {
		canceled = true;
	}

	public boolean isCanceled() {
		return canceled;
	}
}
```

```
import java.util.concurrent.*;

public class EvenChecker implements Runnable {
	private IntGenerator generator;

	private final int id;

	public EvenChecker(IntGenerator g, int ident) {
		generator = g;
		id = ident;
	}

	public void run() {
		while (!generator.isCanceled()) {
			int val = generator.next();
			// 加上这句话就感觉可以无限循环了
			//System.out.println("id is " + id + " val is " + val + " current thread is " + Thread.currentThread());
			if (val % 2 != 0) {
				System.out.println("id is " + id + " val is " + val + " not even current thread is " + Thread.currentThread());
				generator.cancel();
			}
		}
	}

	public static void test(IntGenerator g, int count) {
		System.out.println("Press Conctol-C to exit");
		ExecutorService es = Executors.newCachedThreadPool();
		for (int i = 0; i < count; i++) {
			es.execute(new EvenChecker(g, i));
		}
		es.shutdown();
	}

	public static void test(IntGenerator g) {
		test(g, 10);
	}
}
```

```
import java.util.concurrent.*;

class EvenGenerator extends IntGenerator {
	private int currentEvenvalue = 0;

	public /*synchronized*/ int next() {
		++currentEvenvalue;      // Danger point here
		// Thread.yield();
		++currentEvenvalue;
		return currentEvenvalue;
	}
}


public class Demo {
	public static void main(String[] args) {
		EvenChecker.test(new EvenGenerator());
	}
}
```

一个任务有可能在另一个任务执行第一个对`currentEvenvalue`的递增操作之后，但是没有执行第二个操作之前，调用`next()`方法。这将使这个值处于“不恰当”的状态。

Brian的同步规则：

> 如果你正在写一个变量，它可能接下来将被另一个线程读取，或者正在读取一个上一次已经被另一个线程写过的变量，那么你必须使用同步，并且，读写线程都必须用相同的监视器同步。

### 使用显式的Lock对象


```
public class Demo implements Runnable {
	private int i = 0;
	public int getValue() { return i; }

	private synchronized void evenIncrement() {
		i++;
		i++;
	}

	public void run() {
		while (true) {
			evenIncrement();
		}
	}

	public static void main(String[] args) {
		ExecutorService es = Executors.newCachedThreadPool();
		Demo d = new Demo();
		es.execute(d);
		while (true) {
			int value = d.getValue();
			if (value % 2 != 0) {
				System.out.println(value);
				System.exit(0);
			}
		}
	}
}
```

### 原子性与易变性

原子性是指某个操作，要么让这个操作执行完，要么就不执行这个操作。

Java中对变量的读取和赋值都是原子操作，但long、double类型除外，只有使用volatile修饰之后long、double类型的读取和赋值操作才具有原子性。

易变性有两层含义：
1. 可见性
2. 有序性


#### 可见性

Java虚拟机会为每个线程分配一块专属的内存，称之为工作内存；不同的线程之间共享的数据会被放到主内存中。工作内存主要包含方法的参数、局部变量（在函数中定义的变量），这些变量都是线程私有的，不会被其他线程共用。实例的属性、类的静态属性都是可以被共享的，每个线程在操作这些数据都是先从主内存中读取到工作内存再进行操作，操作结束后再写入到主内存中。可见性要求线程对共享变量修改后立即写入到主内存中，线程读取共享变量时也必须去主内存中重新加载，不能直接使用工作内存中的值。


```
import java.util.concurrent.*;

class NewThread implements Runnable {
	public /*volatile*/ static long value;

	public void run() {
		while (Demo.run) {
			value++;
		}
		System.out.println("Done");
	}
}

public class Demo {
	public /*volatile*/ static boolean run = true;

	public static void main(String[] args) throws Exception {
		ExecutorService es = Executors.newCachedThreadPool();
		es.execute(new NewThread());
		es.shutdown();
		Thread.sleep(100);
		run = false;
		System.out.println("run: " + run);
		System.out.println("value: " + NewThread.value);
		Thread.sleep(100);
		System.out.println("value: " + NewThread.value);
	}
}
```

#### 有序性

易变性另一层含义就是有序性，是指禁止CPU对指令重排优化，默认情况下CPU会对指令进行合理的重排优化，重排优化仅保证单线程运行时结果的正确性，不保证执行顺序。

### 临界区

防止多个线程同时访问方法内部的部分代码而不是防止访问整个方法。这一部分代码被称为临界区。这也被称为同步控制块。

```
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import java.util.*;

class Pair {
	private int x;
	private int y;

	public Pair(int x, int y) {
		this.x = x;
		this.y = y;
	}

	public Pair() {
		this(0, 0);
	}

	public int getX() {
		return x;
	}

	public int getY() {
		return y;
	}

	public void incrementX() {
		x++;
	}

	public void incrementY() {
		y++;
	}

	public String toString() {
		return "x: " + x + ", y: " + y;
	}

	public void checkState() {
		if (x != y) {
			throw new PairValuesNotEqualException();
		}
	}

	public class PairValuesNotEqualException extends RuntimeException {
		public PairValuesNotEqualException() {
			super("Pair values not equal: " + Pair.this);
		}
	}
}

abstract class PairManager {
	AtomicInteger checkCounter = new AtomicInteger(0);

	protected Pair p = new Pair();

	private List<Pair> storage = Collections.synchronizedList(new ArrayList<Pair>());

	public synchronized Pair getPair() {
		return new Pair(p.getX(), p.getY());
	}

	protected void store(Pair p) {
		storage.add(p);

		try {
			TimeUnit.MILLISECONDS.sleep(50);
		} catch (InterruptedException e) {

		}
	}

	public abstract void increment();
}

class PairManager1 extends PairManager {
	public synchronized void increment() {
		p.incrementX();
		p.incrementY();
		store(getPair());
	}
}

class PairManager2 extends PairManager {
	// 同步代码块会比同步整个方法运行得多一些
	public void increment() {
		Pair temp;
		synchronized (this) {
			p.incrementX();
			p.incrementY();
			temp = getPair();
		}
		store(temp);
	}
}

class PairManipulator implements Runnable {
	private PairManager pm;

	public PairManipulator(PairManager pm) {
		this.pm = pm;
	}

	public void run() {
		while (true) {
			pm.increment();
		}
	}

	public String toString() {
		return "Pair: " + pm.getPair() + " checkCounter = " + pm.checkCounter.get();
	}
}

// 跟踪运行测试的频度
class PairChecker implements Runnable {
	private PairManager pm;

	public PairChecker(PairManager pm) {
		this.pm = pm;
	}

	public void run() {
		while (true) {
			pm.checkCounter.incrementAndGet();
			pm.getPair().checkState();
		}
	}
}


public class Demo {
	static void testApproaches(PairManager p1, PairManager p2) {
		ExecutorService es = Executors.newCachedThreadPool();

		PairManipulator pm1 = new PairManipulator(p1);
		PairManipulator pm2 = new PairManipulator(p2);

		PairChecker pc1 = new PairChecker(p1);
		PairChecker pc2 = new PairChecker(p2);

		es.execute(pm1);
		es.execute(pm2);
		es.execute(pc1);
		es.execute(pc2);

		es.shutdown();

		try {
			TimeUnit.MILLISECONDS.sleep(100);
		} catch (InterruptedException e) {
			System.out.println("sleep interrupted");
		}

		System.out.println("pm1: " + pm1 + "\npm2: " + pm2);
		System.exit(0);
	}


	public static void main(String[] args) {
		PairManager p1 = new PairManager1();
		PairManager p2 = new PairManager2();
		testApproaches(p1, p2);
	}
}


/*
pm1: Pair: x: 200, y: 200 checkCounter = 56
pm2: Pair: x: 201, y: 201 checkCounter = 669459870
*/
``` 

上面的`PairManager`类的结构，它的一些功能在基类中实现，并且其一个或多个抽象方法在派生类中定义，这种结构在设计模式中称为模板方法。

muyu: ？？？？？


## 在其他对象上同步

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.concurrent.atomic.*;
import java.util.*;

class DualSynch {
	private Object syncObject = new Object();

	public synchronized void f() {
		for (int i = 0; i < 5; i++) {
			System.out.println("f() " + Thread.currentThread());
			Thread.yield();
		}
	}

	public void g() {
		synchronized (syncObject) {
			for (int i = 0; i < 5; i++) {
				System.out.println("g() " + Thread.currentThread());
				Thread.yield();
			}
		}
	}
}


public class Demo {
	public static void main(String[] args) {
		final DualSynch ds = new DualSynch();
		new Thread() {
			public void run() {
				ds.f();
			}
		}.start();
		ds.g();
	}
}


/*
g() Thread[main,5,main]
f() Thread[Thread-0,5,main]
f() Thread[Thread-0,5,main]
g() Thread[main,5,main]
f() Thread[Thread-0,5,main]
g() Thread[main,5,main]
g() Thread[main,5,main]
f() Thread[Thread-0,5,main]
g() Thread[main,5,main]
f() Thread[Thread-0,5,main]
*/
```

`synchronized(this)`后，那么其他的`synchronized`方法和临界区就不能被调用了。

### 线程本地存储

```
import java.util.concurrent.*;
import java.util.*;

class Accessor implements Runnable {
	private final int id;

	public Accessor(int id) {
		this.id = id;
	}

	public void run() {
		boolean b = Thread.currentThread().isInterrupted();
		while(!b) {
			Demo.increment();
			System.out.println(this);
			Thread.yield();
		}
	}

	public String toString() {
		return "#" + id + ":" + Demo.get();
	}
}

public class Demo {
	private static ThreadLocal<Integer> value = new ThreadLocal<Integer>() {
		private Random rand = new Random(26);
		// 初始值
		protected Integer initialValue() {
			Integer i = rand.nextInt(100000);
			System.out.println("initialValue is " + i + " currentThread is " + Thread.currentThread());
			return i;
		}
	};

	public static void increment() {
		value.set(value.get() + 1);
	}

	public static int get() {
		return value.get();
	}

	public static void main(String[] args) throws Exception {
		ExecutorService es = Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) {
			es.execute(new Accessor(i));
		}
		TimeUnit.MILLISECONDS.sleep(5);  // Run for a while
		es.shutdownNow();              // All Accessors will quit
		System.exit(1);
	}
}


/*
initialValue is 65104 currentThread is Thread[pool-1-thread-2,5,main]
initialValue is 10493 currentThread is Thread[pool-1-thread-1,5,main]
#0:10494
initialValue is 1532 currentThread is Thread[pool-1-thread-5,5,main]
initialValue is 50364 currentThread is Thread[pool-1-thread-3,5,main]
#2:50365
initialValue is 77139 currentThread is Thread[pool-1-thread-4,5,main]
#4:1533
#0:10495
#1:65105
#0:10496
......
*/
```

ThreadLocal创建的变量只能给自己使用。

### 终止任务

```
import java.util.concurrent.*;
import java.util.*;

class Count {
	private int count = 0;
	private Random rand = new Random(26);

	public synchronized int increment() {
		int temp = count;
		if (rand.nextBoolean()) {
			Thread.yield();
		}
		return (count = ++temp);
	}

	public synchronized int value() {
		return count;
	}
}

class Entrance implements Runnable {
	private static Count count = new Count();
	private static List<Entrance> entrances = new ArrayList<Entrance>();
	private int number = 0;
	private final int id;
	private static volatile boolean canceled = false;

	public static void cancel() {
		canceled = true;
	}

	public Entrance(int id) {
		this.id = id;
		entrances.add(this);
	}

	public void run() {
		while (!canceled) {
			synchronized(this) {
				++number;
			}
			System.out.println(this + " Total: " + count.increment());
			try {
				TimeUnit.MILLISECONDS.sleep(100);
			} catch (InterruptedException e) {
				System.out.println("sleep interrupted");
			}
		}
		System.out.println("Stopping " + this);
	}

	public synchronized int getValue() {
		return number;
	}

	public String toString() {
		return "Entrance " + id + ": " + getValue();
	}

	public static int getTotalCount() {
		return count.value();
	}

	public static int sumEntrances() {
		int sum = 0;
		for (Entrance e : entrances) {
			sum += e.getValue();
		}
		return sum;
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		ExecutorService es = Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) {
			es.execute(new Entrance(i));
		}
		TimeUnit.SECONDS.sleep(3);
		Entrance.cancel();
		es.shutdown();
		if (!es.awaitTermination(250, TimeUnit.MILLISECONDS)) {
			System.out.println("Some tasks were not terminated!");
		}
		System.out.println("Total: " + Entrance.getTotalCount());
		System.out.println("Sum of Entrances: " + Entrance.sumEntrances());
	}
}
```

### 在阻塞时终结

进入阻塞状态的原因：

1. 调用`sleep`
2. 调用`wait`，直到线程得到`notify`或`notifyAll`
3. 任务在等待某个输入/输出完成
4. 任务试图在某个对象上调用同步控制方法，但是对象锁不可用，因为另一个任务已经获取了锁

中断：Future对象可以中断线程

```
import java.util.concurrent.*;
import java.io.*;

class SleepBlocked implements Runnable {
	public void run() {
		try {
			TimeUnit.SECONDS.sleep(100);
		} catch (InterruptedException e) {
			System.out.println("SleepBlocked InterruptedException");
		}
		System.out.println("Exiting SleepBlocked.run()");
	}
}

class IOBlocked implements Runnable {
	private InputStream in;

	public IOBlocked(InputStream in) {
		this.in = in;
	}

	public void run() {
		try {
			System.out.println("Waiting for read():");
			in.read();
		} catch (IOException e) {
			if(Thread.currentThread().isInterrupted()) {
				System.out.println("Interrupted from blocked I/O");
			} else {
				throw new RuntimeException(e);
			}
		}
		System.out.println("Exiting IOBlocked.run()");
	}
}

class SynchronizedBlocked implements Runnable {
	public synchronized void f() {
		while (true) {
			Thread.yield();
		}
	}

	public SynchronizedBlocked() {
		new Thread() {
			public void run() {
				f();
			}
		}.start();
	}

	public void run() {
		System.out.println("Trying to call f()");
		f();
		System.out.println("Exiting SynchronizedBlocked.run()");
	}
}

public class Demo {
	private static ExecutorService es = Executors.newCachedThreadPool();

	public static void test(Runnable r) throws InterruptedException {
		Future<?> f = es.submit(r);
		TimeUnit.MILLISECONDS.sleep(100);
		System.out.println("Interrupting " + r.getClass().getName());
		f.cancel(true);
		System.out.println("Interrupt sent to " + r.getClass().getName());
	}


	public static void main(String[] args) throws Exception {
		test(new SleepBlocked());
		test(new IOBlocked(System.in));
		test(new SynchronizedBlocked());
		TimeUnit.SECONDS.sleep(3);
		System.out.println("Aborting with System.exit(0)");
		System.exit(0);
	}
}


/*
Interrupting SleepBlocked
Interrupt sent to SleepBlocked
SleepBlocked InterruptedException
Exiting SleepBlocked.run()
Waiting for read():
Interrupting IOBlocked
Interrupt sent to IOBlocked
Trying to call f()
Interrupting SynchronizedBlocked
Interrupt sent to SynchronizedBlocked
Aborting with System.exit(0)
*/
```

从输出中可以看出，你能够中断`sleep()`的调用，但不能中断`synchronized `和I/O操作的线程，这意味着I/O操作具有锁住你的多线程的潜在可能。

```
// IO是阻塞的，NIO是非阻塞的

import java.util.concurrent.*;
import java.io.*;
import java.net.*;
import java.nio.*;
import java.nio.channels.*;

class NIOBlocked implements Runnable {
	private final SocketChannel sc;

	public NIOBlocked(SocketChannel sc) {
		this.sc = sc;
	}

	public void run() {
		try {
			System.out.println("Waiting for read() in " + this);
			sc.read(ByteBuffer.allocate(1));
		} catch (ClosedByInterruptException e) {
			System.out.println("ClosedByInterruptException");
		} catch (AsynchronousCloseException e) {
			System.out.println("AsynchronousCloseException");
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
		System.out.println("Exiting NIOBlocked.run() " + this);
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		ExecutorService es = Executors.newCachedThreadPool();
		ServerSocket ss = new ServerSocket(8080);
		InetSocketAddress isa = new InetSocketAddress("localhost", 8080);
		SocketChannel sc1 = SocketChannel.open(isa);
		SocketChannel sc2 = SocketChannel.open(isa);
		Future<?> f = es.submit(new NIOBlocked(sc1));
		es.execute(new NIOBlocked(sc2));
		es.shutdown();
		TimeUnit.SECONDS.sleep(1);
		f.cancel(true);
		TimeUnit.SECONDS.sleep(1);
		sc2.close();
	}
}


/*
Waiting for read() in NIOBlocked@7eb4afb1
Waiting for read() in NIOBlocked@5645c898
ClosedByInterruptException
Exiting NIOBlocked.run() NIOBlocked@7eb4afb1
AsynchronousCloseException
Exiting NIOBlocked.run() NIOBlocked@5645c898
*/
```

### 被互斥所阻塞

下面的示例说明了同一个互斥可以被同一个任务多次获得。

```
public class Demo {
	public synchronized void f1(int count) {
		if (count-- > 0) {
			System.out.println("f1() calling f2() with count " + count);
			f2(count);
		}
	}

	public synchronized void f2(int count) {
		if (count-- > 0) {
			System.out.println("f2() calling f1() with count " + count);
			f1(count);
		}
	}


	public static void main(String[] args) throws Exception {
		final Demo multiLock = new Demo();
		new Thread() {
			public void run() {
				multiLock.f1(10);
			}
		}.start();
	}
}


/*
f1() calling f2() with count 9
f2() calling f1() with count 8
f1() calling f2() with count 7
f2() calling f1() with count 6
f1() calling f2() with count 5
f2() calling f1() with count 4
f1() calling f2() with count 3
f2() calling f1() with count 2
f1() calling f2() with count 1
f2() calling f1() with count 0
*/
```

**非多线程**就是按顺序执行啊？？？

### Lock是可被中断的

```
import java.util.concurrent.*;
import java.util.concurrent.locks.*;

class BlockedMutex {
	private Lock lock = new ReentrantLock();

	public BlockedMutex() {
		lock.lock();
	}

	public void f() {
		try {
			lock.lockInterruptibly();
			System.out.println("lock acquired in f()");
		} catch (InterruptedException e) {
			System.out.println("Interrupted from lock acquisition in f()");
		}
	}
}

class Blocked implements Runnable {
	BlockedMutex blocked = new BlockedMutex();

	public void run() {
		System.out.println("Waiting for f() in BlockedMutex");
		blocked.f();
		System.out.println("Broken out of blocked call");
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		Thread t = new Thread(new Blocked());
		t.start();
		TimeUnit.SECONDS.sleep(1);
		System.out.println("Issuing t.interrupt()");
		t.interrupt();
	}
}


/*
Waiting for f() in BlockedMutex
Issuing t.interrupt()
Interrupted from lock acquisition in f()
Broken out of blocked call
*/
```

### wait()与notifyAll()

```
import java.util.concurrent.*;

class Car {
	private boolean waxOn = false;

	public synchronized void waxed() {
		waxOn = true;
		notifyAll();
	}

	public synchronized void buffed() {
		waxOn = false;
		notifyAll();
	}

	public synchronized void waitForWaxing() throws InterruptedException {
		while (waxOn == false) {
			wait();
		}
	}

	public synchronized void waitForBuffing() throws InterruptedException {
		while (waxOn == true) {
			wait();
		}
	}
}

class WaxOn implements Runnable {
	private Car car;

	public WaxOn(Car c) {
		car = c;
	}

	public void run() {
		try {
			while (!Thread.interrupted()) {
				System.out.println("Wax On! ");
				TimeUnit.MILLISECONDS.sleep(200);
				car.waxed();
				car.waitForBuffing();
			}
		} catch (InterruptedException e) {
			System.out.println("Exiting via interrupt");
		}
		System.out.println("Ending Wax On task");
	}
}

class WaxOff implements Runnable {
	private Car car;

	public WaxOff(Car c) {
		car = c;
	}

	public void run() {
		try {
			while (!Thread.interrupted()) {
				car.waitForWaxing();
				System.out.println("Wax Off! ");
				TimeUnit.MILLISECONDS.sleep(200);
				car.buffed();
			}
		} catch (InterruptedException e) {
			System.out.println("Exiting via interrupt");
		}
		System.out.println("Ending Wax Off task");
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		Car car = new Car();
		ExecutorService es = Executors.newCachedThreadPool();
		es.execute(new WaxOff(car));
		es.execute(new WaxOn(car));
		TimeUnit.SECONDS.sleep(5);
		es.shutdown();
	}
}


/*
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
......
*/
```

### notify和notifyAll

```
import java.util.concurrent.*;
import java.util.*;

class Blocker {
	synchronized void waitingCall() {
		try {
			while (!Thread.interrupted()) {
				wait();
				System.out.print(Thread.currentThread() + " ");
			}
		} catch (InterruptedException e) {

		}
	}

	synchronized void prod() {
		notify();
	}

	synchronized void prodAll() {
		notifyAll();
	}
}

class Task implements Runnable {
	static Blocker blocker = new Blocker();

	public void run() {
		blocker.waitingCall();
	}
}

class Task2 implements Runnable {
	static Blocker blocker = new Blocker();

	public void run() {
		blocker.waitingCall();
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		ExecutorService es = Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) {
			es.execute(new Task());
		}
		es.execute(new Task2());

		Timer timer = new Timer();
		timer.scheduleAtFixedRate(new TimerTask() {
			boolean prod = true;
			public void run() {
				if (prod) {
					System.out.print("\nnotify() ");
					Task.blocker.prod();
					prod = false;
				} else {
					System.out.print("\nnotifyAll() ");
					Task.blocker.prodAll();
					prod = true;
				}
			}
		}, 400, 400);
		TimeUnit.SECONDS.sleep(5);
		timer.cancel();
		System.out.println("\nTimer canceled");
		TimeUnit.MILLISECONDS.sleep(500);
		System.out.print("Task2.blocker.prodAll() ");
		Task2.blocker.prodAll();
		TimeUnit.MILLISECONDS.sleep(500);
		System.out.print("\nShutting down");
		es.shutdown();
		System.exit(0);
	}
}


/*

notify() Thread[pool-1-thread-1,5,main] 
notifyAll() Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-5,5,main] Thread[pool-1-thread-4,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-2,5,main] 
notify() Thread[pool-1-thread-1,5,main] 
notifyAll() Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-4,5,main] Thread[pool-1-thread-5,5,main] 
notify() Thread[pool-1-thread-1,5,main] 
notifyAll() Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-5,5,main] Thread[pool-1-thread-4,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-2,5,main] 
notify() Thread[pool-1-thread-1,5,main] 
notifyAll() Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-4,5,main] Thread[pool-1-thread-5,5,main] 
notify() Thread[pool-1-thread-1,5,main] 
notifyAll() Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-5,5,main] Thread[pool-1-thread-4,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-2,5,main] 
notify() Thread[pool-1-thread-1,5,main] 
notifyAll() Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-4,5,main] Thread[pool-1-thread-5,5,main] 
Timer canceled
Task2.blocker.prodAll() Thread[pool-1-thread-6,5,main] 
*/
```

### 生产者与消费者

#### wait/notify方案

```
//Restaurant.java

import java.util.concurrent.*;
import java.util.*;

class Meal {
	private final int orderNum;

	public Meal(int orderNum) {
		this.orderNum = orderNum;
	}

	public String toString() {
		return "Meal " + orderNum;
	}
}

class WaitPerson implements Runnable {
	private Restaurant restaurant;

	public WaitPerson(Restaurant r) {
		restaurant = r;
	}

	public void run() {
		try {
			while (!Thread.interrupted()) {
				synchronized (this) {
					while (restaurant.meal == null) {
						wait();
					}
				}
				System.out.println("Waitperson got " + restaurant.meal);
				synchronized (restaurant.chef) {
					restaurant.meal = null;
					restaurant.chef.notifyAll();
				}
			}
		} catch (InterruptedException e) {
			System.out.println("WaitPerson interrupted");
		}
	}
}

class Chef implements Runnable {
	private Restaurant restaurant;
	private int count = 0;

	public Chef(Restaurant r) {
		restaurant = r;
	}

	public void run() {
		try {
			while (!Thread.interrupted()) {
				synchronized (this) {
					while (restaurant.meal != null) {
						wait();
					}
				}
				if (++count == 10) {
					System.out.println("Out of food, closing");
					restaurant.es.shutdownNow();
				}
				System.out.println("Order up! ");
				synchronized (restaurant.waitPerson) {
					restaurant.meal = new Meal(count);
					restaurant.waitPerson.notifyAll();
				}
				TimeUnit.MILLISECONDS.sleep(100);
			}
		} catch (InterruptedException e) {
			System.out.println("Chef interrupted");
		}
	}
}

class Restaurant {
	Meal meal;
	ExecutorService es = Executors.newCachedThreadPool();
	WaitPerson waitPerson = new WaitPerson(this);
	Chef chef = new Chef(this);

	public Restaurant() {
		es.execute(chef);
		es.execute(waitPerson);
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		new Restaurant();
	}
}


/*
Order up! 
Waitperson got Meal 1
Order up! 
Waitperson got Meal 2
Order up! 
Waitperson got Meal 3
Order up! 
Waitperson got Meal 4
Order up! 
Waitperson got Meal 5
Order up! 
Waitperson got Meal 6
Order up! 
Waitperson got Meal 7
Order up! 
Waitperson got Meal 8
Order up! 
Waitperson got Meal 9
Out of food, closing
Order up! 
WaitPerson interrupted
Chef interrupted
*/
```

### Lock和Condition对象

```
// WaxOMatic2.java

import java.util.concurrent.*;
import java.util.concurrent.locks.*;

class Car {
	private boolean waxOn = false;
	private Lock lock = new ReentrantLock();
	private Condition condition = lock.newCondition();

	public void waxed() {
		lock.lock();
		try {
			waxOn = true;
			condition.signalAll();
		} finally {
			lock.unlock();
		}
	}

	public void buffed() {
		lock.lock();
		try {
			waxOn = false;
			condition.signalAll();
		} finally {
			lock.unlock();
		}
	}

	public void waitForWaxing() throws InterruptedException {
		lock.lock();
		try {
			while (waxOn == false) {
				condition.await();
			}
		} finally {
			lock.unlock();
		}
	}

	public void waitForBuffing() throws InterruptedException {
		lock.lock();
		try {
			while (waxOn == true) {
				condition.await();
			}
		} finally {
			lock.unlock();
		}
	}
}

class WaxOn implements Runnable {
	private Car car;

	public WaxOn(Car c) {
		car = c;
	}

	public void run() {
		try {
			while (!Thread.interrupted()) {
				System.out.println("Wax On! ");
				TimeUnit.MILLISECONDS.sleep(200);
				car.waxed();
				car.waitForBuffing();
			}
		} catch (InterruptedException e) {
			System.out.println("Exiting via interrupt");
		}
		System.out.println("Ending Wax On task");
	}
}

class WaxOff implements Runnable {
	private Car car;

	public WaxOff(Car c) {
		car = c;
	}

	public void run() {
		try {
			while (!Thread.interrupted()) {
				car.waitForWaxing();
				System.out.println("Wax Off! ");
				TimeUnit.MILLISECONDS.sleep(200);
				car.buffed();
			}
		} catch (InterruptedException e) {
			System.out.println("Exiting via interrupt");
		}
		System.out.println("Ending Wax Off task");
	}
}

public class Demo {
	public static void main(String[] args) throws Exception {
		Car car = new Car();
		ExecutorService es = Executors.newCachedThreadPool();
		es.execute(new WaxOff(car));
		es.execute(new WaxOn(car));
		TimeUnit.SECONDS.sleep(5);
		es.shutdownNow();
	}
}


/*
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Wax Off! 
Wax On! 
Exiting via interrupt
Ending Wax Off task
Exiting via interrupt
Ending Wax On task
*/
```

### 生产者-消费者与队列

LinkedBlockingQueue、ArrayBlockingQueue和SynchronousQueue

```
// TestBlockingQueues.java

import java.util.concurrent.*;
import java.io.*;

class LiftOffRunner implements Runnable {
	private BlockingQueue<LiftOff> rockets;

	public LiftOffRunner(BlockingQueue<LiftOff> queue) {
		rockets = queue;
	}

	public void add(LiftOff lo) {
		try {
			rockets.put(lo);
		} catch(InterruptedException e) {
	        System.out.println("Interrupted during put()");
	    }
	}

	public void run() {
		try {
			while (!Thread.interrupted()) {
				LiftOff rocket = rockets.take();
				rocket.run();
			}
		} catch(InterruptedException e) {
	        System.out.println("Interrupted during run()");
	    }
	    System.out.println("Exiting LiftOffRunner");
	}
}

public class Demo {
	static void getkey() {
		try {
			new BufferedReader(new InputStreamReader(System.in)).readLine();
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	static void getkey(String message) {
		System.out.println(message);
		getkey();
	}

	static void test(String msg, BlockingQueue<LiftOff> queue) {
		System.out.println(msg);
		LiftOffRunner runner = new LiftOffRunner(queue);
		Thread t = new Thread(runner);
		t.start();
		for (int i = 0; i < 5; i++) {
			runner.add(new LiftOff(5));
		}
		getkey("Press 'Enter' (" + msg + ")");
		t.interrupt();
		System.out.println("Finished " + msg + " test");
	}


	public static void main(String[] args) throws Exception {
		test("LinkedBlockingQueue", new LinkedBlockingQueue<LiftOff>());
		test("ArrayBlockingQueue", new ArrayBlockingQueue<LiftOff>(3));
		test("SynchronousQueue", new SynchronousQueue<LiftOff>());
	}
}


/*
*/
```

### 死锁

































































