---
layout: post
title: OC Runtime(一)：Object Model part II：iVar Layout
categories: [01 Objective-C]
tags: [Sending Message]
number: [0.14.1.2]
fullview: false
shortinfo: Object Model描述了OC中描述了实例变量，实力方法，类变量，类方法以及继承关系。本文将详细介绍以上各方面在内存中的分布。

---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Object Model 概述 ##

### 1.1 Runtime 3种对象 ###

objc runtime有3种对象。

1. **instance**：类型为``id``，即指向``objc_object``的指针；``objc_object``只有1个值，即类型为``Class``的变量``isa``。

2. **class**：类型为``Class``，即指向``objc_class``的指针；``objc_class``继承自``objc_object``，因此也有类型为``Class``的变量``isa``，指向**metaClass**；``objc_class``还会存储类的实例变量表，实例方法表，协议表，实例大小等；另外``objc_class``还有1个``Class``类型的变量``superClass``，指向父类。root class(即``NSObject``)的``superClass``指向nil。

3. **metaClass**：类型也为``Class``，存储了类变量，类方法；并且有1个``Class``类型的变量``superClass``，指向父类**metaClass**。root meta class的``superClass``指回root class(即``NSObject``)。其中**class**和**metaClass**都以单例存储在内存中。

### 1.2 instance的创建 ###

{% highlight objc linenos %}
size_t = size = cls->instanceSize();
id objc = (id) calloc(1,size);
obj.isa = (uintptr_t)cls;
{% endhighlight %}

分析：``instanceSize``是编译时决定。存储在class里；``calloc``分配1块连续的内存，内存中前4Byte存储了cls地址。

### 1.3 instance，class，metaClass对应例子 ###

{: .img_middle_lg}
![对应关系](/assets/images/posts/01 Objectiev C/2016-03-14-OC Runtime(一)_Object Model/对应关系.png)

## 2 题目 ##

### 2.0 基础 ###

{% highlight objc linenos %}
//都指向Father类
[Father class]
[father class]

//指向参数的isa
Class cls = object_getClass(father);                //Father类
Class metaCls = object_getClass([Father class]);    //Father元类

BOOL res1 = [NSObject class] == [[NSObject class] class];               //1
BOOL res2 = [NSObject class] == [[[NSObject class] class] class];       //1
BOOL res3 = [NSObject class] == object_getClass([NSObject class]);      //0
BOOL res4 = object_getClass([NSObject class]) == object_getClass(object_getClass([NSObject class])); //1
BOOL res9 = [NSObject class] == class_getSuperclass(object_getClass([NSObject class]));//1

{% endhighlight %}

分析：``[NSObject class]``指向``NSObject``类,因此后面无论加多少个class还是指向``NSObject``类；但是``object_getClass(id obj)``返回的是isa变量，因此``object_getClass([NSObject class])``返回的是``NSObject``元类。结合**Object Model**很容易得出1，1，0，1，1的结论。

### 2.1 NSObject类方法输出 ###

{% highlight objc linenos %}

//  NSObject+Sark.h

#import <Foundation/Foundation.h>
        
@interface NSObject (Sark)
+(void)say;
@end
        

//  NSObject+Sark.m

#import "NSObject+Sark.h"
        
@implementation NSObject (Sark)
-(void)say{
    NSLog(@"NSObject Sark instance method:say");
}
@end
        
[NSObject say]; //output: NSObject Sark instance method:say
[[NSObject new] say]; //编译通不过，因为-say没有在头文件申明
{% endhighlight %}

分析:类别扩展了NSObject的类方法``+(void)say``，但没有提供实现；在.m文件里提供了实例方法``-(void)say``的实现。当调用``[NSObject say]``的时候，NSObject的metaClass找不到vTable里对应的IMP，就通过继承链在父类里找，而我们上面介绍过NSObject的元类的父类指向了自己。因此就到NSObject类里找IMP，也就是实例方法``-(void)say``，找到IMP后调用就输出了``NSObject Sark instance method:say``。

### 2.2 NSObject自省方法 ###

{% highlight objc linenos %}

BOOL res1_1 = [(id)[NSObject class] isKindOfClass:[NSObject class]];    //1
BOOL res1_2 = [(id)[NSObject class] isMemberOfClass:[NSObject class]];  //0
        
Class cls = object_getClass(father);
Class metaCls = object_getClass([Father class]);
    
BOOL res2_1 = [(id)[Father class] isKindOfClass:[Father class]];        //0
BOOL res2_2 = [(id)[Father class] isMemberOfClass:[Father class]];      //0
BOOL res2_3 = [(id)[Father class] isKindOfClass:metaCls];               //1
BOOL res2_4 = [(id)[Father class] isMemberOfClass:metaCls];             //1
BOOL res3_1 = [(id)father isKindOfClass:[Father class]];                //1
BOOL res3_2 = [(id)father isMemberOfClass:[Father class]];              //1


