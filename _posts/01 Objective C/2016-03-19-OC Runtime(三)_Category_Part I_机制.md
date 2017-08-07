---
layout: post
title: OC Runtime(三)：Category Part I：机制
categories: [01 Objective-C]
tags: [isa Swizzling]
number: [0.14.1.2]
fullview: false
shortinfo: 在Objective C中, Key Value Observing 是用来对Observing Pattern的Apple官方实现。 它的实现原理用的是isa Swizzling。但是apple对其细节没有多讲。本文用三种方式来实现Oberving Pattern：普通设计模式, isa Swizzling和Method Swizzling。希望通过本文能让您对Observing Pattern有一个全面的认识。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1. category ##

### 1.1 category的编译 ###



{% highlight objc linenos %}
//MyClass.h
#import <Foundation/Foundation.h>
@interface MyClass : NSObject
- (void)printName;
@end

@interface MyClass(MyAddition)
@property(nonatomic, copy) NSString *name;
- (void)printName;
@end

//MyClass.m
#import "MyClass.h"
@implementation MyClass
- (void)printName{
    NSLog(@"%@",@"MyClass");
}
@end

@implementation MyClass(MyAddition)
- (void)printName{
    NSLog(@"%@",@"MyAddition");
}
@end

{% endhighlight %}

用clang重写。

{% highlight objc linenos %}
$ clang -rewrite-objc sark.m
{% endhighlight %}

同级目录下会生成sark.cpp，这就是objc代码重写成c++(基本就是c)的实现。
打开生成的文件，发现茫茫多，排除include进来的header，自己的代码都在文件尾部了，看看上面的category被编译器搞成什么样子了。

{% highlight objc linenos %}
static struct /*_method_list_t*/ {
    unsigned int entsize;  // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[1];
} _OBJC_$_CATEGORY_INSTANCE_METHODS_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    1,
    {{(struct objc_selector *)"printName", "v16@0:8", (void *)_I_MyClass_MyAddition_printName}}
};

static struct /*_prop_list_t*/ {
    unsigned int entsize;  // sizeof(struct _prop_t)
    unsigned int count_of_properties;
    struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_prop_t),
    1,
    {{"name","T@\"NSString\",C,N"}}
};

extern "C" __declspec(dllexport) struct _class_t OBJC_CLASS_$_MyClass;

static struct _category_t _OBJC_$_CATEGORY_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = 
{
    "MyClass",
    0, // &OBJC_CLASS_$_MyClass,
    (const struct _method_list_t *)&_OBJC_$_CATEGORY_INSTANCE_METHODS_MyClass_$_MyAddition,
    0,
    0,
    (const struct _prop_list_t *)&_OBJC_$_PROP_LIST_MyClass_$_MyAddition,
};
static void OBJC_CATEGORY_SETUP_$_MyClass_$_MyAddition(void ) {
    _OBJC_$_CATEGORY_MyClass_$_MyAddition.cls = &OBJC_CLASS_$_MyClass;
}
#pragma section(".objc_inithooks$B", long, read, write)
__declspec(allocate(".objc_inithooks$B")) static void *OBJC_CATEGORY_SETUP[] = {
    (void *)&OBJC_CATEGORY_SETUP_$_MyClass_$_MyAddition,
};
static struct _class_t *L_OBJC_LABEL_CLASS_$ [1] __attribute__((used, section ("__DATA, __objc_classlist,regular,no_dead_strip")))= {
    &OBJC_CLASS_$_MyClass,
};
static struct _category_t *L_OBJC_LABEL_CATEGORY_$ [1] __attribute__((used, section ("__DATA, __objc_catlist,regular,no_dead_strip")))= {
    &_OBJC_$_CATEGORY_MyClass_$_MyAddition,
};
static struct IMAGE_INFO { unsigned version; unsigned flag; } _OBJC_IMAGE_INFO = { 0, 2 };

{% endhighlight %}

我们可以看到：

1. 首先编译器生成了实例方法列表`OBJC$_CATEGORY_INSTANCE_METHODSMyClass$_MyAddition`和属性列表`OBJC$_PROP_LISTMyClass$_MyAddition`，两者的命名都遵循了公共前缀+类名+category名字的命名方式，而且实例方法列表里面填充的正是我们在`MyAddition`这个category里面写的方法`printName`，而属性列表里面填充的也正是我们在MyAddition里添加的name属性。还有一个需要注意到的事实就是category的名字用来给各种列表以及后面的category结构体本身命名，而且有`static`来修饰，所以在同一个编译单元里我们的category名不能重复，否则会出现编译错误。

