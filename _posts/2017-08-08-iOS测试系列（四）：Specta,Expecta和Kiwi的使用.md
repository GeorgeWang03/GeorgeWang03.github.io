---
layout:     post
title:      "iOS测试系列（四）：Specta,Expecta和Kiwi的使用"
date:       2017-07-27 20:11:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - 测试
    - Specta
    - Expecta
    - Kiwi
---

## 前言

前面文章相继介绍了XCTest和OCMock的使用，本篇文章继续介绍其他相关的三个BDD测试框架：[Specta](https://github.com/specta/specta)、[Expecta](https://github.com/specta/expecta) 和 [Kiwi](https://github.com/kiwi-bdd/Kiwi/)，这三个框架都是出自Github作者[Orta](https://github.com/orta)之手，他最出名的开源框架莫过于Cocoapods了。

### Specta

Specta是一个类似于Ruby测试框架RSpec的轻量级BBD测试框架，其为[DSL(Domain-Specific Language)](https://en.wikipedia.org/wiki/Domain-specific_language)模式，让测试更加接近于自然语言描述，更加易懂。他有以下特点：

* 容易集成到项目中 
* 基于XCTest编写，可以很好地与XCTest配合使用

**Specta一般形式如下：**

		// ATestClass.m
		#import <Specta/Specta.h> // 导入头文件，如果是静态库使用 #import "Specta.h"

		SharedExamplesBegin(MySharedExamples)
		// Global shared examples are shared across all spec files.

		sharedExamplesFor(@"foo", ^(NSDictionary *data) {
    		__block id bar = nil;
    		beforeEach(^{
        		bar = data[@"bar"];
		    });
		    it(@"should not be nil", ^{
		        XCTAssertNotNil(bar);
		    });
		});

		SharedExamplesEnd
		
		SpecBegin(Thing)		// 名字为Thing的测试类
		
		// 描述内容
		describe(@"Thing", ^{		// 一组实例
		  sharedExamplesFor(@"another shared behavior", ^(NSDictionary *data) {
		    // Locally defined shared examples can override global shared examples within its scope.
		  });
		
		  beforeAll(^{ 
		    // 该block内的代码在后续的每个it/example、beforeEach之前运行
		  });
		
		  beforeEach(^{
		    // 该block内的代码在后续的每个it/example之前运行
		  });
		  
		  it(@"should do stuff", ^{	// 一个单一的测试
		    // 内部做代码初始化、行为流程
		    // 最后做断言
		  });
		
		  it(@"should do some stuff asynchronously", ^{ // 另外一个谓语和宾语
		    waitUntil(^(DoneCallback done) { // 异步等待
		      done(); // 该调用必须有
		    });
		  });
		
		  // 行为描述
		  itShouldBehaveLike(@"a shared behavior", @{@"key" : @"obj"});
		
		  itShouldBehaveLike(@"another shared behavior", ^{
		    // 懒加载描述
		    return @{@"key" : @"obj"};
		  });
		
		  describe(@"Nested examples", ^{ // 内嵌主语
		    it(@"should do even more stuff", ^{
		      // ...
		    });
		  });
		
		  pending(@"pending example"); // 附加例子
		
		  pending(@"another pending example", ^{
		    // ...
		  });
		
		  afterEach(^{
		    // 在每个it/example之后运行
		  });
		
		  afterAll(^{
		    // 在每个it/example、beforeEach之后运行
		  });
		});
	
		SpecEnd


其中某些关键词可以有别名，如：

* `beforeEach`和`afterEach`等价于`before`和`after`
* `it`等价于`example`和`specify`
* `describe`等价于`context`
* `itShouldBehaveLike`等价于`itBehavesLike`
* `pending`等价于把`x`加到`describe`、`context`、`it`、`example`、`specify`前面

此外，Specta和XCTest相似的功能，就是只运行单个`describe`或`example`，也称为`focus`，只需要把`f`字母加到特定的`describe`或`example`前面（别名同理）

下面看一个具体的例子（引用自objc.io）：

		SpecBegin(Car)
		    describe(@"Car", ^{
		
		        __block Car *car;
		
		        // Will be run before each enclosed it
		        beforeEach(^{
		            car = [Car new];
		        });
		
		        // Will be run after each enclosed it
		        afterEach(^{
		            car = nil;
		        });
		
		        // An actual test
		        it(@"should be red", ^{
		            expect(car.color).to.equal([UIColor redColor]);
		        });
		
		        describe(@"when it is started", ^{
		
		            beforeEach(^{
		                [car start];
		            });
		
		            it(@"should have engine running", ^{
		                expect(car.engine.running).to.beTruthy();
		            });
		        });
		
		        describe(@"move to", ^{
		
		            context(@"when the engine is running", ^{
		
		                beforeEach(^{
		                    car.engine.running = YES;
		                    [car moveTo:CGPointMake(42,0)];
		                });
		
		                it(@"should move to given position", ^{
		                    expect(car.position).to.equal(CGPointMake(42, 0));
		                });
		            });
		
		            context(@"when the engine is not running", ^{
		
		                beforeEach(^{
		                    car.engine.running = NO;
		                    [car moveTo:CGPointMake(42,0)];
		                });
		
		                it(@"should not move to given position", ^{
		                    expect(car.engine.running).to.beTruthy();
		                });
		            });
		        });
		    });
		SpecEnd

上面例子对`Car`这个类做测试，通过多个上下文嵌套（`describe`/`context`），结合不同的条件（`beforeEach`)，来作出不同的断言（`it`)；当我们某个测试失败时，我们会收到一段很明确的错误信息，比如：汽车启动后应该移动到指定位置这个用例测试失败，那么我们会收到`Car move to when the engine is running should move to given position`这么一段话。这样子非常接近自然语言的描述会让我们很快知道错误出在哪里。

