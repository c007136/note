# JAVA编程思想学习八
> 第18章 ～ 第19章

## 第18章：Java I/O系统

### File类

File类，它既能代表一个特定文件的名称，又能代表一个目录下的一组文件的名称。

```
// 输入：java Demo \\w+\\.java
// 
import java.util.regex.*;
import java.io.*;
import java.util.*;

public class Demo {
  public static void main(String[] args) {
    File path = new File(".");
    String[] list;
    if (args.length == 0) {
    	list = path.list();
    } else {
    	System.out.println("arg is " + args[0]);
    	list = path.list(new DirFilter(args[0]));
    }
    Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
    System.out.println("----------");
    for (String dirItem : list) {
    	System.out.println("dirItem is " + dirItem);
    }
  }
}

// 策略模式的运用
class DirFilter implements FilenameFilter {
	private Pattern pattern;
	public DirFilter(String regex) {
		pattern = Pattern.compile(regex);
		System.out.println("pattern is " + pattern);
	}

	public boolean accept(File dir, String name) {
		System.out.println("dir is " + dir + " name is " + name);
		return pattern.matcher(name).matches();
	}
}
```

```
import java.util.regex.*;
import java.io.*;
import java.util.*;

public class Demo {
  public static void main(String[] args) {
    File currentDir = new File("./");
    String[] fileNames = currentDir.list((file, name) -> name.endsWith(".java"));
    for (String fileName : fileNames) {
    	System.out.println(fileName);
    }
  }
}
```

### 封装为工具类

```
import java.util.*;

public class PPrint {
  public static String pformat(Collection<?> c) {
    if(c.size() == 0) return "[]";
    StringBuilder result = new StringBuilder("[");
    for(Object elem : c) {
      if(c.size() != 1)
        result.append("\n  ");
      result.append(elem);
    }
    if(c.size() != 1)
      result.append("\n");
    result.append("]");
    return result.toString();
  }
  public static void pprint(Collection<?> c) {
    System.out.println(pformat(c));
  }
  public static void pprint(Object[] c) {
    System.out.println(pformat(Arrays.asList(c)));
  }
}
```

```
import java.util.regex.*;
import java.io.*;
import java.util.*;

public final class Directory {
	public static File[] local(File dir, final String regex) {
		return dir.listFiles(new FilenameFilter() {
			private Pattern pattern = Pattern.compile(regex);
			public boolean accept(File dir, String name) {
				return pattern.matcher(new File(name).getName()).matches();
			}
		});
	}

	public static File[] local(String path, final String regex) {
		return local(new File(path), regex);
	}

	public static class TreeInfo implements Iterable<File> {
		public List<File> files = new ArrayList<File>();
		public List<File> dirs = new ArrayList<File>();

		public Iterator<File> iterator() {
			return files.iterator();
		}

		void addAll(TreeInfo other) {
			files.addAll(other.files);
			dirs.addAll(other.dirs);
		}

		public String toString() {
			return "dirs: " + PPrint.pformat(dirs) + "\n\nfiles: " + PPrint.pformat(files);
		}
	}

	public static TreeInfo walk(File start, String regex) {
		return recurseDirs(start, regex);
	}

	public static TreeInfo walk(String start) {
		return recurseDirs(new File(start), ".*");
	}

	static TreeInfo recurseDirs(File startDir, String regex) {
		if (startDir.listFiles().length == 0) {
			System.out.println("startDir is wrong directory");
			return null;
		}

		TreeInfo result = new TreeInfo();
		for (File item : startDir.listFiles()) {
			if (item.isDirectory()) {
				result.dirs.add(item);
				result.addAll(recurseDirs(item, regex));
			} else {
				if (item.getName().matches(regex)) {
					result.files.add(item);
				}
			}
		}
		return result;
	}

	public static void main(String[] args) {
		if (args.length == 0) {
			System.out.println(walk("."));
		} else {
			for (String arg : args) {
				System.out.println(walk(arg));
			}
		}
	}
}
```

```
import java.util.regex.*;
import java.io.*;
import java.util.*;

public class Demo {
  public static void main(String[] args) {
    PPrint.pprint(Directory.walk("./").dirs);
  }
}
```