2. 其次，编译器生成了category本身`OBJC$_CATEGORYMyClass$_MyAddition`，并用前面生成的列表来初始化category本身。

3. 最后，编译器在DATA段下的`objc_catlist section`里保存了一个大小为1的`category_t`的数组`L_OBJC_LABELCATEGORY$`（当然，如果有多个category，会生成对应长度的数组^_^），用于运行期category的加载。
到这里，编译器的工作就接近尾声了，对于category在运行期怎么加载，请接着看。

### 1.2 category的runtime动态加载 ###

#### 1.2.1 调用栈 ####

Runtime的初始化过程中，对``Category``动态加载的调用栈简单整理如下：

{% highlight c linenos %}

void _objc_init(void)
└──void map_images(unsigned count, const char * const paths[],const struct mach_header * const mhdrs[])
    └──void map_images_nolock(unsigned mhCount, const char * const mhPaths[], const struct mach_header * const mhdrs[])
        └──void _read_images(header_info **hList, uint32_t hCount, int totalClasses, int unoptimizedTotalClasses)
            └──static Class realizeClass(Class cls)
                └──static void methodizeClass(Class cls)
                    └──static void attachCategories(Class cls, category_list *cats, bool flush_caches)


{% endhighlight %}

关键函数是``attachCategories``。

#### 1.2.2 相关结构体 ###

在理解该函数前，我们首先更新下``Class``的内存分布知识以及该方法会用到的几个结构体。

{% highlight c linenos %}
struct category_t {
    const char *name;   //类的名字，而不是category括号里的名字
    classref_t cls;     //cls要扩展的类对象，编译期间这个值是不会有的，在app被runtime加载时才会根据name对应到类对象
    struct method_list_t *instanceMethods;  //该category里所有-号方法
    struct method_list_t *classMethods;     //该category里所有+号方法
    struct protocol_list_t *protocols;      //该category里实现的协议
    struct property_list_t *instanceProperties; //该category里所有的property，不过它们不会@synthesiz实例变量，需要的话得通过objc_setAssociatedObject和objc_getAssociatedObject方法绑定。
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};

{% endhighlight %}

可见一个`category`持有了一个`method_list_t`类型的数组，`method_list_t` 又继承自`entsize_list_tt`，这是一种泛型容器:

{% highlight c linenos %}
struct method_list_t : entsize_list_tt {
    // 成员变量和方法
};
 
template 
struct entsize_list_tt {
    uint32_t entsizeAndFlags;
    uint32_t count;
    Element first;
};
{% endhighlight %}

可以简单理解`method_list_t`是一个存储`method_t`类型元素的容器。

另外category_list，定义如下：
{% highlight c linenos %}
struct locstamped_category_list_t {
    uint32_t count;
    locstamped_category_t list[0];
};
struct locstamped_category_t {
    category_t *cat;
    struct header_info *hi;
};
typedef locstamped_category_list_t category_list;
{% endhighlight %}

除了标记存储的`category`的数量外，`locstamped_category_list_t`结构体还声明了一个长度为零的数组，这其实是 C99 中的一种写法，允许我们在运行期动态的申请内存。

在Objective 2.0中，`Class`结构体已经发生了改变。

{% highlight c linenos %}

struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
 
struct objc_class : objc_object {
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
 
    class_rw_t *data() { 
        return bits.data();
    }
};
{% endhighlight %}

在Objective2.0的`Class`模型中，`cache`和`super_class`没有变，而其他属性被`bits`所取代，并且多了1个`data()`方法返回`class_rw_t`结构体指针。那么什么是`class_data_bits_t`呢。

{% highlight c linenos %}
struct class_data_bits_t {
    uintptr_t bits;
public:
    class_rw_t* data() {
        return (class_rw_t *)(bits & FAST_DATA_MASK);
    }
}
{% endhighlight %}

可见这个结构体只有一个64位的bits，存储了一个指向`class_rw_t`结构体的指针和三个标志位。它实际上是由3部分组成：