更多相关例子可以参考[objc.io测试专题关于BDD测试介绍](https://objccn.io/issue-15-1/)

此外，Specta还有几个需要注意的地方：

* 如果想用`SPEC_BEGIN` 和 `SPEC_END` 替代 `SpecBegin` and `SpecEnd`，应该在引入头文件之前写上`#define SPT_CEDAR_SYNTAX`
* 如果要使用XCTest Resporter，那么在Test Scheme中，把`SPTXCTestReporter`字段值改为`SPECTA_REPORTER_CLASS`
* 把环境变量`SPECTA_SHUFFLE`设置为`1`启用测试拖拽（test shuffling)



### Expecta

Expecta是基于Objective-C／Cocoa的匹配框架，是Specta、Kiwi的好搭档。
前面文章提到，XCTest自带的断言XCAssert有好几个基础操作，不过基础的断言不太丰富，和Specta／Kiwi也没有很适配。
Expecta不一样，将匹配过程从断言中剥离开，可以很好地适配Specta／Kiwi的DSL断言块。

Expecta有以下几个特点：

* **没有类型限制**，比如数值`1`，并不用关心它是整形还是浮点数
* **链式编程**，可读性高，如：`expect(foo).notTo.equal(1)`
* **反向匹配**，断言不匹配只需加上`.notTo`或者`.toNot`，如：`expect(x).notTo.equal(y)`
* **延时匹配**，可以在链式表达式中加入`.will`、`.willNot`、`.after(interval)`等操作来延时匹配
* **可扩展**，支持增加自定义匹配


**基础匹配API：**

		expect(x).to.equal(y); 
		expect(x).to.beIdenticalTo(y); 
		expect(x).to.beNil(); 
		expect(x).to.beTruthy(); 
		expect(x).to.beFalsy();
		... 更多内容见GitHub或者EXPMatchers.h
		
以上这些都由Expecta默认提供，可以直接使用。

**异步匹配例子：**

		describe(@"Foo", ^{
		  beforeAll(^{
		    // 设置默认延时匹配时间
		    [Expecta setAsynchronousTestTimeout:2];
		  });
		
		  it(@"will not be nil", ^{
		    //	使用默认延时匹配
		    expect(foo).willNot.beNil();
		  });
		
		  it(@"should equal 42 after 3 seconds", ^{
		    // 不使用默认延时匹配，手动设置为3秒
		    expect(foo).after(3).to.equal(42);
		  });
		});