### 输入和输出

编程语言的I/O类库中常使用流这个抽象概念，它代表任何有能力产出数据的数据源对象或者是有能力接收数据的接收端对象。“流”屏蔽了实际的I/O设备中处理数据的细节。



## 第19章：枚举类型

关键字enum可以将一组具名的值的有限集合创建为一种新的类型，而这些具名的值可以作为常规的程序组件使用。

### 向enum中添加新方法

除了不能继承自enum之外，我们基本上可以将enum看作一个常规的类。也就是说，我们可以向enum中添加方法，enum甚至可以有main方法。

```
enum OzWitch {
	WEST("W_E_S_T"),
	NORTH("N_O_R_T_H"),
	EAST("E_A_S_T"),
	SOUTH("S_O_U_T_H");

	private String description;

	private OzWitch(String des) {
		description = des;
	}

	public String getDescription() {
		return description;
	}
}

public class Demo {
  public static void main(String[] args) {
  	for (OzWitch w : OzWitch.values()) {
  		System.out.println("w is " + w.getDescription());
  	}
  }
}
```

### values()的神秘之处

```
import java.lang.reflect.*;
import java.util.*;

enum Explore {
	HERE, 
	THERE
}

public class Demo {
	public static Set<String> analyze(Class<?> enumClass) {
		System.out.println("----- Analyzing " + enumClass + " -----");
		System.out.println("Interfaces:");
		for (Type t : enumClass.getGenericInterfaces()) {
			System.out.println(t);
		}
		System.out.println("Base: " + enumClass.getSuperclass());
		System.out.println("Methods:");
		Set<String> methods = new TreeSet<String>();
		for(Method m : enumClass.getMethods()) {
			methods.add(m.getName());
		}
		System.out.println(methods);
		return methods;
	}


  public static void main(String[] args) {
  	Set<String> exploreMethods = analyze(Explore.class);
  	Set<String> enumMethods = analyze(Enum.class);
  	exploreMethods.removeAll(enumMethods);
  	System.out.println(exploreMethods);
  	OSExecute.command("javap Explore");
  }
}
```

`values()`是由编译器添加的static方法，`valueOf()`其实也是。

### implements

可以实现一个或多个接口

### 随机选取

```
import java.util.*;

class Enums {
	private static Random rand = new Random(47);
	public static <T extends Enum<T>> T random(Class<T> ec) {
		return random(ec.getEnumConstants());
	}
	public static <T> T random(T[] values) {
		return values[rand.nextInt(values.length)];
	}
}

enum Activity { SITTING, LYING, STANDING, HOPPING,
  RUNNING, DODGING, JUMPING, FALLING, FLYING }

public class Demo {
  public static void main(String[] args) {
  	for (int i = 0; i < 20; i++) {
  		System.out.print(Enums.random(Activity.class) + " ");
  	}
  }
}
```

### 使用接口组织枚举

对于enum而言，实现接口是使其子类化的唯一办法。

```
interface Base {
	enum A implements Base {
		A1,
		A2,
		A3
	}

	enum B implements Base {
		B1,
		B2,
		B3
	}
}

public class Demo {
  public static void main(String[] args) {
  	Base b = Base.A.A1;
  	b = Base.B.B1;
  }
}
```

上面的程序，我们可以说“所有东西都是某种类型的Base”。

下面的例子实现“枚举的枚举”

```
interface Base {
	enum A implements Base {
		A1,
		A2,
		A3
	}

	enum B implements Base {
		B1,
		B2,
		B3
	}

	enum C implements Base {
		C1,
		C2,
		C3
	}

	enum D implements Base {
		D1,
		D2,
		D3
	}
}

enum E {
	AA(Base.A.class),
	BB(Base.B.class),
	CC(Base.C.class),
	DD(Base.D.class);

	private Base[] values;
	private E(Class<? extends Base> kind) {
		values = kind.getEnumConstants();
	}

	public Base randomSelection() {
		return Enums.random(values);
	}
}

public class Demo {
  public static void main(String[] args) {
  	for (int i = 0; i < 5; i++) {
		for (E e : E.values()) {
			Base base = e.randomSelection();
			System.out.println(base);
		}
		System.out.println("---");
  	}
  }
}
```

