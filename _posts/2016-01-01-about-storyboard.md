---
layout:     post
title:      "关于Storyboard你也许不知道的小秘密"
subtitle:   ""
date:       2016-01-01 14:30
author:     "Maru"
header-img: "img/post-bg-01.jpg"
tags:
    - iOS
---

## （一）前言
  最近放纵的过了头，笔记好久没有写了，然而LOL的段位并没有并没有因为多包宿而上升，真是个令人悲伤的故事。Xcode总有一些很少用到隐藏很深的小秘密，今天我们就来扒一扒Storyboard，让哥几个爽一爽。

## （二）正文

### 1.Runtime Attribute

> Runtime大法好呀！

我们都知道，Objc是一门动态语言，而这一切都要归功于强大的Runtime库。在日常的使用过程中，我们打交道最多的就是属性面板了（Attributes inspector），改背景颜色啊，改Lable内容啊等等。它非常的方便是没有错，但是它也有局限性。比如设置圆角，在属性面板中并没有这一选项。但是通过Runtime Attribute我们就可以完美的解决问题。当然啦，属性面板能够做的事情，它自然也是不在话下了。

只需要轻轻一点右下角的加号，然后就像代码中KVC的使用方法那样，填上KeyPath，填上Value，一切搞定！

![Attributes inspector](http://upload-images.jianshu.io/upload_images/791315-efabc3992cc557d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)


![Storyboard的样式.png](http://upload-images.jianshu.io/upload_images/791315-d09b47e5c42d2323.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)


![运行后的样式](http://upload-images.jianshu.io/upload_images/791315-5f92862ddaa2dcce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)


### 2.Global Tint

有的时候我们有这样的需求，在Storyboard中所有的Tint Color都设置为其他的颜色，而不是默认的蓝色。那么，我们如果每添加一个控件就在属性面板中设置这种办法实在是太Low了，好在Apple为我们考虑到了这一点，在Storyboard中的File inspector中的Global Tint为我们完美的解决了这一点。为该属性设置了颜色以后，该Storyboard中的所有控件的Tint Color都变成了你所设置的颜色，比如UIButton的字体颜色、UIProgress进度条的颜色等等，是不是方便了很多呢？


![Gobal Tint.png](http://upload-images.jianshu.io/upload_images/791315-82ec5d03f344447b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![6E9F51DE-7C84-48EE-9C30-AF13EB6BD41B.png](http://upload-images.jianshu.io/upload_images/791315-37c7cb034ba1f4ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

### 3.Outlet Collections

Outlet是iOS“拖线”开发中最基础也是最常用的操作了，相信大家也已经用的非常熟练了。Outlet Collections从名字上面我们就可以知道，它是Outlet的一种集合。举个例子吧：有一个UI界面中有很多的UIButton，业务逻辑上他们属于同一块，然后你要在.m中获取他们的引用，原来的方法是一个一个的给他们拖线，命名，然后再拖线...但是有了Collection以后世界就不一样了。步骤如下：


![点击鼠标右键，出现的那个黑色HUD.png](http://upload-images.jianshu.io/upload_images/791315-2ce134ad1b3e8f39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![拖线设置集合名称.png](http://upload-images.jianshu.io/upload_images/791315-21aac11abe471bd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其他几个的控件也是相同操作拖线到该Collections。我们可以看到，得到的引用是一个该控件类的数组集合。也就是说，不同类型的组件不能通过Collections来集合获取。


![屏幕快照 2016-01-01 下午2.30.21.png](http://upload-images.jianshu.io/upload_images/791315-70218e3097741cba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

OK啦，最后我们来测试一下获取那几个Button。


![屏幕快照 2016-01-01 下午2.36.03.png](http://upload-images.jianshu.io/upload_images/791315-d3ac889b40242462.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 4.Storyboard Reference

在使用Storyboard开发中，最为人诟病的可能莫过于多人开发了，唐巧当年在一篇博文中将其称之为“灾难”（详情可见他的个人博客）。机智的苹果也看到了这个的问题，于是Storyboard Reference就出现了。它将一个原本庞大的Storyboard分发成多个子Storyboard，在父本中只能看到子体的引用。在多人开发的过程中，每个人修改自己的那个子体，那么就不会有那么多蛋疼的事情了。


![选中你所要重构的控制器，然后选择Editor->Refactor to storyboard.png](http://upload-images.jianshu.io/upload_images/791315-65e0dc83e3c00698.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![设置新的Storyboard的名字.png](http://upload-images.jianshu.io/upload_images/791315-9710c02fdcaf0701.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![结果.png](http://upload-images.jianshu.io/upload_images/791315-b8150bc4ee2fb66d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然啦，每个人的使用习惯不一样。你也可以选择先新建一个Storyboard，然后再Main.Storyboard中添加一个Storyboard Reference控件，然后在属性面板设置关联，就像下面那样。


![7ADABE74-D7B5-4979-9655-556E009A2968.png](http://upload-images.jianshu.io/upload_images/791315-981c17c8c37c2a8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5.对齐相关

用过PS的人应该都知道辅助线的作业，当然万能的Xcode也是提供了辅助线的功能的。单击某个控件，按下shift+Command+-添加横向辅助线，shift+Command+|添加纵向辅助线，添加的位置都是左右/上下居中的。如果你想要删除辅助线，那么把它拖到最边边就可以啦！


![屏幕快照 2016-01-01 下午3.35.42.png](http://upload-images.jianshu.io/upload_images/791315-3d6f2103465f90ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有的时候我们需要快捷的知道某个控件上下左右的距离，那么我们只需要选择相应的控件,然后按下Command就可以啦，当你鼠标移动到其他的控件的时候，也会智能的显示出与该控件的位置距离参数。如下图:


![距离标注.gif](http://upload-images.jianshu.io/upload_images/791315-0a2ae753746725d1.gif?imageMogr2/auto-orient/strip)



## （三）结语

Storyboard刚出生的时候，它确实有这样那样的缺陷，但是经过几年的发展，苹果不断的改进，它已经成为iOS快速开发不可或缺的重要工具。工程师的时间并不应该浪费在那些无聊乏味的代码上，在这一点上Storyboard一直未忘初心。

关于Storyboard的小技巧我相信还有很多，小伙伴们有什么新发现还记得多多分享哦！

最后的最后，祝大家2016年写出越来越精致的代码，我们Github上见！