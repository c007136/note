# Flutter：导航与路由

大多数应用程序具有多个页面或视图，并且希望将用户从页面平滑过渡到另外一个页面。Flutter的路由和导航功能可以帮助我们管理应用程序中的用户界面之间的命名和过渡。

管理多个用户界面有两个核心概念和类：路由（Route）和导航器（Navigator），路由（Route）是应用程序的“屏幕”或页面的抽象，导航器（Navigator）是管理路由的Widget。

## 显示屏幕路由 ？？

虽然您可以直接创建导航器，但最常见的是使用`WidgetsApp`或`MaterialApp`创建的导航器，您可以使用`Navigator.of`引用该导航器。

一个`MaterialApp`是最简单的设置方式，`MaterialApp`的`home`成为导航器堆栈的root。要在堆栈上push一个新路由，您可以创建一个具有构建器功能的`MaterialPageRoute`实例，它可以创建您想要显示在屏幕上的任何内容。

`MaterialPageRoute`是一种模态路由，可以通过平台自适应过渡来切换屏幕。对于Android，页面推送过渡时向上滑动页面，并将其淡入淡出，弹出过渡则向下滑动页面，该过渡适应平台。在iOS上，页面从右侧滑入，反向弹出，当另一个页面进入以覆盖时，该页面也会在视差中向左移动。 ???

## 命名路由
App中经常管理大量路由，通常也可以通过名称来引用它们，构建这个映射是在`MaterialApp`中的routes参数。`MaterialApp`使用此映射为导航器的`onGenerateRoute`回调创建一个值。

```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      routes: {
        FirstPage.tag: (context) => FirstPage(),
        SecondPage.tag: (context) => SecondPage(),
        ThreePage.tag: (context) => ThreePage(),
        FourPage.tag: (context) => FourPage(),
      },
      home: Scaffold(
        appBar: AppBar(
          title: Text("Navigator"),
        ),
        body: MyHomePage(title: 'Flutter Demo Home Page'),
      ),
    );
  }
}
```

`Map<String, WidgetBuilder> routes`是应用程序的顶级路由表，当使用`Navigaotr.pushNamed`等`Named`相关函数时，程序将在此映射中查找路由名称，如果名称存在，则相关联的`WidgetBuilder`将用于构造一个`MaterialPageRoute`，由这个新的路由来执行适当的过渡。如果在`Map<String, WidgetBuilder> routes`找不到对应的名称，则会调用`onGenerateRoute`回调来构建页面，如果`onGenerateRoute`中也没有找到对应的名称，则会执行`onUnknownRoute`回调。

## 路由可以返回一个值
`push`相关函数返回类型都是`Future`，`Future`将在路由弹出时解析，而`Future`的值是`pop`方法的`result`参数。

```
Navigator.of(context).pop("sss");

Navigator.of(context).pushNamed(SecondPage.tag).then(
  (Object o) {
    print("object is ${o}");
  },
);
```

## 定制路由
您可以创建自己的路由类，比如`PopupRoute`、`ModalRoute`或者`PageRoute`的子类，以控制用于显示路由，包括路由模态屏障的颜色以及路由的动画转换等。`PageRouteBuilder`类可以根据回调来定义路由，下面是例子。

```
// 主要研究定制路由

import 'package:flutter/material.dart';

void main() {
  runApp(new MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: MyHomePage(title: 'Home'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => new _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  void _openNewPage() {
    Navigator.of(context).push(
      PageRouteBuilder(
        opaque: false,
        pageBuilder: (BuildContext context, _, __) {
          return NewPage();
        },
        transitionsBuilder: (_, Animation<double> animation, __, Widget child) {
          return FadeTransition(
            opacity: animation,
            child: ScaleTransition(
              scale: animation,
              child: child,
            ),
            // child: RotationTransition(
            //   turns: Tween<double>(begin: 0.5, end: 1.0).animate(animation),
            //   child: child,
            // ),
          );
        },
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Text(
          '点击浮动按钮打开定制页面',
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _openNewPage,
        child: Icon(Icons.open_in_new),
      ),
    );
  }
}

class NewPage extends StatefulWidget {
  @override
  _NewPageState createState() => _NewPageState();
}

class _NewPageState extends State<NewPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("New Page"),
      ),
      body: Center(
        child: Text('定制页面路由'),
      ),
    );
  }
}
```

## 嵌套路由

一个App中可以有多个导航器，将一个导航器嵌套在另外一个导航器下面可以创建一个内部的`routes`。例子如下：

```
// 主要研究嵌套路由

import 'package:flutter/material.dart';
import 'package:navigator/page1.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Home(),
      // theme: ThemeData.dark(),
      theme: ThemeData(primaryColor: Colors.blue),
    );
  }
}

class Home extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return new _HomeState();
  }
}

class _HomeState extends State<Home> {
  int _currentIndex = 0;
  final List<Widget> _children = [
    HomeNavigator(),
    PlaceholderWidget('Profile'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _children[_currentIndex],
      bottomNavigationBar: new BottomNavigationBar(
        onTap: onTabTapped,
        currentIndex: _currentIndex,
        items: [
          new BottomNavigationBarItem(
            icon: new Icon(Icons.home),
            title: new Text('Home'),
          ),
          new BottomNavigationBarItem(
            icon: new Icon(Icons.person),
            title: new Text('Profile'),
          ),
        ],
      ),
    );
  }

  void onTabTapped(int index) {
    setState(() {
      _currentIndex = index;
    });
  }
}

class PlaceholderWidget extends StatelessWidget {
  final String text;

  PlaceholderWidget(this.text);

  @override
  Widget build(BuildContext context) {
    return new Center(
      child: new Text(text),
    );
  }
}

class HomeNavigator extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Navigator(
      initialRoute: 'home',
      onGenerateRoute: (RouteSettings settings) {
        WidgetBuilder builder;
        switch (settings.name) {
          case 'home':
            builder = (BuildContext context) => new HomePage();
            break;
          case 'demo1':
            builder = (BuildContext context) => new FirstPage();
            break;
          default:
            throw new Exception('Invalid route: ${settings.name}');
        }

        return new MaterialPageRoute(builder: builder, settings: settings);
      },
    );
  }
}

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Home'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('demo1'),
          onPressed: () {
            Navigator.of(context).pushNamed('demo1');
          },
        ),
      ),
    );
  }
}
```



## 参考链接：
1. [Flutter路由和导航](https://juejin.im/post/5bbaf7bf5188255c8d0fe309)
2. [Flutter进阶——路由和导航](https://blog.csdn.net/hekaiyou/article/details/72853738)