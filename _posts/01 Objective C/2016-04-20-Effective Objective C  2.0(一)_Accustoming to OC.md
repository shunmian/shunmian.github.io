---
layout: post
title: EOC(一)：Accustoming yourself to Ojective-C
categories: [01 Objective-C]
tags: [Effective Objective-C]
number: [01.25.1]
fullview: false
shortinfo: 本文是《Effective Objective-C》的系列笔记的第一篇《Accustoming yourself to Ojective-C》，对应书本的第一章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Accustoming yourself to Ojective-C ##

### Item 1：Familiarize Yourself with Objective-C's Roots

1. OC的**Message Sending**机制和其他OOP语言的**Funcion Call**不一样。前者在运行时决定object是哪个类(黑魔法isa swizziling)和该类的哪个方法(黑魔法 method swizziling)。后者在编译的时候已经决定object是哪个类和该类的哪个方法。OC的这种动态绑定(Dynamic Binding)的机制会在[Item 11]()具体介绍。

2. ``NSString *string1 = @"Hi"``会在stack上开辟``string1``变量的空间，指向在heap上开辟``@"Hi"``字符串对象的地址。stack上的变量不需要内存管理，因为frame pop的时候自动会clean；而heap上的对象需要内存管理，OC通过reference counting而不是malloc和delloc来管理，见[Item 29]()。

3. stack上的变量，例如struct，包括``CGRect``,``CGPoint``等，比heap上的对象更有效，因为不需要内存管理等overhead。因此在遇到性能瓶颈时，如果数据结构只包含非object type的数据(int,float,double,char)等，可以用struct来优化。

### Item 2：Minimize Importing Headers in Headers

1. Always import the header in the very deepest point possible.

考虑下面``EOCPerson类``

Version 1
{% highlight objc linenos %}

//EOCPerson.h
#import "EOCEmployee.h"

@interface EOCPerson:NSObject
@property(nonatomic,strong)EOCEmployee *employee;
@end


//EOCPerson.m
#import "EOCPerson.h"

@implementation EOCPerson
//implmentation of method
@end
{% endhighlight %}

Version 2，better than version 1 since EOCEmployee is not visible for any other objects who import "EOCPerson.h"

{% highlight objc linenos %}

//EOCPerson.h
@class EOCEmployee

@interface EOCPerson:NSObject
@property(nonatomic,strong)EOCEmployee *employee;
@end


//EOCPerson.m
#import "EOCPerson.h"
#import "EOCEmployee.h"

@implementation EOCPerson
//implmentation of method
@end


{% endhighlight %}

### Item 3：Prefer Literal Syntax over the Equivalent Methods

1. 用``NSString``，``NSNumber``，``NSArray``，``NSDicitonary``等literal syntax来初始化更紧凑易读。

2. 可用下标literal syntax来读写数组或字典。

3. 若NSArray或NSDictionary的Object为nil，则会抛出异常(及时抛出异常是好事，相比于它们非literal syntax的初始化，只会停在第一个nil的object处，会带来难debug的问题)，需要确定不会有nil。

{% highlight objc linenos %}

//NSString
NSString *string1 = @"Hi";

//NSNumber
NSNumber *intNumber = @1;
NSNumber *floatNumber = @1.2;
NSNumber *doubleNumber = @1.234567;
NSNumber *boolNumber = @YES;
NSNumber *charNumber = @'a';
NSNumber *expressionNumber = @(intNumber * floatNumber);

//NSArray, NSMutableArray
NSArray *array = @[@1,@2,@3];
NSMutableArray *mutableArray = [@[@1,@2,@3] mutableCopy];
mutableArray[0]=@10;

NSArray *array1 = @[obj1,obj2,obj3]; //if any of the obj item is nil, exception will raise "attempt to insert nil object from..."
NSArray *array2 = [NSArray arrayWithObjects:obj1,obj2,obj3,nil] //if obj2 is nil, then array2 only contain obj1, no exception will raise, which is bad practice

//NSDictionary, NSMutableDictionary
NSDictionary *dictionary = @{@1:@10,
                             @2:@20,
                             @3:@30};
NSMutableDictionary *mutableDictionary = [@{@1:@10,
                                            @2:@20,
                                            @3:@30} mutableCopy];
mutableDictionary[@1]=@100;
{% endhighlight %}

### Item 4：Prefer Typed Constants to Preprocessor #define

1. 避免#define来定常数，因为没有type信息。

2. 用type constant时，若是local，则用``Static NSString *const StringConstant``在.m文件里声明和定义；若是global，则用``Extern NSString *const UserStringConstant``在.h文件里声明和用``NSString *const UserStringConstant``在.m文件里定义，记得global要prefix class name作为namespace因为它们是global，有可能会和其他global常量conflict。

{% highlight objc linenos %}
//User.h
extern NSString *const UserStringConstant;
@interface User : NSObject
@end


//User.m
#import "User.h"
//define, no type information, bad practice
#define kStringConstant @"123" 
//type constants, has type, good practice
    //for local, no prefix: static means local class iVar,
    //                      NSString *const means pointer content cannot change(raise error if try to change)
static NSString *const StringConstant = @"StringConstant";
    //for external,define in .m here as following with prefix classname as namespace
    //also "extern NSString *const UserStringConstant;" in .h so that other object when import "User.h" can use UserStringConstant
NSString *const UserStringConstant = @"UserStringConstant";
@implementation User
...
@end


{% endhighlight %}


### Item 5：Use Enumerations for States, Options, and Status Codes

1. 用enumeration来定义discrete states，error code 或者函数的options。

2. 用``NSENUM``和``NS_OPTIONS``来定义enumeration时，显示指定type而不是让compiler来指定会更好。

3. 如果options可以组合，可以用2的倍数来定义：用or来组合；and来提取。

4. 用``switch``使用enumeration时不要有``defaul``，因为当你增加enumeration新的选项时，compiler就会提醒你``swithc``没有处理新的选项。

{% highlight objc linenos %}

# 2,3
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};

UIViewAutoresizing resizing = UIViewAutoresizingFlexibleWidth |
                              UIViewAutoresizingFlexibleHeight;
if(resizing & UIViewAutoresizingFlexibleWidth){
    // UIViewAutoresizingFlexibleWidth is set
}

# 2,4
typedef NS_ENUM(NSUInteger, EOCConnectionState){
    EOCConnectionStateDisconnected,
    EOCConnectionStateConnecting,
    EOCConnectionStateConnected,
}

switch(_currentState){
    EOCConnectionStateDisonnected:
        //Handle disconnected state
        break;
    EOCConnectionStateConnecting:
        //Handle connecting state
        break;
    EOCConnectionStateConnected:
        //Handle connected state
        break;
}

{% endhighlight %}


## 2 Reference ##

- [《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs (Effective Software Development Series)》](https://www.amazon.com/Effective-Objective-C-2-0-Specific-Development/dp/0321917014);