+ (BOOL)isMemberOfClass:(Class)cls {
    return object_getClass((id)self) == cls;
}

- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}

+ (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = object_getClass((id)self); tcls; tcls = class_getSuperclass(tcls)) {
        if (tcls == cls) return YES;
    }
    return NO;
}

- (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = [self class]; tcls; tcls = class_getSuperclass(tcls)) {
        if (tcls == cls) return YES;
    }
    return NO;
}
        
{% endhighlight %}

分析：``isKindOfClass``的实例方法和类方法的实现是一样的，因为实例方法``[self class]``和``object_getClass((id)self)``一样。``isKindOfClass``先返回``isa``，如果和``cls
``一致则返回``YES``，否则从``superClass``的``isa``继续比较判断。因此res1_1中，``NSObject``元类不是``NSObject``类，然后从``NSObject``元类的``superClass``找，根据object model，又指回了``NSObject
``类，因此返回1。res2_1，res2_3，res3_1同理可得。而``isMemberOfClass``的实例方法和类方法的实现也是一样，与``isKindOfClass``不同的地方在于只比较一级``isa``，而不会不相等时沿着继承链继续比较。res1_2，res2_2，res2_4，res3_2的结果也就一目了然了。

### 2.3 self & super关键字 ###

{% highlight objc linenos %}
//  Son.h
#import <Foundation/Foundation.h>
#import "Father.h"
@interface Son : Father
-(void)print;
@end

//  Son.m
#import "Son.h"
@implementation Son
-(void)print{
    NSLog(@"%@",[self class]);  //Son
    NSLog(@"%@",[super class]); //Son
}
/*
msg_sendSuper2(struct objc_super *super, SEL op, ...);
struct objc_super { 
    id receiver; 
    Class current_class
}
*/

@end
{% endhighlight %}

分析：``super``不是1个参数，而是编译器的关键字，转换成``msg_sendSuper2(struct objc_super *super, SEL op, ...)``，该方法跳过本类方法在父类中找对应``IMP``，其中``struct objc_super { id receiver；Class current_class}``的``receiver``作为``IMP``的输入参数。就是说``[super class]``在``Father``类里开始沿着继承链找``class``实例方法，找到后将``self``作为参数输入，因此两个输出都是Son。

### 2.4 What is an Object and its iVar in Memory ###

{% highlight objc linenos %}
//  Father.h
@interface Father : NSObject
@property(nonatomic, copy) NSString *firstName;
-(void)speak;

//  Father.m
#import "Father.h"

@implementation Father
-(void)speak{
    NSLog(@"my name is :%@",self.firstName);
}
@end

//ViewController.m
- (void)viewDidLoad {
    [super viewDidLoad];
    
//  NSString *name = @"Jim";
    id cls = [Father class];
    void *ptr = &cls;
    [(__bridge id)ptr speak];
    //注释掉@"Jim"，输出:  my name is: <ViewController: 0x7ae295d0>
    //插入@"Jim"，  输出:  my name is: Jim
}
{% endhighlight %}

分析：这里是将instance分配在stack上，id结构体的地址和isa地址是同一块，因为isa是id结构体的第一个(也是唯一一个变量)。当调用``[ptr speak]``的时候，``objc_sendMsg(id self, SEL sel)``穿进去的``self``是分配在stack上的``ptr``，通过``isa``找到``speak``IMP后，调用了``self.firstName``，即ptr偏移4Byte。由于stack的地址是上面高，下面低，因此+4Byte就找到了``name``。

如果``name``不在,则是``viewController``，一种说的通的解释如下。

{% highlight objc linenos %}
//ViewController.m
- (void)viewDidLoad {
    // 以下数字越高表示地址越大
    // 压栈参数1： id self (4)
    // 压栈参数2： SEL _cmd (3)
    [super viewDidLoad]; // objc_msgSendSuper2(struct objc_super, SEL)
    // struct objc_super2 {
    //    self, (1)
    //    self.class (2)
    // }
    // objc_msgSendSuper2的SEL不需要申请栈空间

    id cls = [Sark class];
    void *obj = &cls; // (0)
    [(__bridge id)obj speak];
}
{% endhighlight %}

最后总结成
{% highlight objc linenos %}
//objc中的对象是1个指向ClassObject地址的变量
id obj = &ClassObject
//而对象的实例变量
void *ivar = &obj+offset(N)
{% endhighlight %}


## 3 总结 ##

{: .img_middle_hg}
![Runtime_Object Model](/assets/images/posts/01 Objectiev C/2016-03-14-OC Runtime(一)_Object Model/Runtime_Object Model.png)

## 4 Reference ##

- [《Runtime Programming Guide》](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html);

- [《Understanding the Objective-C Runtime》](http://cocoasamurai.blogspot.hk/2010/01/understanding-objective-c-runtime.html);

- [《重识 Objective-C Runtime - Smalltalk 与 C 的融合》](http://blog.sunnyxx.com/2016/08/13/reunderstanding-runtime-0/);



