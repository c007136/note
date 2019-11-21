# iOS编码规范2－目录结构

> by 木愚

这一篇主要聊聊目录结构，首先我们假设一个app，是TabBar结构的，里面有“首页”、“社区”、“我的”三大模块。  
一个app的代码除了第三方库可以分为两大块：业务代码和公共代码，公共代码又可分为业务相关的公共代码和非业务相关的公共代码。

```
App
│
│———— Business Code
│    │
│    │—— Home
│    │    
│    │—— Social
│    │ 
│    │—— Me
│ 
│ 
│———— Module
│    │ 
│    │—— Business Module
│    │ 
│    │—— Public Module
│    │
```

对于“首页”、“社区”、“我的”三大模块，我们希望：

* 模块与模块之间是零冲突，避免命名的冲突，尤其是分类的函数命名冲突
* 模块是很独立的，和其他的模块耦合度尽可能降低，简单的说抛弃其他模块和TabBarController，也是一个可以独立运行的app。
* 模块内部是可扩展的，模块可以由小模块组成，因为随着业务的迭代式增长，代码会急剧膨胀，这就使得代码难以查找、管理和维护。譬如社区模块，最开始可能只是简单的文章展示，后面会陆陆续续加入“评论”功能、“圈子”功能等。
* 模块的图片等资源是公用的，一来图片资源都比较大，二来一张图片往往在模块的多个页面使用，如果不公用包会急剧增大。

避免冲突最好的方式是加前缀，一般来说是以公司名字的首字母为前缀，好了在讨论之前，我们先假定本人待的公司名称叫“大宇宙木愚科技有限公司”，前缀M。有两个一起码代码的同事：小X，小Z以及本人小Y，公司正在做一个社区类app，展示大宇宙的各种星系、人种、怪物、异事......这是谁的点子来的，难怪融不到资。  
好了我们先来定前缀：

```
Home -- MH
Social -- MS
Me -- MM
业务性相关的公共代码 -- MY
```

至于非业务性相关的公共代码，为了避免小X，小Y，小Z之间的命名冲突，我们这般处理：各人写的私有库带上各人的一个专属字母（比如X、Y、Z），库的前缀依次为：XA，XB，XC......只要不跟常用库冲突即可。这样做也能使得库更加独立，可以为库写自己的分类，哪怕多写点重复代码也无所谓，毕竟独立不依赖别的库，移植起来就方便多了。示例：

```
非业务性相关的公共代码
│  
│———— XAHUD（菊花控件）
│———— XBTextField
│......
│
│———— YANetWork
│———— YBAlertView
│......
│
│———— ZATrack（埋点）
│———— ZBLoopProgressButton
│......
```

对于第四点来说，较为好理解，一个模块下面只有一个Resource目录，也就是最开始社区模块的目录结构可能是：

```
Social(MS)
│
│———— Controller
│———— View
│———— Request
│———— Category
│———— DB
│———— Resource
```

随着业务的迭代发展，可能这样转变：

```
Social(MS)
│
│———— News
│    │
│    │———— Controller
│    │———— View
│    │———— Request
│    │———— Category（给News小模块使用）
│    │———— DB
│    
│———— Comment
│    │
│    │———— Controller
│    │———— View
│    │———— Request
│    │———— Category（给Comment小模块使用）
│    │———— DB
│    
│———— Circle
│    │
│    ......
│    
│———— Category（给整个Social模块使用）
│    
│———— Resource
```

最终的目录结构可能会是这个样子：

```
App
│
│———— Business Code
│    │
│    │———— Home(MH)
│    │
│    │———— Social(MS)
│    │    │———— News
│    │    │    │
│    │    │    │———— Controller
│    │    │    │———— View
│    │    │    │———— Request
│    │    │    │———— Category（给News小模块使用）
│    │    │    │———— DB
│    │    │
│    │    │———— Comment
│    │    │    │
│    │    │    │———— Controller
│    │    │    │———— View
│    │    │    │———— Request
│    │    │    │———— Category（给Comment小模块使用）
│    │    │    │———— DB
│    │    │    
│    │    │———— Circle
│    │    │    │
│    │    │    ......
│    │    │    
│    │    │———— Category（给整个social模块使用）
│    │    │    
│    │    │———— Resource
│    │
│    │————Me(MM)
│
│
│———— Module
│    │
│    │———— Business Module(MY)
│    │    │
│    │    │———— MYGoldCoin(做任务获取金币)
│    │    │
│    │    │......
│
│———— Public Module
│    │  
│    │———— XAHUD（菊花控件）
│    │———— XBTextField
│    │......
│    │
│    │———— YANetWork
│    │———— YBAlertView
│    │......
│    │
│    │———— ZATrack（埋点）
│    │———— ZBLoopProgressButton
│    │......
```

## 关于图片的命名

因为Resource目录是给整个模块使用的，图片的命名要带上类名前缀，这样方便查找、管理和删除。当然View，ViewController等可以去掉，毕竟属于View，还是属于ViewController不好说的。  
示例：

```
MSSocialNewsDetail_Header@2x.png
```

当然也可以这样，表示给整个模块使用的：

```
MSSocial_Header@2x.png
MSSocialComment_Header@2x.png
```



