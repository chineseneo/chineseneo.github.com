---
layout: post
title: "git高级原理分析"
date: 2014-01-14 14:05
comments: true
categories: 
---
#detached HEAD
我们我们应该经常会使用`git checkout HEAD~n`这样的命令来查看历史的某一次提交，那么什么是HEAD呢？有人可能会说HEAD就是一个指向某个commit的reference，这种理解没错，但也不是全对。

如果你在一个git repo的根目录下执行命令`cat .git/HEAD`，得到的结果会有两种可能：

- 一般情况下应该会得到形如`ref: refs/heads/master`的结果，也就是HEAD所指向的refs/heads目录下的master；
- 但是如果执行`cat`命令之前先使用`git checkout`提取某个commit，那么就会得到形如`93acc9c3279113af3fb492234169059811864801`的结果，也就是HEAD的指向是一个commit的hash。

为什么会出现这样的结果？

其实HEAD最主要的功能在于追踪branch。每当一个新的git repo被创建出来的时候，都会产生一个HEAD，并且默认指向`ref: refs/heads/master`，即使这时候master还不存在。在这种情况下，如果连续commit的话，会发现你始终在master这个branch上面，也就是说，HEAD被attach在master上了。

那么，如果我们进行`git checkout HEAD~`这样的操作呢？试试上操作结束后，git会立即这样提醒：

	You are in 'detached HEAD' state. You can look around, make experimental changes and commit them, and you can discard any commits you make in this state without impacting any branches by performing another checkout.
	
这就是`detached HEAD`状态。