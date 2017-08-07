---
layout: post
title: iOS 2D Game - SpriteKit入门(一)
categories: [01 Objective-C]
tags: [SpriteKit]
fullview: false
shortinfo: SpriteKit是Apple官方的2D游戏框架， 让开发者在iOS 和OS 平台上更高效的开发2D游戏。 学习SpriteKit， 最权威的资料应该是苹果官方文档-SpriteKit Programming Guide。该文档很全面但是不适合初学者。 对于初学者， 理解SpriteKit背后的设计rationale才能更好的应用它。 什么是2D游戏设计的几个基本问题， 什么是sprite， SpriteKit 里面有哪些重要的类及其他们之间的关系等等，本文带着您一起梳理一遍...
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. SpriteKit 概览 ##

SpriteKit是Apple官方的2D游戏框架， 让开发者在iOS 和OS 平台上更高效的开发2D游戏。

学习SpriteKit， 最权威的资料应该是苹果官方文档 -
[SpriteKit Programming Guide](https://developer.apple.com/library/ios/documentation/GraphicsAnimation/Conceptual/SpriteKit_PG/Introduction/Introduction.html)。 该文档很全面但是不适合初学者。 对于初学者， 理解SpriteKit背后的设计rationale才能更好的应用它。 那么SpriteKit的设计rationale是什么呢？

对于一个2D游戏来说， 设计应该分为三个部分(以超级马里奥为例):

1. View: 即视图显示，用于展示各个Sprite(精灵)，例如马里奥图片和子弹图片;
2. Physics Model:即物理模型， 包括质量， 体积(2D游戏是面积)， 密度， 碰撞， 地球引力等， 例如超级马里奥碰到水管会弹回来， 往上跳会下落。
3. Action:即 物理模型受到的外部作用力。例如超级马里奥里的空中台阶自己来回移动(开发者给其施加一个永恒的来回运动的作用力)。

这三个方面分别对应SpriteKit里面的SKNode，SKPhysicsWorld & SKPhysicsBody，SKAction。


## 2. SKNode，SKPhysicsWorld & SKPhysicsBody，SKAction理解##
### 2.1 SKNode###

SKNode 是 SpriteKit 显示视图的Building Block。它能提供一个游戏视图的基本属性和方法:


{% highlight objective-c linenos %}
SKNode Class

属性:
.zPosition              // 视图的z方向的距离，用于标定重叠视图的显示顺序;
.xScale                 // x方向的视图放大倍数;
.yScale                 // y方向的视图放大倍数;
.alpha                  // 视图的alpha值;
.hidden                 // 视图是否隐藏;

方法:
-addChild:              // 增加子SKNode，与UIView 的 -addSubView: 类似;
-removeFromParent:      // 从父SKNode移除，与 UIView的removeFromSuperview 类似;
-runAction:             // 运行一个SKAction;
{% endhighlight %}



我们一般不直接用它，它下面有几个子类，其中最常见的是以下4个:

- SKEffectNode: 用于缓存，渲染，加滤镜于图片。它的子类SKScene 用于展示所有的SKNode，是游戏场景;
- SKSpriteNode: 用于展示精灵，如超级马里奥图片;
- SKLabelNode: 用于展示单行文本，如游戏时间;
- SKEmitterNode: 用于展示粒子，例如喷射火焰的岩浆;

下面我们就这四个类做一个简单的介绍。

#### 2.1.1 SKScene####
SKScene 是游戏关口(level)，游戏中的一个场景，例如马里奥的第一关和第二关分别是两个SKCene实例。在这个场景中，包含了所有其他SKNode(或者其子类)，比如马里奥(SKSpriteNode)，游戏时间(SKLabelNode)，喷射火焰的岩浆(SKEmitterNode)。它的主要属性和方法有:
{% highlight objective-c linenos %}
SKScene Class

属性:
.view                   // 关口的父视图，是一个SKView，用来展示各个SKScene关口;
.physicsWorld           // 世界的物理模型，是一个SKPhysicsWorld实例，这个后面会介绍;

方法:
-initWithSize:          // 初始化方法;
+SceneWithSize:         // 初始化类工厂方法;
-didMoveToView:         // 当SKScene实例被SKView展示时调用，类似UIView的-didMoveToSuperview;
-addChild:              // 增加子SKNode，如马里奥(SKSpriteNode);
{% endhighlight %}

#### 2.1.2 SKSpriteNode####
SKSpriteNode是用来展示sprite，那么何为sprite呢，sprite有什么作用呢? Wiki中是这样定义的。

>sprite: two-dimensional image or animation that is integrated into a larger scene.Initially including just graphical objects handled **separately** from the memory bitmap of a video display，this now includes various manners of graphical overlays.

sprite是从整个display独立出来渲染的2D图片。如何理解这句话呢，在sprite出现之前，2D游戏要渲染一帧图片(比如马里奥在一个蓝天白云的背景前)，需要把整个图片(马里奥+背景)计算完后再渲染，其中背景的渲染在每一帧中都重复。sprite的出现正是为了避免这一重复。马里奥是一个sprite，他在一个固定的背景前跳跃，只需要将马里奥的每一帧渲染出来叠在背景(背景不需要重复计算渲染)即可，这就是为什么sprite被称为从整个display独立出来渲染的2D图片。sprite的思想在几十年前就已经有了，SpriteKit只是沿袭了2D游戏设计中运用sprite这一思想，用SKSpriteNode来表示sprite类。我们来看下SKSpriteNode的属性和方法

{% highlight objective-c linenos %}

SKSpriteNode Class

属性:
.size                       // 大小;
.physicsBody                // 物体的物理模型，是一个SKPhysicsBody实例，这个后面会介绍;

方法:
+spriteNodeWithImageNamed:  // 类工厂方法，用图片创建sprite;

{% endhighlight %}

#### 2.1.3 SKLabelNode####
SKLabelNode是SpriteKit用来展示text，它的方法和属性如下。有一点需要注意的是它只能显示单行文本。

{% highlight objective-c linenos %}
SKLabelNode Class

属性:
.fontSize               // 字体大小;
.color                  // 字体颜色;
.fontName               // 字体名字;

方法:
-initWithFontNamed:     // 通过字体名字初始化方法;
{% endhighlight %}
#### 2.1.4 SKEmitterNode ####
SKEmitterNode是SpriteKit用来展示粒子系统的，下面介绍下它的常见使用方法。

1. subclass一个SpriteKit Particles Files ( ctrl + N --> iOS，Resources，SpriteKit Particles Files --> choose 1 of the eight template --> save，你会得到一个.sks 和.png 文件，点击.sks文件:

    ![SKEmitterNode_1](/assets/images/posts/2016-02-20/SKEmitterNode_1.png)
    ![SKEmitterNode_2](/assets/images/posts/2016-02-20/SKEmitterNode_2.png)
    ![SKEmitterNode_3](/assets/images/posts/2016-02-20/SKEmitterNode_3.png)
    ![SKEmitterNode_4](/assets/images/posts/2016-02-20/SKEmitterNode_4.png)
2. 在.sks文件右侧调整各参数，例如有粒子平均产生率(BirthRate) 单位是个/秒。右边的range是一个分布，在平均产生率上 ±  range/2 分布。particle texture 是粒子的纹路，你可以选择自己加入的图片文件。
3. 调整完成后如何在code中调用.sks文件呢:

{% highlight objc linenos %}

SKEmitterParticle * fireParticles = [NSKeyedUnarchiver unarchiveObjectWithFile:[[NSBundle mainBundle] pathForResource:@"FireParticle" ofType:@"sks"]];

fireParticles.particleBirthRate = 1;
fireParticles.position = CGPointMake(0, -180);

{% endhighlight %}
     用NSBundle读取.sks 文件，然后可以进一步修改其属性。这样就完成了SpriteKit Particles Files 的子类化和应用。


### 2.2 SKPhysicsWorld & SKPhysicsBody ###
SpriteKit 里表示物理模型的有两个类，SKPhysicsWorld & SKPhysicsBody，前者属于SKScene，后者属于SKNode其他子类。SKPhysicsWorld 和 SKPhysicsBody 都继承与NSObject。我们先来看看SKPhysicsWorld。

#### 2.2.1 SKPhysicsWorld ####
对于一个物理世界来说，例如我们的地球，有一些属性决定了我们日常生活的基础，如重力加速度。
{% highlight objective-c linenos %}
SKPhysicsWorld Class

属性:
.gravity:                   // 重力加速度了， 通过CGVectorMaker(0，-9.8)获得现实生活一个重力加速度。第一个参数是x轴，向右为正，第二个参数是y轴，向上为正;
.contactDelegate            // SKPhyicsContactDelegate 实例;
{% endhighlight %}

又比如物体碰撞后的处理者，可以理解为物体碰撞后有一个裁判需要对此进行处理，在SpriteKit中是SKPhyicsContactDelegate。而SKPhysicsWorld 的contactDelegate属性就指向这样一个delegate，也就是说由物理世界充当裁判的角色。

{% highlight objective-c linenos %}
SKPhysicsContactDelegate Class

方法:
-didBeginContact:       // 碰撞开始;
-didEndContact:         // 碰撞结束;
{% endhighlight %}
对于SKPhysicsContactDelegate两个方法的使用我们后面介绍。

#### 2.2.2 SKPhysicsBody ####
SKPhysicsBody代表物理模型里的物体，它有质量，体积(2D 游戏里是面积)，密度，线速度，角速度，自旋速度等。

{% highlight objective-c linenos %}
SKPhysicsBody Class

属性:
.mass                       // 质量，单位kg;
.area                       // 面积，单位m*m;
.density                    // 密度，单位kg/m*m;
.friction                   // 表面粗糙度，0.0-1.0;
.restitution                // 碰撞时，反射速度/入射速度，0.0-1.0;
.linearDamping              // 物体线速度受液体或者气体(空气)摩擦的影响， 0.0-1.0;
.dynamic                    // YES，动态; NO，静态，与SKPhysicsWorld相连，不受碰撞影响;
.categoryBitMask            // 自身的contact & collision ID;
.contactTestBitMask         // 外界contact测试的ID集合;
.collisionBitMask           // 外界collision的ID集合;

方法:
+bodyWithRectangleOfSize:   // 创建Volume-physicsBody的一种类工厂方法;
+bodyWithEdgeLoopFromRect:  // 创建Edge-physicsBody的一种类工厂方法;
-applyImpulse:              // 被施加外部瞬时力;
-applyForce:                // 被施加外部永恒力;
-runAction:                 // 运行一个SKAction实例;

{% endhighlight %}

 .dynamics 是一个BOOL，设置为NO时，静止(可以理解为与SKPhyisicsWorld相连)，位置不受碰撞前后影响， 例如马里奥里的乌龟壳，碰到水管后反弹，水管静止，这里水管的dynamic就是NO， 乌龟壳的dynmaics就是YES。

 要创建一个在框里永远碰撞而不停下的ball，应设置如下friction，restituition， linearDamping:
 {% highlight objc linenos %}

ball.friction = 0;
ball.restitution = 1;
ball.linearDamping = 0;

{% endhighlight %}

SKPhysicsBody的创建分为两种，一种是Volume-physicsBody，就是有体积(2D里是面积)的物体，有体积意味着有质量，受牛顿定律影响;另一种是只有边缘没有体积的Edge-physicsBody，如上面那个装永动球的箱子，我们只关心箱子的边框，用 +bodyWithEdgeLoopFromRect: 创建它并设置其dynamics = NO 即可。

下面重点要介绍的是接触和碰撞。
SpriteKit为每个物体在接触和碰撞时设定了一个身份证——categoryBitMask，该属性唯一标定了接触和碰撞时每个物体的身份，是一个32 bit的数，我们下面先看code再解释:
{% highlight objc linenos %}
static const uint32_t MariaCategory              = 0x1;
static const uint32_t TortoiseCategory           = 0x1 << 1;
static const uint32_t BulletCategory             = 0x1 << 2;
{% endhighlight %}

我们设置了3种uint32_t的静态常量，分别代表马里奥，子弹，乌龟。
{% highlight objec linenos %}
maria.categoryBitMask = MariaCategory;
tortoise.categoryBitMask = TortoiseCategory;
bullet.categoryBitMask = BulletCategory;
{% endhighlight %}

 然后分别给maria，tortoise以及bullte的categoryBitMask设置为相应值。同时我们需要在以下情形中判断contact发生并作出相应，如马里奥碰到乌龟，马里奥die;子弹碰到乌龟，乌龟die:

 {% highlight objc linenos %}
maria.contactTestBitMask = TortoiseCategory;
tortoise.contactTestBitMask = BulletCategory | MariaCategory;
bullet.contactTestBitMask = TortoiseCategory;
{% endhighlight %}

 上面将maria感兴趣的contact ID集设置为乌龟，乌龟设置为子弹和马里奥，子弹设置为乌龟。用32位非常便于取或操作，同时也限定了一个游戏场景里的碰撞接触的身份证只能有32个。

 {% highlight objc linenos %}
@interface GameScene : SKScene <SKPhysicsContactDelegate>
@end

-(void)moveToParent:(SKNode * )parent{
    self.physicsWorld.contactDelegate = self;
}

-(void)didBeginContact:(SKPhysicsContact * )contact{
    //我们来简单举一个子弹打到乌龟，乌龟die的contact测试。
    SKPhysicsBody * tortoise;

    if(contact.bodyA.categoryBitMask == BulletCategory &&
        contact.bodyB.categoryBitMask == TortoiseCategory){
        tortoise = contact.bodyB;
    }
    if(contact.bodyB.categoryBitMask == BulletCategory &&
        contact.bodyA.categoryBitMask == TortoiseCategory){
        tortoise = contact.bodyA;
    }
    NSLog(@"tortoise die: %@", tortoise);
}
{% endhighlight %}


这样当子弹打到乌龟，GameScene 作为SKPhysicsContactDelegate就会响应-didBeginContact: 方法，然后判断情形，如果是，则打印出来”tortoise die:tortoise的object信息”。

了解了categoryBitMask 和contactTestBitMask， 我们再来看collisionBitMask就简单了，它表示物体之间会不会intersect，也是32位数。默认是32个1，表示与任何物体碰撞都不会穿过那个物体。如果你需要将某个物体穿过另一个物体，比如子弹打到马里奥，直接穿过(当然游戏中不可能发生):

{% highlight objec linenos %}
maria.collisionBitMask = ~BulletCategory;
{% endhighlight %}
最后力的施加我们在SKAction里介绍。

### 2.3. SKAction ###
SKAction表示一个动作，由SKNode得 -runAction: 方法执行，它有几十个方法。下面列出几个比较典型的， 它的实例化大部分都是类工厂方法:

{% highlight objective-c linenos %}
SKAction Class

初始化方法:
+moveByX:y:duration:                    // 移动一个SKNode;
+rotationByAngule:duration:             // 转动一个SKNode;
+hide                                   // 隐藏一个SKNode， 对应的还有 +unhide;
+applyForce:duration:                   // 施加力;
+playSoundFileNamed:waitForCompletion:  // 播放音频文件;
+removeFromParent                       // 将SKNode从父node里移除;
+waitForDuration:                       // 等待的动作，在SKAction group和sequence里用到;

+group:                                 // 组合几个SKAction，从时间起点，这几个SKAction同时开始;
+sequence:                              // 串联几个SKAction，从时间起始点，下一个的开始在上一个结束后才执行;
+repeatActionForever:                   // 永远重复SKAction;
-reversedAction:                        // 逆向一个SKAction，如原来从左到右2秒，现在从右到左两秒;

+runBlock:queque:                       // 在队列里执行SKAction;
+customActionWithDuration:actionBlock:  // 定制SKAction;

{% endhighlight %}
例如马里奥中需要一个空中台阶，“”从左到右2秒，等待0.5秒，从右到左2秒，等待0.5秒 ”的sequence永远运行下去，code如下:
{% highlight objc linenos %}
SKAction * moveToRight = [SKAction moveByX:view.bounds.size.width-self.platform.size.width y: 0 duration:2];
SKAction * moveToLeft = [moveToRight reversedAction];
SKAction * wait = [SKAction waitForDuration:0.5];
SKAction * moveToRightAndLeft = [SKAction sequence:@[moveToRight，wait，moveToLeft，wait]];
SKAction * moveToRightAndLeftForever = [SKAction repeatActionForever:moveToRightAndLeft];
[self.platform runAction:moveToRightAndLeftForever];
{% endhighlight %}

## 3.其它类 ##
在SKView中，场景的不同切换要用到SKTransition，self指一个scene:

{% highlight objc linenos %}

SKTransition * doorOpenTransition = [SKTransition doorsOpenHorizontalWithDuration:1.5];
[self.view presentScene:winScene transition:doorOpenTransition];

{% endhighlight %}
## 4 总结 ##
以上对于SKSpriteKit里的大部分常用类及其用法做了介绍，相信读者能够自己画出SKSpriteKit的类图关系，再回过头来跟着online的SKSpriteKit的tutorial做一遍的时候，自然就了然于心。后面还有一篇从各个小topic来看SKSpriteKit。
