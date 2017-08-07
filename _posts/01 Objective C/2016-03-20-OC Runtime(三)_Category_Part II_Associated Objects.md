---
layout: post
title: OC Runtime(三)：Category Part II：Associated Objects
categories: [01 Objective-C]
tags: [Associated Objects]
number: [0.14.1.2]
fullview: false
shortinfo: 在Objective C 中，一旦类被定义好了，想扩展它的iVar是被禁止的，除非重写改写类本身。我们知道Category可以被用来扩展方法而非iVar，而Associated Objects技术就是用来在遵循实例变量不能扩展的前提下，增加property的。这样在外部看来可以用dot notatoin access 所有的属性，就像达到了iVar可以被扩展的假象。本文就来详细介绍一下Objective C Runtime的Associated Objects。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. iVar VS Property ##

Obective C 的类，本质是`struct objc_class`结构体，它的定义如下：

{% highlight c linenos %}
typedef struct objc_class *Class; 
struct objc_class { 
   Class isa; 
   Class super_class; 
   long version ; 
   long info ; 
   long instance_size ; 
   struct objc_ivar_list *ivars ; 
   struct objc_method_list **methodLists ; 
   struct objc_cache *cache ; 
   struct objc_protocol_list *protocols ; 
} ;
{% endhighlight %}

`objc_ivar_list *ivars`是一个存储了`objc_ivar`的list。在Objective C中我们知道`@property`是声明了setter和getter的语法糖,
 `@synthesis` 是实现了setter和getter的语法糖(Xcode 4.4及之后的版本可以省略)。在类中声明的每一个property都被一个`iVar` backup（property 名字前加_）。`objc_method_list **methodLists`是一个指针的指针。通过修改该指针指向的指针的值，就可以实现动态地为某一个类增加成员方法。这也是Category实现的原理。同时也说明了为什么Category只可为对象增加成员方法，却不能增加成员变量。

 关于`objc_ivar_list`, 我们看下面代码。

{% highlight objc linenos %}
Person.h

@interface Person : NSObject{
    NSString *ivarName;
}
@property (nonatomic, strong) NSString *propertyName;
@end


Person.m

@implementation Person
@synthesize propertyName = _propertyName;//可省略
-(instancetype)init{
    if(self = [super init]){
        ivarName = @"ivar";
        _propertyName = @"property";
    }
    return self;
}
@end
{% endhighlight %}

我们创建了一个`Person`类，其有一个iVar为NSString *类型的`ivarName`，一个property为NSString *类型的`propertyName`。我们打印Person类的`objc_ivar_list *ivars`:

{% highlight objc linenos %}

Person *person = [[Person alloc] init];
    
id personClass = [person class];
unsigned int ivarNumber = 0;
printf("Person objc_ivar_list: ");
Ivar *ivarArray = class_copyIvarList(personClass, &ivarNumber);
for(int i = 0; i < ivarNumber; i++){
    Ivar ivar = ivarArray[i];
    printf("%s, ",ivar_getName(ivar) );
}

//输出为:
//Person objc_ivar_list: ivarName, _propertyName, 
//
{% endhighlight %}

可见在一个类的本身定义中, iVar和Property都被加在`objc_ivar_list`中。

## 2 Associated Objects增加属性  ##

### 2.1在匿名类中增加属性  ###

我们知道`Category`可以用来扩展方法，但扩展不了类的iVar。Associated Objects 的出现就是为了解决这一问题。它让`Category`增加属性，就好像类本身定义中的属性一样可以用dot notation进行存取。但是它不加入类的`objc_ivar_list`中，而是存储在一个哈希表中。我们来看下面代码，给`Person+AssociatedObjects`增加一个属性类型为NSString *`的associatedObjctName`:


{% highlight objc linenos %}

Person+AssociatedObjects.h

#import "Person.h"
#import <objc/runtime.h>

@interface Person (AssociatedObjects)
@property (nonatomic, copy) NSString *associatedObjcName;
@end


Person+AssociatedObjects.m

@implementation Person (AssociatedObjects)

-(NSString *)associatedObjcName{
    return objc_getAssociatedObject(self, @selector(associatedObjcName));
}

