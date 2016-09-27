---
layout:     post
title:      "Objective-C 内存布局与对象生成"
date:       2015-11-17 9:04:18
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - iOS
    - Objective-C
    - 内存
---

一直以来零碎的，连续的看过一些关于OC的内存布局及对象生成方案的文章，没对这方面的知识进行一个总结是心头大患 😔😔。

接下来在这篇文章中，我将会介绍这方面的相关知识，只要内容有：

**1.Object-C的类对象内存布局**

**2.Object-C对象如何生成**

废话不多说，我们开始吧！🎁🍺👇👇

## 一、Object-C的类对象内存布局
 	我们知道Object-C是又前端编译器clang转换为C++/C进而编译为机器码的；我们还知道Object-C中的类是单继承的；
	对于Object-C的一个类对象实例，或许你曾听说它有一个isa指针,这个isa有什么作用呢？
	通过下面一张图可能你会发现一些端倪 
	![oc_memory_struct](/img/post_img/2015-11-17-objectivec/oc_memory_struct.jpg)  	是的，类对象中除了isa指针还有superclass指针。
 isa顾名思义，is a class。。。来指明这是一个什么类的对象。
	根据上图，isa指针指向一个metaClass元类对象，这个对象是全局的，而且只有一个，metaClass存放类的静态变量和类方法，NSObject类对象的isa指向自身，而superClass指针则指向父类对象，NSObject类对象的superClass指针指向nil。

当然，类对象和元类对象都拥有这两个指针，只是有所不同，从图中我们同样可以看到metaClass的isa指针指向的是NSObject元类对象，NSObject元类对象的isa指针则是指向自身；而其superClass则都指向父类的元类对象。

当然，类中的信息不仅仅有isa和superClass指针，从runtime文件中我们可以找到Class的结构定义如下：

	struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
    #if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
    #endif
	} OBJC2_UNAVAILABLE;

其中包括**ivars**、**methodLists**和**protocol**等重要的信息。

* `objc_ivar_list`是类对象所持有的对象实例，最常见的就是平时声明**@property**时，用下划线表示的_someProperty变量；
* `objc_method_list`是存放所有实例方法的列表，调用实例对象的方法时，会在此列表中遍历寻找对应的函数入口；
* `objc_protocol_list`存放该类所遵循的协议；
* 值得注意的是，``objc_cache``，这是什么东西？其实这是objc对方法访问做优化的一个缓存，由于方法列表有可能很大，如果每次都在列表里面遍历，那效率自然就下降，特别是对于一些经常访问的方法，如tableView的`tableView:cellForRowsAtIndexPath:`，高频率的访问带来的高频率遍历会耗费很多时间，而有了cache之后，在其中访问高频方法，速度会提升不少。
* 细心的读者会主要到图中还有一个`objc_object`，这是一个什么东西呢？  这其实就是我们平常所声明的实例对象，其通过clang转换为以下结构：

		struct objc_object {
    	Class isa;
		};

* 其中只有一个isa类对象，就是指向对应的类对象内存地址，如上图所示。 既然提到oc的对象指针，那么我们自然会想到oc中的特殊指针id。 在runtime文件中我们同样可以看到id的定义：

	typedef struct objc_object *id;
 这也就不难理解为什么id指针不用 * 而其他的要有 * 。
 所以我们平常所提到的**isa**有三种含义，第一是**实例对象的结构体中的isa**对象，第二是**类中的isa**，其指向**metaClass**类对象，第三是**元类中的isa**,其指向NSObject元类对象。
	平常我们调用对象的变量，指针都通过第一类isa也就是实例对象中的isa去寻找相关的内容。

## 二、Object-C对象如何生成
有了上面的描述之后，相信大家对于objc中的类已经有一定的理解，甚至对其如何生成有一定的概念了。

那么，一个Object-C对象在运行时如何被如何创建呢？

可以这么说，当我们`alloc/init`一个对象时，runtime会创建oc独有的objc_object结构体对象，在创建结构体时会创建其中的isa，然后根据类的相关继承定义，由上往下，由父类往子类去创建相关的类对象，并把其中每个类对象的isa指向全局的meta元类对象。

`alloc/init`如何执行？

一般我们在oc中创建一个对象的方式是`[[Object alloc] init]`,这相比起C++的new方法，多了一层函数调用。alloc会先在内存中申请对象的内存空间,init作为构造函数把相关的初始化值赋给对象，对于如NSString,NSDictionary等Foundation框架的对象，由于未能确定其数据长度，在alloc时会先指向一个placeHolder对象，当init的时候再申请相关的内存地址，其中的原因关乎内存字节对齐，由于这部分超出了本文的讨论范围，所以不讲，详情移步google。


## 三、总结
Objective-C基于C，其类对象在C语言层面是结构体，由clang转换而成，要探究Objective-C的内存布局，必须从C语言的结构体层面入手。经过分析，我们了解到其大体结构，主要的内容由isa、ivar、method_list等，并且由isa和superclass串联起整个布局。Objective-C对象生成时，实质是生成C的结构体实例，并对串联的isa、superclass指针做初始化，起到布局作用，在oc语言层面的alloc/init使得对象的内存分配和初始化分开，过程更加明确。总而言之，了解和掌握Objective-C的内存布局及对象的创建过程，有助于平时开发和调试，也是提升开发能力的一个要求。

	