**自定义匹配：**

`EXPMatchers+beKindOf.h`

		#import "Expecta.h"
		
		EXPMatcherInterface(beKindOf, (Class expected));
		// 第一个参数是匹配器名称，第二个是参数序列，可以多个，为匹配器传入参数
		// 如：(beKindOf, (Class expected, float bar))
		
		#define beAKindOf beKindOf


`EXPMatchers+beKindOf.m`

		#import "EXPMatchers+beKindOf.h"
		
		EXPMatcherImplementationBegin(beKindOf, (Class expected)) {
		  BOOL actualIsNil = (actual == nil);
		  BOOL expectedIsNil = (expected == nil);
		
		  prerequisite(^BOOL {
		    return !(actualIsNil || expectedIsNil);
		    // 编写代码，当`.Not`时返回`NO`
		  });
		
		  match(^BOOL {
		    return [actual isKindOfClass:expected];
		    // 返回`YES`或`NO`来判断匹配结果，actual为传入的值
		  });
		
		  failureMessageForTo(^NSString * {
		    if (actualIsNil)
		      return @"the actual value is nil/null";
		    if (expectedIsNil)
		      return @"the expected value is nil/null";
		    return [NSString
		        stringWithFormat:@"expected: a kind of %@, "
		                          "got: an instance of %@, which is not a kind of %@",
		                         [expected class], [actual class], [expected class]];
		    // 匹配器匹配成功的信息
		  });
		
		  failureMessageForNotTo(^NSString * {
		    if (actualIsNil)
		      return @"the actual value is nil/null";
		    if (expectedIsNil)
		      return @"the expected value is nil/null";
		    return [NSString
		        stringWithFormat:@"expected: not a kind of %@, "
		                          "got: an instance of %@, which is a kind of %@",
		                         [expected class], [actual class], [expected class]];
		    // 匹配器匹配失败的信息
		  });
		}
		EXPMatcherImplementationEnd
	
**自定义例子：**

比如有以下类：

		@interface LightSwitch : NSObject
		@property (nonatomic, assign, getter=isTurnedOn) BOOL turnedOn;
		@end
		
		@implementation LightSwitch
		@synthesize turnedOn;
		@end

如果判断`turnedOn`可以这样：

		expect([lightSwitch isTurnedOn]).to.beTruthy();
		
自定义的话，通过上面的格式可以实现：

		// 定义
		EXPMatcherInterface(isTurnedOn, (void));
		// 使用
		expect(lightSwitch).isTurnedOn();



### Kiwi

Kiwi和Specta一样，是一个BDD测试框架，不过比起轻量级Specta只是提供BDD DSL模式，Kiwi是重量级的，功能丰富，不仅有Specs上下文DSL模式，而且还包含了**期望(Expectations)、模拟(Mocks and Stubs)、更好的异步测试**等功能，可以说是Specta、Expecta、OCMock的结合体。

#### BDD DSL Specs

上下文的格式和Specta类似，不同的是没有太多别名，格式相对固定：

		#import "Kiwi.h"
		    
		    SPEC_BEGIN(SpecName)
		    
		    describe(@"ClassName", ^{
		        registerMatchers(@"BG"); // Registers BGTangentMatcher, BGConvexMatcher, etc.
		        
		        context(@"a state the component is in", ^{
		            let(variable, ^{ // Occurs before each enclosed "it"
		                return [MyClass instance];
		            });
		    
		            beforeAll(^{ // Occurs once
		            });
		    
		            afterAll(^{ // Occurs once
		            });
		    
		            beforeEach(^{ // Occurs before each enclosed "it"
		            });
		
		            afterEach(^{ // Occurs after each enclosed "it"
		            });
		    
		            it(@"should do something", ^{
		                [[variable should] meetSomeExpectation];
		            });
		
		            specify(^{
		                [[variable shouldNot] beNil];
		            });
		    
		            context(@"inner context", ^{
		                it(@"does another thing", ^{
		                });
		    
		                pending(@"something unimplemented", ^{
		                });
		            });
		        });
		    });
		    
		    SPEC_END

