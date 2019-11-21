# iOS编码规范6－define和const

> by 木愚

## 前言

> **能用category，就不要用define或const。**

我们特别喜欢用const或define定义变量或宏在一个全局可用的头文件中，这样就可以轻松愉快地用在任何地方了。譬如：

```
#define IOS6 [[[UIDevice currentDevice] systemVersion] floatValue] >= 6.0
#define IOS7 [[[UIDevice currentDevice] systemVersion] floatValue] >= 7.0

#define RED_COLOR   = ColorWithRGB(189, 77, 1)

// URL
#define MN_LOGIN [NSString stringWithFormat:@"%@/member/login.json", MN_SERVICE]

// notification key
NSString * const fLaunchEnd     =  @"fLaunchEnd";
```

这是一种偷懒的方式，其实也是符合程序员的性格和作风的。程序员的任务之一就是想尽一切办法将你的代码尽可能地简化。这样的方式，其实无意中留出了一个“通道”，大家会把类似的变量或者宏都定义在这个头文件中。譬如：

```
#define IOS6 [[[UIDevice currentDevice] systemVersion] floatValue] >= 6.0
#define IOS7 [[[UIDevice currentDevice] systemVersion] floatValue] >= 7.0
#define IOS8 [[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0
#define IOS9 [[[UIDevice currentDevice] systemVersion] floatValue] >= 9.0


#define RED_COLOR   = ColorWithRGB(189, 77, 1)
#define BLUE_COLOR  = ColorWithRGB(15, 82, 167)

// URL
#define MN_LOGIN [NSString stringWithFormat:@"%@/member/login.json", MN_SERVICE]
#define MN_ACCOUNT [NSString stringWithFormat:@"%@/myspace/account.json", MN_SERVICE]

// notification key
NSString * const fLaunchEnd     = @"fLaunchEnd";
NSString * const fScreenStatus  = @"fScreenStatus";
```

随着项目越来越大，代码越来越多，问题也就悄悄地出现了——编译时间越来越多了，这其实是十分影响开发效率的。解决的方法还是采取“分而治之”的方法。

## 解决方法

譬如对上面的IOS6、IOS7、IOS8和IOS9来说，可以写在UIDevice+Extensions中：

```
- (BOOL)greaterThanOrEqualTo6
{
    return [[self systemVersion] floatValue] >= 6.0f;
}

//  用的时候
if ([[UIDevice current] greaterThanOrEqualTo6]) {
    ...
}
```

而对于所谓的RED\_COLOR和BLUE\_COLOR，可以写在UIColor+Extensions中：

```
+ (UIColor *)colorWithValue:(NSUInteger)rgbValue
{
    return [UIColor colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 green:((float)((rgbValue & 0xFF00) >> 8))/255.0 blue:((float)(rgbValue & 0xFF))/255.0 alpha:1.0];
}

+ (UIColor *)normalRedColor
{
    return [UIColor colorWithValue:0xfd5f3a];
}

+ (UIColor *)normalBlueColor
{
    return [UIColor colorWithValue:0x418ff1];
}
```

同理，对notification key来说，倒可以用const NString ＊ kNotification  
来定义。而对于URL来说，其实大可不必单独定义一个宏，直接放入相应的请求中即可，相应的URL宏本身也只有相应的请求才会用到，即整份代码中只有一处地方用到此宏，定义这样的宏实在是多此一举。  
但也不是不能用define或者const，对于改动不大的，且能省下不少代码量的，用define或者const会更好，如：

```
#define kWindowWidth CGRectGetWidth([UIScreen mainScreen].bounds)
#define kWindowHeight CGRectGetHeight([UIScreen mainScreen].bounds)
```



