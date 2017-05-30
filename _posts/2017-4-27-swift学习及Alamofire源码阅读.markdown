---
layout:     post
title:      "swift学习及Alamofire源码阅读"
date:       2017-4-27 16:12:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - Swift
    - 源码
    - Alamofire
---


## 前言
学习一门语言，我比较排斥从头到尾地阅读语言教材，那样子既容易遗忘，效率又低。更好的方式，应该是阅读知名的第三方库源码，根据出现的语言障碍，去查阅书籍，并熟悉相应知识点。这样子下来，对于语言的印象既深刻，又能学习到知名第三方库的设计思想和编写技巧。如果第三方库基本看懂了，那么对该语言语法就有一定的掌握了。
本篇笔记主要记录Alamofire的源码阅读的内容，关于swift的笔记在另外的文章中。

## 1.Alamofire 概览
* Alamofire是一个基于swift语言的网络请求框架／库，类似于OC的AFNetworking
* Alamofire一共由17个文件组成分别是:
		
		-----------------接口-----------------
		Alamofire.swift // api声明
		-----------------请求-----------------
		Request.swift // 构建请求
		ParameterEncoding.swift // 参数转码
		MultipartFormData.swift // 自定义表单类
		ServerTrustPolicy.swift // 服务器验证规则描述
		-----------------响应-----------------
		Response.swift // 构建响应
		ResponseSerialization.swift // 序列化响应数据
		Validation.swift // 响应数据验证
		Result.swift // 请求结果表示
		AFError.swift // 描述错误类型
		-----------------底层-----------------
		SessionManager.swift // 请求session的管理类，调用cocoa的NSURLSession
		SessionDelegate.swift // 请求session的代理对象，主要实现NSURLSession的代理方法以及拷贝可逃逸回调闭包
		TaskDelegate.swift // 请求task任务的代理对象，主要实现NSURLDataTask的代理方法及对叼操作
		DispatchQueue+Alamofire.swift // GCD扩展，定义多个不同功能的队列
		-----------------其他-----------------
		NetworkReachabilityManager.swift // 网络状况监听类
		Notifications.swift // 定义请求通知
		Timeline.swift // 描述请求有关的时间 结构体
	
	由上述文件我们可以知道，所有文件可分为接口、请求、响应、底层实现、其他等五个功能模块，这样子，由上往下，由里及表，分模块去分析，就能解剖**Alamofire**
	
## 2. Alamofire 文件解析
### 2.1 Alamofire.swift 文件
* Alamofire.swift 文件可分为上下两部分，上部份主要是协议的定义和相关类的一些转化功能的扩展，如：把String、URL、URLComponent转为URL，并且在出错的时候throws弹出。下半部分，则是公开的请求接口，分为数据请求（Data Request）、下载请求（Download Request）、上传请求（Upload Request）和流传输请求（Stream Request）。
* 我们主要关注下半部分，有两个共同点需要注意一下，第一是，所有公开方法都声明为**@discardableResult**，即可忽略返回结果；第二是，所有请求都调用**SessionManager.default.xxx**，不管是**SessionManager.default.request**或者是**SessionManager.default.download**、**SessionManager.default.upload**、**SessionManager.default.stream**，说明了，SessionManager是所有请求的入口和管理中心
* 另外，每个请求接口都会返回**xxRequest**对象，这个是Alamofire**链式（chain）**写法的关键
* 至于各个接口具体怎么使用，这里就不做介绍了

### 2.2 Request.swift 文件
* Request.swift 文件主要是定义了5个类，基类**Request**，以及四个对应不同功能的子类，分别是**DataRequest**、**DownloadRequest**、**UploadRequest**和**StreamRequest**
* 基类**Request**定义了与HTTP请求有关的属性和方法，属性方面，如**代理delegate**，和**Foundation框架中与URL请求**有关的四个属性，分别是**task(NSURLSessionTask)**，**session(NSURLSession)**，**request(NSURLRequest)**和**responseNSHTTPURLResponse**，其余的还有重试次数retryCount、请求时间startTime和endTime，验证闭包集合validations（也是外部调用时经常用到的）
* 基类**Request**在方法方面，提供了初始化方法`init(session: URLSession, requestTask: RequestTask, error: Error? = nil)`，此方法为internal访问等级，一般为SessionManager调用创建request，外部无须用到。其余的还有三个服务器证书验证方法以及三个请求状态控制方法`resume()`、`suspend()`和`cancel`，其主要操作是调用成员属性task对应方法和发送对应的notification，如`Notification.Name.Task.DidResume`

