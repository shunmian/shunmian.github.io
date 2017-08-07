---
layout: post
title: Let's Build Swift.Array
categories: [02 Swift]
tags: [Array]
number: [4.12.1]
fullview: false
shortinfo: 这是一篇对于Swift中的Array的实现的讨论。通过实现我们自己的Array来达到Swift的Array的基本功能和不相上下的性能对于我们提高自己的编程能力大有帮助。
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
下面是Objective-C的NSArray
{% highlight objc linenos %}
/*NSArray is class, reference type*/
NSMutableArray *array1 = [@[@1, @2, @3] mutableCopy];
NSMutableArray *array2 = array1;
[array2 insertObject:@4 atIndex:3];

NSLog(@"%@", array1);   //@[@1,@2,@3,@4]
NSLog(@"%@", array2);   //@[@1,@2,@3,@4],array2&array1 point to the same objects
{% endhighlight %}



下面是Swift的Array
{% highlight swift linenos %}
/*Array is struct, value type*/
var a = [1,2,3]
var b = a
b.append(4)

a   //[1,2,3]
b   //[1,2,3,4]
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
fInt.initializeFrom(eInt, count: 1) //initialize by copy deferenced value from eInt
fInt.memory     //40
{% endhighlight %}

上述例子对`UnsafeMutablePointer`的`debugDescription()`, `hashValue`, `successor()`, `predecessor()`, `distanceTo()`, `move()`, `initializeFrom()`等方法的用法进行了简单的介绍。


## 3 总结 ##
`UnsafeMutablePointer`涉及手动内存管理，具体体现在4个主要方法`alloc()`, `initialize()`, `与destroy()`, `dealloc()`的成对出现上；它可以与C的`COpaguqePointer`与`CFunctionPointer`自由转换；它的属性和方法有`debugDescription()`, `hashValue`, `successor()`, `predecessor()`, `distanceTo()`, `move()`, `initializeFrom()`。`UnsafeMutablePointer`一般不用，正如其名Unsafe, 会带来一定的安全问题。如果碰到一定要用的情况，要注意上述几点。


