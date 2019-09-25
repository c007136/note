# Dart之线程隔离（Isolate）

在Dart中没有进程和线程的概念，所有的Dart代码都是在isolate上运行的，那么isolate到底是什么？接下来我们讨论讨论。

## 同步代码和异步代码


## Demo

```
import 'dart:async';
import 'dart:isolate';

class Home {
  void momIsCooking() {
    print('mom is cooking');
  }

  void isPlayGame() {
    Timer.periodic(Duration(seconds: 1), (_) {
      print("game is funning");
    });
  }

  void monLetMeBuySoySauce() {
    print("sweet, can you help to buy a bottle of soy sauce, please?");
  }

  void isAgreeToBuy() {
    print('ok, mom. I will buy it');
  }

  static buySoySauce(SendPort sendPort) {
    Timer.periodic(Duration(seconds: 2), (_) {
      sendPort.send("I have bought it");
    });
  }
}

void main() async {
  Home home = Home();
  home.momIsCooking();
  home.isPlayGame();
  home.monLetMeBuySoySauce();
  home.isAgreeToBuy();

  ReceivePort receivePort = ReceivePort();
  receivePort.listen((var message) {
    print(message);
  });
  Isolate.spawn(Home.buySoySauce, receivePort.sendPort);
}

/*
mom is cooking
sweet, can you help to buy a bottle of soy sauce, please?
ok, mom. I will buy it
game is funning
game is funning
I have bought it
game is funning
game is funning
I have bought it
game is funning
game is funning
I have bought it
*/
```






## 参考资料
https://pjf.name/blogs/asynchronous-programming-in-dart.html