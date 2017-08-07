---
layout: post
title: UnsafeMutablePointer
categories: [02 Swift]
tags: [pointer]
number: [3.7.8]
fullview: false
shortinfo: 在swift中，对于指针的操作都被限制到最低，能不使用尽量不使用。但是还是有一些场合需要我们使用指针。Swift处理指针主要是UnsafePointer和其可变类UnsafeMutablePointer。本文主要介绍它们的用法。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. Swift中的指针介绍 ##


指针类型在Swift里被`struct UnsafePointer<Memory>`表示，它是一个拥有泛型的struct。它还有一个变体`struct UnsafeMutablePointer<Memory>`。`UnsafePointer` 对应的是C中的常量指针(例如`const char *`)，其他可变指针对应的是`UnsafeMutablePointer`。

我们先来看看`UnsafePointer`在Apple中的定义:

>struct UnsafePointer&ltMemory&gt：A pointer to an object of type Memory. This type provides no automated memory management, and therefore the user must take care to allocate and freee memory appropriately. 

它有三种状态：

1. 内存未分配（例如,pointer 值是null,或者内存被释放了）；
2. 内存分配，但是值未被初始化；
3. 内存分配，值被初始化。

只有第三种情况的pointer才能正常使用。


`UnsafeMutablePointer`在Apple中的定义定义和`UnsafePointer`一样，是其可变类型。我们下面主要介绍`UnsafeMutablePointer`。


## 2. UnsafeMutablePointer 例子##


### 2.1 例1: 基础 ###

{% highlight swift linenos %}
/*0. 变量地址作为指针值*/

var color = UIColor(hue: 0.1, saturation: 0.2, brightness: 0.3, alpha: 0.4)

var h: CGFloat = 0.0
var s: CGFloat = 0.0
var b: CGFloat = 0.0
var a: CGFloat = 0.0

/*
func getHue(hue: UnsafeMutablePointer<CGFloat>, saturation: UnsafeMutablePointer<CGFloat>, brightness: UnsafeMutablePointer<CGFloat>, alpha: UnsafeMutablePointer<CGFloat>) -> Bool
*/

color.getHue(&h, saturation: &s, brightness: &b, alpha: &a)

h   //0.1
s   //0.2
b   //0.3
a   //0.4

/*1.UnsafeMutablePointer<CGFloat>指针变量*/

var hp = UnsafeMutablePointer<CGFloat>.alloc(1) //分配内存
hp.initialize(0.0)                              //初始化值

var sp = UnsafeMutablePointer<CGFloat>.alloc(1) 
sp.initialize(0.0)                              

var bp = UnsafeMutablePointer<CGFloat>.alloc(1) 
bp.initialize(0.0)                              

var ap = UnsafeMutablePointer<CGFloat>.alloc(1) 
ap.initialize(0.0)                              

color.getHue(hp, saturation: sp, brightness: bp, alpha: ap)

hp.memory   //0.1， hp解引用的值
sp.memory   //0.2
bp.memory   //0.3
ap.memory   //0.4

hp.destroy()    //析构,和initialize对应
hp.dealloc(1)   //释放内存，和alloc对应

sp.destroy()
sp.dealloc(1)

bp.destroy()
bp.dealloc(1)

ap.destroy()
ap.dealloc(1)

{% endhighlight %}

从上述例子中可以看出，一个`UnsafeMutablePointer`的4个主要方法`alloc()`, `initialize()`, `与destroy()`, `dealloc()`是成对出现的,这就是Apple定义中所说的我们要手动管理内存。

### 2.2 例2: 与c指针的相互转换 ###
{% highlight swift linenos %}
/*COpaquePointer convert from/to UnsafeMutablePointer*/

var str = "Hello, pointer"
var sp1 = UnsafeMutablePointer<String>.alloc(1)
sp1.initialize(str)
sp1.memory  //"Hello, pointer"

/*Opaque pointers are used to represent C pointers to types that cannot be represented in Swift, 
such as incomplete struct types*/
var cop = COpaquePointer(sp1)               //convert UnsafeMutablePointer to COpaquePointer
var sp2 = UnsafeMutablePointer<String>(cop) //convert COpaquePointer to UnsafeMutablePointer

