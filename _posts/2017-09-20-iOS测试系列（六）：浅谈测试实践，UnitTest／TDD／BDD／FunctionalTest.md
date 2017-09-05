---
layout:     post
title:      "iOS测试系列（六）：浅谈测试实践，UnitTest／TDD／BDD／FunctionalTest"
date:       2017-09-20 21:40:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - 测试实践
    - 单元测试
    - TDD
    - BDD
    - 集合测试
---

### 前言

前面的多篇文章中，分别介绍了**XCTest、OCMock、Specta／Expecta／Kiwi、KIF**等测试框架，虽然也只是众多测试框架中的一部分，但已经算是比较主流的。框架有了便有了工具，但我们还需要一些理论的支持，才能完成测试这一环节。什么是理论呢，我想应该和设计模式一个道理，有了语言和框架的支持，还不至于让我们写出优秀的代码，我们还需要的是一些实践经验，也就是设计模式／架构模式的支持。业务代码中我们有MVC／MVVM／MVP／VIPER等，而测试中也有一套测试模式。

从测试范围来说，可以分为：

* 单元测试（UnitTest）
* 集成测试（IntegrationTest）
* 系统测试（System Test）

从开发模式来说有：

* TDD（Test-Driven-Development）测试驱动开发
* BDD（Behaviour-Driven-Development）行为驱动开发

从测试方法来说有：

* 手动测试
* 自动化测试

虽然有多种分类，但他们都是相辅相成的，比如说，UnitTest-TDD-BDD这三者是平时用得较多的一种，给我们展示了测试什么／什么时候测试／如何测试的方案。

下面简要介绍一下这几种分类。

### 单元测试（UnitTest）

单元测试，顾名思义，就是测试单一的代码代码片段，通常是一个类的方法或者一个模块的某个函数，通过mock／stub模拟外部依赖，进而测试代码的核心功能。一旦测试通过，后续我们对代码进行小修改，依然可以使用该测试。单元测试也是XCTest的一个重要部分，还记得XCTest的每个测试方法吗，那就是一个测试用例，也可以说一个单元测试。

以下就是一个XCTest Unit Test：

		- (void)testThatItDoesURLEncoding {
		    // given
		    NSString *searchQuery = @"$content$amp;?@";
		    HTTPRequest *request = [HTTPRequest requestWithURL:@"/search?q=%@", searchQuery];
		
		    // when
		    NSString *encodedURL = request.URL;
		
		    // then
		    XCTAssertEqualObjects(encodedURL, @"/search?q=%24%26%3F%40");
		}
		
通过设定初始值、参数，运行代码，最后验证结果是否正确。
其实在前面的文章中，单元测试介绍得很多了，这里不再赘述。
单元测试的缺点就是，太过于依赖于代码的实现，代码有大变动的时候，测试代码重用率很低。

### 集成测试（IntegrationTest）

集成测试是在单元测试的基础上，将所有模块组合成子系统或系统，进行集成测试。其主要的目的是，暴露出模块集成之后，单元测试所不能暴露的问题；有些问题虽然在单元测试中没出现，但是在集成测试中还是很容易找到。集成测试也是伴随着整个组装过程来进行的，比如说，我们的系统是由N个代码块组成M个小集成组件，再由M个小组件组成I个子系统，最后由I个子系统组合成一个完整的系统，这就是一个逐层集成的过程，而测试也应该伴随每一次的集成，以保证集成的正确。

### 系统测试（System Tests）

系统测试是**将经由许多集成测试而组合出来的最终系统进行测试的一种测试**。
系统测试包括了**功能测试（Acceptance／Functional Test）和健壮性测试（Robustness Test）**。
**功能测试：**根据系统需求说明的描述，对系统逐个功能点进行测试，检测是否达到预定需求。
**健壮性测试：**健壮性包括**容错能力**和**恢复能力**，一般进行**恢复测试、安全测试、压力测试**。


### 测试驱动开发（Test-Driven-Development）

测试驱动开发，是在代码编写之前先写好测试用例，代码实现之后加以测试验证，如下流程：

* 依据需求编写测试用例
* 运行测试，这时测试必须失败，因为代码没有实现，如果通过了是不正常的
* 编写代码让测试通过
* 再次运行测试检测是否通过
* 重构代码
* 重复第一步

TDD的好处是，测试的覆盖率高，让代码可靠性大大提高；但是相对的，由于覆盖内容比较多，任务量也就变大，时间耗费变多。
TDD就是一种开发模式，以测试驱动，让我们知道，什么时候应该测试。

### 行为驱动开发（Behaviour-Driven-Development）