整理下代码

```
enum E {
	AA(Base.A.class),
	BB(Base.B.class),
	CC(Base.C.class),
	DD(Base.D.class);

	private Base[] values;
	private E(Class<? extends Base> kind) {
		values = kind.getEnumConstants();
	}

	interface Base {
		enum A implements Base {
			A1,
			A2,
			A3
		}

		enum B implements Base {
			B1,
			B2,
			B3
		}

		enum C implements Base {
			C1,
			C2,
			C3
		}

		enum D implements Base {
			D1,
			D2,
			D3
		}
	}

	public Base randomSelection() {
		return Enums.random(values);
	}	
}

public class Demo {
  public static void main(String[] args) {
  	for (int i = 0; i < 5; i++) {
		for (E e : E.values()) {
			E.Base base = e.randomSelection();
			System.out.println(base);
		}
		System.out.println("---");
  	}
  }
}
```

### 使用EnumSet替代标志

基本用法

```
import java.util.*;

enum A {
	STAIR1,
	STAIR2,
	LOBBY,
	OFFICE1,
	OFFICE2,
	OFFICE3,
	OFFICE4,
	BATHROOM,
	UTILITY,
	KITCHEN
}

public class Demo {
	public static void main(String[] args) {
		EnumSet<A> points = EnumSet.noneOf(A.class);
		points.add(A.BATHROOM);
		System.out.println(points);
		points.addAll(EnumSet.of(A.STAIR1, A.STAIR2, A.KITCHEN));
		System.out.println(points);
		points = EnumSet.allOf(A.class);
		points.removeAll(EnumSet.of(A.STAIR1, A.STAIR2, A.KITCHEN));
		System.out.println(points);
		points.removeAll(EnumSet.range(A.OFFICE1, A.OFFICE4));
		System.out.println(points);
		points = EnumSet.complementOf(points);
		System.out.println(points);
	}
}
```

### 使用EnumMap

EnumMap是一种特殊的Map，它要求其中的键（key）必须来自一个enum。由于enum本身的限制，所以EnumMap在内部可由数组实现。

下面的例子演示了命令模式的用法。一般来说，命令模式首先需要一个只有单一方法的接口，然后从该接口实现具有各自不同的行为的多个子类。

```
import java.util.*;

enum A {
	STAIR1,
	STAIR2,
	LOBBY,
	OFFICE1,
	OFFICE2,
	OFFICE3,
	OFFICE4,
	BATHROOM,
	UTILITY,
	KITCHEN
}

interface Command {
	void action();
}

public class Demo {
	public static void main(String[] args) {
		EnumMap<A, Command> em = new EnumMap<A, Command>(A.class);
		em.put(A.KITCHEN, new Command() {
			public void action() {
				System.out.println("Kitchen fire");
			}
		});

		em.put(A.BATHROOM, new Command() {
			public void action() {
				System.out.println("Bathroom alert");
			}
		});

		for(Map.Entry<A, Command> e : em.entrySet()) {
			System.out.print(e.getKey() + " : ");
			e.getValue().action();
		}
	}
}
```

### 常量相关的方法

```
import java.util.*;
import java.text.*;

enum A {
	DATE_TIME {
		String getInfo() {
			return DateFormat.getDateInstance().format(new Date());
		}
	},
	CLASSPATH {
		String getInfo() {
			return System.getenv("CLASSPATH");
		}
	},
	VERSION {
		String getInfo() {
			return System.getProperty("java.version");
		}
	};

	abstract String getInfo();
}

public class Demo {
	public static void main(String[] args) {
		for (A a : A.values()) {
			System.out.println(a.getInfo());
		}
	}
}
```

上面的代码也称为**表驱动的代码（driven code）**，其本质是：从表里查询信息来代替逻辑语句。

命令模式的另外一种实现方式。

对上面的代码执行`javap -c A`会得到如下结果：

```
public static final A DATE_TIME;

public static final A CLASSPATH;

public static final A VERSION;

......
```

