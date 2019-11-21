# iOS编码规范8－少用继承2

> by 木愚

## 前言

前文聊到用category（主要讲的是Method Swizzling），这篇文章就主要来讲讲用如何用组合的方式解决一些问题。  
木愚最早在设计网络请求基础类时BaseRequestManager，利用了Objective-C的多态性开放了一些接口，项目组的其它成员可以继承这个基础类定制自己的需求，设计如下：

```
BaseRequestManager
{
// 发起请求
- (void)loadData;

// 上传接口
- (void)uploadData;

// 取消请求
- (void)cancelRequest;

// 请求接口文件名
- (void)requestFileName;

// 请求方式
- (void)requestType;

// 请求参数
- (void)requestParam;

// 请求时间
- (void)requestTime;

// 请求公共参数
- (void)requestPublicParam;

......
}
```

然而用着用着，木愚和项目组的成员们发现一些问题。

* 哪些函数是允许子类覆盖的？  
* 哪些函数是不允许子类覆盖的？  
* 哪些函数是子类必须覆盖的？  
* 哪些函数是子类非必须覆盖的？  

当然可以利用注释解决这个问题，类似这般：

```
BaseRequestManager
{
/*
  以下接口不允许子类覆盖
*/
// 发起请求
- (void)loadData;

// 上传接口
- (void)uploadData;

// 取消请求
- (void)cancelRequest;

/*
  以下接口子类必须覆盖
*/
// 请求接口文件名
- (void)requestFileName;

// 请求方式
- (void)requestType;  // POST、GET等

// 请求参数
- (void)requestParam;

/*
  以下接口子类非必须覆盖，子类没有覆盖的话，以父类为准
*/
// 请求时间
- (void)requestTime;

// 请求公共参数
- (void)requestPublicParam;

......
}
```

但是有时候时间紧迫或者一时没注意了，还是会出事情，譬如漏掉了下面这个函数

```
- (void)requestType;
```

程序能跑，但是请求就是失败，经验丰富的可能一下子就找到问题了，经验较少的或者新员工就会耗费不少时间在这儿。

于是木愚开始各种Google并请教大神，终于找到一个较好的解决方法。

## 解决方法

```
BaseRequestManager
{
// 发起请求
- (void)loadData;

// 上传接口
- (void)uploadData;

// 取消请求
- (void)cancelRequest;

@property (weak, nonatomic) id<RequestManagerDataSource> dataSource;
}

/**
 负责网络请求所需要的数据生成
 */
@protocol RequestManagerDataSource <NSObject>

@required
- (NSDictionary *)requestParam;
- (NSUInteger)requestType;
- (NSString *)requestFileName;
......

@optional
- (NSTimeInterval)requestTime;
- (NSDictionary *)requestPublicParam;
......

@end
```

这样做呢其实是面向接口编程的思想（IOP），好处：

* 子类不需要关心父类的逻辑，只要负责完成代理中相应函数就可以了  
* 代理规范好了哪些函数是必须实现的，哪些函数是非必须实现的。至于不在代理中的函数是建议覆盖的。