* 此外，在介绍四个子类之前，还有两个扩展需要介绍，分别是**CustomStringConvertible**和**CustomDebugStringConvertible**，从名字也可以看成他们的作用，相当于description，前者包含了http方法、url和响应状态码，后者是比较具体的debug，这也提醒了我们，在编码时不妨养成良好的习惯，为类添加CustomStringDescription扩展

* 子类**DataRequest**，主要用于简单的HTTP请求，**GET**和**POST**。其有三个属性，**request(NSURLRequest)**、**progress**和**dataDelegate**，各自作用从字面可以理解，其中**dataDelegate**必须**向下转换(as!)**为DataTaskDelegate类型，也就是类型符合；**DataRequest**提供了两个方法，`open func stream(closure: ((Data) -> Void)? = nil) -> Self`和`open func downloadProgress(queue: DispatchQueue = DispatchQueue.main, closure: @escaping ProgressHandler) -> Self`，分别用于设置流闭包和下载进度闭包操作

* 子类**DownloadRequest**，下载请求，有三个部分，帮助类型、属性、方法；帮助类型方面，有下载选项**DownloadOptions**（用于下载完成时文件转移的操作）、下载文件位置闭包**DownloadFileDestination**以及一个enum类型的**Downloadable**；属性方面，有**request(NSURLRequest)**、**resumeData(用于下载暂停或失败，重新开始的dataOffset)**、**progress（下载进度）**以及**downloadDelegate（下载代理）**；方法有两个，`cancel()`和`downloadProgress(queue: DispatchQueue = DispatchQueue.main, closure: @escaping ProgressHandler) -> Self`，前者用于取消下载，后者用于设置下载过程闭包执行队列

* 子类**UploadRequest**，上传请求，内容和前两个子类大致相同，三个属性分别为**request(NSURLRequest)**、**uploadProgress（上传进度）**和**uploadDelegate（上传代理)**，方法只有一个，`uploadProgress(queue: DispatchQueue = DispatchQueue.main, closure: @escaping ProgressHandler) -> Self`，用于设置上传进度闭包和执行队列

* 子类**StreamRequest**，流请求，没有定义属性和方法

* 从四个子类内容看来，它们对应着普通请求、下载请求、上传请求和流传输请求。值得注意的是，它们都有相似的，遵循**TaskConvertible协议**的**enum类型---xxxable**，其中唯一的方法`func task(session: URLSession, adapter: RequestAdapter?, queue: DispatchQueue) throws -> URLSessionTask`，用于创建返回对应的task，如downTask和streamTask，并对urlRequest做适配

* 还有一点需要重点讲的是，Request的delegate/taskDelegate与各个子类的**xxTaskDelegate**都是同一个作用，就是用于创建**TaskDelegate.swift**文件中的各种代理实例，处理来自**URLSessionDelegate**的请求，如接收数据，错误等

### 2.3 ParameterEncoding.swift 文件
* 该文件主要功能是对请求参数做转码，如**URL转码**、**JSON转码**和**PropertyList转码**，指定参数转码的方式如下：

		Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters, encoding: URLEncoding.httpBody)
		Alamofire.request("https://httpbin.org/post", method: .post, parameters: parameters, encoding: JSONEncoding(options: []))
		
  不管是**URLEncoding**还是**JSONEncoding**，都是一个结构体，调用时把结构体实例传给request，做参数转码

* ParameterEncoding.swift 文件中定义了三个转码结构体，除了上面提到的两种，还有一个**PropertyListEncoding**，它们都遵守`protocol ParameterEncoding`协议，协议声明了一个转码方法`func encode(_ urlRequest: URLRequestConvertible, with parameters: Parameters?) throws -> URLRequest`，这个也是request做抽象调用的方法，用于转码，各个结构体实现该方法中，除了把参数转成目标格式，还要相应设置HTTP头部的contentType，比如JSON格式设置为`Content-Type: application/json`

### 2.4 MultipartFormData.swift 文件
* 功能是多重数据的组织，包括文件、参数等，一般用于上传，比较复杂，后面有时间再看

### 2.5 ServerTrustPolicy.swift 文件
* 用于服务器身份验证，比较复杂，属于进阶用法，后面再看