其中，`specify`和`it`的不同点是，前者为无描述实例，后者有描述；`let`是为每一个`it`实例定义一个共同的变量；`registerMatchers`为匹配器定义前缀。


#### Expectations

Kiwi的期望用法和Expecta不一样，Expecta用链式表达式，而Kiwi用的是Objective-C方法，相对没那么简洁，不过可读性还是可以的，如下：

		id car = [Car car];
	   [[car shouldNot] beNil];
	   [[car should] beKindOfClass:[Car class]];
	   [[car shouldNot] conformToProtocol:@protocol(FlyingMachine)];
	   [[[car should] have:4] wheels];
	   [[theValue(car.speed) should] equal:theValue(42.0f)];
	   [[[car should] receive] changeToGear:3];		
表达式中的`should`和`shouldNot`相当于Expecta的`.to`和`.notTo`，如果car为nil，`should`和`shouldNot`依然能保证后续匹配进行。

**非对象类型需要用`theValue()`包裹起来**

**Kiwi还支持调用期望**，只需要通过`[[[car should] receive] someMethod]`，后续如果方法调用了，则通过验证

**Kiwi支持的基础匹配类型：**

* **值类型与数字：**

		[[subject should] equal:(id)anObject]
		[[subject should] beLessThan:(id)aValue]
		[[subject should] beGreaterThan:(id)aValue]

* **字符串：**

		[[subject should] containString:(NSString*)substring]
		[[subject should] containString:(NSString*)substring options:(NSStringCompareOptions)options]
		[[subject should] startWithString:(NSString*)prefix]
		[[subject should] endWithString:(NSString*)suffix]

* **正则表达式：**
		
		[[subject should] matchPattern:(NSString*)pattern]
		[[subject should] matchPattern:(NSString*)pattern options:(NSRegularExpressionOptions)options]


* **数量变化：**

		[[theBlock(^{ ... }) should] change:^{ return (NSInteger)count; }]
		[[theBlock(^{ ... }) should] change:^{ return (NSInteger)count; } by:+1]
		[[theBlock(^{ ... }) should] change:^{ return (NSInteger)count; } by:-1]

		如：
			[[theBlock(^{
        		[array addObject:@"foo"];
    		}) should] change:^{ return (NSInteger)[array count]; } by:+1];


* **对象类型测试：**
		
		[[subject should] beKindOfClass:(Class)aClass]
		[[subject should] beMemberOfClass:(Class)aClass]


* **集合类型：**

		[[subject should] beEmpty]
		[[subject should] containObjectsInArray:(NSArray *)anArray]
		[[[subject should] haveAtLeast:(NSUInteger)aCount] collectionKey]


* **消息验证：**

Kiwi支持方法调用验证，同时stub，这一点和OCMock类似

		[[subject should] receive:(SEL)aSelector]
		[[subject should] receive:(SEL)aSelector withCount:(NSUInteger)aCount]
		[[subject should] receive:(SEL)aSelector andReturn:(id)aValue]
		[[subject should] receive:(SEL)aSelector withArguments:(id)firstArgument, ...]

		如：
			id subject = [Robot robot];
			[[subject should] receive:@selector(speak:afterDelay:whenDone:) withArguments:@"Hello world",any(),any()];
			[subject speak:@"Hello world" afterDelay:3 whenDone:nil];
			
* **通知：**

		[[@"MyNotification" should] bePosted];
		[[@"MyNotification" should] bePostedWithObject:(id)object];
		
* **异步与异常：**

		[[subject shouldEventually] receive:(SEL)aSelector]
		
		[[theBlock(^{ ... }) should] raiseWithName:]
		
		