-(void)setAssociatedObjcName:(NSString *)associatedObjcName{
    objc_setAssociatedObject(self, @selector(associatedObjcName), associatedObjcName, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
@end

{% endhighlight %}

我们在`Person(AssociatedObjects)`中增加了属性类型为NSString *的`associatedObjctName`,并用runtime的API实现了其setter和getter方法。我们再来运行下面代码:


{% highlight objc linenos %}
Person *person = [[Person alloc] init];
person.associatedObjcName = @"associated objects";
NSLog(@"associatedObjcName: %@", person.associatedObjcName);
    
id personClass = [person class];
unsigned int ivarNumber = 0;
Ivar *ivarArray = class_copyIvarList(personClass, &ivarNumber);
printf("Person objc_ivar_list: ");
for(int i = 0; i < ivarNumber; i++){
    Ivar ivar = ivarArray[i];
    printf("%s, ",ivar_getName(ivar) );
}

//输出为:
//associatedObjcName: associated objects
//Person objc_ivar_list: ivarName, _propertyName, 
//
{% endhighlight %}

我们可以看见`objc_ivar_list`并没有新加入的associated objects。

### 2.2 在Runtime API中动态增加属性  ###
当用runtime API 动态创建类，添加方法和实例变量过程中，当类创建完毕后（调用`objc_registerClassPair`后）再用`class_addIvar(...)`添加的iVar不会出现在类的`objc_ivar_list`中，见如下代码。

{% highlight objc linenos %}
 //dynamically create a Peron Class
 Class person = objc_allocateClassPair([NSObject class], "Person", 0);
     // add a instance method
 Method description = class_getInstanceMethod([NSObject class], @selector(description));
 const char *methodType = method_getTypeEncoding(description);
 class_addMethod([person class], NSSelectorFromString(@"greeting"), (IMP)greeting, methodType);
        // add a instance variable before register
 const char *nameBeforeRegister = "nameBeforeRegister";
 class_addIvar(person, nameBeforeRegister, sizeof(id), rint(log2(sizeof(id))), @encode(id));
    
  //register Person Class
 objc_registerClassPair(person);
    
     // add a instance variable after register
 const char *nameAfterRegister = "nameAfterRegister";
 class_addIvar(person, nameAfterRegister, sizeof(id), rint(log2(sizeof(id))), @encode(id));
    
 //create a instance
 id personInstance = [[person alloc] init];
     //call the method
 NSLog(@"%@", ((id (*)(id, SEL))objc_msgSend)(personInstance, NSSelectorFromString(@"greeting")));
     //dynamically add a variable(an associated object) to the personInstance and get it
 NSString *firstname = @"Johnson";
 objc_setAssociatedObject(personInstance, nameBeforeRegister, firstname, OBJC_ASSOCIATION_COPY_NONATOMIC);
 NSLog(@"%@", objc_getAssociatedObject(personInstance,nameBeforeRegister));
    
 NSString *lastname = @"Lu";
 objc_setAssociatedObject(personInstance, nameAfterRegister, lastname, OBJC_ASSOCIATION_COPY_NONATOMIC);
 NSLog(@"%@", objc_getAssociatedObject(personInstance,nameAfterRegister));
    

    
 int ivarNum = 0;
 Ivar *ivars = class_copyIvarList([person class], &ivarNum);
 for(int i = 0; i < ivarNum; i++){
     Ivar ivar = ivars[i];
     printf("%s ",ivar_getName(ivar));
 }
 printf("\n");

//输出为:
//Hello World!
//Johnson
//Lu
//nameBeforeRegister 
//
{% endhighlight %}

可见当用runtime API动态创建类时，当创建完毕后，添加的iVar就不加入到`objc_ivar_list`.

### 2.3. Runtime API介绍  ###

以上介绍了Associated Objects的实现，我们现在来看看其runtime API,包括以下3个方法：

1. `objc_setAssociatedObject` 用于给对象添加关联对象，传入 nil 则可以移除已有的关联对象；
2. `objc_getAssociatedObject` 用于获取关联对象；
3. `objc_removeAssociatedObjects` 用于移除一个对象的所有关联对象。



关于前两个函数中的 key 值是我们需要重点关注的一个点，一般来说，有以下三种推荐的 key 值：

1. 声明 static char kAssociatedObjectKey; ，使用 &kAssociatedObjectKey 作为 key 值;
2. 声明 static void *kAssociatedObjectKey = &kAssociatedObjectKey; ，使用 kAssociatedObjectKey 作为 key 值；
3. 用 selector ，使用 getter 方法的名称作为 key 值。

由于第3种方式最为优雅和简洁，在上面的例子中我们用的就是selector。


## 3 Non-Fragile iVars ##

> **Fragile Base Class Problem**：在某些编程语言中，父类必须得通过重新编译所有子类才可以改变ivar Layout。例如，C++在父类里增加data members或virtual member functions会打破子类的**二进制兼容**，即使增加的members是私有的和对子类不可见的。

### 3.1 为什么Non Fragile ivars很关键 ###


Runtime有两个版本，**Legacy**和**Modern**。当编译类的时候，ivar layout已经确定下来，比如object的ivar的offset和size等。所以你的ivar layout很可能看起来是下图中的**base class**。当我们子类化``NSView``为``PetShopView``的时候，增加了``kittens``和``puppies``两个ivar，我们期待的ivar layout是下图中的**expected class**。

在**Legacy**版本，当Mac OS X Def Leopard出来后，AppKit开发者在NSView里增加了`touchedPaws` iVar。这个时候，原来的`PetShopView`里的`kittens`就会被新加的`touchedPaws`所覆盖。这种Non Fragile ivars的特性必须通过重新编译`PetShopView`才能运行。如果更不幸一点，`PetShowView`类是来自第三方提供的静待库，我们就只能干等库作者跟新版本了。

在**Modern**版本。程序启动后，runtime加载MyObject列的收，通过计算基类的大小，runtime动态调整了`PetShowView` ivar的layout，即向后移动了8个字节。于是我们的程序无序编译，就能运行。

{: .img_middle_lg}
![SendingMessage](/assets/images/posts/01 Objectiev C/2016-03-18-OC Runtime(三)_Category_Part II_Associated Objects/Non-Fragile iVars.png)

### 3.2 如何寻址类成员变量 ###

主要通过`class_ro_t`里的`instanceStart`和`instanceSize`来判断，
当子类的instanceStart小于父类的instanceSize时，需要调整。

{% highlight objc linenos %}

struct class_ro_t {
    ...
    uint32_t instanceStart;
    uint32_t instanceSize;
    ...
}

{% endhighlight %}

### 3.3 为什么Objective-C类不能动态添加成员变量而可以添加方法属性和协议 ###

> 为什么可以通过分类动态添加方法，属性，协议而不能添加添加成员变量呢。简单的回答是方法，属性，协议属于`Procedure`，存储在类和元类(都是单例)里，内存中只有一份；而实例变量属于`Data`，存储在同样属于`Data`的对象(可以有多个)内存里。若给类增加``ivar_t``，来动态修改类成员变量布局，那么已经创建出的对象就不符合类定义了，变成了无效对象。

这也解释了为什么[2.2 在Runtime API中动态增加属性](#runtime-api)，在通过`objc_registerClassPair`注册了类后，后面用`class_addIvar`方法增加的实例变量是无效的。

## 4 总结 ##
Associated Objects用于扩展类的属性，使得外部在用dot notation存取属性时分不出两者的区别，这解决了Objective C属性扩展的问题。但是在内部实现中，Associated Objects和类本身的iVar和Property还是有区别的：

1. 类的定义结束后(无论是Objective-C还是runtime API动态创建类)`objc_ivar_list`是不能变得。
2. 关联类和被关联类在内存中是分开存储的。被关联类的`objc_ivar_list`只存储其本身的iVar和Property。
3. 这同样体现在用runtime动态创建类中，`class_addIvar(...)`在`objc_registerClassPair`前和后调用时的情况。当`class_addIvar(...)`在`objc_registerClassPair`前调用时，加入的iVar是在类本身的定义中，存储在其`objc_ivar_list`；在`objc_registerClassPair`后调用，`class_addIvar(...)`不可以再动态增加类属性(因为如果可以动态增加，之前创建的类就无效了，data可以有多份，但是procedure只能有一份，ivar的定义在类的单例里，可以看做是procedure，这也解释了为什么不能在分类里添加ivar而只能添加方法或属性(方法的一种)。

全文总结参考[该图]({{site.baseurl}}/01%20objective-c/2016/03/12/OC-Runtime(零)_Runtime概述.html#runtime-1)。

