---
layout:     post
title:      "iOS测试系列（二）：在XCode上测试（XCTest的使用）"
date:       2017-07-27 20:11:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - 测试
---

## 前言

说到iOS/macOS的测试，第一个想到的就是XCTest了，毕竟是Xcode原生的测试框架及测试解决方案。XCTest是Xcode5集成的，Xcode6引进了性能管理，Xcode7引进来UI测试；此外Xcode Test可以在服务端Xcode上执行，可以用于持续集成自动测试。可见，如今XCTest功能也是比较地完善了，平时再结合一些开源测试框架，会让我们非常方便地进行测试。
本篇笔记着重于整体的XCTest的使用介绍，至于详细的XCTest API，详见[文档](https://developer.apple.com/documentation/xctest)

## 1. 开始
### 1.1 测试导航栏
我们可以在测试导航栏创建、管理、运行以及review测试用例

![](../img/post_img/2017-07-28-XCTest/test_001.png)

由上图可以看到，将测试导航栏目录展开，分为三层，第一层是**TestBundle**，第二层是**TestClasses**，第三层是**TestMethod**；由于第一层是**TestBundle**，因此平时我们测试需要用到的图片等资源文件，也可以一起放在里面，测试的时候根据相应`bundle`读取就行

每个测试类和测试用例后面有一个开始按钮，表示执行该测试，成功或者失败会有相应的提示：

![](../img/post_img/2017-07-28-XCTest/test_002.png)

在导航栏面板左下角加号按钮是添加测试target或测试用例，右下角两个按钮分别是只显示活跃用例和错误用例

### 1.2 为工程添加测试
现在工程创建完Xcode一般会默认帮我们创建一个TestTarget，这里介绍先假设没有创建，以下步骤：

* **创建target**，如下，与普通target一样创建

![](../img/post_img/2017-07-28-XCTest/test_003.png)

![](../img/post_img/2017-07-28-XCTest/test_004.png)

  最后：

![](../img/post_img/2017-07-28-XCTest/test_005.png)

* **运行测试，观看结果：**

![](../img/post_img/2017-07-28-XCTest/test_006.png)

* `setUp`和`tearDown`方法，这两个方法在测试类的一开始调用以及结束时调用，我们可以把一些通用代码放在里面，比如经常会用到的测试实例的创建及释放


## 2.测试基础
### 2.1 明确的测试范围
我们的工程一般由很多的小代码部件一层一层地组成，好的测试要测试每个部件，每一层的代码，XCTest可以帮我们完成这个任务；我们可以通过一个简单的测试方法来测试某个功能，也可以通过一系列的方法来测试，取决于功能的性质；我们把工程代码分成越多的部件，在开发过程我们越能高效地去测试，由于部件多起来之后测试也多，因此每个测试要尽可能快完成，这样做它们才能伴随我们调试不断快速运行，节省开发时间。

测试驱动开发（TDD）具体概念在**objc专题笔记**里面介绍过，就是测试用例先写，然后开发迭代实现代码，只要行为不变，就可以不断用测试用例来验证代码的正确性；即使不是TDD，测试用例也可以帮我们验证代码正确性，比如修复bug之后，通过其来验证

### 2.2 性能测试
XCTest提供了时间记录方法，通过baseline图显示多次测试的结果，以更直观地表示关于代码性能变化

如下，在block`measureBlock `中，写入需要性能测试的代码：

![](../img/post_img/2017-07-28-XCTest/test_006.png)

运行之后会显示：

![](../img/post_img/2017-07-28-XCTest/test_008.png)

### 2.3 界面测试
前面提到的**功能**和**性能**测试，总体来说都属于单元测试，它们的目标是，测试组件的行为是否正确，以及组建与组件之间的关系是否正确；用户通过界面来调用我们底层写好的这些小组件所集合起来的功能，且用户操作的粒度比较大，不像某个部件只执行一个单一功能，通常是一个动作序列，因此，比较难通过单元测试来进行，Xcode提供了**UI Test**来进行界面测试，我们可以通过它模拟用户操作，然后检测结果的正确性

### 2.4 App和库测试
Xcode提供了两种类型单元测试上下文，分别是：

* **App Test：**检测应用代码行为是否正确

* **Library Test：**在应用运行期间检测动态框架的行为正确性

### 2.5 XCTest
具体接口功能参照 [XCTest Framework Reference](https://developer.apple.com/reference/xctest)

### 2.6 从哪里开始着手测试
* **单元测试**应该从最基础的开始，比如我们一般都以MVC为架构，测试就从Model类及方法开始，接下去再是控制器，因为控制器最为复杂，关联的内容最多（其中还涉及到网络等复杂操作）；有比如是Library，我们应该从API入手，测试相关类的行为。

* **UI测试**从最基本的业务流程入手，用测试框架模拟相关交互；UI测试粒度比较大，一开始返回的大量结果难分析，随着测试的进行，我们可以改善测试的力度从而让流程更加清晰

## 3.编写测试类和方法

### 3.1 Test Targets, Test Bundles, and the Test Navigator
关于三者的介绍，前面提过，不赘述。
除了一个测试类对应一个实际功能类，我们还可以用测试类来分组，比如计算器，分为**基础计算测试**、**进阶计算测试**、**显示测试**

![](../img/post_img/2017-07-28-XCTest/test_009.png)

### 3.2 创建测试类

![](../img/post_img/2017-07-28-XCTest/test_010.png)

可以选的测试类有**UI Test** 和 **Unit Test Case**

![](../img/post_img/2017-07-28-XCTest/test_011.png)

值得注意的是，所有测试类都继承自**XCTestCase**

![](../img/post_img/2017-07-28-XCTest/test_012.png)

![](../img/post_img/2017-07-28-XCTest/test_013.png)

### 3.3 测试类的结构

默认创建的单元测试类为一个`.m`文件，里面包含了以下四个方法：

* `- (void)setUp`：在每个测试用例开始前调用，可以做一些测试准备工作，为可选方法
* `- (void)tearDown`：在每个测试用例结束后调用，可以做一些测试收尾工作，为可选方法
* `- (void)testExample`：默认创建的测试用例
* `- (void)testPerformanceExample`：性能测试方法

此外，我们还可以实现`+ (void)setUp`和`+ (void)tearDown`方法，它们在所有测试用例开始前／结束后执行

### 3.4 测试执行流程

当我们默认执行测试时，系统找到所有的测试类，并执行每个测试方法；我们也可以选择性地执行某些测试而已，比如，在`scheme`中disable某个用例，或者直接在测试导航栏中每个测试用例后面的运行按钮，单独执行某个测试。

默认流程如下：

上一个测试类 -> 当前类`+ (void)setUp` -> \[ `- (void)setUp` -> 测试方法 -> `- (void)tearDown` ] (循环直至当前类测试方法全部执行完) -> 当前类`+ (void)tearDown` -> 下一个测试类

### 3.5 测试方法

测试方法以`test`为前缀，没有参数，返回值为`void`，方法中用**断言**来判断测试的正确性：

		- (void)testColorIsRed {
		   // Set up, call test subject API. (Code could be shared in setUp method.)
		   // Test logic and values, assertions report pass/fail to testing framework.
		   // Tear down. (Code could be shared in tearDown method.
		}
		
### 3.6 编写异步任务的测试用例

Xcode6 引进了异步任务测试功能，主要是调用XCTestCase基类的方法`waitForExpectationWithTimeout:handler:`，使用如下:

		- (void)testDocumentOpening
		{
    		// Create an expectation object.
    		// This test only has one, but it's possible to wait on multiple expectations.
    		XCTestExpectation *documentOpenExpectation = [self expectationWithDescription:@"document open"];
 
    		NSURL *URL = [[NSBundle bundleForClass:[self class]]
                              URLForResource:@"TestDocument" withExtension:@"mydoc"];
    		UIDocument *doc = [[UIDocument alloc] initWithFileURL:URL];
    		[doc openWithCompletionHandler:^(BOOL success) {
        		XCTAssert(success);
        		// Possibly assert other things here about the document after it has opened...
 
        		// Fulfill the expectation-this will cause -waitForExpectation
        		// to invoke its completion handler and then return.
        		[documentOpenExpectation fulfill];
    		}];
 
    		// The test will pause here, running the run loop, until the timeout is hit
    		// or all expectations are fulfilled.
    		[self waitForExpectationsWithTimeout:1 handler:^(NSError *error) {
        		[doc closeWithCompletionHandler:nil];
    		}];
		}
		
### 3.7 编写性能测试

性能测试主要通过`XCTestCase`的基类方法`measureBlock:`，如：

		- (void)testPerformanceExample {
		    // This is an example of a performance test case.
		    [self measureBlock:^{
		        // Put the code you want to measure the time of here.
		    }];
		}
		
每次运行，block里面的代码会运行10次，记录每次时间，并计算平均时间，最后用baseline图显示

### 3.8 编写UI测试

UI测试和单元测试模式差不多，不同的是其关注点在UI记录和使用XCTest UI Testing API

### 3.9 在Swift中写测试用例

在swift中，测试的时候会涉及到类及类方法读取权限的问题，只有在至少`public`情况下才读得到，Xcode提供了以下解决方法：

* 在BuildSetting中把`Enable Testability`设置为YES，这会让swift标记的元素的访问等级提高
* 在上个步骤前提下，在`import somemodule`之前添加`@testable`，会让`internal`和`public`的类或类方法访问等级提升为`open`，让`internal`的其他非类元素访问等级提升为`public`

不过除了上面提到的等级，`file-private`和`private`在@testable作用下并没有效果

### 3.10 测试断言

* 断言一般由`判断条件`、`字符串format`、`字符串参数`组成，参数可选，在XCTest中，断言有以下分类：

	* **Unconditional Fail**：`XCTFail`，失败时候抛出
	* **Equality Tests**：用于断言两个表达式相等或不相等，如：`XCTAssertEqual`、`XCTAssertEqualWithAccuracy`、`XCTAssertNotEqual`、` XCTAssertGreaterThan`
	* **Boolean Tests**：断言布尔表达式真假，如：`XCTAssertTrue`、`XCTAssertFalse`
	* **Nil Tests**：空断言，如：`XCTAssertNil`、`XCTAssertNotNil`
	* **Exception Tests**：断言表达式抛出或不抛出异常，如：`XCTAssertThrows`、`XCTAssertThrowsSpecific`、`XCTAssertNoThrow`

### 3.11 不同语言中的断言（OC、Swift）

* 断言在不同语言环境下用法会有一些差别，拿**Equal**来说，`XCTAssertEqualObjects`用于判断对象是否相等，而`XCTAssertEqual`则是判断非对象类型是否相等
* 对于非对象类型的断言，在OC中，可以通过==, !=, <=, <, >=, >来操作的类型便能断言；在Swift中，遵从`Equatable`和`Comparable`协议的类型，可以用于非对象断言
* Swift中NSObject也遵从`Equatable`，可以用对象断言，不过没必要
* 在OC中，断言中允许隐式转化，Swift中不允许

## 4.运行测试／观看结果

### 4.1 运行测试命令
* 一共有以下三种方式运行测试：

	* **测试导航栏**：前面提到，不赘述
	* **代码编辑框快捷键**：需要提一点，除了每个测试用例前面的运行单个用例按钮，在类`@implementation`前面的按钮可以运行类里面所有用例
	* **Product菜单**：
		* **Product > Test**：运行当前Scheme，Common+U快捷键
		* **Product > Build for > Testing 和 Product > Perform Action > Test without Building**：快捷键Shift-Command-U／Control-Command-U
		* **Product > Perform Action > Test <testName>**：运行当前点击的测试用例，快捷键Control-Option-Command-U
		* **Product > Perform Action > Test Again <testName>**：重新运行最后一次运行的测试，快捷键Control-Option-Command-G

### 4.2 测试结果显示
除了之前提到的**测试导航栏**和**代码编辑框**可以显示测试用例的测试结果，还有以下途径可以看到测试结果：

* **Report导航栏**：点击某个测试运行记录，右边面板会显示有关测试结果

![](../img/post_img/2017-07-28-XCTest/test_014.png)

如图中的性能测试，点击时间会显示出具体性能测试结果：

![](../img/post_img/2017-07-28-XCTest/test_015.png)

点击Logs面板还可以看到详细描述

![](../img/post_img/2017-07-28-XCTest/test_016.png)

* **Debug Console**：调试框输出，这个和上面的Logs输出内容差不多

![](../img/post_img/2017-07-28-XCTest/test_017.png)

### 4.3 Scheme 和 Test Target

可以在Scheme管理中控制哪些**Bundle、Class、Method**有效

![](../img/post_img/2017-07-28-XCTest/test_018.png)

### 4.4 应用测试与库测试的Build Setting

![](../img/post_img/2017-07-28-XCTest/test_019.png)

由前面的介绍我们知道，测试target可以分为应用和库测试，而他们对应的Build Setting也还不一样，新建Test Target的时候，选择要测试的目标（上图所示），Xcode会帮我们配置默认信息；创建完之后我们还可以在Target面板中查看具体配置信息：

![](../img/post_img/2017-07-28-XCTest/test_020.png)

![](../img/post_img/2017-07-28-XCTest/test_021.png)

## 5.调试测试代码
* 测试失败的时候，有可能是真的失败，也有可能是我们的测试代码写错，所以我们要排除测试代码的问题：

	* 测试逻辑是否正确，实现代码是否正确？
	* 假设内容是否正确？有没有用错数据
	* 是否用正确的断言来验证

* Xcode有特别的工具帮助我们调试测试代码：

	* 测试失败断点，这个和平常我们使用的**Exception Breakpoint**类似，会在断言失败的地方停下，供我们调试；

		![](../img/post_img/2017-07-28-XCTest/test_022.png)
		
	* 前面提到的**Product > Perform Action > Test Again <testName>**在测试用例调试时候会很有用
	* 在**Assistant Editor**下拉菜单中有两个特殊工具：

		![](../img/post_img/2017-07-28-XCTest/test_023.png)
		
		* **Test Callers category**：比如我们在测试用例中调用了某个方法失败了，可以通过这个功能查找其他地方是否有用到该方法，并验证是否调用成功，这样可以来断定是哪里出问题
		* **Test Classes category**：和上面的类型，只不过是查找当前类中的测试用例是否在其他类出现过，我们新添加测试用例时，可以来判断其他地方是否测试过
		
	* **Exception Breakpoint**在测试运行时同样起作用

## 6.代码覆盖
Xcode7 提供了查看测试代码覆盖率查看的功能

### 6.1 打开代码覆盖功能
* 代码覆盖功能由LLVM提供，其通过方法或函数调用率来计算代码覆盖率，适用于单元测试和UI测试，开启的步骤：
	* 选择测试Scheme
		![](../img/post_img/2017-07-28-XCTest/test_024.png)
	* 选择Test面板，并为`Gather coverage data`打勾
		![](../img/post_img/2017-07-28-XCTest/test_025.png)

* **值得一题的是，代码覆盖收集会影响测试的性能表现，所以是否适用需要衡量一下**


### 6.2 代码覆盖在测试中的作用
* 作用如下：
	* 当我们测试的时候，那些代码被运行了
	* 有多少测试是全面的测试
	* 那些代码还没被测试到

* 测试结束之后，在导航的`Coverage`面板中，分别显示不同实现文件，有哪些方法被测试到了，以及文件测试百分比
	![](../img/post_img/2017-07-28-XCTest/test_026.png)

* 在每个方法名称后面的跳转小按钮，可以跳转到对应的代码编辑区域，右边显示着某行代码被调用的次数，没覆盖到的代码会高亮提醒

	![](../img/post_img/2017-07-28-XCTest/test_027.png)
	![](../img/post_img/2017-07-28-XCTest/test_028.png)
	![](../img/post_img/2017-07-28-XCTest/test_029.png)


## 7.自动化测试
* 自动化测试一般结合服务器持续继承来实现，这样做能让我们持续跟踪代码的情况，明确需求，也让多人合作的项目能够更好的进行，具体优点如下：

	* 服务端测试可以减轻客户端开发设备的压力，让我们专注代码编写和调试
	* 团队成员都在服务端测试，维持良好的测试连贯性，这样每个成员都能收到最新build的工程
	* 可以自定义测试时间，是在新commit的时候，还是周期性的时间，或者任何其他时间
	* 测试一直以相同方式进行，时刻给到开发人员编译问题和测试结果
	* 项目可以在不同设备上测试，只要把设备连接到服务器上

* 使用命令行，我们可以通过脚本来进行自动化编译和测试
	可以这样：
	
			xcodebuild test -project MyAppProject.xcodeproj -scheme MyApp -destination 'platform=OS X,arch=x86_64'
			
	也可以这样：
	
			xcodebuild test -project MyAppProject.xcodeproj -scheme MyApp -destination 'platform=Simulator,name=iPhone,OS=8.1'
			
	还可以同时多个设备：
	
			xcodebuild test -project MyAppProject.xcodeproj -scheme MyApp
		-destination 'platform=OS X,arch=x86_64'
		-destination 'platform=iOS,name=Development iPod touch'
		-destination 'platform=Simulator,name=iPhone,OS=9.0'

	详细使用参见`man xcodebuild`
	
* **通过ssh使用xcodebuild**：当我们远程登录macOS系统时，会建立一个**Aqua会话**，如果没有正确建立，使用xcodebuild的ssh链接也无法建立；此外，UIKit／AppKit也要求该会话建立，所以我们的测试需要正确建立**Aqua会话**；为了保证会话建立，我们在ssh到服务器之后，要用有效的用户登录主机系统
	
* 持续集成主要通过Xcode Server，具体操作详见 [Xcode Server持续集成](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/xcode_guide-continuous_integration/index.html#//apple_ref/doc/uid/TP40013292)

## 8.UI测试

* UI测试让我们可以模拟用户交互，然后检测空间的属性和状态来达到测试的目的；UI测试还提供了**UI Recording**功能，就是手动交互自动产生交互代码，而不用自己去写代码；UI测试提供详细的测试报告，包括失败时的截图；
* UI测试基于** XCTest framework 和 Accessibility**两个技术
* UI测试和单元测试步骤类似，都是建立target，添加实现文件，编写测试用例；不同的是，不像单元测试，去调用内部方法和函数，而是通过UI交互来检测交互结果控件的属性和状态，内部方法和函数等对于UI测试来说是不可见的

* UI测试要求**Xcode 7, macOS 10.11, 以及 iOS 9**

* UI测试基于以下三个类：

	* XCUIApplication
	* XCUIElement
	* XCUIElementQuery

* **UI Recording**会将转化好的代码输出到实现文件中，具体步骤如下：

	* 创建UITest Target
	* 把光标移动到测试方法上
	* 开始UI记录，这时应用会运行，我们开始点击UI，这时系统也开始记录我们的操作
	* 当我们操作完了，结束UI记录
	* 添加断言到输出的代码之后

* **编写UI测试**，大概步骤：

	* 找到（XCUIElementQuery）指定的控件（XCUIElement）
	* 明确该控件执行什么操作
	* 点击或者其他事件发送给控件
	* 使用断言来检测空间结果状态

* **UI测试实现文件大部分和单元测试一样，不过有以下不同**：

		- (void)setUp {
    		[super setUp];
 
    		// Put setup code here. This method is called before the invocation of each test method in the class.
 
    		self.continueAfterFailure = NO;
    		[[[XCUIApplication alloc] init] launch];
		}
		
	`self.continueAfterFailure = NO`表示某个操作失败的时候，不继续后面的测试，这是必须的，因为UI测试，多数时候后面的操作是基于前面的操作，一旦前面的错误，后续也很可能会错误；
	`[[[XCUIApplication alloc] init] launch]`用于启动应用
	
* **UI测试和单元测试一样，除了功能测试，也可以测试性能，一样在`measureBlock`中写入要测试的代码**

## 9.编写可测试的代码

* **声明接口要求（Define API requirements）**：必须声明或用文档明确指出方法／函数的功能、输入输出的范围、什么条件下会发生异常、返回值类型等

* **边写代码边写测试用例（Write test cases as you write code）**：每写一个方法或者函数时，要顺便写测试用例，这比写完代码再一起写测试用例容易很多

* **检测辩解条件（Check boundary conditions）**：如果参数有明确的范围，那么我们要检测边界情况

* **使用负面测试（Use negative tests）** ：检测代码再错误输入的情况下，是否表现期望的异常结果

* **编写全面的测试用例（Write comprehensive test cases）**：通过不同的方式。或者说调用不同的方法来测试同一个行为，这一点不同于平时的单一测试

* **测试刚修复的bug（Cover your bug fixes with test cases）**：无论什么时候，新修复的bug要用测试用例重新测试





