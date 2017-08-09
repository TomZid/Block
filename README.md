# Malloc Block

看下面的代码
```Objective-C
- (void)viewDidLoad {
    [super viewDidLoad];
    int __block a = 10;
    void(^block)(void);
    //打印a数值和指针
    NSLog(@"a == %i &a == %p", a ,&a);
    block = ^{
        //block 捕获变量作为block的成员变量，改变被捕获的变量，外部变量也会被改变
        a = 11;
        //打印a数值和指针
        NSLog(@"a block == %i &a == %p", a ,&a);
    };
    block();
    //打印a数值和指针
    NSLog(@"a == %i &a == %p", a ,&a);
}
```
这是打印的最终结果，可以看出在block内部改变的外部变量指针发生了改变。
```
 a == 10 &a == 0x7fff52d124b8
 a block == 11 &a == 0x60800023fc38
 a == 11 &a == 0x60800023fc38
```
这个是什么原因呢，经过分析不难发现其中的道理。
block引用了外部变量，那么block初始化的isa指针变为NSMallocBlock，也就是说是堆中的block，同时引用的变量在block结构体中声明为成员变量保存在堆中。
在block中对捕获的变量进行修改为了保持与外部变量的一致，那么外部变量的指针发生了改变，指针指向了新的地址也就是改变过后的值的地址。
保守估计苹果的做法是在block中对引用的变量进行了深拷贝的同时还保留了一份指针引用，方便在修改引用时做指针改变。

苹果官方文档解释：
> __block storage is similar to, but mutually exclusive of, the register, auto, and static storage types for local variables https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/bxVariables.html#//apple_ref/doc/uid/TP40007502-CH6-SW6
