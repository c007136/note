# iOS编码规范1－基础相关

> by 木愚

## 前言

请先阅读苹果官方的规范[《Apple Coding Guidelines for Cocoa》](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html#//apple_ref/doc/uid/10000146-SW1)， 另外请遵守下面内容。

## 括号

括号在Objective-C中，推荐的方式是“在同一行语句中打开在新一行语句中关闭”，类似：

```
if (user.isHappy) {
    //注释1我很开心
    code1
    code2
} else {
    //注释2我不开心
    code1
    code2
}
```

然而，我认为在用于方法、代码中有嵌套括号和ifelse时，需要采取这样的方式。  
应该：

```
- (void)viewDidLoad
{
}

// 函数与函数之间空一行 
- (void)viewWillAppear
{
}

// 函数与函数之间空一行 
- (void)viewDidAppear
{
}
```

不应该：

```
- (void)viewDidLoad {
}

- (void)viewWillAppear {
}

- (void)viewDidAppear {
}
```

应该：

```
for (....) 
{
    if (1)
    {

    }
}
```

不应该：

```
for (....) {
    if (1) {

    }
}
```

应该：

```
if
{
}
else
{
}
```

不应该：

```
if {
} else {
}
```

理由是：  
1. 代码量超出5行后，括号的对齐就尤为关键。  
2. 对于ifelse来说，注释和调试更为方便。  
这样的格式：

```
// 案例一
if (...)
{
}
// 案例二
else if (...)
{
}
// 案例三
else
{
}
```

好于：

```
if (...) {  // 案例一
    code1
} else {    // 案例二
    code2
}
or
if (...) {
    //注释1我很开心
    code1
    code2
} else {
    //注释2我不开心
    code1
    code2
}
```

这样调试：

```
//if
//{
//    //do something
//}
//else
{
    //do something
}
```

方便于：

```
/*if {
    // do somethig
} else */{
    // do something
}
```

故而在此规定，函数的括号使用这样的方式

```
- (void)fun1
{
}

- (void)fun2
{
}
```

而行数较多的代码也推荐上面的方式，行数较少的代码可使用这样的方式

```
if (1) {
}
```

## 短代码风格

> **将长代码拆分成若干短代码**

应该：

```
NSDictionary * dict = self.newsTableDatas[indexPath.section];
NSNumber * ID = dict[kQuestionDataFabricantCellQuestionID];
```

不应该：

```
NSNumber * ID = self.newsTableDatas[indexPath.section］[kQuestionDataFabricantCellQuestionId];
```

## BOOL变量

> **不可将布尔变量直接与YES、NO或者 1、0 进行比较。**

应该:

```
if (flag)
if (!flag)
```

不应该：

```
if (flag == YES)
if (flag == 1)
if (flag == NO)
if (flag == 0)
```

更多信息，参考[这里](http://www.jianshu.com/p/678ee3ee4a64)和[NSHipster.cn](http://nshipster.cn/bool/)

## 命名

> **使用驼峰式命名，且长的描述性的命名会更好**

应该：

```
UIButton *settingButton;
```

不应该：

```
UIButton *setBut;
```

#### 1.常量命名

作用域不超出.m文件，在常量前边加字母k作为标记（当然记得加上static，否则在不同.m文件中重复命名会报编译错误）  
应该：

```
static const NSTimeInterval kAnimationDuration = 0.3
```

而不应该：

```
const NSTimeInterval animationDuration = 0.3
```

作用域超出.m文件，命名加相关类名前缀，并加上extern  
应该：

```
extern NSString * const AFURLResponseSerializationErrorDomain;
```

不应该：

```
extern NSString * const errorDomain;
```

#### 2.枚举命名

枚举类型命名要加相关类名前缀，枚举值命名要加枚举类型前缀，并使用宏NS\_ENUM来固定枚举的基本类型。  
示例：

```
typedef NS_ENUM(NSInteger, SDImageCacheType) {
    SDImageCacheTypeNone,
    SDImageCacheTypeDisk,
    SDImageCacheTypeMemory
};
```

## instancetype

init方法和类构造方法返回类型应该返回instancetype而不是id，类方法可直接返回相应的类型。

```
- (instancetype)init
```

```
@interface UIImage (MultiFormat)

+ (UIImage *)sd_imageWithData:(NSData *)data;

@end
```

更多信息，参考这里[NSHipster.com](http://nshipster.com/instancetype/)

## 条件语句

条件语句主体为了防止出错应该使用大括号包围，即使条件语句主体能够不用大括号编写\(如，只用一行代码\)。这些错误包括添加第二行代码和期望它成为if语句；还有，even more dangerous defect可能发生在if语句里面一行代码被注释了，然后下一行代码不知不觉地成为if语句的一部分。除此之外，这种风格与其他条件语句的风格保持一致，所以更加容易阅读。  
应该：

```
if (!error) {
  return success;
}
```

不应该：

```
if (!error)
  return success;
```

或

```
if (!error) return success;
```

另外当使用条件语句编码时，左手边的代码应该是"golden" 或 "happy"路径。也就是不要嵌套if语句，多个返回语句也是OK。  
应该：

```
- (void)someMethod {
  if (![someOther boolValue]) {
    return;
  }
  //Do something important
}
```

不应该：

```
- (void)someMethod {
  if ([someOther boolValue]) {
    //Do something important
  }
}
```

## Literals（字面值）

NSString、NSDictionary、NSArray和NSNumber的字面值应该在创建这些类的不可变实例时被使用。请特别注意nil值不能传入NSArray和NSDictionary字面值，因为这样会导致crash。  
应该：

```
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone": @"Kate", @"iPad": @"Kamal", @"Mobile Web": @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingStreetNumber = @10018;
```

不应该：

```
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingStreetNumber = [NSNumber numberWithInteger:10018];
```

## CGRect函数

当访问CGRect里的x, y, width, 或 height时，应该使用CGGeometry函数而不是直接通过结构体来访问。  
应该：

```
CGRect frame = self.view.frame;
CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
CGRect frame = CGRectMake(2*x, 2*y, width, height);
```

不应该：

```
CGRect frame = self.view.frame;  
CGFloat x = frame.origin.x;  
CGFloat y = frame.origin.y;  
CGFloat width = frame.size.width;  
CGFloat height = frame.size.height;  
CGRect frame = CGRectMake(2*x, 2*y, width, height);
```

更多信息，参考[这里](http://www.jianshu.com/p/aacb34e5aadb)，其实看了这篇文章，本人觉得还是把“应该”改为“推荐”更好，毕竟负的width，heigth不常见。

