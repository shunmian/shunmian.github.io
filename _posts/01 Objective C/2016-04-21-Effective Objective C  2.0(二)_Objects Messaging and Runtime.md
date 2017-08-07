---
layout: post
title: EOC(二)：OC Runtime (Object Hierarchy & Messaging)
categories: [01 Objective-C]
tags: [Effective]
number: [01.25.1]
fullview: false
shortinfo: 本文是《Effective Objective-C》的系列笔记的第二篇《OC Runtime (Object Hierarchy & Messaging)》，对应书本的第二章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Object Hierarchy & Messaging ##

### Item 6：Understand Properties

1. @property = _iVar + setter + getter。To prevent @property automatically create _iVar + setter + getter，use @dynamic；To customize your own iVar's name (not _iVar)，use @synthesis。

2. @property attribues：atomicity(nonatomic，atomic)，read/write(readwrite，readonly)，memeory-management(assing，strong，weak，copy)，method name(getter，setter)。

{% highlight objc linenos %}
//  EOCCat.h
#import <Foundation/Foundation.h>

@interface EOCCat : NSObject
@property(nonatomic,strong) NSString *name0;
@property(nonatomic,strong) NSString *name1;
@property(nonatomic,strong) NSString *name2;

@property(nonatomic,strong,readonly) NSString *nameReadOnly;
@property(nonatomic,copy) NSString *nameCopy;
@property(nonatomic,assign,getter=isMale)BOOL male;
-(instancetype)initWithNameCopy:(NSString *)nameCopy;
-(void)changeNameCopy:(NSString *)nameCopy;
@end


//  EOCCat.m
#import "EOCCat.h"

@interface EOCCat()
@property(nonatomic, strong, readwrite) NSString *nameReadOnly;
@end

@implementation EOCCat
@synthesize name1 =_myName1;
@dynamic name2;

-(instancetype)initWithNameCopy:(NSString *)nameCopy{
    if(self = [super init]){
        _name0 = @"name0";
        _myName1 = @"myName1";
        //_name2 error, since dynamic forbidden its _iVar+setter+getter auto create
        //in init, one cannot use setter, thus [nameCopy copy] is used to conform to _nameCopy's copy attributes
        _nameCopy = [nameCopy copy];
        _nameReadOnly = @"readOnly1";
    }
    return self;
}

-(void)changeNameCopy:(NSString *)nameCopy{
    //nameCopy will be copied to self.nameCopy
    self.nameCopy = nameCopy;
    //self.nameReadOnly can only be changed once it redeclared as readwrite in .m
    self.nameReadOnly = @"readOnyly2";
}
{% endhighlight %}

### Item 7：Access Instance Variables Primarily Directly When Accessing Them Internally

1. For external access，property usage is without doubt。

