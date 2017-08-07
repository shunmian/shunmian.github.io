---
layout: post
title: EOC(四)：Protocols and Categories
categories: [01 Objective-C]
tags: [Effective]
number: [01.25.1]
fullview: false
shortinfo: 本文是《Effective Objective-C》的系列笔记的第四篇《Protocols and Categories》，对应书本的第四章。
---
目录
{:.article_content_title}


* TOC
{:toc}

---
{:.hr-short-left}

## 1 Protocols and Categories ##

### Item 23：Use Delegate and Data Source Protocols for Interobject Communication ###

1. Use `Data Source` to provide data to `client object`;
2. Use `Delegate` to respond to `client object`'s event;
3. Including `client object` parameter in protocol methods so that one `Data Source` or `Delegate` can responds to multiple `client object`

{% highlight objc linenos %}
-(void)networkFecher:(EOCNetworkFecther *)fecher
      didReceiveData:(NSData*)data{
    if(fetcher == self.myFetcherA){
        //Handle data
    }else{
        //Handle data
    }
}

{% endhighlight %}

### Item 24: Use Categories to Break Class Implementations into Manageable Segments ###

1. Use categories to split a class implementation into more manageable fragments.

2. Create a category called ``Private`` to hide implementation detail of methods that should be considered as private.

### Item 25: Always Prefix Category Names on Third-Party Classes ###

1. Always prepend your naming prefix to the names of categories you add to classes that are not your own.

2. Always prepend your naming prefix to the method names within categories you add to classes that are not your own. However prepend makes code autocompletion difficult since what in your mind is the method name after prefix. The suggestied solution was to use suffix `method_ABC` insted of prefix `ABC_method` Nowadays the Xcode autocompletion is more advanced to return matching even the keyword you typed in the middle of the method. As a result, stick to prepend your naming prefix to the method names within categories.

{% highlight objc linenos %}
@interface NSString(ABC_HTTP)

-(NSString n*)ABC_urlEncodedString;
{% endhighlight %}

### Item 26: Avoid Properties in Categories ###

1. Keep all property declarations for encapsulated data in the main interface definition.

2. Prefer accessor methods to property declarations in categories, unless it is a class-continuation category.

### Item 27: User the Class-Continuation Category to Hide Implementation Detail 

1. Use the class-continuation category to add private properties, private methods, private protocols;

2. Redeclare properties in the class-continuation category as read-write if they are read-only in the main inteface, if the setter accessor is requred internally within the class.


### Item 28: Use a Protocol to Provide Anonymous Objects ###

1. Protocols can be used to provide some level of anonymity to types. The type can be reduced to an ``id`` type that implements a protocol's methods.

2. Use anonymous objects when the type (class name) should be hidden.

3. Use anonymous objects when the type is irrelevant, and the fact that the object responds to certain methods (the ones defined in the protocol) is more important. 

## 2 Reference ##

- [《Effective Objective-C 2.0: 52 Specific Ways to Improve Your iOS and OS X Programs (Effective Software Development Series)》](https://www.amazon.com/Effective-Objective-C-2-0-Specific-Development/dp/0321917014);