### 洗车的例子

```
import java.util.*;

class CarWash {
  public enum Cycle {
    UNDERBODY {
      void action() { System.out.println("Spraying the underbody"); }
    },
    WHEELWASH {
      void action() { System.out.println("Washing the wheels"); }
    },
    PREWASH {
      void action() { System.out.println("Loosening the dirt"); }
    },
    BASIC {
      void action() { System.out.println("The basic wash"); }
    },
    HOTWAX {
      void action() { System.out.println("Applying hot wax"); }
    },
    RINSE {
      void action() { System.out.println("Rinsing"); }
    },
    BLOWDRY {
      void action() { System.out.println("Blowing dry"); }
    };
    abstract void action();
  }

  EnumSet<Cycle> cycles = EnumSet.of(Cycle.BASIC, Cycle.RINSE);

  public void add(Cycle cycle) { 
  	cycles.add(cycle); 
  }

  public void washCar() {
    for(Cycle c : cycles)
      c.action();
  }

  public String toString() {
  	return cycles.toString();
  }
}

public class Demo {
	public static void main(String[] args) {
		CarWash wash = new CarWash();
    	System.out.println(wash);
    	wash.washCar();
    	wash.add(CarWash.Cycle.BLOWDRY);
    	wash.add(CarWash.Cycle.BLOWDRY); // Duplicates ignored
    	wash.add(CarWash.Cycle.RINSE);
    	wash.add(CarWash.Cycle.HOTWAX);
    	System.out.println(wash);
    	wash.washCar();
	}
}
```

匿名函数的可能实现方法：

```
import java.util.*;

class IInter {
	public IInter(String s) {
		text = s;
	}

	public String value() {
		return text;
	}

	private String text;

	public void action() {

	}
}

class Cycle {
	public IInter washAction(String s) {
		return new IInter(s) {
			public void action() {
				System.out.println(super.value());
			}
		};
	}
}

class CarWash {
	static private Cycle c = new Cycle();
	
	static final IInter BASIC = c.washAction("The basic wash");
	static final IInter RINSE = c.washAction("Rinsing");
	static final IInter HOTWAX = c.washAction("Applying hot wax");

	static public Set<IInter> set = new HashSet();
	
	public void add(IInter i) {
		set.add(i);
	}
}


public class Demo {
	public static void main(String[] args) {
		CarWash wash = new CarWash();
		wash.add(CarWash.BASIC);
		wash.add(CarWash.RINSE);
		wash.add(CarWash.HOTWAX);

		for (IInter i : wash.set) {
			i.action();
		}
	}
}
```

### 覆盖方法

```
enum E {
	A,
	B,
	C {
		void f() {
			System.out.println("overridden method");
		}
	};

	void f() {
		System.out.println("default method");
	}
}


public class Demo {
	public static void main(String[] args) {
		for (A a : A.values()) {
			System.out.print(a + ": ");
			a.f();
		}
	}
}
```

### 使用enum的职责链

在职责链（Chain of Responsibility）设计模式中，程序员以多种不同的方式来解决一个问题，然后将它们链接在一起。当一个请求到来时，它遍历这个链，直到链中的某个解决方案能够处理该请求。

我们以一个邮局的模型为例。邮局需要以尽可能通用的方式来处理每一封邮件，并且要不断尝试处理邮件，直到该邮件最终被确定为一封死信。其中的每一次尝试可以看作为一个侧露（也是一个设计模式），而完整的处理方式列表就是一个职责链。

