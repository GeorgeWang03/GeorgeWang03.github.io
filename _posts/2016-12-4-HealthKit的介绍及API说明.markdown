---
layout:     post
title:      "HealthKit的介绍及API说明"
date:       2016-12-4 20:24:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - WYChart
    - 开源
    - 绘制
    - 交互
---


## 总览
* 内容主要参照[官方文档](https://developer.apple.com/reference/healthkit)

* Health Kit是iOS 8.0引进的新框架，其可以记录手机传感器、穿戴设备收集到的健康数据，苹果系统会自动合并这些来自不同设备的数据，智能化合并；不过iPad不支持Health Kit

* Health Kit的健康数据存储在一个加密的叫做**HealthKit store**的数据库中，通过类**HKHealthStore**可以取得这些数据；苹果手表和手机分别有一个**store**，苹果手表为了节省空间，会定时删除旧的数据，可以通过方法**earliestPermittedSampleDate**来设置存储时间

* 用户拥有决定任何一个种类健康数据的访问权限控制，可以允许某些数据供第三方app使用；iOS 10.0之后要使用健康数据要在Info.plist中加入  NSHealthShareUsageDescription 和 NSHealthUpdateUsageDescription 键
* Health Kit保存多种类型的数据，如下：
	**Characteristic data:** 保存不会变动的数据，如身高、体重、性别等，可以通过 `dateOfBirthWithError:`, `bloodTypeWithError:`, `biologicalSexWithError:`等方法获取，这些数据智能用户手动在Health App中设置
	**Sample data:** 样本数据，表示在某个时间点的样本，所有Sample Data继承自类HKSample又继承自HKObject
	**Source data:** 表示样本来源，**HKSourceRevision**表示存放样本的应用或设备的信息，**HKDevice**表示收集样本的硬件设备的信息
	**Deleted objects:** 临时存放被Health Store删除的条目的UUID，参照类**HKAnchoredObjectQuery** 和 **HKDeletedObject**
	
* 属性
	**HKObject:** 有**UUID**、**Metadata**、**Source Revision**、**Device**，其中**Metadata**是一个字典，里面除了存放系统预置的键值对，还可以存放自定义键值对，供不同app使用
	**HKSample:** **Type**、**Start date**、**End date**，**Type**表示样本的类型，如睡眠分析样本、步数样本，后面两个为起始时间，如果样本是对应某个时间，那么开始时间和结束时间相同
				**HKSample** 还分为四个子类 **HKCategorySample**（分类数据，睡眠分析之类的)、**HKQuantitySample**（用数字表示的数据，体重、升高、步数）、**HKCorrelation**（包含多个样本，表示食物或者血压样本）、**HKWorkout**（表示运动数据，如跑步、游泳；这类数据一般有**类型**、**持续时间**、**距离**、**消耗能力**等属性）
				
## 使用Health Kit

1.打开HealthKit capabilities

		 当我们在Xcode项目中引入Health Kit的时候，xcode自动帮我们把其加到支持Health Kit的设备，如果我们不想在某个设备上使用，可以在Info.plist的**Required device capabilities**数组中删除对应的设备
		 
2.调用下面代码判断是否在当前设备上可用：

		if ([HKHealthStore isHealthDataAvailable]) {
    		// add code to use HealthKit here...
		}

3.创建health store，每个app只需要一个：

		self.healthStore = [[HKHealthStore alloc] init];

4.为应用授权，弹出健康数据权限页面，这个参照官方demo

5.在Info.plist中设置授权时的提醒信息，**NSHealthShareUsageDescription**设置读提醒信息，**NSHealthUpdateUsageDescription**设置写提醒信息

6.确保在读写数据之前用户授权了，否则会返回授权失败错误信息
7.当用户授权某个类型的数据权限之后，可以**创建数据到app的healthStore中**或者**读取健康数据**

## 创建样本到Health Store
1. 通过**HealthKit Constants**和**HKObjectType的子类**创建类型实例
2. 用上面的类型实例创建**HKSample**实例子
3. 用Health Store的`saveObject:withCompletion:`方法保存数据

