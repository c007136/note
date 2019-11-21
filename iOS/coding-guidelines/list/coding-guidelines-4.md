# iOS编码规范4－纯代码编写UI

> by 木愚

## 前言

> **采用纯代码的方式编写UI代码，抛弃IB的方式。**

IB（Interface Builder），这是一种所见即所得的方式，目前苹果提供了两种形式：xib和storyboard（本质是一样的）。IB简单、易上手、界面上UI元素清晰明了，这样的方式怎能让程序员不喜欢呢。当然有利必有弊的。

## IB的缺点

* 自定义控件十分麻烦，可复用性差  
  一个客户端项目中难免会有可复用的控件，而客户端开发又是需求变化极度频繁的。譬如想将项目所有的按钮颜色由红色改为蓝色，如果是IB的方式，那基本得一个一个地改过去，这对程序员来说简直是灾难式的需求。如果是纯代码开发，那么或继承或category，都能轻松地解决这个问题。

* BUG和错误难以定位  
  IB的实现方式对程序员来说是不开放的，封闭的，这就导致无法调试。无法调试就意味着错误难以定位。当遇到莫名其妙的BUG或错误，工期又特别赶的时候（这也是经常遇到的），那份焦急谁尝过谁知道。

* 属性不直观  
  IB的各种属性是统一放在一个窗口中，至于设置了哪个属性，需要一个一个地检查过去。

* 在版本控制中，IB改动了什么不好对比  
  有个好习惯是在提交代码时review你自己的代码。大半部分时候，我们都不会记得自己写过什么代码。花个5-10分钟时间，review一下自己的代码是十分有必要的，这能减少BUG和错误，对他人对自己负责。然而对IB来说，如果不仔仔细细查看，一般来说，是不知道改过了很什么。纯代码的话一眼瞧过去基本上就看出个大概，毕竟这是自己写的代码。

* 在版本控制中，IB的冲突是头疼的问题  
  IB是会附加上Xcode和操作系统的版本号，如果不一样的话，该文件就会被修改。哪怕Xcode和操作系统的版本是一致的，有时候打开xib或者storyboard也会被修改。（这个随着Xcode的升级冲突问题越来越少了，但问题依旧存在），这对于多人协作的团对来说，冲突就不可避免了，有冲突就意味着工作效率低下。

## 如何更优雅地编写纯代码

洋洋洒洒地数落了一番IB，肯定会想着如何更优雅地编写纯代码，让纯代码的方式也能像IB一样清晰明了。根据规范2，可以先将代码这样写：

```
- (void)viewDidLoad
{
    self.testLabel.frame = ....;
    [self.view addSubview:self.testLabel];

    self.testButon.frame = ...;
    [self.view addSubview:self.testButton];
}

...

#pragma mark - getter and setter
- (UILabel *)testLabel
{
    if (nil == _testLabel) {
        _testLabel = [[UILabel alloc] init];
        ....
    }
    return _testLabel;
}

- (UIButton *)testButton
{
    if (nil == _testButton) {
        _testButton = [[UIButton alloc] init];
        ...
    }
    return _testButton;
}
```

这样处理，已然清晰不少。但为了使UI的层次关系和位置关系更为清晰直观，我们需要更进一步：

> **所有的UI变量都声明为成员变量，且按照UI控件的层次关系组织代码；对于相同层次来说，按照位置关系组织代码，原则上按照从左到右或从上到下，并用注释标明好。  
> 在ViewControler中；在viewDidLoad做addSubView的事情，单开一个函数做布局的事情（autolayout另外讨论），在init，dealloc里做Notification的事。  
> 在View中，在init做addSubview的事情，在layoutSubviews中做布局的事情，至于Notification的监听之类的事情放入init，dealloc中（一般来说这是没有的）。**

```
变量的声明
// UI
@property (nonatomic, strong) UIView      * view1;
@property (nonatomic, strong) UIView      * view2;
// view1
@property (nonatomic, strong) UIButton    * button1;
@property (nonatomic, strong) UIButton    * button2;
// view2
@property (nonatomic, strong) UILabel     * label1;
@property (nonatomic, strong) UILabel     * label2;
// popup view（弹出框）
@property (nonatomic, strong) PopupView   * popupView1;
@property (nonatomic, strong) PopupView   * popupView2;

- (void)viewDidLoad
{
    // self view
    [self.view addSubview:self.view1];
    [self.view addSubview:self.view2];

    // view1
    [self.view1 addSubview:self.button1];
    [self.view1 addSubview:self.button2];

    // view2
    [self.view2 addSubView:self.label1];
    [self.view2 addSubView:self.label2];
}

- (void)viewWillAppear
{
    // self view
    self.view1.frame = ...
    self.view2.frame = ...

    // view1
    self.button1.frame = ...
    self.button2.frame = ...

    // view2
    self.label1.frame = ...
    self.label2.frame = ...
}

...

#pragma mark - getter and setter
- (UIView *)view1
{
    if (nil == _view1) {
    }
    return _view1;
}
```