```
import java.util.*;

class Mail {
	enum GeneralDelivery {YES,NO1,NO2,NO3,NO4,NO5}
	enum Scannability {UNSCANNABLE,YES1,YES2,YES3,YES4}
	enum Readability {ILLEGIBLE,YES1,YES2,YES3,YES4}
	enum Address {INCORRECT,OK1,OK2,OK3,OK4,OK5,OK6}
	enum ReturnAddress {MISSING,OK1,OK2,OK3,OK4,OK5}

	GeneralDelivery generalDelivery;
	Scannability scannability;
	Readability readability;
	Address address;
	ReturnAddress returnAddress;

	static long counter = 0;
	long id = counter++;

	public String toString() { return "Mail " + id; }

	public String details() {
		return toString() +
		  ", General Delivery: " + generalDelivery +
		  ", Address Scanability: " + scannability +
		  ", Address Readability: " + readability +
		  ", Address Address: " + address +
		  ", Return address: " + returnAddress;
	}

	public static Mail randomMail() {
		Mail m = new Mail();
		m.generalDelivery = Enums.random(GeneralDelivery.class);
		m.scannability = Enums.random(Scannability.class);
		m.readability = Enums.random(Readability.class);
		m.address = Enums.random(Address.class);
		m.returnAddress = Enums.random(ReturnAddress.class);
		return m;
	}

	public static Iterable<Mail> generator(final int count) {
		return new Iterable<Mail> () {
			int n = count;
			public Iterator<Mail> iterator() {
				return new Iterator<Mail>() {
					public boolean hasNext() {
						return n-- > 0;
					}

					public Mail next() {
						return randomMail();
					}

					public void remove() {
						throw new UnsupportedOperationException();
					}
				};
			}
		};
	}
}

class PostOffice {
	enum MailHandler {
		GENERAL_DELIVERY {
			boolean handle(Mail m) {
				switch(m.generalDelivery) {
					case YES:
						System.out.println("Using general delivery for " + m);
                        return true;
                    default:
                    	return false;
				}
			}
		},
		MACHINE_SCAN {
			boolean handle(Mail m) {
				switch (m.scannability) {
					case UNSCANNABLE:
						return false;
					default:
						switch (m.address) {
							case INCORRECT:
								return false;
							default:
								System.out.println("Delivering "+ m + " automatically");
								return true;
						}
				}
			}
		},
		VISUAL_INSPECTION {
	      boolean handle(Mail m) {
	        switch(m.readability) {
	          case ILLEGIBLE: return false;
	          default:
	            switch(m.address) {
	              case INCORRECT: return false;
	              default:
	                System.out.println("Delivering " + m + " normally");
	                return true;
	            }
	        }
	      }
	    },
	    RETURN_TO_SENDER {
	      boolean handle(Mail m) {
	        switch(m.returnAddress) {
	          case MISSING: return false;
	          default:
	            System.out.println("Returning " + m + " to sender");
	            return true;
	        }
	      }
	    };

		abstract boolean handle(Mail m);
	}

	static void handle(Mail m) {
		for (MailHandler handler : MailHandler.values()) {
			if (handler.handle(m)) {
				return;
			}
		}
		System.out.println(m + " is a dead letter");
	}
}


public class Demo {
	public static void main(String[] args) {
		for (Mail mail : Mail.generator(2)) {
			System.out.println(mail.details());
			PostOffice.handle(mail);
			System.out.println("-----------");
		}
	}
}

/*
Mail 0, General Delivery: NO2, Address Scanability: UNSCANNABLE, Address Readability: YES3, Address Address: OK1, Return address: OK1
Delivering Mail 0 normally
-----------
Mail 1, General Delivery: NO5, Address Scanability: YES3, Address Readability: ILLEGIBLE, Address Address: OK5, Return address: OK1
Delivering Mail 1 automatically
-----------
*/
```

### 使用enum的状态机

一个状态机可以具有有限个特定的状态，它通常根据输入，从一个状态转移到下一个状态，不过也可能存在瞬时状态，而一旦任务执行结束，状态机就会立刻离开瞬时状态。