* 当然每个子类样本都有一套创建实例的过程，和上面的过程不一定一样，如：

  **HKQuantitySample：**计量样本的单位要符合系统定义的单位类型，我们创建**HKQuantity**实例，比如**HKQuantityTypeIdentifierHeight**表示这个样本用长度作为单位表示
  ![hk_001](/img/post_img/2016-12-4-HealthKit/hk_001.png)
  
  **HKCategorySample：**分类样本必须用系统指定的分类枚举变量，如**HKCategoryTypeIdentifierSleepAnalysis**表示使用**HKCategoryValueSleepAnalysis**枚举，因此我们要用该类型枚举去创建实例子
  ![hk_001](/img/post_img/2016-12-4-HealthKit/hk_002.png)
  
  **HKCorrelation：**关联数据中，**correlation’s type identifier**必须先定义好自身以及所有会包含的数据类型，不用亲自把包含的数据添加到store，HKCorrelation会自己添加
  ![hk_001](/img/post_img/2016-12-4-HealthKit/hk_003.png)
  
  **HKWorkout：** 运动样本不需要和上面几个不同，它不需要指定**HKWorkoutType**类型，只要通过**HKWorkoutActivityType**指定运动类型，最后要关联一些如能量或距离的样本以表示该运动过程；另外，不同系统记录运动数据也不一样，watchOS通过**HKWorkoutSession**记录，然后移动距离依据能量消耗，锻炼依据时间，最后计算并添加到活动环；iOS10+则不用做额外工作，时刻记录着，然后添加到活动环；iOS 9不添加到活动环
  
  ![hk_001](/img/post_img/2016-12-4-HealthKit/hk_004.png)
  
* 保存数据时，自定义的样本包含越多的细节样本能更好表示样本，消耗越短时间的运动样本，如步数，则是越短时间记录一次越好，最好小于1小时，不要超过24小时

## 读取样本
* Health Kit提供了三种主要的途径用于读取数据，分别是**Direct method calls**、**Queries**、**Queries**
* **Direct method calls** 是Health Store提供的直接用方法调用的方式去读取**characteristic data**也就是不可变的身高、性别等数据
* **Queries** 是由Health Store返回的数据快照，一般的查询是一般进行，通过完成block回调。有几种类型的查询，分别返回不同类型的数据：
	* 样本查询：最常见的查询，可以对样本做排序和过滤等操作，参照**HKSampleQuery**
	
	* 锚对象查询：用于查询被添加到store或从store删除的条目，或者说是修改的数据，第一次返回所有store中匹配到的，第二次返回自从第一次之后添加或删除的，参照**HKAnchoredObjectQuery**
	* 统计查询：在匹配到的数据的基础上做总数、最大最小、平均值查询，参照**HKStatisticsQuery**
	* 统计集合查询：和上一个查询的不同在于，本查询是基于一个固定的时间段内的数据，如整一天的卡路里消耗、每5分钟的步数，参照**HKStatisticsCollectionQuery**
	* 关系查询：查询保存在**关系样本（correlation data）**中的不同类型数据，为复杂查询，参照**HKCorrelation**
	* 来源查询：顾名思义用于查询指定设备／app来源的样本数据，参照**HKSourceQuery**
	* 活动总结查询(Activity summary query)：查询某项运动的总结，每个条目包含用户一天的运动描述，参照**HKActivitySummaryQuery**
	* 文档查询：参照**HKDocumentQuery**

* **Long running queries** 长时间查询，在后台启动查询任务，当health store有数据变化是，通知app并更新App内容，和普通查询**Queries**最大的不同就是这个是长时间的随着变化而不断查询的，其中的**Observer query**还可以指定为真正的后台任务，数据更新时唤醒app
	* **观察者查询(Observer query)** 当指定匹配到的数据有变化的时候，通知app
	* **锚对象查询(Anchored object query)** 长时间的关于修改数据的查询，只限于修改部分的数据，不能做真正的后台查询
	* **统计集合查询** 长时间查询，如果相关数据修改了，那么重新做一次普通的统计集合查询
	* **活动总结查询(Activity summary query)** 也是长时间查询，数据变化是重新做一次普通查询

## 单位和数量（Units and Quantities）

* 类**HKUnit**用于表示一个单位，支持简单的单位，如：米、秒、磅，或者复杂的单位，如：米／秒，**HKUnit**提供了一些创建复杂单位的操作	
* 类**HKQuantity**可以存储指定单位的数量，还可以在不同单位间转换
* 可以用**NSMeasurementFormatter**来本地化数据单位

## 线程（Threading）

* Health Kit 是线程安全的

## 数字签名

* Health Kit 的数据还可以加上数字签名，防止被篡改，暂时用户到这个功能，需要的时候再仔细研究

## 使用Health Kit的好处

* 数据收集、数据处理、社交使用 的分割，使得我们只需要集中关注我们所关心的点，让其他工作交与其他人，这样的分割让用户可以选择自己喜欢的app类型，如：有关体重的、有关步数的；而且不同app之间可以共用同一套健康数据，不同设备来源的数据也可以合并
* 开发者不用担心如何在app之间共享数据，Health APP已经帮我们处理了，而且不同App还可以控制不同的数据类型权限
* 提供丰富的数据集合，让开发者能更好地分析并为用户提出健康的建议
* 构建了一个好的生态系统，不同的应用使用同一套数据，会使得healthKit越来越强大，我们的应用可以收集与其他强大应用共有的数据