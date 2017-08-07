---
layout: post
title: OC Runtime(二)：Messaging Part IV：KVC
categories: [01 Objective-C]
tags: [KVO]
number: [0.14.9.1]
fullview: false
shortinfo: Key Value Observing 是Objective-C对Observer Pattern的内置实现。它的功能非常强大, 同时API设计糟糕,相应的坑也不少。本文对Key Value Observing进行介绍。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. Key Value Observing 介绍 ##

Observer Pattern 是"GOF"提到的24种面向对象设计模式中的一种， 它可以让subject(被观察者)的iVar改变时通知observer（观察者）。Key Value Observing 是Cocoa对于Observing Pattern的具体实现。 这在Model-View-Controller模式里提供了一种在Controller里，Model和View各自变化实时更新彼此的实现。
KVO实现的原理是subject拥有观察者们的引用(array)，在iVar的setter的更改值前后加入通知观察者。令人疑惑的是Cocoa的KVO的API在没有重写subject的setter的同时又实现了通知观察者，这里用到了isa swizzling的方法，我们暂时不讲。本文着重介绍Key Value Observing的API应用, 包括自动观察，手动观察，依赖值观察，以及KVO的各种坑及应对方法。

## 2. Key Value Observing 三种应用  ##

Key Value Observing 的应用逻辑遵循 ”订阅“-”响应“-”取消订阅“的基本方式。

### 2.1. 自动观察  ###

先来看一段代码。

{% highlight objc linenos %}
Bank.h
@class Person;
@interface Bank : NSObject
@property (nonatomic, strong) NSNumber* accountDomestic;
@property (nonatomic, strong) Person* person;
-(instancetype)initWithPerson:(Person *)person;
@end

Bank.m
@implementation Bank
-(instancetype)initWithPerson:(Person *)person{
    if(self = [super init]){
        _person = person;
        [self addObserver:_person forKeyPath:NSStringFromSelector(@selector(accountDomestic)) options:NSKeyValueObservingOptionOld|NSKeyValueObservingOptionNew context:bankContext];
    }
    return self;
}
-(void)dealloc{
    [self removeObserver:_person forKeyPath:NSStringFromSelector(@selector(accountDomestic))];
}
@end


Person.h
@interface Person : NSObject
@end

Person.m
@implementation Person
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
    if([object isKindOfClass:[Bank class]]){
        if([keyPath isEqualToString:NSStringFromSelector(@selector(accountDomestic))]){
            NSString *oldValue = change[NSKeyValueChangeOldKey];
            NSString *newValue = change[NSKeyValueChangeNewKey];
            NSLog(@"--------------------------");
            NSLog(@"accountDomestic old value: %@", oldValue);
            NSLog(@"accountDomestic new value: %@", newValue);
        }
    }
}
@end


ViewController.m
@interface ViewController ()
@property (nonatomic, strong) Bank *bank;
@property (nonatomic, strong) Person *person;
@property (nonatomic, strong) NSNumber *delta;
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    self.person = [[Person alloc] init];
    self.delta = @10;
    self.bank = [[Bank alloc] initWithPerson:self.person];
}
- (IBAction)accountBalanceIncreaseButtonDidTouch:(id)sender {
    self.bank.accountDomestic = self.delta;
    int temp = [self.delta intValue];
    temp += 10;
    self.delta = [NSNumber numberWithInt:temp]; 
}
//输出:按两次button
//--------------------------
//accountDomestic old value: <null>
//accountDomestic new value: 10
//--------------------------
//accountDomestic old value: 10
//accountDomestic new value: 20
//
{% endhighlight %}
我们先来看"订阅"。这里`Bank`类有一个iVar是`accountDomestic`,还有一个客户`Person`,在`Bank`初始化时, `Person`订阅了`accountDomestic`的KVO服务, 被加入到观察者中。

{% highlight objc linenos %}
- (void)addObserver:(NSObject *)observer 
         forKeyPath:(NSString *)keyPath 
            options:(NSKeyValueObservingOptions)options 
            context:(nullable void *)context;
{% endhighlight %}
这里`observer`是`person`,观察者;`keyPath`是iVar的名字, 应该是`@"accountDomestic"`,由于手动输入只能在运行时才能确认对错，用`NSStringFromSelecotr(@selector(accountDomestic))`替代更为合适; `option`是一个`NSKeyValueObservingOptions`的enum，可以有四个值，一般用的是旧值`NSKeyValueObservingOptionOld`和新值`NSKeyValueObservingOptionNew`; `context`通常是null也可以用作唯一的ID来区别这个类的观察和父类的观察，这个我们后面讲。

下面我们看”响应“。Person类实现了以下方法。

