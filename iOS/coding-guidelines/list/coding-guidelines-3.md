# iOS编码规范3－变量的声明、初始化和使用

> by 木愚

## 前言

变量涉及到声明、初始化、使用三个问题，如何组织这些代码是个头疼的问题。

> **规范二：所有的变量都采取@property的形式声明且用懒加载的方式初始化和使用变量，并按照声明变量的顺序置于文件的尾部**

## 变量的声明和初始化

变量声明的方式主要可以分为两种方式。

```
方式一
UILabel          * _textLabel;
方式二
@property (nonatomic, strong) NSString       * textLabel;
```

对于方式一来说，初始化一般是这样的：

```
- (void)viewDidLoad
{
    _textLabel = [[UILabel alloc] init];
    _textLabel.frame = ....;
    _textLabel.text = @"...";
    _textLabel.textColor = @"....";
    ....
    [self.view addSubview:_textLabel];
}
```

而对于方式二，则可以采取这样的方式初始化

```
- (void)viewDidLoad
{
    // do something
    _textLabel.frame = ....;
    [self.view addSubview:_textLabel];
}
...
//  文件的尾部
- (UILabel *)textLabel
{
    if (nil == _textLabel) {
        _textLabel.text = @"";
        _textLabel.textColor = ...;
        ....
    }
    return _textLabel;
}
```

对于方式二来说，有如下优点：

1.有一个统一的初始化代码地方；

2.将繁琐细节的代码从逻辑代码中抽离出；

3.可明确变量的原子性、存取器控制以及内存管理。

## 变量的使用

在上文中我们用了这样的方式使用变量。

```
- (void)viewDidLoad
{
    // do something
    _textLabel.frame = ....;
    [self.view addSubview:_textLabel];
}
```

其实这样的方式是不推荐的，  
1. 初始化的代码无法被调用，  
2. 易跟局部变量混淆在一起，  
3. 代码风格不统一。

```
- (void)viewDidLoad
{
    // 无法调用getter方法
    _textLabel.frame = ....;
    [self.view addSubview:_textLabel];

    UILabel textLabel;
    textLabel = [[UILabel alloc] init]
    textLabel.frame = ...;
    ...
}

// getter方法
- (UILabel *)textLabel
{
    if (nil == _textLabel) {
        _textLabel.text = @"";
        _textLabel.textColor = ...;
        ....
    }
    return _textLabel;
}
```

故推荐self.的方式，即Accessor Method的方式，只有当真正要用的时候才加载，延迟加载，提高效率（懒加载的由来）。

```
- (void)viewDidLoad
{
    // do something
    self.textLabel.frame = ....;
    [self.view addSubview:self.textLabel];

    UILabel textLabel;
    textLabel = [[UILabel alloc] init]
    textLabel.frame = ...;
    ...
}

//  可以被调用。
- (UILabel *)textLabel
{
    if (nil == _textLabel) {
        _textLabel.text = @"";
        _textLabel.textColor = ...;
        ....
    }
    return _textLabel;
}
```

## Accessor Method的争议

争议主要在init和dealloc函数中，苹果开发者文档[内存管理](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html)中有这么一句话：Don’t Use Accessor Methods in Initializer Methods and dealloc，至于为什么苹果并未说明。我们还是先看看如下代码：

```
// 头文件
////////////////////////////////

#import <Foundation/Foundation.h>
@interface IFather : NSObject
@property (nonatomic, strong) NSString               * fatherString;
@end

//
//  Child
//
@interface IChild : IFather
@property (nonatomic, strong) NSString               * childString;
@end 


// .m文件
////////////////////////////////

@implementation IFather
- (id)init
{
    self = [super init];
    if (self) {
        NSLog(@"init in Father");

        // 会崩溃
        self.fatherString = @"FatherString";
    }
    return self;
}
@end

//
//  Child
//
@implementation IChild
- (id)init
{
    self = [super init];
    if (self) {
        NSLog(@"init in Child");
        self.childString = @"I am really ChildString";
        self.fatherString = @"ChildString";
    }
    return self;
}

- (void)setFatherString:(NSString *)fatherString
{
    [super setFatherString:fatherString];

    NSString * string = [NSString stringWithString:self.childString];
    NSLog(@"string in Child is %@", string);
}
```

当执行IChild \* ic = \[\[IChild alloc\] init\]会先调用IFather的初始化函数，又因为多态性会调用子类的setter函数，这就跳过了子类的初始化函数了，这十分危险。子类的函数是基于子类的初始化完成为前提来进行的，即只有子类初始化完成了，子类的其它函数才是可用的。上面的例子就会发生崩溃。同理，dealloc也是如此，具体可参考[这里](http://blog.smilexiaofeng.com/blog/2015/08/11/why-do-not-use-accessor-in-init-and-dealloc/)。

但我推荐的方式是，在ViewController和View中（也就是逻辑代码中）仍然采取Accessor Method的方式，一方面为了代码结构的清晰，另外一方面并不推荐使用继承的方式，具体可参考后面的规范6。至于其它的地方（也就是通用模块中），还是遵循苹果开发者文档的原则吧：Don’t Use Accessor Methods in Initializer Methods and dealloc。

