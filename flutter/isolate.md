# isolate

## Dart是一种单线程语言

首先，大家需要牢记，Dart是单线程的并且Flutter依赖于Dart。

> **重点**
> 
> **Dart同一时刻只执行一个操作，其他操作在该操作之后执行，这意味着只要一个操作正在执行，它就不会被其他Dart代码中断**

```
void myBigLoop(){
    for (int i = 0; i < 1000000; i++){
        _doSomethingSynchronously();
    }
}
```

在上面的例子中，myBigLoop()方法在执行完成前永远不会被中断。因此，如果该方法需要一些时间，那么在整个方法执行期间应用将会被阻塞。

## Dart执行模型

那么在幕后，Dart是如何管理操作序列的执行呢（the sequence of operations to be executed?）

为了回答这个问题，我们需要看一下Dart的Event Loop。

当你启动一个Flutter（或任何Dart）应用时，将创建并启动一个新的线程process（在Dart中为Isolate）。该线程将是你在整个应用中唯一需要关注的。

那么，此线程创建后，Dart会自动：

1. 初始化2个FIFO队列（MicroTask和Event）；
2. 并且当该方法执行完成后，执行main方法；
3. 启动Event Loop

在该线程的整个生命周期中，一个被称为Event Loop的内部且隐藏的process将决定你代码的执行方法以及顺序，这依赖于Microtask队列和Event队列的内容。

事件循环其实就是死循环，由一个内部时钟切片着（cadenced）。在每个时钟周期内，如果没有其他Dart代码执行，则执行一下操作

```
void eventLoop(){
    while (microTaskQueue.isNotEmpty){
        fetchFirstMicroTaskFromQueue();
        executeThisMicroTask();
        return;
    }

    if (eventQueue.isNotEmpty){
        fetchFirstEventFromQueue();
        executeThisEventRelatedCode();
    }
}
```

正如我们看到的，MicroTask队列优先于Event队列，那这2个队列的作用是什么呢？

## MicroTask队列

MicroTask队列用于非常简短且需要异步执行的内部动作，这些动作需要在其他事情完成之后并将执行权送还给Event队列之前

作为MicroTask的一个例子，你可以设想必须在资源关闭后立即释放它。由于关闭过程可能需要一些时间才能完成，你可以按照以下方式编写代码：

```
MyResource myResource;

...

void closeAndRelease() {
    scheduleMicroTask(_dispose);
    _close();
}

void _close(){
    // The code to be run synchronously
    // to close the resource
    ...
}

void _dispose(){
    // The code which has to be run
    // right after the _close()
    // has completed
}
```

这是大多数时候你不必使用的东西。比如，在整个Flutter源代码中scheduleMicroTask()方法仅被调用7次。

最好优先考虑使用Event队列。

## Event队列

Event队列适用于以下操作：

* 外部事件如
   * I/O
   * 手势
   * 绘图
   * 计时器
   * 流
   * ......
* futures

事实上，每次外部事件被触发时，要执行的代码都会被Event队列所引用。

一旦没有任何micro task运行。Event Loop将优先考虑Event队列中的第一项并执行它。

值得注意的是，Future操作也通过Event队列处理。

## Future

Future是一个异步执行并且在未来的某一个时刻完成（或失败）的任务。

当你实例化一个Future时：

* Future的一个实例会被创建出来并且会被记录在由Dart管理的内部数组中；
* 需要由此Future执行的代码直接被推送到Event队列中去；
* 该Future实例返回一个状态（=incomplete）；
* 如果存在下一个同步代码，执行它（非Future的执行代码）。

只要Event Loop从Event循环中获取它，被Future引用的代码将像其他任何Event一样被执行。

当该代码被执行并将完成（或失败）时。then()或catchError()方法将直接被触发。

为了说明这一点，我们来看下面的例子：

```
void main(){
    print('Before the Future');
    Future((){
        print('Running the Future');
    }).then((_){
        print('Future is complete');
    });
    print('After the Future');
}
```

如果我们运行该代码，输出将如下所示：

```
Before the Future
After the Future
Running the Future
Future is complete
```

这是完全正确的，因为执行流程如下：

1. `print(‘Before the Future’)`
2. 将`(){print(‘Running the Future’);}`添加到Event队列
3. `print(‘After the Future’)`
4. Event Loop获取步骤二的代码并执行它
5. 当代码执行完毕时，它会查找`then()`语句并执行它

需要记住一些非常重要的事情：

> Future并非并行执行，而是遵循Event Loop处理事件的顺序规则执行。

## Async方法

当你使用async关键字作为方法声明的后缀时，Dart会将其理解为：

* 该方法的返回值是一个Future；
* 它同步执行该方法的代码直到第一个await关键字，然后它暂停该方法其他部分的执行（it runs synchronously the code of that method up to the very first await keyword, then it pauses the execution of the remainder of that method）； 
* 一旦由await关键字引用的Future执行完成，下一行代码将立即执行。

了解这一点是非常重要的，因为很多开发者认为await暂停了整个流程直到它执行完成，但事实并非如此，他们忘记了Event Loop的运作模式.....