1. Mac OSX只使用47位内存地址；

2. 前17位空余出来，提供给retain/release和alloc/dealloc；

3. 由于内存对齐，后散文都是0，可以用来做标志位。

{% highlight c linenos %}
// class is a Swift class
#define FAST_IS_SWIFT           (1UL<<0)
// class or superclass has default retain/release/autorelease/retainCount/
//   _tryRetain/_isDeallocating/retainWeakReference/allowsWeakReference
#define FAST_HAS_DEFAULT_RR     (1UL<<1)
// class's instances requires raw isa
#define FAST_REQUIRES_RAW_ISA   (1UL<<2)
// data pointer
#define FAST_DATA_MASK          0x00007ffffffffff8UL
{% endhighlight %}

`FAST_DATA_MASK` 这个 16 进制常量的二进制表示恰好后三位为0，且长度为47位: 11111111111111111111111111111111111111111111000，我们通过这个掩码做按位与运算即可取出正确的指针地址。

bits中包含了一个指向`class_rw_t`结构体的指针，它的定义如下:

{% highlight c linenos %}
struct class_rw_t {
    uint32_t flags;
    uint32_t version;
 
    const class_ro_t *ro;
 
    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;
}
{% endhighlight %}

注意到有一个名字很类似的结构体`class_ro_t`，这里的 `rw` 和 `ro` 分别表示 ‘readwrite’ 和 ‘readonly’。因为`class_ro_t`存储了一些由编译器生成的常量。如果阅读`class_ro_t` 结构体的定义就会发现，旧版本实现中类结构体中的大部分成员变量现在都定义在`class_ro_t`和`class_rw_t`这两个结构体中了。感兴趣的读者可以自行对比，本文不再赘述。

`class_rw_t`结构体中还有一个`methods`成员变量，它的类型是`method_array_t`，继承自`list_array_tt`。

{% highlight c linenos %}
template {
    struct array_t {
        uint32_t count;
        List* lists[0];
    };
}
class method_array_t : public list_array_tt
{% endhighlight %}
这是1个二维数组，和旧版的`objc_method_list_t **methodLists`类似。

#### 1.2.3 关键函数attachCategories ###

对``Category``中方法的解析并不复杂，首先来看一下``attachCategories``的简化版代码:

{% highlight c linenos %}
static void attachCategories(Class cls, category_list *cats, bool flush_caches) {
    if (!cats) return;
    bool isMeta = cls->isMetaClass();
 
    method_list_t **mlists = (method_list_t **)malloc(cats->count * sizeof(*mlists));
    // Count backwards through cats to get newest categories first
    int mcount = 0;
    int i = cats->count;
    while (i--) {
        auto& entry = cats->list[i];
 
        method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            mlists[mcount++] = mlist;
        }
    }
 
    auto rw = cls->data();
 
    prepareMethodLists(cls, mlists, mcount, NO, fromBundle);
    rw->methods.attachLists(mlists, mcount);
    free(mlists);
    if (flush_caches  &&  mcount > 0) flushCaches(cls);
}
{% endhighlight %}

代码比较容易理解，即遍历`category_list`里的每个`category`，将`method_list`指针放入`mlist`。`mlist`是个二级指针，里面的值是指向`method_list`数组的地址。也就是说，若`MyClass`有两个category(`Category1`，`Category2`)，对应的方法分别是`method1_1`，`method1_2`和`method2_1`，`method2_2`。则mlist指向了个二维数组[[method1_1,method1_2],[method2_1,method2_2]]。

在获取了所有`category`里的method后，就要加入到原来的类的method里：
{% highlight c linenos %}
    auto rw = cls->data();
    rw->methods.attachLists(mlists, mcount);
{% endhighlight %}

rw是一个`class_rw_t`类型的结构体指针。根据runtime中的数据结构，它有一个 methods 结构体成员，并从父类继承了`attachLists`方法，用来合并`category` 中的方法:

{% highlight c linenos %}
void attachLists(List* const * addedLists, uint32_t addedCount) {
    if (addedCount == 0) return;
    uint32_t oldCount = array()->count;
    uint32_t newCount = oldCount + addedCount;
    setArray((array_t *)realloc(array(), array_t::byteSize(newCount)));
    array()->count = newCount;
    memmove(array()->lists + addedCount, array()->lists, oldCount * sizeof(array()->lists[0]));
    memcpy(array()->lists, addedLists, addedCount * sizeof(array()->lists[0]));
}
{% endhighlight %}

