---
layout: post
title: OC Memory Managment(四)：Weak Strong Dance
categories: [01 Objective-C]
tags: [MemoManamement]
number: [5.5.1]
fullview: false
shortinfo: Swift 和 Objective-C 内存管理的一个重要问题是防止循环引用引起的内存泄露，其中对于匿名函数和self的循环引用问题最为常见。解决方式之一就是"Weak Strong Dance"。本文探讨循环引用的的问题，以及Weak Strong Dance的原理，
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. Retain Cycle 介绍 ##
在面向对象编程中, 引用类型分配在堆中(如类的实例，匿名函数), 值类型分配在栈中(如整数，浮点数)。Swift中的类型分为class，struct，enum：其中只有class是引用类型，其实例分配在堆中；struct(Array, Dictionary, String, Set)和enum都是值类型，其实例分配在栈中。对于分配在堆中的类的实例，我们要对其进行内存管理，Swift 和Objective-C都是用ARC(而不是垃圾回收机制)来对内存进行管理，包括对于内存泄露的处理。本文对Swift和Objective-C中的循环引用进行讨论。

{: .img_middle_mid}
![面向对象编程](/assets/images/posts/2016-03-07/面向对象编程.png)

## 2. Retain Cycle 问题及解决 ##
Retain Cycle 的Apple的定义:

>A problem, known as a retain cycle, can therefore arise if two objects may have <b>cyclical references</b>—that is, they have a strong reference to each other (either directly, or through a chain of other objects each with a strong reference to the next leading back to the first).

这里"cyclical reference"非常重要，它指出了Retain Cycle的核心，即循环强引用。有循环强引用才有循环引用问题；没有循环强引用就没有循环引用问题。仔细阅读上面这句话，别看它类似一句“废话”，它几乎可以回答所有在Stack OverFlow上面的循环引用的问题。我们将用下面几个例子来解释Retain Cycle和它的解决方法Weak Strong Dance。

### 2.1 Retain Cycle 问题 ###
其中一个很典型的与匿名函数（Objective-C 中是 block）相关的Retain Cycle 如下面这个Objective-C的例1:

{% highlight objc linenos %}
#import "ViewController.h"

@interface ViewController ()
@property (nonatomic, strong) UIView * rectangleView;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.rectangleView = [[UIView alloc] initWithFrame:CGRectMake(10, 10, 100, 20)];
    self.rectangleView.backgroundColor = [UIColor greenColor];
    [self.view addSubview:self.rectangleView];

}

-(void)viewDidAppear:(BOOL)animated{
    [super viewDidAppear:animated];
    [UIView animateWithDuration:5 animations:^{
        CGPoint center = self.rectangleView.center;
        center.x +=50;
        center.y +=50;
        self.rectangleView.center = center;
    }];
}
@end
{% endhighlight %}

在`[UIView animationWithDuration: animations:]`的方法中，我们用block捕获了`self.rectangleView` 同时也捕获了`self`，请问这里有Retain Cycle吗？答案是没有。因为block 虽然强引用了self, 但是self 没有强引用block。

那么何时self才会强引用block引起Retain Cycle呢，请看下面这个例2：

{% highlight objc linenos %}
#import "ViewController.h"

typedef void (^AnimationBlock)();

@interface ViewController ()
@property (nonatomic, strong) UIView *rectangleView;
@property (nonatomic, copy) AnimationBlock animationBlock;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.rectangleView = [[UIView alloc] initWithFrame:CGRectMake(10, 10, 100, 20)];
    self.rectangleView.backgroundColor = [UIColor greenColor];
    [self.view addSubview:self.rectangleView];
    self.animationBlock = ^{
        CGPoint center = self.rectangleView.center;
        center.x +=50;
        center.y +=50;
        self.rectangleView.center = center;
    };
}

-(void)viewDidAppear:(BOOL)animated{
    [super viewDidAppear:animated];
    [UIView animateWithDuration:5 animations:self.animationBlock];
}
@end
{% endhighlight %}

在这个例子里，block作为一个属性被self捕获，同时self也和第一个例子一样在block被其捕获，因此产生了循环引用。
因此请牢记下面这句话：

>有循环强引用才有循环引用问题。

### 2.2 Retain Cycle 解决方法：Weak Strong Dance ###
那么如何解决循环引用问题呢。相信大家对此都不陌生，那就是打破"循环强引用"，要么解决"循环"，要么解决"强"。显然解决""循环""会影响我们的业务逻辑，因此只能从解决"强"入手---将其中一个强引用设为弱引用，我们可以将例2改为如下例3：