```
import 'dart:async';

void main(List<String> args) async {
  methodA();
  await methodB();
  await methodC('main');
  methodD();
}

methodA() {
  print('method A');
}

methodB() async {
  print('method B start');
  await methodC('B');
  print('method B end');
}

methodC(String from) async {
  print('method C start from $from');

  Future(() {
    print('method C running Future from $from');
  }).then((_) {
    print('method C end of Future from $from');
  });

  print('method C end from $from');
}

methodD() {
  print('method D');
}
```

输出结果为：

```
method A
method B start
method C start from B
method C end from B
method B end
method C start from main
method C end from main
method D
method C running Future from B
method C end of Future from B
method C running Future from main
method C end of Future from main
```

如果你最初希望示例代码中仅在所有代码末尾执行methodD()，那么你应该按照以下方式编写代码：

```
import 'dart:async';

void main(List<String> args) async {
  methodA();
  await methodB();
  await methodC('main');
  methodD();
}

methodA() {
  print('method A');
}

methodB() async {
  print('method B start');
  await methodC('B');
  print('method B end');
}

methodC(String from) async {
  print('method C start from $from');

  await Future(() {
    print('method C running Future from $from');
  }).then((_) {
    print('method C end of Future from $from');
  });

  print('method C end from $from');
}

methodD() {
  print('method D');
}
```

输出结果为：

```
method A
method B start
method C start from B
method C running Future from B
method C end of Future from B
method C end from B
method B end
method C start from main
method C running Future from main
method C end of Future from main
method C end from main
method D
```

我想向你演示的最后一个例子如下。 运行 method1 和 method2 的输出是什么？它们会是一样的吗？

```
import 'dart:async';

void main(List<String> args) async {
  //method1();
  method2();
}

void method1() {
  List<String> myArray = <String>['a', 'b', 'c'];
  print('before loop');
  myArray.forEach((String value) async {
    await delayedPrint(value);
  });
  print('end of loop');
}

void method2() async {
  List<String> myArray = <String>['a', 'b', 'c'];
  print('before loop');
  for (var i = 0; i < myArray.length; i++) {
    await delayedPrint(myArray[i]);
  }
  print('end of loop');
}

Future<void> delayedPrint(String value) async {
  await Future.delayed(Duration(seconds: 1));
  print('delayedPrint: $value');
}
```

method1()输出结果:

```
before loop
end of loop
delayedPrint: a(after 1 second)
delayedPrint: b(directly after)
delayedPrint: c(directly after)
```

method2()输出结果：

```
before loop
delayedPrint: a(after 1 second)
delayedPrint: b(1 second later)
delayedPrint: c(1 second later)
end of loop(right after)
```

你是否清楚它们行为不一样的区别以及原因呢？

答案基于这样一个事实，method1 使用 forEach() 函数来遍历数组。每次迭代时，它都会调用一个被标记为 async（因此是一个 Future）的新回调函数。执行该回调直到遇到await，而后将剩余的代码推送到Event 队列。一旦迭代完成，它就会执行下一个语句：“print(‘end of loop’)”。执行完成后，事件循环 将处理已注册的 3 个回调。**muyu: 不能理解？？？**

## 多线程

因此，我们在Flutter中如何并行运行代码呢？这可能吗？

是的，这多亏了Isolates

### Isolates是什么？

正如前面解释过的，Isolate是Dart中的线程

然而，它与常规“线程”的实现存在较大差异，这也是将其命名为“Isolate”的原因。

> Isolate在Flutter中并不共享内存。不同Isolate之间通过消息进行通信。

### 每个Isolate都有自己的事件循环

每个Isolate都拥有自己的Event Loop及队列（MicroTask和Event）。这意味着在一个Isolate中运行的代码与另外一个Isolate不存在任何关联。

多亏了这一点，我们可以获得并行处理的能力。

### Isolate的简单例子

```
import 'dart:async';
import 'dart:io';
import 'dart:isolate';

main(List<String> args) {
  start();
  print("start");
}

Isolate isolate;
int i = 0;

void start() async {
  final receive = ReceivePort();

  isolate = await Isolate.spawn(runTimer, receive.sendPort);
  receive.listen((dynamic data) {
    // i始终等于0，表明isolate之间内存不共享
    print('Receive :$data; i: $i');

    if (data is String) {
      if (data.contains('5')) {
        stop();
        exit(0);
      }
    }
  });
}

void runTimer(SendPort port) {
  int counter = 0;
  Timer.periodic(const Duration(seconds: 1), (_) {
    counter++;
    i++;
    final msg = "notification $counter";
    print('Send: $msg; i: $i');
    port.send(msg);
  });
}

void stop() {
  print('stop isolate');
  isolate?.kill(priority: Isolate.immediate);
  isolate = null;
}

/*
start
Send: notification 1; i: 1
Receive :notification 1; i: 0
Send: notification 2; i: 2
Receive :notification 2; i: 0
Send: notification 3; i: 3
Receive :notification 3; i: 0
Send: notification 4; i: 4
Receive :notification 4; i: 0
Send: notification 5; i: 5
Receive :notification 5; i: 0
stop isolate
*/
```

