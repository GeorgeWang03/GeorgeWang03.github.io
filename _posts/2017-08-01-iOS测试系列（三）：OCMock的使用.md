---
layout:     post
title:      "iOS测试系列（三）：OCMock的使用"
date:       2017-07-27 20:11:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - 测试
    - OCMock
---

## 前言

在前面几两篇文章中介绍了iOS测试概述和XCTest的使用，如今的XCTest已经功能很完善，但平时的测试有一些实用的功能XCTest还是没提供，比如Mock、Stub；因此，我们需要借助一些开源框架来帮助我们更好地测试，比如**OCMock、GHUnit、Specta、Expecta、Kiwi**等，这边文章我们主要介绍集Mock、Stub功能一身的**OCMock**，也是测试中人气比较高的一个框架。

OCMock是GitHub上开源作者[Erik Doernenburg](https://github.com/erikdoe)所创作的，能和XCTest很好地配合。

OCMock主要以下面几方面来介绍：

		* **模拟对象（mock object）**
		* **stub方法**
		* **交互验证**
		* **参数约束**
		* **模拟类方法**
		* **部分模拟**
		* **严格模拟与期望**
		* **观察者模拟**

具体内容参考[OCMock文档](http://ocmock.org/reference/#mocking-class-methods)

## 模拟对象（mock object）
模拟对象就是创建一个具有目标功能的实例，分别有类实例模拟、协议模拟、部分模拟、观察者模拟、严格模拟。

**类模拟**：主要是创建一个类实例，可以模拟实例方法及类方法，不管是在当前类还是基类，用法如下：

		id classMock = OCMClassMock([SomeClass class]);
		
classMock相当于一个类实例。

**协议模拟**：协议模拟是创建一个实例，让其具有协议的所定义的功能：

		id protocolMock = OCMProtocolMock(@protocol(SomeProtocol));
		
protocolMock实例具有协议的功能。

**部分模拟**：部分模拟是模拟自一个对象实例，在该模拟实例上stub部分实例方法，没有stub的部分功能照常：

		id partialMock = OCMPartialMock(anObject);
		
**观察者模拟**：顾名思义，创建一个接收通知的实例：

		id observerMock = OCMObserverMock();
		
**严格模拟**：主要是相对类实例模拟和协议模拟来说，他们相对一般的模拟，不同点是严格模拟调用没有stub的方法会直接crash，而不是返回nil：

		id classMock = OCMStrictClassMock([SomeClass class]);
		id protocolMock = OCMStrictProtocolMock(@protocol(SomeProtocol));
		
## stub方法

### 让某个方法返回指定对象或非对象值

		OCMStub([mock someMethod]).andReturn(anObject);
		OCMStub([mock aMethodReturningABoolean]).andReturn(YES);
		
前者是返回指定对象，后者是返回值类型，不过值类型要明确，不能返回不同类型，否则会异常

### 让某个方法交由代理对象实现

		OCMStub([mock someMethod]).andCall(anotherObject, @selector(aDifferentMethod));
		
当`someMethod`调用时，实际调用的是`anotherObject`的`@selector(aDifferentMethod)`，参数和返回值都作用和来自于`aDifferentMethod`方法

### 委托block实现某个方法

		OCMStub([mock someMethod]).andDo(^(NSInvocation *invocation)
    		{ /* block that handles the method invocation */ });
    		
原理和上面的方法代理类似，不过是通过block，而参数和返回值也在`invocation`中处理

### block参数调用

一般的方法参数中，有可能存在block参数，有时我们想模拟回调block，这就需要stub block：

		OCMStub([mock someMethodWithBlock:[OCMArg invokeBlock]]);
		OCMStub([mock someMethodWithBlock:([OCMArg invokeBlockWithArgs:@"First arg", nil])]);
		
block模拟，非对象参数需要转成value类型，参数超过一个的时候整个`[OCMArg invokeBlockWithArgs:...]`表达式要用`()`括号包围起来

### 模拟异常、通知

模拟一个方法一旦调用就抛出异常

		OCMStub([mock someMethod]).andThrow(anException);
		
模拟一个方法一旦调用就发送一个通知

		OCMStub([mock someMethod]).andPost(aNotification);
		
关于stub的操作还可以串联起来，如指定发送通知与指定返回值

		OCMStub([mock someMethod]).andPost(aNotification).andReturn(aValue);
		

### 转发与省略

可以让stub方法实际调用转发到原来的对象实例／类实例：

		OCMStub([mock someMethod]).andForwardToRealObject();
		
也可以不做任何事情，相当于调用没有效果：

		OCMStub([mock someMethod]).andDo(nil);
		
总而言之，方法stub是OCMock一大功能特点，也是XCTest所不具备的

## 交互验证

交互验证主要是验证方法是否调用，同时还能stub

		id mock = OCMClassMock([SomeClass class]);
		OCMStub([mock someMethod]).andReturn(myValue);

		/* run code under test */

		OCMVerify([mock someMethod]);
		
如果`someMethod`没有被调用，那么测试将会失败；同时我们注意到，上面还stub了该方法，这两个操作不矛盾。

## 参数约束
在上面的stub实例中，我们并没有提到参数（除了block），OCMock同样支持参数的约束，也就是只约束传入指定参数的调用

**无限制约束**：

		OCMStub([mock someMethodWithAnArgument:[OCMArg any]])
		OCMStub([mock someMethodWithPointerArgument:[OCMArg anyPointer]])
		OCMStub([mock someMethodWithSelectorArgument:[OCMArg anySelector]])
		
更多类型的无限制约束可以参考`OCMArg.h`，无限制约束，不管传入任何参数，stub都起作用

**忽略非对象约束**：

		[[[mock stub] ignoringNonObjectArgs] someMethodWithIntArgument:0]
		
这种约束表示不管参数中的非对象参数传什么值都能通过，但是对象参数照样可以作限制

**匹配限制**：`OCMArg.h`中提供了多个匹配方法，用于限制参数，如下：

		OCMStub([mock someMethod:aValue)
		OCMStub([mock someMethod:[OCMArg isNil]])
		OCMStub([mock someMethod:[OCMArg isNotNil]])
		OCMStub([mock someMethod:[OCMArg isNotEqual:aValue]])
		OCMStub([mock someMethod:[OCMArg isKindOfClass:[SomeClass class]]])
		OCMStub([mock someMethod:[OCMArg checkWithSelector:aSelector onObject:anObject]])
		OCMStub([mock someMethod:[OCMArg checkWithBlock:^BOOL(id value) { /* return YES if value is ok */ }]])
		
需要注意的是最后一个block判断，调用时参数传给block，我们返回YES或者NO作为判断结果

## 模拟类方法
模拟类方法和上面提到的stub没有多大区别：

		OCMStub([classMock aClassMethod]).andReturn(@"Test string");

不过这里只能stub类方法，不能stub实例方法，如果mock没有释放，那么stub的类方法会一直保留；
如果想停止类模拟，需要调用`[classMock stopMocking]`，这个方法在mock对象释放时会自动调用

至于**交互验证**，上面也提到过，如：`OCMVerify([classMock aClassMethod])`
如果有实例方法和类方法同名，那类方法stub时必须通过`ClassMethod()`指出：

		OCMStub(ClassMethod([classMock ambiguousMethod])).andReturn(@"Test string");

## 部分模拟（Partial mocks）

部分模拟其实也就是对象实例模拟。

		OCMStub([partialMock someMethod]).andReturn(@"Test string");
		
部分模拟通过创建子类实例实现。**交互验证**和**停止模拟**与类模拟格式一致：

		OCMVerify([partialMock someMethod]); // 交互验证
		[partialMock stopMocking]; // 停止模拟
		 
## 严格模拟和期望

**期望-运行-验证**：除了上面提到的单个方法调用验证，OCMock可以批量验证

		id classMock = OCMClassMock([SomeClass class]);
		OCMExpect([classMock someMethodWithArgument:[OCMArg isNotNil]]);

		/* run code under test, which is assumed to call someMethod */

		OCMVerifyAll(classMock)

最后的`OCMVerifyAll`会验证前面的期望是否有效，只要有个一没调用，就会出错

**严格模拟**：严格模拟会让所有没stub的方法的调用都抛出异常

		id classMock = OCMStrictClassMock([SomeClass class]);
		[classMock someMethod]; // this will throw an exception
		
此外，在`OCMExpect`设定期望同时，我们也可以stub：

		OCMExpect([classMock someMethod]).andReturn(@"a string for testing");
		
相当于stub该方法返回指定对象

**延时验证**：该功能用于等待异步操作会比较多

		OCMVerifyAllWithDelay(mock, aDelay);
		
其中`aDelay`为预期最长等待时间

**顺序验证**：

		[mock setExpectationOrderMatters:YES];
		
当我们通过上面调用设置顺序验证开启时，如果方法调用没有按照预设进行，那么会报错

## 观察者模拟

		id observerMock = OCMObserverMock();
		[notificatonCenter addMockObserver:aMock name:SomeNotification object:nil];
		[[mock expect] notificationWithName:SomeNotification object:[OCMArg any]];
		
		OCMVerifyAll(observerMock);
		
观察者验证是严格的，所有的通知必须被期望，如果收到未期望的通知，会报错

## 进阶话题

**快速出错**：通过`OCMReject`来指定某个方法调用会抛出异常，就如同**严格模拟**，只不过后者是所有为stub方法都会抛出异常

		id mock = OCMClassMock([SomeClass class]);
		OCMReject([mock someMethod]);
		
快速出错的异常可能不导致测试失败，因为方法的调用栈终点不在测试方法中，这种情况下，要通过`OCMVerifyAll`来让异常重现

**方法模拟**：也可以用于初始化方法，如：`alloc`、`new`、`copy`等，不过这个不是很好的做法，这并不符合依赖注入原则；另外，`init`方法不能被模拟。

**方法混编**：OC中，方法混编是动态地把某个实例方法实现替换为另外的实现，OCMock中的代理调用就是这个作用：

		id partialMock = OCMPartialMock(anObject);
		OCMStub([partialMock someMethod]).andCall(differentObject, @selector(differentMethod));
		
## 限制

**只能同时模拟一个类实例**：同个时间内，只能存在一个类实例，直至当前类实例释放才能重新模拟，如果不释放，那么在其他测试用例中都还会存在

		id mock1 = OCMClassMock([SomeClass class]);
		OCMStub([mock1 aClassMethod]);
		id mock2 = OCMClassMock([SomeClass class]);
		OCMStub([mock2 anotherClassMethod]);
		
上面这种操作是不行的

**对于同个方法，先stub后expect是不行的**：因为先stub的话，所有的调用都会变成stub，这样子即使过程调用该方法，最后`OCMVerifyAll`验证也会失败；解决的办法是，在`OCMExpect`上顺便stub，比如：`OCMExpect([mock someMethod]).andReturn(@"a string")`，或者将stub置于expect之后

**部分模拟不适用于某些类**：如`NSString`和`NSDate`，这些"toll-free bridged"的类，否则会抛出异常

**某些方法不能stub**：如：`init`、`class`、`methodSignatureForSelector`、`forwardInvocation`这些

**NSString与NSArray的类方法不能stub，否则无效**

**NSObject的方法调用不能验证，除非在子类中重写**

**苹果核心类的私有方法调用不能被验证，如以`_`开头的方法**

**延时验证方法调用不支持，暂时只支持`期望-运行-验证`模式的延时验证**

**OCMock不支持多线程**

## 总结

OCMock对于我们使用XCTest来时，是一个很好的补充，特别是在Mock、Stub方面，但也会有局限，特别是不支持多线程的、以及类实例模拟只能有一个，这些要求我们使用的时候要小心，否则会埋下隐患。不过，一般的测试，OCMock依然是一个好帮手。