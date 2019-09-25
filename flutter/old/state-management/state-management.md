# 状态管理

在Flutter中，状态管理最复杂的问题了。难在哪里呢？不外乎状态管理需要做到：

* 方便拿到`State`
* 能够将更新动作高效地通知给依赖组件
* 能够精准的做到最小更新


## 第三方库
### Scoped_model

https://juejin.im/post/5b97fa0d5188255c5546dcf8

缺点，侵入式
技术实现： Listenable/InheritedWidget

### Redux

![](./images/1.bmp)

问题一个state要对应一个store吗？

https://juejin.im/post/5ba26c086fb9a05ce57697da

### BLoC

https://juejin.im/post/5bb6f344f265da0aa664d68a

### RxDart

https://juejin.im/post/5bcea438e51d4536c65d2232

### Provide

https://juejin.im/post/5c6d4b52f265da2dc675b407