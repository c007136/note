# iOS编码规范5－代码组织

> by 木愚

## 前言

> **遵循特定的结构组织代码，并用\#pragma mark标注。** 

> 1. lifecycle  
> 2. delegate  
> 3. action  
> 4. response  
> 5. public method  
> 6. private method  
> 7. super  
> 8. getter and setter

另外对于每个delegate，都要老老实实写在上面，方便查看delegate的方法（command＋点击）。

```
#pragma mark - lifecycle
- (void)viewDidLoad
{
}

#pragma mark - UITableViewDelegate and UITableViewDataSource

...

#pragma mark - RequestManagerCallbackDelegate

...
```

## 另外

对于一些不能好归类为上面的划分区域，那另外单独开辟代码区域。类似MJRefresh的SEL方法：

```
#pragma mark - life circle
- (void)viewDidLoad
{
}

#pragma mark - UITableViewDelegate and UITableViewDataSource

...

#pragma mark - RequestManagerCallbackDelegate

...

#pragma mark - MJRefresh

...
```