sp2.memory      //"Hello,pointer"
sp1.destroy()   // return the pointer to uninitialize state
sp1.dealloc(1)  // dealocate the memory

/*CFunctionPointer convert from/to UnsafeMutablePointer*/


var f: () -> String = {
    return "Hello from C"
}

var fp1 = UnsafeMutablePointer<() -> String>.alloc(1)
fp1.initialize(f)

var cop2_1 = COpaquePointer(fp1)                            //convert UnsafeMutablePointer to COpaquePointer
var cfp = CFunctionPointer<() -> String>(cop2_1)            //convert COpaquePointer to CFunctionPointer
var cop2_2 = COpaquePointer(cfp)                            //convert CFunctionPointer to COpaquePointer
var fp2 = UnsafeMutablePointer<() -> String>.alloc(cop2_2)  //convert COpaquePointer to UnsafeMutablePointer

fp2.memory      // calls anonymous function and returns "Hello from C"
fp1.destroy()   // return the pointer to uninitialize state
fp1.dealloc(1)  // dealocate the memory
{% endhighlight %}

从上述例子可以看出Swift的`UnsafeMutalbePointer` 可以和C的`COpaguqePointer`与`CFunctionPointer` 自由转换

### 2.3 例3: 地址操作 ###


{% highlight swift linenos %}
/*UnsafeMutablePointer的绝对地址*/
let aInt = UnsafeMutablePointer<Int>.alloc(1)
aInt.initialize(10)
let bInt = UnsafeMutablePointer<Int>.alloc(1)
bInt.initialize(100)

aInt.debugDescription               //memory address as string
aInt.hashValue                      //memory address as integer

bInt.debugDescription
bInt.hashValue

bInt.successor().debugDescription
bInt.successor().hashValue

bInt.predecessor().debugDescription
bInt.predecessor().hashValue

bInt.distanceTo(aInt)               //memory address distance, -249200

let c = bInt.move()                 //return bInt.memory and destroy bInt
bInt.initialize(100)


/*UnsafeMutablePointer的绝相对地址*/
var cInt = aInt.advancedBy(3)       // advance by sizeOf(Int) * 3 = 24
cInt.debugDescription
bInt.distanceTo(cInt)               // -249197


/*多个UnsafeMutablePointer指向同一个地址*/
let hashV = bInt.hashValue
var dInt = UnsafeMutablePointer<Int>(bitPattern: hashV) //dInt 和 bInt指向同一个地址
bInt.memory = 36
dInt.memory         //36

bInt.destroy()
bInt.dealloc(1)     //now dInt is nil


/*UnsafeMutablePointer从另一个变量copy*/

var eInt = UnsafeMutablePointer<Int>.alloc(1)
eInt.initialize(40)

var fInt = UnsafeMutablePointer<Int>.alloc(1)
var gInt = UnsafeMutablePointer<Int>.alloc(1)
fInt.initializeFrom(eInt, count: 1)     //initialize by copy deferenced value from eInt
gInt.moveInitializeFrom(eInt, count: 1) //initialize by copy deferenced value from eInt + destroy eInt

fInt.memory     //40
{% endhighlight %}

上述例子对`UnsafeMutablePointer`的`debugDescription()`, `hashValue`, `successor()`, `predecessor()`, `distanceTo()`, `move()`, `initializeFrom()`等方法的用法进行了简单的介绍。


## 3 总结 ##
`UnsafeMutablePointer`涉及手动内存管理，具体体现在4个主要方法`alloc()`, `initialize()`, `与destroy()`, `dealloc()`的成对出现上；它可以与C的`COpaguqePointer`与`CFunctionPointer`自由转换；它的属性和方法有`debugDescription()`, `hashValue`, `successor()`, `predecessor()`, `distanceTo()`, `move()`, `initializeFrom()`。`UnsafeMutablePointer`一般不用，正如其名Unsafe, 会带来一定的安全问题。如果碰到一定要用的情况，要注意上述几点。


