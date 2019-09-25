# Rumtime学习4

> 消息转发相关

消息转发，也就是`forwardInvocation`，还有两个作用。

## 多继承

转发和继承相似，可以用于为Objective-C添加一些多继承的效果。

![](./images/4-1.gif)

在上图中`Warrior`和`Diplomat`没有继承关系，但是`Warrior`将`negotiate`消息转发给了`Diplomat`后，就好似`Diplomat`是`Warrior`的超类一样。

消息转发弥补了Objective-C不支持多继承的性质，也避免了因为多继承导致单个类变得臃肿复杂。它将问题分解得很细，只针对想要借鉴的方法才转发，而且转发机制是透明的（如何理解透明？？？）。

## 替代者对象（Surrogate Objects）

参考[官网](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW10)

不是很明白？？？