这段代码用`realloc`函数将原来的空间拓展，然后把原来的数组复制到后面，再把新数组复制到前面。需要注意的是，无论执行哪种逻辑，参数列表中的方法都会被添加到二维数组的前面。而我们简单的看一下runtime在查找方法时的逻辑:
{% highlight c linenos %}
static method_t *getMethodNoSuper_nolock(Class cls, SEL sel){
    for (auto mlists = cls->data()->methods.beginLists(), 
              end = cls->data()->methods.endLists(); 
         mlists != end;
         ++mlists) {
        method_t *m = search_method_list(*mlists, sel);
        if (m) return m;
    }
 
    return nil;
}
 
static method_t *search_method_list(const method_list_t *mlist, SEL sel) {
    for (auto& meth : *mlist) {
        if (meth.name == sel) return &meth;
    }
}
{% endhighlight %}

可见搜索的过程是按照从前向后的顺序进行的，一旦找到了就会停止循环。因此category 中定义的同名方法不会替换类中原有的方法，但是对原方法的调用实际上会调用category 中的方法。

## 2 Cateogry题目 ##

>若有多个同类的category声明和定义了和原类相同的方法，那么当给该类的实例发送该方法消息时，是会执行哪个？

由第1部分可知，分类方法都会在本类方法前面，因此调用的是分类的方法(本类的也存在，只不过在后面，代码从开头找方法，找到就返回)。对于多个category之间，build Phase的Compile Sources(后进先出)里的顺序决定了先执行哪个。因此下图代码会输出。


{: .img_middle_lg}
![Category Compile Order](/assets/images/posts/01 Objectiev C/2016-03-17-OC Runtime(三)_Category_Part I_机制/Category Compile Order.png)

{% highlight c linenos %}

//  MyClass.h
@interface MyClass : NSObject
-(void)printName;
@end

//  MyClass.m
#import "MyClass.h"
@implementation MyClass
-(void)printName{
    NSLog(@"%@",@"MyClass");
}
@end


//  MyClass+Category1.h
#import "MyClass.h"
@interface MyClass (Category1)
-(void)printName;
@end

//  MyClass+Category1.m
#import "MyClass+Category1.h"
@implementation MyClass (Category1)
-(void)printName{
    NSLog(@"%@",@"MyClass+Category1");
}
@end


//  MyClass+Category2.h
#import "MyClass.h"
@interface MyClass (Category2)
-(void)printName;
@end

//  MyClass+Category2.m
#import "MyClass+Category2.h"
@implementation MyClass (Category2)
-(void)printName{
    NSLog(@"%@",@"MyClass+Category2");
}
@end


MyClass *myClass = [MyClass new];
NSLog(@"normal category method call ------");
[myClass printName];

unsigned int methodCount = 0;
Method * methodPtr = class_copyMethodList([myClass class], &methodCount);
NSLog(@"iterate category method call ------");
for (unsigned int i = 0; i < methodCount; i++){
    Method method = methodPtr[i];
    SEL methodName = method_getName(method);
    NSString *methodNameString = [NSString stringWithUTF8String:sel_getName(methodName)];
    IMP imp = method_getImplementation(method);
    ((void(*)(id,SEL))imp)(myClass,@selector(printName));
}
/*输出:
 normal category method call ------
 MyClass+Category2
 iterate category method call ------
 MyClass+Category2
 MyClass+Category1
 MyClass
 */
{% endhighlight %}

若编译顺序为`Category2`在`Category1`前，则正常调用会输出``MyClass+Category1``而不是``MyClass+Category2``。

## 3 总结 ##

全文总结参考[该图]({{site.baseurl}}/01%20objective-c/2016/03/12/OC-Runtime(零)_Runtime概述.html#runtime-1)。

## 4 参考资料 ##
- [objc category的秘密](http://blog.sunnyxx.com/2014/03/05/objc_category_secret/);

- [结合 category 工作原理分析 OC2.0 中的 runtime](http://ios.jobbole.com/87623/);




