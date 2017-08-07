---
layout: post
title: EOC(七)：The System Framework
categories: [01 Objective-C]
tags: [Effective]
number: [01.25.1]
fullview: false
shortinfo: 本文是《Effective Objective-C》的系列笔记的第七篇《The System Framework》，对应书本的第七章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 The System Framework ##




### Item 47：Familiarize Yourself with the System Frameworks ###

{: .img_middle_hg}
![iOS Framework Overview](/assets/images/posts/01 Objectiev C/2016-04-26-EOC(七)：The System Framework/iOS Framework Overview.png)

### Item 48：Prefer Block Enumeration to  for loops ###

1. Enumerating collections can be achieved in four ways. The for loop is the most basic, followed by enumeration using ``NSEnumerator`` and fast enumeration. The most modern and advanced way is using the block-enumeration methods.

2. Block enumeration alows you to perform the enumeration **concurrently**, withou any additional code, by making use of GCD. This cannot easily be achieved with the other enumeration techniques.

3. Alter the block signature to indicate the precise types of objects if you know them.

### Item 49：Use Toll-Free Bridging for Collections with Custom Memory-Management Semantics ###

1. Toll-free bridging allows you to cast between Foundation's Objective-C objects and CoreFoundation's C data structures.

2. Dropping down to CoreFoundation to creat a collection allows you to specify various callbacks that are used when the colleciton handles its contents. Casting this through toll-free bridging allows you to end up with an Objective-C collection that has custom memory-management semantics.

### Item 50：Use NSCache Instead of NSDictionary for Caches ###

1. Consider using ``NSCache`` in the place of ``NSDictionary`` objects being used as caches. Caches provide optimal pruning behavior, thread safety, and don't copy keys, unlike a dictionary.

2. Use the count limite and cost limits to manipulate the metrics that define when objects are pruned from the cache. But never rely on those metrics to be hard limits; they are purely guidance for the cache.

3.Use ``NSPurgeableData`` objects with a cache to provide autopurging data that is also automatically removed from the cache when purged.

4. Caches will make your applications more responsive if used correctly. Cache only data that is expensive to recalculate, such as data that needs to be fetched fromt he network or read from disk.

### Item 51：Keep initialize and load Implementations Lean ###

1. Classes go through a load phase in which they have the ``load`` method called on them if it has been implemented. This method may also be present in categories whereby the class load always happens before the category load. Unlike other mehtods, the ``load`` method does not participate in overriding.

2. Before a class is used for the first time, it is sent the ``initialize`` method. This method does participate in overriding, so it is usually best tocheck which class is being initialized.

3. Both implementations of ``load`` and ``initialize`` should be kept lean, which helps to keep applications responsive and reduces the likelihood that interdependency cycles will be introduced.

4. Keep ``initialize`` methods for setting up global state that cannot be done at compile time.

### Item 52：Remember that NSTimer Retains Its Target ###

1. An ``NSTimer`` object retains its target until the time is invalidated either because it fires or through an explicit call to invalidate.

2. Retain cycles are easy to introduce through the use of repeating timers and do so if the target of a timer retains the timer. This may happen directly or indirectly through other objects in the object graph.

3. An extension to ``NSTimer`` to use blocks can be used to break the retain cycle. Until this is made part of the public ``NSTimer`` interface, the functionality must be added through a category.




## 2 Reference ##

- [《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs (Effective Software Development Series)》](https://www.amazon.com/Effective-Objective-C-2-0-Specific-Development/dp/0321917014);

