---
layout: post
title: EOC(六)：Blocks and GCD
categories: [01 Objective-C]
tags: [Effective]
number: [01.25.1]
fullview: false
shortinfo: 本文是《Effective Objective-C》的系列笔记的第六篇《Blocks and GCD》，对应书本的第六章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Blocks and GCD ##

### Item 37：Understand Blocks ###

1. Blocks are lexical closures for C, C++, and Objective-C

2. Blocks can optionally take parameters and optionally return values.

3. Blocks can be stack allocated, heap allocated, or global. A stack-allocated block can be copied onto the heap, at which point it is reference counted just like standard Objective-C objects.

### Item 38：Create typedefs for Common Block Types ###

1. Use type definitions to make it easier to use block variables.

2. Always follow the naming conventions when defining new types such that you do not clash with other types.

3. Don't be afraid to define multiple types for the same block signature. You may want to refactor one  place that uses a certain block type by changing the block signature but not another.



### Item 39：Use Handler Blocks to Reduce Code Separation ###

1. Handler blocks have the benefit of being associated with an object directly rather than delegation, which often requires switching based on the object if multiple instances are being observed.

2. When designing an API that uses handler blocks, consider passing a queue as a parameter, to designate the queue on which the block should be enqueued.

### Item 40：Avoid Retain Cycles Introduced by Blocks Referencing the Object Owning Them ###

1. Be aware of the potential problem of retain cycles introduced by blocks that capture objects that directly or indirectly retain the block.

2. Ensure that retain cycles are broken at an opportune moment, but never leave responsibility to the consumer of your API.


### Item 41：Prefer Dispatch Queues to Locks for Synchronization ###


1. Dispatch queues can be used to provide synchronization semantics and offer a simpler alternative to @symchronized blocks or NSLock objects.


2. Mixing synchronous and asynchronous dispatches can provide the same syncrhonized behavior as with normal blocking but withou blocking the calling thread in the asynchronous dispatches.


3. Conurrent queues and barrier blocks can be used to make synchronized behavior more efficient, such as **Reader vs Writer Problem** , [see more]({{site.baseurl}}/01%20objective-c/2016/04/03/OC-Concurrency(二)_GCD-part-I_用法详解.html#using-dispatch-barriers) 



{% highlight objc linenos %}
_syncQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

-(NSString*)someString {
	__block NSString *localSomeString; 
	dispatch_sync(_syncQueue, ^{
		localSomeString = _someString; });
	return localSomeString; 
}

-(void)setSomeString:(NSString*)someString { 	
	dispatch_barrier_async(_syncQueue, ^{
		_someString = someString; });
}

{% endhighlight %}


### Item 42：Prefer GCD to performSelector and Friends ###

1. The performSelector family of methods is potentially dangerous with respect to memory management. If it has no way of determining what selector is going to be performed, the ARC compiler cannot insert the appropriate memory-management calls.

2. The family of methods is very limited with respect to the return type and the number of parameters that can be sent to the method.

3. The methods that allow performing a selector on a different thread are better replaced with certain GCD calls using blocks.

{% highlight objc linenos %}
// Using performSelector:withObject:afterDelay:
[self performSelector:@selector(doSomething) withObject:nil afterDelay:5.0];

// Prefer using dispatch_after
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC)); 
dispatch_after(time, dispatch_get_main_queue(), ^(void){
	[self doSomething]; 
});

{% endhighlight %}

### Item 43：Know When to Use GCD and When to Use Operation Queues ###

1. Dispatch queues are not the only solution to multithreading and task management.

2. Operation queues provide a high-level, Objective-C API that can do most of what plain GCD do. Operation can do extra operation such as **Cancelling perations**,**Operation dependencies**,**Key-Value Observing of operation properties**,**Operation priorities**,**Reuse of operations**.

### Item 44：Use Dispatch Groups to Take Advantage of Platform Scaling ###

1. Dispatch groups are used to group a set of tasks. You can optionally be notified when the group finishes executing.

2. Dispatch groups can be used to execute multiple tasks concurrently through a concurrent dispatch queue. In this case, GCD handles the scheduling of multiple tasks at the same time, based on system resources. Writing this yourself would require a lot of code. [See more]({{site.baseurl}}/01%20objective-c/2016/04/03/OC-Concurrency(二)_GCD-part-I_用法详解.html#using-dispatch-barriers).

### Item 45：Use dispatch_once for Thread-Safe Single-Time Code Execution ###

1. Thread-safe single-code execution is a common task. GCD provides an easy-to-use tool for this with the dispatch_once funtion.

2. The token should be declared in static or global scope such that it is the same token being passed in for each block that should executed once.

### Item 46：Avoid dispatch_get_current_queue ###

1. The ``dispatch_get_current_queue`` function does not in general perform how you would expect. It has been deprecated and should now be used only for debugging.

2. Queue-specific data can be used to solve the usual reason for using ``dispatch_get_current_queueu``, which is to avoid dealocks owing to nonreentrant code.



## 2 Reference ##

- [《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs (Effective Software Development Series)》](https://www.amazon.com/Effective-Objective-C-2-0-Specific-Development/dp/0321917014);

