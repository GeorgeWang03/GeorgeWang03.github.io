---
layout:     post
title:      "创建自己的 Github 开源库并发布到 Cocoapods 插件库"
date:       2016-10-9 17:13:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - Github
    - Cocoapods
    - OpenSource
---


**前记：本文用于记录开源库的创建过程备忘以及初学者参考**

开发者们平时在**Github**查看各种功能的开源库时，是否想过开源库从何而来？开源库如何创建？使用**Cocoapods**时的`pod 'someLibrary'`从何而来？

最近发布了个人第一个严格意义上的开源库[**WYChart**]('https://github.com/GeorgeWang03/WYChart')，便想着把这个过程记录下来，以便后来者参考。本文将主要从Cocoapods创建公开库来说明Github + Cocoapods开源库创建全过程。

## 一、创建Cocoapods工程库

### 1. 在你存放开工程库的工程文件夹目录下（比如我的是~/Desktop/iOS/Pods)，执行下面命令：


		pod lib create MyLibrary

接着会出现以下问题，即工程库为Objective-C还是Swift，`What language do you want to use?? [ Swift / ObjC ]`，根据你的需要输入，比如输入`Objc`，然后Enter；

再接着是 `Would you like to include a demo application with your library? [ Yes / No ]`，也就是是否包含**例子工程(demo)**，需要的话选择`Yes`；

然后是，`Which testing frameworks will you use? [ Specta / Kiwi / None ]`，选择测试框架，不需要的话选`None`；

接着是，`Would you like to do view based testing? [ Yes / No ]`，是否要基于视图测试；

最后是`What is your class prefix?`输入工程的前缀，所有的文件都会加上这个前缀。

成功之后会有以下信息：
	
		Ace! you're ready to go!
		We will start you off by opening your project in Xcode
		open 'WYAnimations/Example/WYAnimations.xcworkspace'
		
即工程库创建成功，🍺🍺🎆🎆🍻🍻
	
	
### 2. 工程库目录简介

工程库创建完之后，Xcode界面会弹出:

![XcodeTemplate](/img/post_img/2016-10-10-Github_Cocoapods/Xcode_Template.png)
	
由上往下可以看到主要的文件夹有**Podspec Metadata**，**Example for yourLibrary**，**Development Pods - YourLibrary**。
	**Podspec Metadata**中分别有**.podspec**、**README.md**和**LICENSE**，**.podspec**为工程库的信息，详情格式google，**README.md**中可以对工程库做介绍，cocoapods会默认帮你生成一个介绍的模版:
	
![README](/img/post_img/2016-10-10-Github_Cocoapods/README_DEFAULT.png)
	
值得注意的是，**README**文件中常见的小图标，如![icon](https://img.shields.io/cocoapods/l/WYAnimations.svg?style=flat) 等，可以在[这里](http://shields.io/)找到并定制。
	**LICENSE**默认是**MIT**，你可以根据需要自行选择如Apache或 [其它的](http://choosealicense.com/) 。
	
**Example for yourLibrary**便是存放Demo例子的文件夹，如同平时创建Xcode工程时一样。
	**Development Pods - YourLibrary**，可以看到里面有**ReplaceMe.m**文件，顾名思义，就是用你的组件库内容替换它，也即是工程库文件的存放地。
	
### 3. 开发

在了解并创建完上述的工程之后，就可以编写你的工程库以及demo例子，如果工程和例子已经写好，那么直接复制替换到文件夹中就可以进入下一步发包了。

## 二、加入版本控制并推到Github

在Github上创建一个工程库，并在刚刚创建的本地工程库目录中，执行下面命令：

	$ git init  //初始化git仓库
	$ git add . //暂存所有修改
	$ git commit -m "first commit"  //提交修改
	$ git remote add origin git@github.com:YourUserName/	  YourRepository.git  //添加远程仓库
	$ git push -u origin master  //把本地内容推到Github远程仓库
	
至此，你便可以随时将修改更新到Github仓库上。

## 三、把版本的工程库发布到Cocoapods公开库或私有库

备注：这里主要介绍公开库，私有库👉[点击这里](https://guides.cocoapods.org/making/private-cocoapods)，公开库英文原版介绍👉[点击这里](https://guides.cocoapods.org/making/getting-setup-with-trunk)

### 1.创建pod帐号

如果你已经创建过，可以跳过这一步。

在终端输入(pod帐号不需要密码，只是一个token存放在你的机器上)：

		$ pod trunk register 你的邮箱 '你的用户名' --description='关于你的机器的描述'
		
创建完，或者创建过之后，执行下面命令：

		$ pod trunk me
		
会出现如下信息：

		- Name:     GeorgeWang  //用户名
  		- Email:    georgewang003@gmail.com  //邮箱
  		- Since:    September 18th, 08:44  //时间
  		- Pods:	// 创建过的开源库
    		- WYChart
  		- Sessions:

### 2.发布开源库

⚠️ 需要注意的是，cocoapods对发布的开源库要求没有任何错误或警告信息，而且要有证书（这一点由于我们在上面的创建中已经生成证书所以不用担心）。

确定你的工程没有问题，以及版本正确之后，你需要在你的项目目录中创建一个**git分支**，标注版本号，如`0.1.0`，这样才会被cocoapods接受。

最后，在工程库目录执行下述命令：

		pod trunk push [NAME.podspec]
		
如果成功的话，工程库便发布到cocoapods上了。

可以在其它工程里面的**podfile**尝试加入`pod YourRepository`，然后`pod install`，如果成功，即工程库发布成功。🍻🍻🎆🎆 


**后记：发布流程至此结束，如果你对开源库有兴趣，行动吧，并推广你的开源库，让更多人知道你！**