**Kiwi还支持自定义匹配器，**[详见文档](https://github.com/kiwi-bdd/Kiwi/wiki/Expectations)

#### Mock 和 Stub

Kiwi提供像OCMock类似的Mock和Stub功能，关于Mock和Stub在上一篇文件已经介绍很多，这里主要介绍不同之处，详情还是参考Github上的[文档](https://github.com/kiwi-bdd/Kiwi/wiki)。

Kiwi的Mock同样分对象实例Mock、类实例Mock、协议Mock。

**Mock**

		// 实例mock
		id carMock = [Car mock];
		[ [carMock should] beMemberOfClass:[Car class]];
		
		// null实例mock	
		id carNullMock = [Car nullMock];
		[ [theValue(carNullMock.currentGear) should] equal:theValue(0)];
		
		// 协议mock	
		id flyerMock = [KWMock mockForProtocol:@protocol(FlyingMachine)];
		[ [flyerMock should] conformToProtocol:@protocol(FlyingMachine)];
		
		// null协议mock	
		id flyerNullMock = [KWMock nullMockForProtocol:@protocol(FlyingMachine)];
		[flyerNullMock takeOff];
		
这里需要注意一下的是NULL Mock，默认情况下，如果模拟的对象收到非预期的内容会抛出异常，NULL Mock则不会抛出异常，如我我们不关心非预期内容，可以这样做。NULL Mock其实和OCMock的默认处理是一样的。

**Class Mock**：有两种形式，一种是**NSObject Category**，另一种是**Class Name**

		// NSObject Category
		[SomeClass mock]
		[SomeClass mockWithName:(NSString *)aName]
		[SomeClass nullMock]
		[SomeClass nullMockWithName:(NSString *)aName]
		
		// Class Name
		[KWMock mockForClass:(Class)aClass]
		[KWMock mockWithName:(NSString *)aName forClass:(Class)aClass]
		[KWMock nullMockForClass:(Class)aClass]
		[KWMock nullMockWithName:(NSString *)aName forClass:(Class)aClass]
		
		
**Stub**

方法stub有两种形式，**Selector**和**MessagePattern**

		// selector
		[subject stub:(SEL)aSelector]
		[subject stub:(SEL)aSelector andReturn:(id)aValue]
		
		// message pattern
		[[subject stub] *messagePattern*]
		[[subject stubAndReturn:(id)aValue] *messagePattern*]
		
stub通常放在`it`模块之内；对于没有stub的方法，返回nil或0

**参数捕获**

有时我们需要捕获某些方法的参数，比如block，然后手动调用，以更好地模拟实际流程；在OCMock也有类似功能：

		id robotMock = [KWMock nullMockForClass:[Robot class]];
		KWCaptureSpy *spy = [robotMock captureArgument:@selector(speak:afterDelay:whenDone:) atIndex:2];
	
		... running
		
		void (^block)(void) = spy.argument;
		block();
		
关于更多Kiwi的Mock和Stub注意事项见GitHub[文档](https://github.com/kiwi-bdd/Kiwi/wiki)。

#### 异步测试

执行异步任务，可以使用异步测试，主要是`expectFutureValue()`、`shouldEventually`、`shouldEventuallyBeforeTimingOutAfter(interval)`，如：

		[[expectFutureValue(myObject) shouldEventually] beNonNil];
		
		[[expectFutureValue(fetchedData) shouldEventuallyBeforeTimingOutAfter(2.0)] equal:@"expected response data"];
		
		
相反的还有`shouldNotEventually`和`shouldNotEventuallyBeforeTimingOutAfter()`


## 总结

BDD即行为驱动开发，BDD测试是以行为为目标来测试类或组件的功能，相比起单元测试，其不关心内部实现，只关心API提供的行为功能。

BDD测试的DSL能让测试描述更加接近自然语言，可读性更高。[Specta](https://github.com/specta/specta) 和 [Kiwi](https://github.com/kiwi-bdd/Kiwi/)
就是提供BDD DSL描述的两个测试框架，前者为轻量级，后者重量级、功能丰富，测试时，Specta通常搭配Expecta匹配框架使用。实际测试时，选择哪个框架，要根据实际情况来选择，如果用到的功能不过，那么Specta+Expecta够用，而且是链式表达式，可读性高，如果需要Mock和Stub，还可以结合OCMock；而如果要求功能丰富，甚至要Mock和Stub，那么Kiwi也是个不错的选择。