**行为驱动开发**是**测试驱动开发**的进化，关注点在于设计，后续测试通过行为描述来验证，而行为的描述是需求拟定者、开发人员、测试人员共有的语言（Domain Specific Language，DSL）编写的；行为驱动开发中的测试相比单元测试，不关注代码的实现，而关注代码的行为，也就是测试行为的结果的正确性。BDD测试关注API行为而不关注内部代码实现，带来的好处就是，只要API行为不变，测试的代码就一直有效，这一点也是**多数自动化测试采用BDD**的重要原因。

在前面的文章中，我们所介绍的**Specta／Kiwi**都是BDD测试框架，下面是前面文章用到的一个例子：

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
			
			        ... 省略 ...
			    });
			SpecEnd
			
这就是BDD DSL的测试描述／代码，通过DSL描述，让测试更加接近自然语言，可读性提高。


### 不同测试模式之间的关系

上述的这么多个测试模式，他们并非孤立存在。很明显的是，就测试规模来说，先有单元测试，再有集成测试，最后才是系统测试。集成测试不能先于单元测试，系统测试不能先于集成测试，否则很多问题将会无从发现。而TDD和BDD也不是孤立的，单元测试是这两者的基础，明确了测试什么，TDD则提出了什么时候进行测试，BDD给出了如何测试，他们构成了一个测试方案，给我们日常测试开发指明方向。日常开发可能比较少用到TDD，因此，单元测试和BDD会是主要的。

### 测试方案

有了这么多种测试模式，我们就可以定制一套日常开发方案了，比如按照以下流程：

* **单元测试／BDD**，在我们实现每个类，每个方法的过程中，尽可能地编写测试用例进行单元测试；如果有可能的话，还要更多地使用BDD进行测试，特别是当我们目标是自动化测试的时候。比如，本人日常开发时用到比较多的MVVM模式，那么测试的时候，就应该在**View-Controller-ViewModel-Model**四个部分的编写过程中，逐步地进行单元测试；特别是**ViewModel**，其中包含大量的业务代码和视图逻辑，我们应该确保它的正常运行；而**Controller**是多个部分的枢纽，有时候Controller中也包括View，因此对Controller的测试是比较麻烦的，在精力有限的情况下，我们可以对部分核心代码着重测试（比如界面的跳转逻辑、响应逻辑）。其实**单元测试／BDD**也是我们日常开发最为常见的，因为它伴随着整个开发过程，这个阶段我们用到的框架有**XCTest Unit Test / OCMock / Specta / Kiwi**。

* **集成测试**，集成测试可以是对整个类的行为测试，也可以是如我们的ViewController的测试，又或者是一个小业务模块的测试；这一部分的测试情况、代码集成的规模比较多，对外部的依赖也可能比较大，因此是有一定的测试难度的，应该具体情况具体分析；对于我们平时的业务代码，特别是某个控制器，更多的应该是结合KIF等界面测试框架来进行；而对于某些功能性组件，比如说网络框架、存储框架等，就要不同对待，一般是结合BDD做行为测试，可以使用框架有**OCMock/Specta**。

* **系统测试**，系统测试中的功能测试，可以结合KIF／Appium／Frank等来进行；而健壮性测试的话，App端比较多的是测试网络、安全、容错等，有些企业已经提供了自动化测试的解决方案，这个有兴趣的可以Google一下，本文不多讨论。

### 总结

测试按规模可以分为单元测试、集成测试、系统测试，也可以依据开发模式分为TDD／BDD，根据测试方法也可以分为自动化测试和手动测试。在这个系列的文章中，自动化测试介绍得比较少，由于自动化测试还需要配合XcodeServer和持续集成等技术，因此这个系列中暂时不讨论，后续有机会再研究。测试是一个庞大的任务，也是非常讲究实践经验的一个过程，好的测试不但可以及时发现问题，还是提高开发效率，节约开发成本。如今除了Apple官方提供的测试框架，开源社区也为我们提供了很多的可选、优秀的测试框架，这让我们的测试变得简单起来。好的测试框架加上一套好的测试方法论，才能保证测试更好进行。


### 参考文档

* **[Wikipedia: Unit Test](https://en.wikipedia.org/wiki/Unit_testing)**

* **[Wikipedia: Integration testing](https://en.wikipedia.org/wiki/Integration_testing)**

* **[Wikipedia: System testing](https://en.wikipedia.org/wiki/System_testing)**

* **[Behavior-Driven Development](https://www.objc.io/issues/15-testing/behavior-driven-development/)**

* **[What’s the difference between Unit Testing, TDD and BDD?](https://codeutopia.net/blog/2015/03/01/unit-testing-tdd-and-bdd/)**
