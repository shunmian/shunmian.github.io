---
layout: post
title: EOC(五)：Memory Management
categories: [01 Objective-C]
tags: [Effective]
number: [01.25.1]
fullview: false
shortinfo: 本文是《Effective Objective-C》的系列笔记的第五篇《Memory Management》，对应书本的第五章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Memory Management ##

### Item 29：Understand Reference Counting ###

1. Reference-counting memory management is based on a counter that is incremented and decremented. An object is created with a count of at least 1. An object with a positive retain count is alive. When the retain count drops to 0, the object is destroyed.

2. As it goes through its life cycle, an object is retained and released by other objects holding references to it. Retaining and releasing increments and decrements the retain count respectively.

### Item 30：User ARC to Make Reference Counting Easier ###

<ol>

<li> Method-Naming Rules Applied by ARC. <code>alloc</code>,<code>new</code>,<code>copy</code>,<code>mutableCopy</code> methods return an object owned by the caller method. 

{% highlight objc linenos %}

+ (EOCPerson*)newPerson {
    EOCPerson *person = [[EOCPerson alloc] init]; return person;
    /**
    * The method name begins with 'new', and since 'person' * already has an unbalanced +1 retain count from the
    * 'alloc', no retains, releases, or autoreleases are
    * required when returning.
    */
}
+ (EOCPerson*)somePerson {
    EOCPerson *person = [[EOCPerson alloc] init]; return person;
    /**
    * The method name does not begin with one of the "owning"
    * prefixes, therefore ARC will add an autorelease when
    * returning 'person'.
    * The equivalent manual reference counting statement is:
    *
    */
}

- (void)doSomething {
    EOCPerson *personOne = [EOCPerson newPerson]; 
    // ...

    EOCPerson *personTwo = [EOCPerson somePerson]; 
    // ...

    /*
    * At this point, 'personOne' and 'personTwo' go out of scope, therefore ARC needs to clean them up as required. 
    * - 'personOne' was returned as owned by this block of code, so it needs to be released.
    * - 'personTwo' was returned not owned by this block of code, so it does not need to be released. 
    * The equivalent manual reference counting cleanup code is:
    * [personOne release];
    */
}

{% endhighlight %}
</li>


<li> Memory-Management Semantics of Variables. <code>_strong</code>,<code>_unsafe_unretained</code>,<code>__weak</code>,<code>_autoreleasing</code>.
</li>

<li> ARC Handling of Instance Variables. ARC also handles the memory management of instance variables. Doing so requires ARC to automatically generate the required cleanup code during deallocation. 

{% highlight objc linenos %}
-(void) dealloc{
    [_foo release];
    [_bar release];
    [super dealloc];
}

{% endhighlight %}

</li>

<li> Overriding the Memory-Management Methods  </li>

</ol>

### Item 31：Release References and Clean Up Observation State Only in dealloc ###

1. The ``dealloc`` method should be used only to release references to other objects and to unregister anything that needs to be, such as **Key-Value Observing(KVO)** or ``NSNotificationCenter`` notifications.

2. If an object holds onto system resources, such as file descriptors, there should be a method for releasing these resources. It should be the contract with the consumer of such a class to call this `close` method when finished using the resources.

3. Method calls should be avoided in ``dealloc`` methods in case those methods try to perform asynchronous work or end up assuming that the object is in a normal state, which it won't be.

### Item 32：Beware of Memory Management with Exception-Safe Code ###

1. When exceptions are caught, care should be taken to ensure that any required cleanup is done for objects created within the try block.

2. By default, ARC does not emit code that handles cleanup when exceptions are thrown. This can be enabled with a compiler flag but produces code that is larger and comes with a runtime cost.

### Item 33：Use Weak References to Avoid Retain Cycles ###

1. Retain cycles can be avoided by making certain references weak.

2. Weak references may or may not be autonilling. Autonilling is a new feature introduced with ARC and is implemented in the runtime. Autonilling weak references are always safe to read, as they will never contain a reference to a deallocated object.

### Item 34：Use Autorelease Pool Blocks to Reduce High-Memory Waterline ###

<ol>

<li> Autoreleas pools are arranged in a stack, with an object being added to the topmost pool when it is sent the autorelease message. </li>

<li> Correct application of autorelease pools can help reduce the high-memory waterline of an application.
{% highlight objc linenos %}
NSArray *databaseRecords = /* ... */; 
NSMutableArray *people = [NSMutableArray new]; 
for (NSDictionary *record in databaseRecords) {
    @autoreleasepool { 
        EOCPerson *person =[[EOCPerson alloc] initWithRecord:record]; 
        [people addObject:person];
    } 
}
{% endhighlight %}
</li>

<li> Modern autorelease pools using the new <code>@autoreleasepool {} </code> syntax are cheap. </li>

</ol>

### Item 35：Use Zombies to Help Debug Memory-Management Problems ###

1. When an object is deallocated, it can optionally be turned into a zombie instead of being deallocated. This feature is turned on by using the envrionment flag NSZombieEnabled.

2. An object is turend into a zombie by manipulating its isa pointer to change the object's class to a special zombie class. A zombie class responds to all selectors by aborting the application after printing a message to indicate what message was sent to what object.

### Item 36：Avoid Using retainCount ###

1. The retain count of an object might seem useful but usually is not, because the absolute retain count at any given time does not give a complete picture of an object's lifetime.

2. When ARC came along, the ``retainCount`` method was deprecated, and using it causes a compiler error to be emitted.

## 2 Reference ##

- [《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs (Effective Software Development Series)》](https://www.amazon.com/Effective-Objective-C-2-0-Specific-Development/dp/0321917014);

