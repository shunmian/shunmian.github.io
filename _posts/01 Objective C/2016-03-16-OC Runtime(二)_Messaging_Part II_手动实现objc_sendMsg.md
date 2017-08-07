---
layout: post
title: OC Runtime(二)：Messaging Part II：手动实现objc_sendMsg
categories: [01 Objective-C]
tags: [Messaging,isa swizziling,method swizzling]
number: [0.14.1.2]
fullview: false
shortinfo: Objective-C中的消息发送和C中的函数调用有着本质的区别。后者在编译阶段已经确定了函数的具体实现, 而前者在运行时还可以更改消息发送的具体实现，这给Objective-C注入了崭新的动态活力。而这都得益于Objective-C的Runtime系统。本文在介绍Runtime Messaging机制后，实现两个黑魔法，isa swizziling和method swizzling。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. Messaging机制 ##

首先回顾一下Objective-C 消息发送过程:

{: .img_middle_lg}

![SendingMessage](/assets/images/posts/01 Objectiev C/2016-03-15-OC Runtime(二)_Messaging/sending message.png)

简单来看下这张图，当`[foo doSomething:@"param1" and @"param2"]`执行时，会转换成`objc_msgSend(id foo, SEL @selector(doSomething:and:),@"param1",@"param2")`, 而这个函数的执行过程就是这张图的流程:

{: .nestedList}
- 1 调用 `IMP lookUpImpOrNil(Class cls, SEL name)`
  - 1.1 查找`foo`的`isa`，通过`@selector(doSomething:and:)`查找`cache`，若找不到则下一步;
  - 1.2 查找本类vtable，找不到则通过`superClass`沿着继承链找，最终找到函数的实现, 类型为IMP的函数指针,若找不到则转到2;

- 2 调用`_objc_fowardMsg(Classcls, SEL name)`进行消息转发，
  + 2.1 `+ (BOOL) resolveInstanceMethod:(SEL)sel`, 如找不到则2.2;
  + 2.2 `-(id)forwardingTargetForSelector:(SEL)sel`, 如找不到则2.3;
  + 2.3 `-(void)forwardInvocation:(NSInvocation *)inv`, 如找不到则2.4;
  + 2.4 `-(void)doesNotRecognizeSelector:(SEL)sel`。

关于手动实现``objc_sendMsg``[这里]()会另开一篇文章。

这个过程中，swizzling可以发生在:

- 1.1 调换isa(isa Swizzling)来达到查找IMP的目的(如Objective-C对KVO的实现,原类没有改写setter，如何实现KVO，关键在于isa swizzling创建了一个中间类，重写了setter;
- 1.2 调换@selector和IMP在vtable上的对应关系(Method Swizzling)来动态实现selector和IMP的绑定。



## 2. Method Swizzling

### 2.1 API

介绍了消息发送之后，我们来具体看看Method Swizzling。 既然是Method Swizziling，那什么是`Method`呢，在`/usr/include/objc`中的我们可以找到这样的定义:

{% highlight objc linenos %}
typedef objc_method *Method
struct objc_method {
  SEL method_name       OBJC2_UNAVAILABLE; //method 的 selector，SEL
  char *method_types    OBJC2_UNAVAILABLE; //method 的 输入输出参数类型, signature
  IMP method_imp        OBJC2_UNAVAILABLE; //method 的 实现，IMP
} 
{% endhighlight %}

一个Method是一个指向objc_method结构体的指针，这个结构体包含三个变量，分别是SEL，signature和IMP。一个vtalbe的entry既是Method, 第一列是SEL,第二列是IMP。一个SEL对应一个IMP，如下图。

{: .img_middle_lg}
![SendingMessage](/assets/images/posts/01 Objectiev C/2016-03-15-OC Runtime(二)_Messaging/vtable.png)

因此，如果你可以更改SEL和IMP的动态对应关系，则可以做出一些奇妙的事情。

我们来建一个Swzizzle类，代码如下:
{% highlight objc linenos %}

Swizzle.h

# import <Foundation/Foundation.h>
# import <objc/runtime.h>
@interface Swizzle : NSObject
+(void)swizzleClass:(id)objecClass fromSelector:(SEL)sel1 toSelector:(SEL)sel2;
@end


Swizzle.m

@implementation Swizzle


+(void)swizzleClass:(id)objecClass fromSelector:(SEL)sel1 toSelector:(SEL)sel2{
  
  Method originalMethod = class_getInstanceMethod(objecClass, sel1);
  Method destinationMethod = class_getInstanceMethod(objecClass, sel2);
  
  IMP originIMP = method_getImplementation(originalMethod);
  IMP destinationIMP = method_getImplementation(destinationMethod);
  
  BOOL isAdded = class_addMethod(objecClass, sel1, destinationIMP, method_getTypeEncoding(destinationMethod));
  
  if(isAdded){
    class_replaceMethod(objecClass, sel2, originIMP, method_getTypeEncoding(originalMethod));
  }else{
    method_exchangeImplementations(originalMethod, destinationMethod);
  }
}
@end
{% endhighlight %}

Swizzle类只有一个类方法`+(void)swizzleClass:(id)objecClass fromSelector:(SEL)sel1 toSelector:(SEL)sel2`,返回void, 输入第一参数是要执行method swizzle的类名，第二个和第三个参数是要交换的selector。

首先我们获取两个selector对应的method(`class_getInstanceMethod)`, 然后通过method获取其IMP(`method_getImplementation`)。
然后我们在原本的类里尝试添加新的方法，该方法的selector是sel1，IMP是destinationIMP，返回一个bool值表示是否添加成功。如果添加成功，则将原来的method2替换掉(`class_replaceMethod`), 令其sel2指向IMP2。若果失败，则表示类里面已经存在method1，那么就简单将method1和method2的IMP交换既可。这样就防止了一种情况：该类继承于父类，却没有重写父类的方法，如果简单用`method_exchangeImplementations(originalMethod, destinationMethod)`,则会将父类的originalMethod的IMP和该类的destinationMethod的IMP交换，这不是我们想要的。


### 2.2 应用1: NSString lowerCase upperCase方法的交换

好了我们来看看如何用这个Swizzle,

{% highlight objc linenos %}
NSString *originalString = @"hello,WORLD!";
NSLog(@"original string  : %@",originalString);
NSLog(@"before Method Swizzle---------");
NSLog(@"lower case String: %@", [originalString lowercaseString]);
NSLog(@"upper case string: %@", [originalString uppercaseString]);
NSLog(@"after  Method Swizzle---------");
[Swizzle swizzleClass:[originalString class]
       fromSelector:@selector(lowercaseString)
         toSelector:@selector(uppercaseString)];
NSLog(@"lower case String: %@", [originalString lowercaseString]);
NSLog(@"upper case string: %@", [originalString uppercaseString]);

//输出：
//original string  : hello,WORLD!
//before Method Swizzle---------
//lower case String: hello,world!
//upper case string: HELLO,WORLD!
//after  Method Swizzle---------
//lower case String: HELLO,WORLD!
//upper case string: hello,world!

{% endhighlight %}

可以看到在Method Swizzle后，同样是`lowercaseString`和`uppercaseString`的结果交换了，有没有一个感觉自己是个黑客了呢。

### 2.3 应用2: 打印当前UIViewController的名字

下面这个场景对于应用Method Swizzle来说再合适不过了

> Method Swizzle 应用场景: 给你一个Xcode project修改代码。在修改前你想要了解出现在手机上的当前UIView对应得UIViewController到底是什么类, 比如在`ViewDidAppear`加入`NSLog(@"%@: [self class]")`让其自行打印。由于custmozied的各种视图控制器继承自如UITableViewController, UITabViewController, UINavigationViewController,而它们都继承自UIViewController。因此最笨的方法是你需要重写customized每一个视图控制器中的`ViewDidAppear`，聪明一点就是重写其父类(那也有好多个)。再聪明一点是重写UIViewController。问题是UIViewController我们没有其源码！这个时候我们就可以用Method Swizzle.

代码如下。

{% highlight objc linenos %}
UIViewController + Swizzle.h

# import <UIKit/UIKit.h>
# import "Swizzle.h"

@interface UIViewController (Swizzle)
-(void)swizzle_viewDidAppear:(BOOL)animated;
@end


UIViewController + Swizzle.m

@implementation UIViewController (Swizzle)

+(void)load{
  [Swizzle swizzleClass:[self class] fromSelector:@selector(viewDidAppear:) toSelector:@selector(swizzle_viewDidAppear:)];
}

-(void)swizzle_viewDidAppear:(BOOL)animated{
  [self swizzle_viewDidAppear:animated];
  NSLog(@"the current appeared ViewController: %@",[self class]);
}


@end
{% endhighlight %}

我们在声明了一个`-(void)swizzle_viewDidAppear:(BOOL)animated;`方法，在它的实现里调用了`[self swizzle_viewDidAppear:animated]`，然后加入一句打印自身类的语句。在`+(void)load`里我们Method Swizlle两者的实现。
首先category的`+(void)load`保证其在尽可能很早的时候已经执行Method Swizzle（`+(id)initialize`不行，因为其只会在第一次调用方法时才执行，因此有可能太迟，有可能永不执行。）然后当你的各个视图控制器的`-(void)viewDidAppear:(BOOL)animated`执行时，其IMP是line20-21。而line20调用`[self swizzle_viewDidAppear:animated]`时，其IMP又是UIViewController`-(void)viewDidAppear:(BOOL)animated`的IMP。不会有循环。

当你在各种视图控制器中切换时，他们的类就自行打印出来了，Cool！

{% highlight objc linenos %}
//输出：
//the current appeared ViewController: UITabBarController
//the current appeared ViewController: UINavigationController
//the current appeared ViewController: PBBibleBookViewController
//the current appeared ViewController: UIInputWindowController
//the current appeared ViewController: PBBibleVolumeViewController
//the current appeared ViewController: PBBibleChapterViewController
//
{% endhighlight %}

### 2.4 注意点

在用Method Swizzling的时候，有一些坑需要注意：

1. 要尽可能早的执行swizzle，因此在`+(void)load`而不是在`+(id)initialize`里;

2. 用GCD的`dispatch_once`保证其线程安全且执行一次;

3. 要注意方法命名冲突，用swizzle_前缀比较合适，表示Method Swizzle的方法。如果这会让你记不起原来的方法名，则可以将swizzle作为后缀,即`-(void)viewDidAppear_Swizzle:(BOOL)animated`。



### 2.5 小结

Method Swizzle在sending message的过程中寻找IMP这一步动态注入代码，这使得当你没有源码时依然可以重写其方法。这比子类化重写和category重写要有更多优势。比如你没有实例化控制权，子类化重写则无济于事；category重写会覆盖掉原方法，而通常我们需要的是扩展功能同时保留原有功能。Method Swizzle提供了另一种优雅的解决方案，其中的细节需要读者细细品味。

## 3 ISA Swizzling (OC KVO的实现) ##

### 3.1 Observing Pattern 介绍 ##

Obeserving Pattern 是Gang Of Four里面提到的24种面向对象设计模式之一。使得subject(被观察者)的iVar改变的时候，observer(观察者)能够得到提醒。这里有一个tricky的地方是beginner刚开始思考这个问题的时候，observer如果一直主动观察着subject iVar的变化，那observer就干不了其他事情了，或者另开一个线程。而问题的解决方法其实是observer被动接受subject发出的iVar的变化， 即subject拥有observer的引用，在iVar的setter里提醒observer。

{: .img_middle_lg}
![SendingMessage](/assets/images/posts/01 Objectiev C/2016-03-15-OC Runtime(二)_Messaging/Observer Pattern.png)

上图是Observing Pattern的UML。Subject是一个接口，声明了三个方法：

1. addObserver(Observer), 注册observer(即将observer加入到observers数组)；
2. notify(), 通知observers(observers数组里的每个observer都调用其update()方法)；
3. removeObserver(Observer), 注销observer(即observers数组里删除observer)。

Observer也是一个接口，声明了一个方法：update(),用来Subject iVar改变时响应。 

ConcreteSubjectA实现了Subject接口，在iVar的setter里加了notify(), 因此在每一次iVar值改变时，都会提醒observers里的每一个observer相应update()。ConcreteObserver实现了Observer接口。这个UML理解起来比较清晰。

### 3.2 三种实现 ###

我们知道Objective C对 Observer Pattern的实现是用Key Value Observeing。我们下面用三种方法在Objective C中实现Observer Pattern：普通实现(即按照上面UML图来实现)， isa Swizzling实现KVO， method Swizzling实现KVO。

#### 3.2.1 普通实现 ####

普通实现比较简单，代码如下

{% highlight objc linenos %}

@implementation ConcreteSubjectA

-(NSMutableArray *)Observers{
    if(!_Observers){
        _Observers = [NSMutableArray array];
    }
    return _Observers;
}

-(void)LAL_addObserver:(id<LAL_Observer>)observer WithBlock:(LALObservingBlock)block{
    [observer setObservingBlock:block];
    [self.Observers addObject:observer];

}

-(void)LAL_notify{
    for (id<LAL_Observer> observer in self.Observers) {
        [observer LAL_update];
    }
}

-(void)LAL_removeObserver:(id<LAL_Observer>)observer{
    [self.Observers removeObject:observer];
}

-(void)setAge:(NSNumber *)age{
    _age = age;
    [self LAL_notify];//iVar age 改变后通知Observers.
}
@end

{% endhighlight %}

在这里，重写age的setter，加入通知Observers是关键。

#### 3.2.2 KVO实现: isa Swizzling ###

对于熟悉Key Value Observing API的同学来说，“订阅”-“响应”-“取消订阅” 也对应着上述`addObserver(Observer)`,`notify()`和`removeObserver(Observer)`这三个步骤，具体请参考[Key Value Observing]({{site.baseurl}}/objective-c/2016/02/18/Key-Value-Observing.html){:target="_blank"}。


1. 订阅：`- (void)addObserver:forKeyPath:options:context:`


2. 响应：`- (void)observeValueForKeyPath: ofObject:change:context:`

3. 取消订阅：`- (void)removeObserver:forKeyPath:`


令人疑惑的是KVO缺少改写Subject iVar setter这一步，而这一步正是Observing Pattern的关键。Apple是如何做到这一步的呢，我们来看下[Introduction to Key-Value Observing Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOImplementation.html#//apple_ref/doc/uid/20002307-BAJEAIEE){:target="_blank"}对于KVO实现原理的介绍：

><b>Key-Value Observing Implementation Details</b>: Automatic key-value observing is implemented using a technique called isa-swizzling.The isa pointer, as the name suggests, points to the object's class which maintains a dispatch table. This dispatch table essentially contains pointers to the methods the class implements, among other data.When an observer is registered for an attribute of an object the isa pointer of the observed object is modified, pointing to an intermediate class rather than at the true class. As a result the value of the isa pointer does not necessarily reflect the actual class of the instance.You should never rely on the isa pointer to determine class membership. Instead, you should use the class method to determine the class of an object instance.

Apple用了一种isa Swizzling的方法，创建了一个中间类，使得原来的类变成中间类的父类，原来的实例变成中间类的实例;并改写了其class方法，返回依然是原来的类用来欺骗开发者。这样的做法有两个好处：

1. 开发者专注于“订阅”-“响应”-“取消订阅”API,而无需关注重写iVar setter， 重写由KVO自己完成；
2. isa Swizzling创建了中间类，因此原来的类的实现细节不会由于KVO而改变, KVO的改变体现在中间类里。这体现了封装和扩展的思想。

下面我们就来自行实现以下KVO用isa Swizzling的方法。我们给NSObject创建一个KVO匿名类，里面有两个方法，订阅和取消订阅,并且声明了一个LALKVOBlock类型，用来响应(官方KVO没有支持block或者selector,所有响应逻辑都在`-(void)observeValueForKeyPath:ofObject:change:context:`里执行，这也是官方KVO广为诟病的一个原因)：

{% highlight objc linenos %}
#import <Foundation/Foundation.h>
typedef void (^LALKVOBlock)(id obsever, id subject, NSString *keyPath, id oldValue, id newValue);

@interface NSObject (LALKVO)

-(void)LALKVO_addObserver:(id)observer ForKeyPath:(NSString *)keyPath withBlock:(LALKVOBlock)block;

-(void)LALKVO_removeObserver:(id)observer ForKeyPath:(NSString *)keyPath;

@end
{% endhighlight %}


`-(void)LALKVO_addObserver:ForKeyPath:withBlock:`主要逻辑如下:

1. keyPath是否有效;
2. 若有效则看中间类是否存在,若不存在则创建(实例isa指向中间类，中间类的super_class指向原类，并改写实例的class方法，使其指向原类)；
3. keyPath对应的setter中间类是否已经重写，若没有则重写；
4. 增加observer到observers数组。

{% highlight objc linenos %}
-(void)LALKVO_addObserver:(id)observer ForKeyPath:(NSString *)keyPath withBlock:(__autoreleasing LALKVOBlock)block{
    //assume the original class is Person
    Class originalClass = [self class];
    
    //step1: keyPath is valid or not.
    SEL keyPathSetter = NSSelectorFromString(setterForGetter(keyPath));
    Method keyPathSetterMethod = class_getInstanceMethod(originalClass, keyPathSetter);
    if(!keyPathSetterMethod){
        NSLog(@"the keyPath is not valid!");
        return;
    }
    
    //step2: intermediate Class exist or not, if not create it LALKVO_Person as the intermediate Class.
    const char *originalClassName = class_getName(originalClass);
    const char *intermediateClassName = getIntermediateClassName(originalClassName);
    if(!objc_getClass(intermediateClassName)){
        createIntermediateClass(self, originalClassName, intermediateClassName);
    }
    
    //step3: override the setter in LALKVO_Person.
        //LALKVO_Person 是否有setter
    Class intermediateClass = object_getClass(self);

    if(!hasSelector(self, keyPathSetter)){
        //没有则创建.
        Method originalKeyPathSetterMethod = class_getInstanceMethod(originalClass, keyPathSetter);
        const char* types = method_getTypeEncoding(originalKeyPathSetterMethod);
        class_addMethod(intermediateClass, keyPathSetter, (IMP)kvo_setter, types);
    }
    //setp4: add the observer to the subject.
    NSMutableArray *observeInfoArray = objc_getAssociatedObject(self, kLALKVOObserverInfoArray);
    if(!observeInfoArray){
        observeInfoArray = [[NSMutableArray alloc] init];
        objc_setAssociatedObject(self, kLALKVOObserverInfoArray, observeInfoArray, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    ObserveInfo *observeInfo = [[ObserveInfo alloc] initWithSubject:self keyPath:keyPath observer:observer block:block];
    if(!observeInfoArrayContainsObserveInfo(observeInfoArray, observeInfo)){
        [observeInfoArray addObject:observeInfo];
    }
}
{% endhighlight %}

其中最关键的是第三步，获取原类的setter IMP,执行它，代码如下:
{% highlight objc linenos %}
void kvo_setter(id obj, SEL _cmd, id newValue){
    
    NSString *setterName = NSStringFromSelector(_cmd);
    NSString *getterName = getterForSetter(setterName);
    
    //get the originalClass getter IMP and implement it.
    Method getterMethodFromOriginalClass = class_getInstanceMethod(class_getSuperclass(object_getClass(obj)), NSSelectorFromString(getterName));
    IMP getterIMPFromOriginalClass = method_getImplementation(getterMethodFromOriginalClass);
    id (*getterIMPFromOriginalClassCasted)(id,SEL) = (void *)getterIMPFromOriginalClass;
    id oldValue = getterIMPFromOriginalClassCasted(obj,NSSelectorFromString(getterName));
    
    
    //get the originalClass setter IMP, and implement it.
    Method setterMethodFromOriginalClass = class_getInstanceMethod(class_getSuperclass(object_getClass(obj)), _cmd);
    IMP setterIMPFromOriginalClass = method_getImplementation(setterMethodFromOriginalClass);
    void (*setterIMPFromOriginalClassCasted)(id, SEL, id) = (void *)setterIMPFromOriginalClass;
    setterIMPFromOriginalClassCasted(obj, _cmd, newValue);
    
    //call the block for each observer.
    NSMutableArray *observerInfoArray = objc_getAssociatedObject(obj, kLALKVOObserverInfoArray);
    for (ObserveInfo *observeInfo in observerInfoArray){
        if([observeInfo.keyPath isEqualToString:getterName]){
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
                observeInfo.block(observeInfo.observer, observeInfo.subject, observeInfo.keyPath, oldValue, newValue);
            });
        }
    }
}
{% endhighlight %}

源码在这里。

#### 3.2.3 KVO实现: Method Swizzling ###

不知道同学们注意到没有，其实isa Swizzling实现KVO的本质是创建中间类，然后在中间类的setter里用Method Swizzling重写。Method Swizzling的介绍请见传送门[OC Runtime(三)： Method Swizziling]({{site.baseurl}}/objective-c/2016/03/16/OC-Runtime(三)_method-swizzling.html){:target="_blank"}。创建中间类的好处是使得原类的实现不被KVO改变。实际上我们完全可以不用isa Swizlling而只用Method Swizzling来实现KVO, 代码如下:


{% highlight objc linenos %}
#import <Foundation/Foundation.h>
typedef void (^LALKVOBlock)(id obsever, id subject, NSString *keyPath, id oldValue, id newValue);
static NSNumber *LALKVO_overrideKeyPathSetterFlag;
static void (*originalSetterIMP)(id, SEL,id);

@interface NSObject (LALKVO)

-(void)LALKVO_addObserver:(id)observer ForKeyPath:(NSString *)keyPath withBlock:(LALKVOBlock)block;
-(void)LALKVO_removeObserver:(id)observer ForKeyPath:(NSString *)keyPath;

+(NSNumber *)overrideFlag;
+(void)setOverrideFlag:(NSNumber *)flag;
@end

{% endhighlight %}

我们增加了一个静态类变量来标志setter是否被重写，并且实现了该类变量的setter和getter。同时我们设置了一个静态函数指针用来存储原来的setter的实现。

{% highlight objc linenos %}
-(void)LALKVO_addObserver:(id)observer ForKeyPath:(NSString *)keyPath withBlock:(__autoreleasing LALKVOBlock)block{
    
    Class originalClass = object_getClass(self);
    //step1: keyPath is valid or not
    SEL keyPathSetter = NSSelectorFromString(setterForGetter(keyPath));
    Method keyPathSetterMethod = class_getInstanceMethod(originalClass, keyPathSetter);
    if(!keyPathSetterMethod){
        NSLog(@"the keyPath is not valid!");
        return;
    }
    //step2: override the setter class, 查看overrideFlag
        //self 是否已经改写setter
    if(![[object_getClass(self) overrideFlag] boolValue]){
        //获取original Setter Method and IMP
        Method originalKeyPathSetterMethod = class_getInstanceMethod(originalClass, keyPathSetter);
        originalSetterIMP =(void *) method_getImplementation(originalKeyPathSetterMethod);

        method_setImplementation(originalKeyPathSetterMethod, (IMP)kvo_setter);
    }
    
    // setp3: add the observer to the subject.
    NSMutableArray *observeInfoArray = objc_getAssociatedObject(self, kLALKVOObserverInfoArray);
    if(!observeInfoArray){
        observeInfoArray = [[NSMutableArray alloc] init];
        objc_setAssociatedObject(self, kLALKVOObserverInfoArray, observeInfoArray, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    ObserveInfo *observeInfo = [[ObserveInfo alloc] initWithSubject:self keyPath:keyPath observer:observer block:block];
    if(!observeInfoArrayContainsObserveInfo(observeInfoArray, observeInfo)){
        [observeInfoArray addObject:observeInfo];
    }
}
{% endhighlight %}

kvo_setter的code如下:

{% highlight objc linenos %}
void kvo_setter(id obj, SEL _cmd, id newValue){
    
    NSString *setterName = NSStringFromSelector(_cmd);
    NSString *getterName = getterForSetter(setterName);
    
    //get the class getter IMP, and implement it.
    Method getterMethodFromOriginalClass = class_getInstanceMethod(object_getClass(obj), NSSelectorFromString(getterName));
    IMP getterIMPFromOriginalClass = method_getImplementation(getterMethodFromOriginalClass);
    id (*getterIMPFromOriginalClassCasted)(id,SEL) = (void *)getterIMPFromOriginalClass;
    id oldValue = getterIMPFromOriginalClassCasted(obj,NSSelectorFromString(getterName));
    
    //get the class setter IMP, and implement it.
    originalSetterIMP(obj, _cmd, newValue);
    
    NSMutableArray *observerInfoArray = objc_getAssociatedObject(obj, kLALKVOObserverInfoArray);
    for (ObserveInfo *observeInfo in observerInfoArray){
        if([observeInfo.keyPath isEqualToString:getterName]){
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
                observeInfo.block(observeInfo.observer, observeInfo.subject, observeInfo.keyPath, oldValue, newValue);
            });
        }
    }
    LALKVO_overrideKeyPathSetterFlag = [NSNumber numberWithBool:YES];
}

{% endhighlight %}
源码在这里。


### 3.3 小结 ##

本文用三种方法在Objective-C里实现了Observing Pattern, 包括普通实现，KVO isa Swizlling实现和KVO Method Swizzling实现。后两种需要读者对Objective-C的runtime有一定的熟悉。通过自己实现KVO的主要功能，相信定会让你更能体会代码的乐趣和增加自己coding的信心。

## 4 总结 ##