### 2.6 Response.swift 文件
* Response.swift文件主要是四个结构体，DefaultDataResponse/DataResponse、DefaultDownloadResponse/DownloadResponse，其中Default表示服务器返回的原始数据，未经序列化的；而DefaultDataResponse/DataResponse用于普通数据请求和上传，DefaultDownloadResponse/DownloadResponse用于下载，它们四个主要功能都是用于存储与响应相关的值，相当于模型
* **DefaultDataResponse**结构体有五个主要的属性，分别是**request(URLRequest)**、**response(HTTPURLResponse)**、**data(服务器返回)**、**error(请求错误)**和**timeline(请求的时间线)**
* **DataResponse**属性大致与**DefaultDataResponse**相同，增加了两个额外的属性，分别是**result(序列化数据)**和**value(result.value)**
* **DefaultDownloadResponse**有七个主要属性，分别是**request(URLRequest)**、**response(HTTPURLResponse)**、**destinationURL(最终地址／重定向地址)**、**temporaryURL(临时地址)**、**resumeData(下载取消时的contentOffset)**、**error(请求错误)**和**timeline(请求的时间线)**
* **DownloadResponse**相比**DefaultDownloadResponse**增加了两个额外的属性，分别是**result(序列化数据)**和**value(result.value)**
* 不管是**DataResponse**还是**DownloadResponse**都有用于数据转化的扩展，都有两个方法`map<T>(_ transform: (Value) -> T)`和`func flatMap<T>(_ transform: (Value) throws -> T)`，它们的区别是，前者不会throw error，后者会；值得一提的是，无论是服务器的原始数据，还是序列化后的数据，都被存储在result(enum)的case绑定元组中，success时为泛型<value>，failure时为Error，详见Result.swift

### 2.7 ResponseSerialization.swift 文件
* ResponseSerialization.swift 文件主要由两个协议和两个序列化结构体和五种类型Request扩展组成，具体如下：

		// 协议
		1.DataResponseSerializerProtocol
		2.DownloadResponseSerializerProtocol
		// 结构体
		1.DataResponseSerializer<Value>: DataResponseSerializerProtocol
		2.DownloadResponseSerializer<Value>: DownloadResponseSerializerProtocol
		// 扩展
		0.extension Request // Timeline
		// 1.Default
		// 2.Data
		// 3.String
		// 4.JSON
		// 5.Plist
		// 上面五种类型都分别对Request、DataRequest和DownloadRequest做扩展，以适应相应的序列化功能
		