```
import java.util.*;
import java.io.*;

enum Input {
  NICKEL(5), 
  DIME(10), 
  QUARTER(25), 
  DOLLAR(100),
  TOOTHPASTE(200), 
  CHIPS(75), 
  SODA(100), 
  SOAP(50),
  ABORT_TRANSACTION {
    public int amount() { // Disallow
      throw new RuntimeException("ABORT.amount()");
    }
  },
  STOP { // This must be the last instance.
    public int amount() { // Disallow
      throw new RuntimeException("SHUT_DOWN.amount()");
    }
  };

  int value; // In cents
  Input(int value) { this.value = value; }
  Input() {}
  public int amount() { return value; }; // In cents
  static Random rand = new Random(47);
  public static Input randomSelection() {
    // Don't include STOP:
    return values()[rand.nextInt(values().length - 1)];
  }
}

enum Category {
	MONEY(Input.NICKEL, Input.DIME, Input.QUARTER, Input.DOLLAR),
	ITEM_SELECTION(Input.TOOTHPASTE, Input.CHIPS, Input.SODA, Input.SOAP),
	QUIT_TRANSACTION(Input.ABORT_TRANSACTION),
	SHUT_DOWN(Input.STOP);

	private Input[] values;
	Category(Input... types) { values = types; }

	private static EnumMap<Input, Category> categories = new EnumMap<Input, Category>(Input.class);
	static {
		for (Category c : Category.class.getEnumConstants()) {
			for (Input type : c.values) {
				categories.put(type, c);
			}
		}
	}

	public static Category categorize(Input input) {
		return categories.get(input);
	}
}

class VendingMachine {
	private static State state = State.RESTING;
	private static int amount = 0;
	private static Input selection = null;

	enum StateDuration { TRANSIENT }

	enum State {
		RESTING {
			void next(Input input) {
				switch (Category.categorize(input)) {
					case MONEY:
						amount += input.amount();
						state = ADDING_MONEY;
						break;
					case SHUT_DOWN:
						state = TERMINAL;
						break;
					default:
						break;
				}
			}
		},
		ADDING_MONEY {
			void next(Input input) {
				switch (Category.categorize(input)) {
					case MONEY:
						amount += input.amount();
						break;
					case ITEM_SELECTION:
						selection = input;
						if (amount < selection.amount()) {
							System.out.println("Insufficient money for " + selection);
						} else {
							state = DISPENSING;
						}
						break;
					case QUIT_TRANSACTION:
						state = GIVING_CHANGE;
						break;
					case SHUT_DOWN:
						state = TERMINAL;
						break;
					default:
						break;
				}
			}
		},
		DISPENSING(StateDuration.TRANSIENT) {
			void next() {
				System.out.println("here is your " + selection);
				amount -= selection.amount();
				state = GIVING_CHANGE;
			}
		},
		GIVING_CHANGE(StateDuration.TRANSIENT) {
			void next() {
				if (amount > 0) {
					System.out.println("Your change: " + amount);
					amount = 0;
				}
				state = RESTING;
			}
		},
		TERMINAL {
			void output() {
				System.out.println("Halted");
			}
		};

		private boolean isTransient = false;

		State() {}
		State(StateDuration sd) { isTransient = true; }

		void next(Input input) {
			throw new RuntimeException("Only call " + "next(Input input) for non-transient states");
		}

		void next() {
			throw new RuntimeException("Only call next() for " + "StateDuration.TRANSIENT states");
		}

		void output() {
			System.out.println(amount);
		}
	}

	static void run(Generator<Input> gen) {
		while (state != State.TERMINAL) {
			state.next(gen.next());
			while (state.isTransient) {
				state.next();
			}
			state.output();
		}
	}
}

class RandomInputGenerator implements Generator<Input> {
	public Input next() {
		return Input.randomSelection();
	}
}

class FileInputGenerator implements Generator<Input> {
	private Iterator<String> input;

	public FileInputGenerator(String fileName) {
		input = new TextFile(fileName, ";").iterator();
	}

	public Input next() {
		if (!input.hasNext()) {
			return null;
		}
		return Enum.valueOf(Input.class, input.next().trim());
	}
}


public class Demo {
	public static void main(String[] args) {
		Generator<Input> gen = new RandomInputGenerator();
		if (args.length == 1) {
			gen = new FileInputGenerator(args[0]);
		}

		VendingMachine.run(gen);
	}
}
```