{% highlight objc linenos %}
- (void)observeValueForKeyPath:(NSString *)keyPath 
                      ofObject:(id)object 
                        change:(NSDictionary<NSString*, id> *)change 
                        context:(void *)context;
{% endhighlight %}
`keyPath`是`Person`类添加的键，本例中是至少有一个为`NSStringFromSelecotr(@selector(accountDomestic))`；`object`是subject(被观察者),这里是`Bank`实例。`change`为一个`NSDictionary`，当订阅时的`option`是用的是旧值`NSKeyValueObservingOptionOld`和新值`NSKeyValueObservingOptionNew`时，可以分别通过`change[NSKeyValueChangeOldKey]`和`change[NSKeyValueChangeNewKey]`来访问;`context`和订阅时的`context`一样。

本例中我们首先判断响应是否来源于Bank,再判断是否是`NSStringFromSelecotr(@selector(accountDomestic))`,然后再执行相关业务。由于KVO的响应不能像`UIControl`的实例方法`-addTarget:action:forControlEvents:`一样可以添加selector, 因此所有的响应都在`-observeValueForKeyPath:ofObject:change:context:`中执行，这是KVO广为诟病的一大缺点之一。

最后我们来看取消订阅:
{% highlight objc linenos %}
- (void)removeObserver:(NSObject *)observer 
            forKeyPath:(NSString *)keyPath;
{% endhighlight %}
当subject`dealloc`时，我们取消订阅。

### 2.2. 手动观察  ###

有时候观察者需要对被观察的某个键进或者几个键的通知发送进行控制, 比如`Bank`还有一个属性`accountForeign`, `Person`同时也观察这个`key`。但是现在`Person`暂时想把`accountForeign`改变时的发送通知给停止掉，或者暂停掉后又想开启。见如下代码。

{% highlight objc linenos %}
Bank.m
@implementation Bank
-(instancetype)initWithPerson:(Person *)person{
    if(self = [super init]){
        _person = person;
        [self addObserver:_person forKeyPath:NSStringFromSelector(@selector(accountDomestic)) options:NSKeyValueObservingOptionOld|NSKeyValueObservingOptionNew context:bankContext];
        [self addObserver:_person forKeyPath:NSStringFromSelector(@selector(accountForeign)) options: NSKeyValueObservingOptionOld|NSKeyValueObservingOptionNew context:bankContext];
    }
    return self;
}

+(BOOL)automaticallyNotifiesObserversForKey:(NSString *)key{
//当key时NSStringFromSelector(@selector(accountForeign)）停止自动发送通知。
    if([key isEqualToString:NSStringFromSelector(@selector(accountForeign))]){
        return NO;
    }else{
        return YES;
    }
}

-(void)dealloc{
    [self removeObserver:_person forKeyPath:NSStringFromSelector(@selector(accountDomestic))];
    [self removeObserver:_person forKeyPath:NSStringFromSelector(@selector(accountForeign))];
}

Person.m
@implementation Person
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
    if([object isKindOfClass:[Bank class]]){
        if([keyPath isEqualToString:NSStringFromSelector(@selector(accountDomestic))]){
            NSString *oldValue = change[NSKeyValueChangeOldKey];
            NSString *newValue = change[NSKeyValueChangeNewKey];
            NSLog(@"--------------------------");
            NSLog(@"accountBalance old value: %@", oldValue);
            NSLog(@"accountBalance new value: %@", newValue);
        }else if([keyPath isEqualToString:NSStringFromSelector(@selector(accountForeign))]){
            NSString *oldValue = change[NSKeyValueChangeOldKey];
            NSString *newValue = change[NSKeyValueChangeNewKey];
            NSLog(@"--------------------------");
            NSLog(@"accountForeign old value: %@", oldValue);
            NSLog(@"accountForeign new value: %@", newValue);
        }
    }
}

ViewController.m
...
@implementation ViewController
...
- (IBAction)accountBalanceIncreaseButtonDidTouch:(id)sender {
    self.bank.accountDomestic = self.delta;
//同时更改accountForeign的值。
    self.bank.accountForeign = self.delta;
    int temp = [self.delta intValue];
    temp += 10;
    self.delta = [NSNumber numberWithInt:temp]; 
}
//输出:按两次button
//--------------------------
//accountDomestic old value: <null>
//accountDomestic new value: 10
//
@end
{% endhighlight %}
这里`+(BOOL)automaticallyNotifiesObserversForKey:(NSString *)key`对`accountForeign`返回`NO`,停止了它的值改变时自动发送通知，虽然它和`accountDomestic`一样实现了”订阅“-”响应“-”取消订阅“。

如果要开启的话，要么更改`+(BOOL)automaticallyNotifiesObserversForKey:(NSString *)key`对`accountForeign`返回`YES`; 要么在`Bank`中插入下列代码:

{% highlight objc linenos %}
-(void)setAccountForeign:(NSNumber *)accountForeign{
    [self willChangeValueForKey:NSStringFromSelector(@selector(accountForeign))];
    _accountForeign = accountForeign;
    [self didChangeValueForKey:NSStringFromSelector(@selector(accountForeign))];
}
{% endhighlight %}
`- (void)willChangeValueForKey:(NSString *)key`和`- (void)didChangeValueForKey:(NSString *)key`分别在setter里面值改变前后通知观察者。这时候在改变它的值得时候，就会收到通知继而触发响应了。

