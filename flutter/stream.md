# stream

对于Stream而言它有两种方式：

* 单订阅
* 广播订阅

对于单订阅来说它只允许你实现一个监听的出口，不允许进行多次监听，另外订阅一旦被取消，你将无法再次订阅。

## 单订阅

```
import 'dart:async';

void main(List<String> args) async {
  StreamController<int> cr = new StreamController();
  cr.stream.listen((data) {
    print("data is ${data}");
  });

  // Bad state: Stream has already been listened to.
  // cr.stream.listen((data) {
  //   print("data is ${data}");
  // });

  cr.sink.add(1);
  cr.sink.add(2);
  cr.sink.add(3);

  cr.close();

  // Bad state: Cannot add event after closing
  // cr.sink.add(4);
}

/*
data is 1
data is 2
data is 3
*/
```

## 广播订阅

广播Stream允许我们可以给Stream添加任意数量的订阅。


```
import 'dart:async';

void main(List<String> args) async {
  StreamController<int> cr = new StreamController();
  Stream stream = cr.stream.asBroadcastStream();

  stream.listen((data) {
    print("data1 is ${data}");
  });

  stream.listen((data) {
    print("data2 is ${data}");
  });

  cr.sink.add(1);
  cr.sink.add(2);
  cr.sink.add(3);

  cr.close();
}

/*
data1 is 1
data2 is 1
data1 is 2
data2 is 2
data1 is 3
data2 is 3
*/
```

## transform

StreamTransformer<S, T>是Stream的检查员，它负责接收stream的数据信息，然后进行处理返回一条新的数据。S表示输入类型，T表示转化后的输出类型。

```
import 'dart:async';

void main(List<String> args) async {
  StreamController<int> cr = new StreamController();
  final transformer =
      StreamTransformer<int, String>.fromHandlers(handleData: (value, sink) {
    if (value == 10) {
      sink.add("you are right");
    } else {
      sink.addError('you are wrong');
    }
  });

  cr.stream.transform(transformer).listen((String data) {
    print("data is ${data}");
  }).onError((error) {
    print("error is ${error}");
  });

  cr.sink.add(11);
  cr.sink.add(10);

  cr.close();
}

/*
error is you are wrong
data is you are right
*/
```

## where

where可以用来筛选一些不想要的数据。

```
import 'dart:async';

void main(List<String> args) async {
  StreamController<int> cr = new StreamController();
  cr.stream.where((value) {
    return value == 10;
  }).listen((data) {
    print("data is ${data}");
  });

  cr.sink.add(11);
  cr.sink.add(10);

  cr.close();
}

/*
data is 10
*/
```

## StreamBuilder

StreamBuilder可以将Stream和Widget关联起来，用来更新UI，被称之为：基于事件流的异步状态控件。

```
import 'dart:async';
import 'dart:math';

import 'package:flutter/material.dart';

class StreamBuilderDemo extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return StreamBuilderDemoState();
  }
}

class StreamBuilderDemoState extends State<StreamBuilderDemo> {
  String _string = "hello";
  StreamController<String> _streamController = new StreamController();

  @override
  void dispose() {
    _streamController.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      home: new Scaffold(
        appBar: new AppBar(
          title: new Text('AVStream'),
        ),
        body: new Center(
          child: new Column(
            children: <Widget>[
              StreamBuilder<String>(
                stream: _streamController.stream,
                initialData: _string,
                builder:
                    (BuildContext context, AsyncSnapshot<String> snapshot) {
                  return new Text(snapshot.data);
                },
              ),
              RaisedButton(
                child: Text(
                  '更新',
                  style: TextStyle(color: Colors.white),
                ),
                color: Colors.blue,
                onPressed: () {
                  Random random = new Random();
                  int rInt = random.nextInt(1000000);
                  String randomText = "update " + rInt.toString();
                  _streamController.sink.add(randomText);
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}

void main(List<String> args) {
  runApp(new StreamBuilderDemo());
}
```

## 参考链接

* [使用 Stream 简单更新 UI](https://zhuanlan.zhihu.com/p/55783611)
* [Dart | 什么是Stream](https://juejin.im/post/5baa4b90e51d450e6d00f12e)
* [Flutter完整开发实战详解(十一、全面深入理解Stream)](https://juejin.im/post/5cc2acf86fb9a0321f042041)