```
import java.io.*;
import java.util.*;

public class TextFile extends ArrayList<String> {
  // Read a file as a single string:
  public static String read(String fileName) {
    StringBuilder sb = new StringBuilder();
    try {
      BufferedReader in= new BufferedReader(new FileReader(
        new File(fileName).getAbsoluteFile()));
      try {
        String s;
        while((s = in.readLine()) != null) {
          sb.append(s);
          sb.append("\n");
        }
      } finally {
        in.close();
      }
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
    return sb.toString();
  }
  // Write a single file in one method call:
  public static void write(String fileName, String text) {
    try {
      PrintWriter out = new PrintWriter(
        new File(fileName).getAbsoluteFile());
      try {
        out.print(text);
      } finally {
        out.close();
      }
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
  }
  // Read a file, split by any regular expression:
  public TextFile(String fileName, String splitter) {
    super(Arrays.asList(read(fileName).split(splitter)));
    // Regular expression split() often leaves an empty
    // String at the first position:
    if(get(0).equals("")) remove(0);
  }
  // Normally read by lines:
  public TextFile(String fileName) {
    this(fileName, "\n");
  }
  public void write(String fileName) {
    try {
      PrintWriter out = new PrintWriter(
        new File(fileName).getAbsoluteFile());
      try {
        for(String item : this)
          out.println(item);
      } finally {
        out.close();
      }
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
  }
  // Simple test:
  public static void main(String[] args) {
    String file = read("TextFile.java");
    write("test.txt", file);
    TextFile text = new TextFile("test.txt");
    text.write("test2.txt");
    // Break into unique sorted list of words:
    TreeSet<String> words = new TreeSet<String>(
      new TextFile("TextFile.java", "\\W+"));
    // Display the capitalized words:
    System.out.println(words.headSet("a"));
  }
}
```

```
// VendingMachineInput.txt
QUARTER; QUARTER; QUARTER; CHIPS;
DOLLAR; DOLLAR; TOOTHPASTE;
QUARTER; DIME; ABORT_TRANSACTION;
QUARTER; DIME; SODA;
QUARTER; DIME; NICKEL; SODA;
ABORT_TRANSACTION;
STOP;
```

### 多路分发

Java只支持单路分发，也就是说，如果要执行的操作包含了不止一个类型未知的对象时，那么Java的动态绑定机制只能处理其中一个的类型。所以，你必须自己来判定其他的类型，从而实现自己的动态绑定行为。

```
import java.util.*;
import java.io.*;

enum Outcome {
	WIN,
	LOSE,
	DRAW
}

interface Competitor<T extends Competitor<T>> {
	Outcome compete(T Competitor);
}

class Enums {
	private static Random rand = new Random(47);

	public static <T extends Enum<T>> T random(Class<T> ec) {
		return random(ec.getEnumConstants());
	}

	public static <T> T random(T[] values) {
		return values[rand.nextInt(values.length)];
	}
}

class RoShamBo {
	public static <T extends Competitor<T>> void match(T a, T b) {
		System.out.println(a + " vs. " + b + ": " + a.compete(b));
	}

	public static <T extends Enum<T> & Competitor<T>> void play(Class<T> rsbclass, int size) {
		for (int i = 0; i < size; i++) {
			match(Enums.random(rsbclass), Enums.random(rsbclass));
		}
	}
}

public enum Demo implements Competitor<Demo> {
	PAPER(Outcome.DRAW, Outcome.LOSE, Outcome.WIN),
	SCISSORS(Outcome.WIN, Outcome.DRAW, Outcome.LOSE),
	ROCK(Outcome.LOSE, Outcome.WIN, Outcome.DRAW);

	private Outcome vPaper;
	private Outcome vScissors;
	private Outcome vRock;

	Demo(Outcome paper, Outcome scissors, Outcome rock) {
		vPaper = paper;
		vScissors = scissors;
		vRock = rock;
	}

	public Outcome compete(Demo it) {
		switch (it) {
			default:
			case PAPER: return vPaper;
			case SCISSORS: return vScissors;
			case ROCK: return vRock;
		}
	}


	public static void main(String[] args) {
		RoShamBo.play(Demo.class, 10);
	}
}


/*
ROCK vs. ROCK: DRAW
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
PAPER vs. PAPER: DRAW
PAPER vs. SCISSORS: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. SCISSORS: DRAW
ROCK vs. SCISSORS: WIN
*/
```