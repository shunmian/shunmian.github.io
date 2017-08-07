---
layout: post
title: OC Memory Managment(二)：Autoreleasing
categories: [01 Objective-C]
tags: [MemoManamement]
number: [5.5.1]
fullview: false
shortinfo: ARC的引入，使得程序员摆脱了MRC那个"黑暗的时代"。本文是对OC ARC的概述，意在使得开发者在写OC代码的时候，了解编译器在背后做了哪些内存管理的事情。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. 引用计数 ##

在ARC时代，为什么我们还要理解MRC下的引用计数？原因主要有二：

1. ARC并非万能。理解引用计数是我们解决循环引用的基础；

2. 了解引用计数可以加深对OC内存管理的理解。

### 1.1 MRC vs ARC###

新开1个工程，在**Build Phases**的`AppDelegate.m`里添加`-fno-objc-arc`，可以开启MRC模式。

{: .img_middle_mid}
![MRC command](/assets/images/posts/01 Objectiev C/2016-03-25-OC Memory Managment(一)_ARC/MRC command.png)

输入如下代码。

{% highlight objc linenos %}

//  AppDelegate.m

#define isARC 1
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
#if isARC == 0      //-fno-objc-arc
    NSObject *object = [[NSObject alloc] init];
    NSLog(@"Reference Count = %u", [object retainCount]);                           //1
    NSObject *anotherObject = [object retain];
    NSLog(@"Reference Count = %u", [anotherObject retainCount]);                    //2
    [anotherObject release];
    NSLog(@"Reference Count = %u", [anotherObject retainCount]);                    //1
    [object release];
    NSLog(@"Reference Count = %u", [anotherObject retainCount]);                    //1, why not 0?
#elif isARC == 1
    NSObject *object = [[NSObject alloc] init];
    NSLog(@"Reference count = %ld", CFGetRetainCount((__bridge CFTypeRef)object));  //1
    //[object retain];
    NSObject *anotherObject = object;
    NSLog(@"Reference count = %ld", CFGetRetainCount((__bridge CFTypeRef)object));  //2
    anotherObject = nil;
    //[object release];
    NSLog(@"Reference count = %ld", CFGetRetainCount((__bridge CFTypeRef)object));  //1
    object = nil;
    //[object release];
    NSLog(@"Reference count = %ld", CFGetRetainCount((__bridge CFTypeRef)object));  //Thread 1: EXC_BREAKPOINT(code=EXC_I386_BPT,subcode=0X0)
#endif
    return YES;
}
{% endhighlight %}

在MRC模式下(`-fno-objc-arc`)，我们可以看到：

1.在实例化`NSObject`时，`object`指向了该实例，因此该实例`retainCount`从0变为1。

2.`anotherObject`也指向该实例，并且`retain`，则`retainCount`从1变为2。如果是用`=object`而不是`=[object retain]`则`retainCount`依旧是1。

3.`anotherObject release`使得该实例`retainCount`-1。

4.`object release`使得该实例`retainCount`-1变为0释放。但是为什么`retainCount`输出还是1呢？因为该对象的内存已经被回收，而我们向一个已经被回收的无效对象发了一个`retainCount`消息，所以它的输出结果应该是不确定的。不将无效对象的这个值从1变成0，可以减少一次内存的写操作，加速对象的回收。

在ARC模式下(删除`-fno-objc-arc`)，retain和release是由系统自动加入到适当的位置的：

1. 实例化过程引用计数和MRC一样。若增加`__weak`修饰符，则`NSObject`实例之初就会被释放。

2. 这和3一样。

3. `object = nil`会使实例`retainCount`减1。这里减1后为0。

4. 向已释放的实例发送`CFGetRetainCount((__bridge CFTypeRef)object)`(ARC下`retainCount`已被禁止)会触发野指针错误。


## 2 ARC4种所有权修饰符： __weak，__strong，__autoreleasing，__unsafe_unretained##

**属性修饰符**对应的4种**所有权修饰符**如下所示：

{: .img_middle_lg}
![ARC4种对象所有权修饰符 vs 属性修饰符](/assets/images/posts/01 Objectiev C/2016-03-25-OC Memory Managment(一)_ARC/ARC4种对象所有权修饰符 vs 属性修饰符.png)


### 2.1 __weak ###

表示弱引用，对应属性修饰符`weak`。弱应用对对象的生命周期无影响。对象在被释放的同时，指向它的弱引用会被自动置为nil，这个技术称为**zeoring weak pointer**。这样有效地防止了野指针的产生。`__weak`用在防止循环引用的场景中(如**delegate**，**block**)和无效的强引用中(如@IBOutlet)。

### 2.2 __strong ###

表示强引用，对应属性修饰符`strong`和`copy`。当所有对象没有任何一个强引用时，才会被释放。如果在声明时不加修饰符，默认是强应用。当需要释放强引用指向的对象时，需要将强引用置nil。

### 2.3 __autoreleasing


### 2.4 __unsafe_unretained

和__weak一样，但对象释放后，不会自空，所以非常不安全。由于ARC是在iOS 5引入的，而这个修饰符主要是兼容iOS 4以及更低版本的设备，现在可以完全忽略这个修饰符了，因为iOS 4早已退出历史舞台很多年。

## 2. Retain Cycle 问题及解决 ##
。


### 2.1 Retain Cycle 问题 ###

{% highlight objc linenos %}

{% endhighlight %}




## 3 总结 ##
。


