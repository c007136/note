# autorelease和autoreleasepool

内存管理一直是Objective-C的重点和难点之一，而只有理解了autorelease的原理，才能了解Objective-C的内存管理机制。

在MRC下，我们需要手动管理内存，写一大堆的retain，release代码，稍不留神就会造成内存泄露。另外调用[object autorelease]可以延迟对象的内存释放。