2. For internal access，use property except in init，setter，getter，dealloc。 Since property encapsulate setter(customization)，getter(lazy instantiate)， memory-managment(_iVar = someValue not retain new value and release old value while self.iVar do) and copy (_iVar = someValue don't copy it while self.iVar = someValue do if @property with copy)。

### Item 8：Understand Object Equality

1. Object euqality必须实现下面两个方法：用 ``isEuqal``(用于两者比较)；``hash``用于collection内object的比较，例如set。前者相同的后者必须相同，后者相同的前者不用相同。两者都属于``NSObject``协议。

2. 对于创建的Class，可以写``isEqualToClass``方法，例如``NSString``有``isEqualToString``，``NSArray``有``isEuqalToArray``，``NSDictionary``有``isEuqalToDictionary``。可以用``isEqualToClass``(不需要check类是否相同)来实现``isEqual``方法(需要check类是否相同，不同返回false)。


{% highlight objc linenos %}
-(BOOL)isEqualToPerson:(EOCPerson *)other{
    if(self == other) return YES; //if points to the same instance, return YES
    
    if(![self.firstName isEqualToString:other.firstName]){
        return NO;
    }
    if(![self.lastName isEqualToString:other.lastName]){
        return NO;
    }
    return YES;
}

-(BOOL)isEqual:(id)object{
    if([object isKindOfClass:[self class]]){
        return [self isEqualToPerson:object];
    }else{
        return [super isEqual:object];
    }
}

-(NSUInteger)hash{
    return [self.firstName hash]^[self.lastName hash];
}

{% endhighlight %}

### Item 9：Use the Class Cluster Pattern to Hide Implementation Detail

1. 类簇可以减少类的种类，子类信息被类簇隐藏，调用者不需要知道子类。比如NSSnumber是一个类簇，如果不使用类簇的话，需要为每一种基本类型都提供一种封装，这样会使类的数量极其庞大，程序员需要记住大量的类，但是如果使用类簇的话，只需要记住NSNumber这一个类就行了，具体使用这个类返回的是什么类型的对象，会在NSNumber类内部做处理和判断，而不需要调用者来关心，调用者只需要知道NSNumber的接口拿来用即可。

2. Subclass abstract class在类簇里和Subclass normal class是完全不一样的。后者可以是一个空的实现，因为super class已经提供了完整实现；而前者需要实现super class接口的函数，因为这部分在super class里只是一个空壳。


{% highlight objc linenos %}
//EOCMployee.h
#import <Foundation/Foundation.h>

typedef NS_ENUM(NSUInteger,EOCEmployeeType){
    EOCEmployeeTypeDeveloper,
    EOCEmployeeTypeDesigner,
    EOCEmployeeTypeFinance,
};

@interface EOCEmployee : NSObject
+(EOCEmployee *)employeeWithType:(EOCEmployeeType)type;
-(void)doWork;
@end



//EOCEmployee.m
#import "EOCEmployee.h"

@interface EOCEmployeeDeveloper : EOCEmployee
@end
@implementation EOCEmployeeDeveloper
-(void)doWork{
    NSLog(@"%@,%@",[self class],NSStringFromSelector(_cmd));
}
@end

@interface EOCEmployeeDesigner : EOCEmployee
@end
@implementation EOCEmployeeDesigner
-(void)doWork{
    NSLog(@"%@,%@",[self class],NSStringFromSelector(_cmd));
}
@end

@interface EOCEmployeeFinance : EOCEmployee
@end
@implementation EOCEmployeeFinance
-(void)doWork{
    NSLog(@"%@,%@",[self class],NSStringFromSelector(_cmd));
}
@end

@interface EOCEmployee()
@property(nonatomic,assign)EOCEmployeeType type;
@end

@implementation EOCEmployee

+(EOCEmployee *)employeeWithType:(EOCEmployeeType)type{
    EOCEmployee *employee;
    switch (type) {
        case EOCEmployeeTypeDeveloper:
            employee = [EOCEmployeeDeveloper new];
            break;
        case EOCEmployeeTypeDesigner:
            employee = [EOCEmployeeDesigner new];
            
        case EOCEmployeeTypeFinance:
            employee = [EOCEmployeeFinance new];
    }
    employee.type = type;
    return employee;
}

-(void)doWork{
    //Subclass implement this.
    //Without class cluster, you need to implement with lots of cumbersome if/else
    if(self.type == EOCEmployeeTypeDeveloper){
        //do developer work
    }else if (self.type == EOCEmployeeTypeDesigner){
        //do designer work
    }else if (self.type == EOCEmployeeTypeFinance){
        //do finance work
    }
}
@end
{% endhighlight %}

### Item 10：Use Associated Objects to Attach Custom Data to Existing Classes

1.对已有的类进行实例变量扩展可以通过category结合runtime实现，具体有3个方法：``objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy policy)``，``objc_getAssociatedObject(id object, void *key)``，``void objc_removeAssociatedObjects(id object)``。

{% highlight objc linenos %}
//EOCPerson+ID.h
#import "EOCPerson.h"

@interface EOCPerson (ID)
@property(nonatomic, strong) NSString *id;
@end

//EOCPerson+ID.m
#import "EOCPerson+ID.h"
#import <objc/runtime.h>
@implementation EOCPerson (ID)

-(void)setId:(NSString *)id{
    objc_setAssociatedObject(self,@selector(id),id,OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

-(NSString *)id{
    return objc_getAssociatedObject(self,@selector(id));
}
@end
{% endhighlight %}

### Item 11：Understand the Role of objc_msgSend

见[OC Runtime(零)：Runtime概述]({{site.baseurl}}/01%20objective-c/2016/03/12/OC-Runtime(零)_Runtime概述.html#runtime-1)和[OC Runtime(二)：Messaging Part I：消息机制和swizzling]({{site.baseurl}}/01%20objective-c/2016/03/15/OC-Runtime(二)_Messaging_Part-I_消息机制和swizzling.html)。

### Item 12：Understand Message Forwarding

见[OC Runtime(零)：Runtime概述]({{site.baseurl}}/01%20objective-c/2016/03/12/OC-Runtime(零)_Runtime概述.html#runtime-1)和[OC Runtime(二)：Messaging Part I：消息机制和swizzling]({{site.baseurl}}/01%20objective-c/2016/03/15/OC-Runtime(二)_Messaging_Part-I_消息机制和swizzling.html)。

### Item 13：Consider Method Swizzling to Debug Opaque Methods

见[OC Runtime(零)：Runtime概述]({{site.baseurl}}/01%20objective-c/2016/03/12/OC-Runtime(零)_Runtime概述.html#runtime-1)和[OC Runtime(二)：Messaging Part I：消息机制和swizzling]({{site.baseurl}}/01%20objective-c/2016/03/15/OC-Runtime(二)_Messaging_Part-I_消息机制和swizzling.html)。

### Item 14：Understand What a Class Object Is

见[OC Runtime(零)：Runtime概述]({{site.baseurl}}/01%20objective-c/2016/03/12/OC-Runtime(零)_Runtime概述.html#runtime-1)和[OC Runtime(一)：Object Model Part I：Object Hierarchy]({{site.baseurl}}/01%20objective-c/2016/03/13/OC-Runtime(%E4%B8%80)_Object-Model-Part-I_Object-Hierarchy.html)。


## 2 Reference ##


- [《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs (Effective Software Development Series)》](https://www.amazon.com/Effective-Objective-C-2-0-Specific-Development/dp/0321917014);

