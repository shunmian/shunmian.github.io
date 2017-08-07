---
layout: post
title: EOC(三)：Interface and API Design
categories: [01 Objective-C]
tags: [Effective]
number: [01.25.1]
fullview: false
shortinfo: 本文是《Effective Objective-C》的系列笔记的第三篇《Interface and API Design》，对应书本的第三章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Interface and API Design ##

### Item 15：Use Prefix Names to Avoid Namespace Clashes ###

Since ``Objective-C`` dosen't have namespace like other language such as ``Java's Package`` and ``Python's Module``，Prefix is the solution that minimize name clash possibility.

1. Choose 3 letter prefix(from your company & application) for your class. Apple reserve the 2 letter prefix for its future use.

2. If your own library use a thrid-party library, be sure to prefix the thrid-party library as well. For example, your library has ``LALClient``, if the thrid-party library has prefixed class as ``XYZData``, you'd better to **reprefix** ``XYZData`` as ``LALXYZData``. By doing so, if other people install your library and also ``XYZData``, there would be no clash; it also allow other people to install a different version of ``XYZData``。This convention is common such as in [``RestKit``](https://github.com/RestKit/RestKit), which use ``AFNetworking``，it **reprefix** it as ``AFRK...``.

### Item 16：Have a Designated Initializer ###

{: .img_middle_hg}
![Designated Initializer](/assets/images/posts/01 Objectiev C/2016-04-22-EOC(三)：Interface and API Design/Designated Initializer.png)

### Item 17：Implement the description Method ###

1. Implement description method to provide a meaningful string description of instances

2. If the object description could do with more detail for use during debugging (``po`` command for ``LLDB``), implement ``debugDescription``(default to ``description``).

{% highlight objc linenos %}
//use dictionary format for description provides compact visualization of object information.
-(NSString *)description{
    return [NSString stringWithFormat:@"<%@:%p,%@>",
        [self class],
        self,
        @{@"title": _title,
          @"labitude":_labitude,
          @"longitude": _longitude
    }
}
{% endhighlight %}

### Item 18：Prefer Immutable Objects ###

1. When possible, create objects that are immutable (readonly property in ``.h``);

2. Extend read-only properties in a class-extention to read-write if the property will be set internally.

3. Provide methods to mutate collections held by objects rather than exposing a mutable collection as a property.

### Item 19：Use Clear and Consistent Naming ###

The conciseness comes at the cost of ambiguity. While Objective-C reach unambiguity via verbosity. The Objective-C code reads like sentences with minimal ambiguity. This is what called **self-documenting(自文档)**, code itself is enough to convery its functionality at most time.

1. Ensure method names are concise but precise to make them read from left to right as a sentence.

2. Avoid using abbreviations of types in method names.

3. Most important, make sure that method names are consistent with your own code or that with which it is being integrated.

### Item 20：Prefix Private Method Names ###

1. Prefix priavate method names so that they are easily distinguished from public methods.

2. Avoid using a single underscroe as the method prefix, since this is reserved by Apple.

A better choice is to use double underscore ``__someMethod`` instead of ``p_someMethod``.

### Item 21：Understand the Objective-C Error Model ###

1. Use exceptions only for tatal errors that should bring down the entire application, such as sending message to abstract class.

2. For nonfatal errors, either provide a delegate method to handle errors or offer an out-parameter ``NSError`` object.

{% highlight objc linenos %}
//out-parameter NSError object
-(BOOL)doSomething:(NSError **)error{
    //some implementation
}

NSError *error = nil;

BOOL ret = [object doSomething:&error];
if(error){
    // There was an error
}


{% endhighlight %}

### Item 22：Understand the NSCopying Protocol ###


**To Be Continued**.



## 2 Reference ##

- [《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs (Effective Software Development Series)》](https://www.amazon.com/Effective-Objective-C-2-0-Specific-Development/dp/0321917014);

