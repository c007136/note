# iOS编码规范7－少用继承1

> by 木愚

## 前言

> **能用category或组合的方式，就不要用继承的方式。**

## 上下文规范

在进一步地讨论这些概念之前，需要达成一个表达上的共识：

```
所有的大写字母都是类或对象，小写字母表示属性或方法。

FOO:{ isLoading, _data, render(), _switch() }   这表示一个FOO对象，isLoading、_data是它的属性，render()、_switch()是它的方法，加下划线表示私有。

A -> B                                          这表示从A派生出了B，A是父类。

A -> B:{ [a, b, c(), d()], e, f() }             []里面是父类的东西，e、f()是派生类的东西

B:{ [ A ], e, f() }                             省略了对父类的描述，用类名A代替，其他同上

B:{ [ A ], e, f(), @c() }                       省略了对父类的描述，函数前加@表示重载了父类的方法。

B:{ [ A,D ], e, f() }                           多继承，B继承了A和D

B<protocol>                                     符合某个protocol接口的对象。

<protocol>:{foo(), bar}                         protocol这个接口中包含foo()这个方法，bar这个属性。

foo(A, int)                                     foo这个函数，接收A类和int类型作为参数。
```

## 继承的缺点

继承意味着高耦合，意味着牵一发而动全身，意味着相应的业务逻辑代码难以被剥离。先来看看一个场景：

最开始产品经理对木愚说：

> 我们的APP点击页面中的UITextField弹出键盘时，都要在键盘上面显示一个带完成按钮的自定义UIView。

木愚心想好简单的任务啊，在原有的BASE\_VC中监听键盘事件，并让其他的VC（如登录页面：LOGIN\_VC）继承于BASE\_VC：

```
BASE_VC:{keyboardTopView, doneButton, keyboardWillShow(), keyboardWillHide()}
LOGIN_VC:{[BASE_VC], textField}
```

过了一段时间，多变的产品经理过来对木愚说：

> 有个搜索页面需要加一个左移，右移按钮方便用户操作

木愚想了想这简单的：

```
BASE_VC:{keyboardTopView, doneButton, keyboardWillShow(), keyboardWillHide()}
LOGIN_VC:{[BASE_VC], textField}
SEARCH_VC:{[BASE_VC], leftButton, rightButton}
```

又过了一段时间，随着业务的发展，产品经理说：

> 需要一个能给用户发送消息的页面，不要关闭按钮了，但需要填写消息的TextField和发送按钮。

木愚懵逼了，肯定是不能改动BASE\_VC，否则其他VC也得跟着改动，只能重写监听函数了，于是变成了这样：

```
BASE_VC:{keyboardTopView, doneButton, keyboardWillShow(), keyboardWillHide()}
LOGIN_VC:{[BASE_VC], textField}
SEARCH_VC:{[BASE_VC], leftButton, rightButton}
MESSAGE_VC:{[BASE_VC], messageTextField, sendButton, @keyboardWillShow}
```

“虽然代码有冗余，也会有多余的变量，但先把需求应付掉再说。”木愚这么想。

过了一段时间，一个做社区模块的同事\(他也弄了一个BASE\_VC：SOCIAL\_BASE\_VC\)过来对木愚说：

> 我也需要键盘显示时带完成按钮的自定义UIView的代码，能告诉我怎么用吗？

木愚说：

> 这只能复制粘贴了......

于是变成了这样：

```
BASE_VC:{keyboardTopView, doneButton, keyboardWillShow(), keyboardWillHide()}
LOGIN_VC:{[BASE_VC], textField}
SEARCH_VC:{[BASE_VC], leftButton, rightButton}
MESSAGE_VC:{[BASE_VC], messageTextField, sendButton, @keyboardWillShow}
SOCIAL_BASE_VC:{keyboardTopView, doneButton, keyboardWillShow(), keyboardWillHide()}
...  //社区模块的其它类
```

代码相当冗余，违背了[一次且仅一次原则](https://zh.wikipedia.org/wiki/一次且仅一次)。这是很不好的现象，当这块代码出现BUG时，就意味着要多次修改，也意味着当出现另类需求时，又得复制粘贴，如此恶性循环下去。

## 解决方法

category其实是一个不太准确的说法，正确的说法叫Method Swizzling。Method Swizzling是利用Objective-C的动态性，实现在运行时偷换selector对应的实现方法，达到给方法hook的目的。之所以这里说成category，是因为在很多时候用category就够了，完全没必要用继承，这里就不展开了。

下面给出解决方法的关键代码。

```
@implementation NSObject (RNSwizzle)

// 这是Method Swizzling的核心方法
+ (void)swizzleSelector:(SEL)origSelector withSelector:(SEL)newSelector
{
    Class class = [self class];
    Method origMethod = class_getInstanceMethod(class, origSelector);
    Method newMethod = class_getInstanceMethod(class, newSelector);
    IMP origIMP = method_getImplementation(origMethod);
    IMP newIMP = method_getImplementation(newMethod);

    BOOL didAddMethod = class_addMethod(self, origSelector, newIMP, method_getTypeEncoding(newMethod));
    if (didAddMethod)
    {
        class_replaceMethod(class, newSelector, origIMP, method_getTypeEncoding(origMethod));
    }
    else
    {
        method_exchangeImplementations(origMethod, newMethod);
    }
}

@end
```

```
@implementation UIViewController (Swizlle)

+ (void)load
{
    [UIViewController swizzleSelector:@selector(viewWillAppear:) withSelector:@selector(zx_viewWillAppear:)];
    [UIViewController swizzleSelector:@selector(viewWillDisappear:) withSelector:@selector(zx_viewWillDisappear:)];
}

- (void)zx_viewWillAppear:(BOOL)animated
{
    [self zx_viewWillAppear:animated];

    if (![self zx_needToPerform]) {
        return;
    }

    NSLog(@"%@ will appear", NSStringFromClass([self class]));

    // 监听键盘事件
}

- (void)zx_viewWillDisappear:(BOOL)animated
{
    [self zx_viewWillDisappear:animated];

    if (![self zx_needToPerform]) {
        return;
    }

    NSLog(@"%@ will disappear", NSStringFromClass([self class]));

    // 取消监听键盘事件
}

- (void)keyboardWillShow:(NSNotification *)notification
{
}

-(void)keyboardWillHide:(NSNotification *)notification
{
}

// 过滤函数
- (BOOL)zx_needToPerform
{
    if ([self isKindOfClass:[AViewController class]]
        || [self isKindOfClass:[BViewController class]])
    {
        return YES;
    }

    return NO;
}
```



