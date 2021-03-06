---
layout:     post
title:      "iOS开发Delegate,Notification,Block使用心得"
subtitle:   ""
iframe:     "http://huangxuan.me/js-module-7day/"
date:       2015-06-08 00:58
author:     "Maru"
header-img: "img/post-bg-01.jpg"
tags:
    - iOS
    - Objc
---


## （一）简要介绍

 1.Delegate(代理、委托)
 代理几乎是iOS开发中最常用的传值方式，在项目中的AppDelegate就是使用的这种设计模式，不仅如此，还有很多原生的控件也使用的这种设计模式，比如：UITextFiled，UITableView等等。官方给出的解释如下：

> Delegation is a simple and powerful pattern in which one object in a program				1	 acts on behalf of, or in coordination with, another object. The delegating object keeps a reference to the other object—the delegate—and at the appropriate time sends a message to it. The message informs the delegate of an event that the delegating object is about to handle or has just handled. The delegate may respond to the message by updating the appearance or state of itself or other objects in the application, and in some cases it can return a value that affects how an impending event is handled. The main value of delegation is that it allows you to easily customize the behavior of several objects in one central object.

具体使用方法不再示例（网上关于Delegate基础使用的博文数不胜数[示例](http://www.2cto.com/kf/201307/229082.html)），主要总结一下Delegate的优点和缺陷。

优点：
1.减少代码的耦合性，使事件监听和事件处理相分离。
2.清晰的语法定义，减少维护成本，较强的代码可读性。
3.不需要创建第三方来监听事件和传输数据。
4.一个控制器可以实现多个代理，满足自定义开发需求，可选必选有较大的灵活性。

缺点:
1.实现委托的代码过程比较繁琐。
2.当实现跨层传值监听的时候将加大代码的耦合性，并且程序的层次结构将变的混乱。
3.当对多个对象同时传值响应的时候，委托的易用性将大大降低。

2.NotificationCenter（通知）
通知也是iOS开发中常用的一种传值响应方法，例如在AVFoundation框架中的MPMoviePlayerController在监听播放状态改变，播放停止等事件时使用的也正是NotificationCenter。官方给出的解释如下：

>An NSNotificationCenter object (or simply, notification center) provides a mechanism for broadcasting information within a program. An NSNotificationCenter object is essentially a notification dispatch table.

>Objects register with a notification center to receive notifications (NSNotification objects) using the addObserver:selector:name:object: or addObserverForName:object:queue:usingBlock: methods. Each invocation of this method specifies a set of notifications. Therefore, objects may register as observers of different notification sets by calling these methods several times.

>Each running Cocoa program has a default notification center. You typically don’t create your own. An NSNotificationCenter object can deliver notifications only within a single program. If you want to post a notification to other processes or receive notifications from other processes, use an instance of NSDistributedNotificationCenter.

NSNotification采用的单例设计模式，当给通知中心注册一个key以后，那么无论在什么地方只要给通知中心发送一个这个key的消息，那么就实现了通信传值，注册的通知的对象就会调用相应的方法。

优点：
1.使用简单，代码精简。
2.解决了同时向多个对象监听相应的问题。
3.传值方便快捷，Context自身携带相应的内容。
缺点：
1.使用完毕后，要时刻记得注销通知，否则将出现不可预见的crash。
2.key不够安全，编译器不会监测是否被通知中心正确处理。
3.调试的时候动作的跟踪将很难进行。
4.当使用者向通知中心发送通知的时候，并不能获得任何反馈信息。
5.需要一个第三方的对象来做监听者与被监听者的中介。

3.Block（代码块）
Block是iOS4.0+ 和Mac OS X 10.6+ 引进的对C语言的扩展，用来实现匿名函数的特性。苹果官方对于Block的解释也是十分的简洁：

>A block is an anonymous inline collection of code, and sometimes also called a "closure".Blocks are a powerful C-language feature that is part of Cocoa application development. They are similar to “closures” and “lambdas” you may find in scripting and programming languages such as Ruby, Python, and Lisp. Although the syntax and storage details of blocks might at first glance seem cryptic, you’ll find that it’s actually quite easy to incorporate blocks into your projects’ code.

翻译过来就是，闭包是一个能够访问其他函数内部变量的函数。Block的使用在这里也不在赘述（[不知道的同学可以点这个链接，我觉得写的很详细](http://blog.csdn.net/totogo2010/article/details/7839061)）。不论是原生的框架还是第三方的框架，我们都可以看到Block的身影，Block强大的功能性和易用性使得深受架构师的喜爱。

官方也有一篇简短的指导文档,有兴趣的同学可以看一下（[官方指导](https://developer.apple.com/library/ios/featuredarticles/Short_Practical_Guide_Blocks/index.html)）。值得注意的是，在这份文档中也提到了几种Block的使用场合。

* 任务完成时回调处理
* 消息监听回调处理
* 错误回调处理
* 枚举回调
* 视图动画、变换
* 排序

优点：
1.语法简洁，实现回调不需要显示的调用方法，代码更为紧凑。
2.增强代码的可读性和可维护性。
3.配合GCD优秀的解决多线程问题。

缺点：
1.Block中得代码将自动进行一次retain操作，容易造成内存泄露。
2.Block内默认引用为强引用，容易造成循环引用。

## （二）注意事项

1.代理
<ul>
	<li>在代理中调用方法的时候使用的是系统的子线程，因此，当使用Delegate进行UI操作的时候，必须调用GCD的主线程方法：
dispatch_async(dispatch_get_main_queue(), <^(void)block>)，在block中写进行的UI操作代码。</li>
</ul>

2.通知
<ul>
	<li>在通知的使用过程中Crash的原因很多情况都是注册观察者以后没有及时的注销观察者，当然这个情况在非ARC时代比较常见，但是这并不是说在现在的ARC时代就不会出现这个问题。往往一旦出现问题就很难追查，所以还是要养成及时注销的习惯。</li>
	<li>由于通知的使用极其简单，往往能够看到很多开发人员在开发过程中滥用NSNoticationCenter的现象。导致到处都是乱七八糟的通知，代码的可维护性和可读性非常差，即便是使用了宏定义也不能完全避免这些问题。比如在Debug的时候，当存在多个Observe的时候，简直就是要人命的感觉，想死的心都有了。在这方面Block和Delegate比Notification强太多了。</li>
</ul>

3.Block
<ul>
	<li>当在Block中引用某个外部变量的时候，Block内部只会进行只读拷贝，这也就意味着，即便你在使用Block之前修改了那个外部变量的值，那么在你使用的Block里面它的值依旧是最开始的那个外部变量的值。如果想要同步外部变量的值，那么就需要在block内部引用变量时，在前面加上__block关键字。</li>
	<li>block本身是像对象一样可以retain，和release。但是，block在创建的时候，它的内存是分配在栈(stack)上，而不是在堆(heap)上。他本身的作于域是属于创建时候的作用域，一旦在创建时候的作用域外面调用block将导致程序崩溃。</li>
</ul>


## （三）使用场景对比

1.回调方法
在日常的开发过程中，我们经常会遇到一些完成之后的处理问题，比如完成网路请求之后的回调，或者页面加载完成之后的回调等。这个时候我们一般使用的是前两者方法，即Block或者Delegate。而在一对一传输回
调的时候明显Block的使用更加的简单高效，只需要在代码块中执行所需要的操作即可。在一对多的情况下，Delegate更加能够发挥出自己的优势。

2.跨层通信
有的时候我们需要实现在两个毫无关联的对象之间的通信，这个时候如果使用Block或者Delegate就势必会增加代码的耦合性，这样对于代码的结构来说是不健康的，因此这个时候使用Notification便是明智的选择。

3.UI响应事件
用户在于App的UI进行互动的时候，总会需要App进行交互响应，这个时候就毫无疑问的使用代理设计模式。而苹果官方给出的建议也是可以肯定的，在Cocoa Touch框架中我们也可以在几乎所有的UI交互控件的头文件里看到Delegate的成员变量，也正是印证了在UI响应事件上Delegate有着绝对的优势。

4.简单值得传递
当需要进行简单值得传递的时候，比如子控件传输给父控件所点击的IndexPath的时候，更加适合使用Block来传值。因为，如果只是为了传这一个简单的值而没有特别的业务处理而定义一个协议，然后实现协议，设置代理再写方法的话将十分麻烦，得不偿失，这个时候简单高效的Block就可以完美的替代Delegate完成任务了。