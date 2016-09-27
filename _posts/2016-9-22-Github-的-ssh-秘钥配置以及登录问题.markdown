---
layout:     post
title:      "Github 的 ssh 秘钥配置 以及登录问题"
date:       2016-9-22 15:22:00
author:     "GeorgeWang"
header-img: "img/post-bg-miui6.jpg"
tags:
    - Git
    - ssh
---

最近在github上部署项目的时候遇到一个问题，将项目拷贝下来之后，修改完再度push上去，却收到 **Permission to user/repo denied to other-user** 的错误，在github help搜索之后，其给出的答案如下

		This error means the key you are pushing with is attached to an account which does not have access to the repository.
		To fix this, the owner of the repository (user) needs to add your account (other-user) as a collaborator on the repository or to a team that has write access to the repository. 
大概意思就是所给出的秘钥无权利访问这个项目。需要让项目所有者将other-user加入到项目组。

先说下最终的解决办法，其实就是以前登录过的账户保留在系统的钥匙串中，所以当前的用户是other-user，自然无权利访问。办法就是在系统钥匙串设置中将以前的邮箱关于github的钥匙串删除，回到终端重新push，输入账户和用户名，便可以。

下面是分界线
***

解决这个问题的时候，更重要的是一开始的误打误撞。

**1.最开始** 以为是git的config问题，于是修改username,useremail等，下面是关于git config的一些设置方法

config文件路径(mac os下) ： ~/.gitconfig

config的设置分为几个等级，global，project，子级默认用上级的设置。

命令行的设置法

git config [--global] user.name yourUsername
git config [--global] user.email yourUsername

在项目根目录下执行时，不加--global选项，为设置项目git配置。

config的作用是，设置分支开发时的作者信息，暂时了解这点，其他的后续再研究。

**2.随后** 在stackoverflow和githelp上搜索，说是公钥的问题，于是研究了一下ssh key的配置

github用rsa非对称秘钥来进行https的加密，用于客户端与github服务器进行通信时的认证，也就是常见的 git@github.com:user/rep 远程仓库通信。

**关于ssh key的部署**，👉 [官方教程]("https://help.github.com/categories/ssh/") 
大概的步骤是

**1.查看系统已经存在的rsa秘钥对**

		$ ls -al ~/.ssh 
如果出现id_dsa与id_dsa.pub类似的私钥公钥秘钥对的话，就证明以前配置过，当然在这里是介绍怎么配置，所以继续往下看

**2.新建秘钥对并添加到ssh代理**

		$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com" 
然后设置秘钥的名字

		Enter a file in which to save the key (/Users/you/.ssh/id_rsa): 
一般自己命名为 **id_rsa_somename**

接下去的问题验证有兴趣的可以到githelp看，这里不做详解

		Enter passphrase (empty for no passphrase): [Type a passphrase]
		Enter same passphrase again: [Type passphrase again] 
直接按enter也可以

**3.把秘钥对添加到代理**

首先查看代理是否可用

		$ eval "$(ssh-agent -s)" 
出现下面结果

		Agent pid 59566(或者其它id) 
证明是有用的，然后

		$ ssh-add ~/.ssh/id_rsa 
id_rsa应该是第2步时你秘钥的命名

**4.把秘钥添加到你的github**

复制秘钥信息到粘贴板

		$ pbcopy < ~/.ssh/id_rsa.pub 
选择github个人的 **setting-SSH and GPG keys-New SSH key or Add SSH key**

输入一个秘钥名字，然后command+v粘贴秘钥内容，最后点击**Add SSH key**确认。

5.检测秘钥对是否能链接成功

终端输入 ：

		ssh -T git@github.com 
如果出现：

		Hi username! You've successfully authenticated, but GitHub does not provide shell access. 
则验证成功，可以正常使用了。

其实如果没配置ssh key的话，提示的错误信息应该是 **Permission to user/repo denied to (other-)user/other-repo** ，所以遇到这个问题时也可以借鉴一下。

如果有其它问题，请到githelp寻求帮助。
		

**以上** 两种就是一开始误打误撞去探究的结果，虽说没有解决问题，但是也学如何配置git的秘钥等内容。

**总而言之**，github的账户切换时，要注意对配置信息和钥匙串的更换，否则会出现Permission ... denied to ... 的错误。
		