{% highlight objc linenos %}
...
- (void)viewDidLoad {
    __weak ViewController *weakSelf = self;
    self.animationBlock = ^{
        CGPoint center = weakSelf.rectangleView.center;
        center.x +=50;
        center.y +=50;
        weakSelf.rectangleView.center = center;
    };
}

-(void)viewDidAppear:(BOOL)animated{
    [super viewDidAppear:animated];
    [UIView animateWithDuration:5 animations:self.animationBlock];
}
...
{% endhighlight %}

这个时候，由于block对self的引用是弱引用，因此打破了循环强引用，所以不会有Retain Cycle问题。但是这个方法不是Weak Strong Dance, 只是其一部分。如果硬要给它起个名字，应该叫Weak Dance 比较合适。这个方法有什么问题呢，我们来看看下面这个例4：

{% highlight objc linenos %}
...
- (void)viewDidLoad {
    __weak ViewController *weakSelf = self;
    self.animationBlock = ^{
        CGPoint center = weakSelf.rectangleView.center;
        center.x +=50;
        center.y +=50;
        weakSelf.rectangleView.center = center;
        weakSelf.rectangleView.backgroundColor = [UIColor blackColor];
    };
}
-(void)viewDidAppear:(BOOL)animated{
    [super viewDidAppear:animated];
    [UIView animateWithDuration:5 animations:self.animationBlock];
}
...
{% endhighlight %}

在例4中，block里面的weakSelf 接连调用了两次它的方法(setter)。因为weakSelf的存在，self 强引用block， block弱引用self。考虑一种情况，当block里的weakSelf调用其方法时，self已经不存在了：这个时候程序不会crash, 因为给nil send message什么都不会发生，但是我们期待的程序并没有执行。考虑第二种情况，当weakSelf调用第一个方法时，self还存在，调用第二个方法时，程序却不存在：这个时候程序不会crash，第一个方法执行，第二个方法不执行。

因此为了解决第二种情况，要么全执行，要么全不执行，我们引入一个strongSelf, 如例5：

{% highlight objc linenos %}
...
- (void)viewDidLoad {
    __weak ViewController *weakSelf = self;
    self.animationBlock = ^{
        ViewController *strongSelf = weakSelf;
        CGPoint center = strongSelf.rectangleView.center;
        center.x +=50;
        center.y +=50;
        strongSelf.rectangleView.center = center;
        strongSelf.rectangleView.backgroundColor = [UIColor blackColor];
    };
}

-(void)viewDidAppear:(BOOL)animated{
    [super viewDidAppear:animated];
    [UIView animateWithDuration:5 animations:self.animationBlock];
}
...
{% endhighlight %}
在例5中，当`self.animationBlock` 作为`[UIView animationWithDuration: animations:]`的最后一个参数运行时，传给strongSelf要么是nil，要么是短暂强引用self，因为strongSelf是block的变量(有生命周期，当block执行完毕，`strongSelf`自动设置为`nil`，viewController的retainCount减1，返回原来的值)。因此在打破循环引用的同时，又解决了第二种情况，要么全执行，要么全不执行。这对于我们的程序非常有利。这就是人们所谓的Weak Strong Dance。

在这里，可以用@weakify(self) 和@strongigy(self)代替上面栗5中的`__weak ViewController *weakSelf = self;` 和 `ViewController *strongSelf = weakSelf;`。weakify 创建了一个新的weak self 来替代原来的self，strongify 则相反。也就是说weakify和strongify使得变量`self`的引用属性在`weak`和`strong`里自由转换。


Weak Strong Dance 在swift中可以结合guard使用，如例6:

{% highlight swift linenos %}
class ViewController: UIViewController{
    var rectangleView: UIView?
    var animationFunction: ()->()
    func viewDidLoad() {

        ...
        self.animationFunction = {
            [weak self] in
            guard let strongSelf = self else{return}
            strongSelf.rectangleView?.backgroundColor = UIColor.redColor()
        }
        ...
    }
}
{% endhighlight %}


## 3 总结 ##
Weak Strong Dance 是用来打破循环引用的问题。对于self和匿名函数的循环引用问题，一个weakSelf可以打破循环引用，但是会引起self执行一部分方法的问题。解决方法是strongSelf， 要么全执行，要么全不执行。strongSelf结合weakSelf就是人们所谓的Weak Strong Dance。希望本文对于循环引用的问题的探讨对您有用。