### 2.3. 依赖值观察  ###
再比如，`Bank`有一个`totalAmount`iVar，它是`accountDomestic`和`accountForeign`的和, 我们希望`accountDomestic`和`accountForeign`任何一个的改变都会使得totalBalance也有一样KVO的功能。这就是所谓的依赖值观察。这里`totalAmount`的"订阅"-"响应"-"取消订阅"的实现和`accountDomestic`或者`accountForeign`一样。唯一不同的是我们在`Bank`里要加入如下代码:

{% highlight objc linenos %}
//重写totalAmount的getter,体现其数值关系。
-(NSNumber *)totalAmount{
    return [NSNumber numberWithInt:([self.accountDomestic intValue] + [self.accountForeign intValue])];
}
//keyPathsForValuesAffectingTotalAmount里返回两个依赖的key作为NSSet。
+(NSSet *)keyPathsForValuesAffectingTotalAmount{
    return [NSSet setWithObjects:NSStringFromSelector(@selector(accountDomestic)),NSStringFromSelector(@selector(accountForeign)), nil];
}
{% endhighlight %}
这样任何一个`accountDomestic`或者`accountForeign`的改变，都会发送`totalAmount`改变的通知。

### 2.4. KeyPath注意事项  ###

结合ReactiveCocoa
{% highlight objc linenos %}
RACObserve(targe,keyPath);
{% endhighlight %}
self->percolates->N
若想观察self.percolates是否指向新的percolates对象，则``RACObserve(self,percolates)``;若想观察self.percolates.N是否指向新的N对象，则``RACObserve(self.percolates, N)``;

该情况在Algo Project中的[PercolateImplementationViewController.m](https://github.com/shunmian/iOS_Algo/blob/master/PercolateImplementationViewController.m)遇到过。

## 3. 坑及应对 ##
KVO有一些坑需要注意, 否则有时候会产生莫名其妙的bug;

1. `keyPath`最好不要用动态输入，如N`SStringFromSelector(@selector(accountDomestic))`代替`@"accountDomestic"`；

2. `context`用来区分本类和super类的通知，例如`Person`继承自`Client`(`Client`还有子类是`Company`),`Client`类里已经实现了对Bank某个属性的观察，这个时候在`Person`里实现响应的时候，应该做如下处理:
{% highlight objc linenos %}
Person.m
//定义了一个静态常数指针，值为自身的地址，用来表示一个KVO的observer的token
static void *const kPersonConext = &kPersonConext;
@implementation Person
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
    //首先判断是不是自身的KVO响应，
    if(context == kPersonConext){
        if([object isKindOfClass:[Bank class]]){
            if([keyPath isEqualToString:NSStringFromSelector(@selector(accountDomestic))]){
                NSString *oldValue = change[NSKeyValueChangeOldKey];
                NSString *newValue = change[NSKeyValueChangeNewKey];
                NSLog(@"--------------------------");
                NSLog(@"accountBalance old value: %@", oldValue);
                NSLog(@"accountBalance new value: %@", newValue);
            }else if([keyPath isEqualToString:NSStringFromSelector(@selector(accountForeign))]){
                NSString *oldValue = change[NSKeyValueChangeOldKey];
                NSString *newValue = change[NSKeyValueChangeNewKey];
                NSLog(@"--------------------------");
                NSLog(@"accountForeign old value: %@", oldValue);
                NSLog(@"accountForeign new value: %@", newValue);
            }
        }
    }else{//不是则调用父类。
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}
{% endhighlight %}
`static void *const kPersonConext = &kPersonConext`用来作为本类observer的token，区分父类。

3. 在"响应"里`-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context`，由于没有selector参数，因此所有的响应都要写在这个函数的实现里，逻辑比较混乱，要多注意。


## 4. 总结 ##
KVO是Cocoa对于Observer Pattern的内置实现，需要我们通过"订阅"-”响应“-”取消订阅“来设置自动, 手动或者值依赖KVO。KVO的API有一些坑需要注意，比如keyPath的选择，context的用法, ”响应“的混乱逻辑等。对于KVO的isa swizzle实现原理，我们将在后面OC Runtime系列会详细介绍。


## 5. 参考资料 ##
- [Key-Value Observing Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html);
- [深入理解KVO](http://zhangbuhuai.com/2015/04/29/understanding-KVO/);
- [Key-Value Observing Done Right](https://mikeash.com/pyblog/key-value-observing-done-right.html);
- [KVO Considered Harmful](http://khanlou.com/2013/12/kvo-considered-harmful/);
- [如何自己动手实现 KVO](http://tech.glowing.com/cn/implement-kvo/);