* 2～5的扩展基本都依赖于扩展1的方法，底层调用构造response，几种序列化过程都基本相同，举个responseJSON的例子（DatRequest.responseJSON实现方式）：

	先给个Alamofire给的JSON序列化例子：
	
		Alamofire.request("https://example.com/users/mattt").responseJSON { (response: DataResponse<Any>) in
    	let userResponse = response.map { json in
        	// We assume an existing User(json: Any) initializer
        	return User(json: json)
    	}

    	// Process userResponse, of type DataResponse<User>:
    	if let user = userResponse.value {
        	print("User: { username: \(user.username), name: \(user.name) }")
    	}
	}
	
	DataRequest关于JSON序列化的扩展，主要有两个方法`func jsonResponseSerializer`和`func responseJSON`，而一般我们外部调用后者，其实现如下：
	
		@discardableResult
    	public func responseJSON(
        queue: DispatchQueue? = nil,
        options: JSONSerialization.ReadingOptions = .allowFragments,
        completionHandler: @escaping (DataResponse<Any>) -> Void)
        -> Self
    	{
        	return response(
            	queue: queue,
            	responseSerializer: DataRequest.jsonResponseSerializer(options: options),
            	completionHandler: completionHandler
        	)
    	}
    	
    我们可以看到，该方法一共有三个参数，第一个是可选的执行队列**queue**，第二个是**JSON序列化options**，第三个是完成时闭包，由Alamofire的JSON序列化例子可以看到序列化操作是在完成时闭包中进行的，通过调用**response.map{// 将json转为自定义模型}**。想知道其过程中发生了什么，就需要知道，在完成之前，和完成时都做了什么工作。
    先说完成前调用responseJSON，回到responseJSON方法，除了刚刚提到的三个参数，方法内部其实是调用了一个`response(queue: responseSerializer: completionHandler:)`方法，该方法是前面提到的**五种类型**中的**default类型扩展**所提供，其实现如下：
    
    	@discardableResult
    	public func response<T: DataResponseSerializerProtocol>(
        queue: DispatchQueue? = nil,
        responseSerializer: T,
        completionHandler: @escaping (DataResponse<T.SerializedObject>) -> Void)
        -> Self
    	{
        	delegate.queue.addOperation {
            	let result = responseSerializer.serializeResponse(
                	self.request,
                	self.response,
                	self.delegate.data,
                	self.delegate.error
            	)

            	var dataResponse = DataResponse<T.SerializedObject>(
                	request: self.request,
                	response: self.response,
                	data: self.delegate.data,
                	result: result,
                	timeline: self.timeline
            	)

            	dataResponse.add(self.delegate.metrics)

            	(queue ?? DispatchQueue.main).async { completionHandler(dataResponse) }
        	}

        	return self
    	}
    	
    该方法把**生成dataResponse及序列化操作**添加到执行队列中，当网络请求有结果时，执行该队列，进而做对应的操作。该方法另一个重点是，参数**responseSerializer**，这是一个服从**DataResponseSerializerProtocol协议**的变量，再看一下**DataResponseSerializerProtocol**内容：
    
    	public protocol DataResponseSerializerProtocol {
    		/// The type of serialized object to be created by this `DataResponseSerializerType`.
    		associatedtype SerializedObject

    		/// A closure used by response handlers that takes a request, response, data and error and returns a result.
    		var serializeResponse: (URLRequest?, HTTPURLResponse?, Data?, Error?) -> Result<SerializedObject> { get }
		}
		
	主要功能是声明一个返回值为**Result\<SerializedObject\>**的枚举类型(这个在Response.swift中提到过它)，它有一个**绑定值SerializedObject**
	
	回到`func response `方法。其方法中用`let result = responseSerializer.serializeResponse`，就是调用上述协议中的闭包生成**Result\<SerializedObject\>**类型实例，进而用于DataResponse的初始化，至于这个实例在DataResponse的作用，那就是我们下面要说的。
	
	在responseJSON例子中，JSON数据转换为自定义模型是在完成时闭包中实现的`let userResponse = response.map {...}`，通过调用response的`map<T>(_ transform: (Value) -> T)`和`func flatMap<T>(_ transform: (Value) throws -> T)`来实现，而response的`map`和`flatMap`核心在于调用**Result\<SerializedObject\>**.map/flatMap，去做模型转换操作，并将result绑定到Result枚举类型中，做**success和failure区分**
	
	细心的读者可能会问，json转自定义模型过程有了，那服务器原始数据转为json字典的过程呢？
	
	回到我们上面提到的`func response `方法。其方法中用`let result = responseSerializer.serializeResponse`，**responseSerializer.serializeResponse闭包**在生成**Result\<SerializedObject\>**之前，其实就做了数据序列化的操纵了，这个要回到`func responseJSON`中，其传给`func response `的**responseSerializer参数**，需要调用另一个扩展方法，`func serializeResponseJSON`，这是关于**Request**的扩展，不是`func responseJSON`所在的**DataRequest**的扩展，其实现如下：
	
		public static func serializeResponseJSON(
        options: JSONSerialization.ReadingOptions,
        response: HTTPURLResponse?,
        data: Data?,
        error: Error?)
        -> Result<Any>
    	{
        	guard error == nil else { return .failure(error!) }

        	if let response = response, emptyDataStatusCodes.contains(response.statusCode) { return .success(NSNull()) }

        	guard let validData = data, validData.count > 0 else {
            	return .failure(AFError.responseSerializationFailed(reason: .inputDataNilOrZeroLength))
        	}

        	do {
            	let json = try JSONSerialization.jsonObject(with: validData, options: options)
            	return .success(json)
        	} catch {
            	return .failure(AFError.responseSerializationFailed(reason: .jsonSerializationFailed(error: error)))
        	}
    	}
    	
    从方法中可以很清楚看到**序列化**`let json = try JSONSerialization.jsonObject(with: validData, options: options)`及**枚举绑定**`.success(json)`的过程
    
    至此，真相大白！其他如String、Data、Plist的过程和JSON类型，这里就不赘述。具体流程，如下图：
    
    ![](/img/post_img/2017-4-27-swift学习及Alamofire源码阅读/img_0.png)
    
    ![](/img/post_img/2017-4-27-swift学习及Alamofire源码阅读/img_1.png)
    
    ![](/img/post_img/2017-4-27-swift学习及Alamofire源码阅读/img_2.png)
    
    ![](/img/post_img/2017-4-27-swift学习及Alamofire源码阅读/img_3.png)
    
 
### 2.8 Result.swift 文件
* 既然上面一直提到Result，这里顺带介绍一下这个文件的作用，其功能相对比较简单，内容主要是一个**枚举类型Result**，有两个case:`success`和`failure`，case success可以**绑定指定泛型<Value>实例**，case failure**绑定Error**；该枚举还有两个方法，分别是`map<T>(transform)`和`flatMap<T>(transform)`，用于上文提到的**模型映射**，并再度绑定回case success
* 根据内容，可以知道该枚举的功能，就是用枚举表示请求结果成功或失败，并且成功时把响应数据绑定到`success`中，失败时把error绑定到`failure`中

### 2.9 Validation.swift 文件
* **Validation.swift 文件**主要用于对服务器响应数据做验证，先看一个使用例子：

		Alamofire.request("https://httpbin.org/get")
    	.validate(statusCode: 200..<300)
    	.validate(contentType: ["application/json"])
    	.responseData { response in
	    	switch response.result {
	    	case .success:
    	    	print("Validation Successful")
	    	case .failure(let error):
    	    print(error)
	    	}
    	}
    	
  该例子验证statusCode是否在200~300之间，contentType是否为`application/json`，使用两个validate，然后在序列化闭包中对response.result做验证
  
* **Validation.swift 文件**内容主要是三个扩展，**Request扩展**，**DataRequest扩展**，**DownloadRequest扩展**，前者是基本的验证，后两个是对不同下载需求的验证
* **Request扩展**有两个主要属性`var acceptableStatusCodes: [Int]`和`fileprivate var acceptableContentTypes: [String]`，以及两个方法`func validate<S: Sequence>(statusCode acceptableStatusCodes: S, response: HTTPURLResponse) -> ValidationResult`和`func validate<S: Sequence>( contentType acceptableContentTypes: S, response: HTTPURLResponse, data: Data?) -> ValidationResult`，它们分别用于验证**状态码**和**contentType**
* **DataRequest扩展**有四个扩展方法，分别为`validate(_:)`、`validate(statusCode:)`、`validate(contentType:)`和`validate()`，第一个是基础方法，用于接收**Validation**类型的闭包，构造另一个验证闭包**let validationExecution**，然后添加到验证队列中`validations.append(validationExecution)`，第二第三个方法分别是验证statusCode和contentType，它们调用**Request的扩展方法**`validate(statusCode:...)`或`validate(contentType:...)`，构造成**Validation**闭包传递给第一个方法去添加到验证队列；方法四是默认验证，调用第二第三个方法验证默认的statusCode和contentType，`validate(statusCode: self.acceptableStatusCodes).validate(contentType: self.acceptableContentTypes)`
* **DownloadRequest扩展**大致和**DataRequest扩展**相同，不赘述

### 2.10 AFError.swift 文件
* 该文件用于对请求错误的描述，主要为枚举类型，主体是枚举`enmu AFError`，包含了四个内部枚举，用于描述其他类型操作错误，分别是：

		enum ParameterEncodingFailureReason // 参数转码错误
		enum MultipartEncodingFailureReason // 表单转码错误
		enum ResponseValidationFailureReason // 响应数据验证错误
		enum ResponseSerializationFailureReason // 数据序列化错误
		
 	每个枚举类型都有关联值，用于关联错误主体
 
* 除了AFError枚举主体，还有三种类型的对所有错误的扩展，分别是：

 			通过布尔值判断是否有这种错误，如：AFERROR.isParameterEncodingError
 			通过属性名读取错误关联的主体值，没错误时返回nil，如：AFError.ResponseValidationFailureReason.acceptableContentTypes // 主要通过枚举绑定值提取
 			错误的藐视，如：AFError.errorDescription
 			
* AFError.swift文件应该关注它对于enum枚举的使用技巧，比如内联枚举，枚举绑定值及提取判断等

### 2.11 SessionManager.swift、SessionDelegate.swift、TaskDelegate.swift 文件
* 经过九九八十一难，终于来到最初的入口，最核心的部分了，SessionManager，这部分要三个文件一起来讲，甚至加上之前的其他文件，因为SessionManager是一个中心，请求的从这里开始，从这里结束，为了方便讲解，先上个图，看看他们之间的关系

 ![](/img/post_img/2017-4-27-swift学习及Alamofire源码阅读/img_4.png)
 
* SessionManager负责session的创建和管理，请求的创建和发起
* SessionManger有10个属性，其中有几个比较有代表性：

  **default**
  
  这个是一个单例，Alamofire.swift中对外接口都是调用该实例。声明如下：
  
  	open static let `default`: SessionManager = {
        let configuration = URLSessionConfiguration.default
        configuration.httpAdditionalHeaders = SessionManager.defaultHTTPHeaders

        return SessionManager(configuration: configuration)
    }()
    
  由此也可以看到swift中单例的创建方式，原理和OC差不多，使用静态常量来表示。由于default是系统关键词，故要加上单引号
  
  **defaultHTTPHeaders**
  
  创建默认HTTP头部信息
  
  **backgroundCompletionHandler**
  
  app进入后台时的完成闭包，需要手动设置和调用
  
  **queue**
  
  每个session私有的任务队列
  
* SessionManager除了属性，剩下的是初始化方法，以及四个不同类型请求的入口，普通数据、下载、上传、流数据，每个部分都有一个通用入口，他们创建请求任务按照以下过程：

		1.调用者通过方法 func request(_ urlRequest: URLRequestConvertible) -> DataRequest 调用
		2.sessionManager通过传入的urlRequest创建适配的urlRequest（用上文提到的Adapter适配器）
		3.调用Request.Requestable.task(session:adapter:queue)创建对应sessionTask // 该方法在Request.swift中提到过
		4.创建对应的xxRequest实例
		5.将xxRequest设置到delegate[task]下标
		6.开始任务
		
 除了请求的创建，剩下的就是错误处理，重试两部分，具体流程并不复杂，详细看源代码
 
* SessionDelegate负责着服务器响应事件的处理，SessionDelegate类的内容可以分为三个部分，**代理闭包属性**、**普通属性和方法**、**SessionDelegate所有代理方法的实现**
 
* **代理闭包属性**如`open var dataTaskDidReceiveResponse: ((URLSession, URLSessionDataTask, URLResponse) -> URLSession.ResponseDisposition)?`，会在下面对应的代理方法中调用，默认都是nil，手动给sessionDelegate设置的时候可以做需要的事件处理
* 属性中，**sessionManager为weak引用**，避免retainCycle；前面提到的sessionDelegate自定义下标，如下：

		open subscript(task: URLSessionTask) -> Request? {
        get {
            lock.lock() ; defer { lock.unlock() }
            return requests[task.taskIdentifier]
        }
        set {
            lock.lock() ; defer { lock.unlock() }
            requests[task.taskIdentifier] = newValue
        }
    	}
    	
   requests声明为字典，`private var requests: [Int: Request] = [:]`；task.taskIdentifier是系统生成的唯一标识；为了多线程操作安全，get和set都加了**锁**
   
* 第三部分就是SessionDelegate代理方法的实现了，基本上遵照以下流程：

		1.sessionDelegate 代理闭包属性 中有对应的实现，那么调用
		2.如果没有对应代理闭包，那么检测是否有request.taskDelegate对应实现，有的话，执行
		3.如果都没有，那只能放弃
		
  不过有一个代理方法需要提一下，就是`open func urlSession(_ session:, task:, didCompleteWithError error:)`，在sessionTask结束时调用，其中不仅调用了上述过程，而且还做了错误处理、request的validations调用、重试处理，可以说，这个方法是sessionManager的出口
  
* TaskDelegate有五个类，如下：

		1.TaskDelegate // 基类
		2.DataTaskDelegate: TaskDelegate, URLSessionDataDelegate // 子类
		3.DownloadTaskDelegate: TaskDelegate, URLSessionDownloadDelegate // 子类
		4.UploadTaskDelegate: DataTaskDelegate // 子类
		
 其功能都是，保存请求的状态信息（如：progress、totalBytesReceived、progressHandler等）以及处理来自sessionDelegate的代理转发。其中基类有一个属性为`open let queue: OperationQueue`，其作用是保存所有完成时的执行任务，在task结束时执行该队列，Request的responsexx闭包都是在该队列中；基类代理方法`func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?)`在任务结束时调用（不管是否出错），其中就有`queue.isSuspended = false`的操作，开启完成时队列，执行完成任务
 
 
* 总而言之，SessionManager、SessionDelegate、Request、TaskDelegate构成了一个请求的核心，如果将SessionManager比做一个请求工厂，那么SessionDelegate就是工厂分部，Request就是某个生成任务，而TaskDelegate就是生产任务协作人员，它们共同合作，不可分割