### 如何启动Isolate

根据你运行Isolate的场景，你可能需要考虑不同的方法。

#### 1.底层解决方案

第一个解决方案不依赖任何软件包，它完全依赖Dart提供的底层API

##### 1.1 第一步：创建并握手

如前所述，Isolate不共享任何内存并通过消息进行交互，因此，我们需要找到一种方法在“调用者”与新的Isolate之间建立通信。

每个Isolate都暴露了一个将消息传递给Isolate的被称为“SendPort”的端口。（个人觉得该名字有一些误导，因为它是一个接收/监听的端口，但毕竟是官方名称。）

这意味着“调用者”和“新的isolate”需要互相知道彼此的端口才能进行通信。这个握手的过程如下所示：

```
//
// The port of the new isolate
// this port will be used to further
// send messages to that isolate
//
SendPort newIsolateSendPort;

//
// Instance of the new Isolate
//
Isolate newIsolate;

//
// Method that launches a new isolate
// and proceeds with the initial
// hand-shaking
//
void callerCreateIsolate() async {
    //
    // Local and temporary ReceivePort to retrieve
    // the new isolate's SendPort
    //
    ReceivePort receivePort = ReceivePort();

    //
    // Instantiate the new isolate
    //
    newIsolate = await Isolate.spawn(
        callbackFunction,
        receivePort.sendPort,
    );

    //
    // Retrieve the port to be used for further
    // communication
    //
    newIsolateSendPort = await receivePort.first;
}

//
// The entry point of the new isolate
//
static void callbackFunction(SendPort callerSendPort){
    //
    // Instantiate a SendPort to receive message
    // from the caller
    //
    ReceivePort newIsolateReceivePort = ReceivePort();

    //
    // Provide the caller with the reference of THIS isolate's SendPort
    //
    callerSendPort.send(newIsolateReceivePort.sendPort);

    //
    // Further processing
    //
}
```

##### 1.2 第二步：向Isolate发送消息

```
//
// Method that sends a message to the new isolate
// and receives an answer
// 
// In this example, I consider that the communication
// operates with Strings (sent and received data)
//
Future<String> sendReceive(String messageToBeSent) async {
    //
    // We create a temporary port to receive the answer
    //
    ReceivePort port = ReceivePort();

    //
    // We send the message to the Isolate, and also
    // tell the isolate which port to use to provide
    // any answer
    //
    newIsolateSendPort.send(
        CrossIsolatesMessage<String>(
            sender: port.sendPort,
            message: messageToBeSent,
        )
    );

    //
    // Wait for the answer and return it
    //
    return port.first;
}

//
// Extension of the callback function to process incoming messages
//
static void callbackFunction(SendPort callerSendPort){
    //
    // Instantiate a SendPort to receive message
    // from the caller
    //
    ReceivePort newIsolateReceivePort = ReceivePort();

    //
    // Provide the caller with the reference of THIS isolate's SendPort
    //
    callerSendPort.send(newIsolateReceivePort.sendPort);

    //
    // Isolate main routine that listens to incoming messages,
    // processes it and provides an answer
    //
    newIsolateReceivePort.listen((dynamic message){
        CrossIsolatesMessage incomingMessage = message as CrossIsolatesMessage;

        //
        // Process the message
        //
        String newMessage = "complemented string " + incomingMessage.message;

        //
        // Sends the outcome of the processing
        //
        incomingMessage.sender.send(newMessage);
    });
}

//
// Helper class
//
class CrossIsolatesMessage<T> {
    final SendPort sender;
    final T message;

    CrossIsolatesMessage({
        @required this.sender,
        this.message,
    });
}
```

##### 1.3 第3步：销毁Isolate

```
//
// 释放一个 isolate 的例程
//
void dispose(){
    newIsolate?.kill(priority: Isolate.immediate);
    newIsolate = null;
}
```


#### 2.一次性计算

如果你只需要运行一些代码来完成一些特定的工作，并且在工作完成之后不需要与Isolate进行交互，那么这里有一个非常方便的称为compute的Helper

主要包含以下功能：

* 产生一个Isolate
* 在该Isolate上运行一个回调函数，并传递一些数据
* 返回回调函数的处理结果
* 回调执行后终止Isolate

> **约束**
>
> 回调函数必须是顶级函数并且不能是闭包或类中的方法（静态或非静态）

例子：

```
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

int syncFibonacci(int n) {
  return n < 2 ? n : syncFibonacci(n - 2) + syncFibonacci(n - 1);
}

void main() {
  compute(syncFibonacci, 20).then((int n) {
    print('n is $n');
  });

  runApp(new MyApp());
}
... ...
```

### 我应该什么时候用Future和Isolate

* 如果一个方法需要几毫秒 => Future
* 如果一个处理流程需要几百毫秒 => Isolate


## 参考资料

* [Dart:并发编程之Isolate](https://juejin.im/post/5cb3e8e46fb9a068646c0b1b)
* [Futures - Isolates - Event Loop](https://www.didierboelens.com/2019/01/futures---isolates---event-loop/)


