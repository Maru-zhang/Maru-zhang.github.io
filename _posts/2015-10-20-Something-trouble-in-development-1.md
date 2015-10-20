---
layout:     post
title:      "开发中遇到的那些莫名其妙的坑-第一弹"
subtitle:   ""
date:       2015-10-20 00:58
author:     "Maru"
header-img: "img/post-bg-01.jpg"
tags:
    - iOS
---


iOS开发中遇到的那些莫名其妙的坑
===========================

>写在前面的话：在苦逼的iOS开发过程中，总会碰到各种各样莫名其妙的问题，有的时候由于Apple官方的限制，有的时候又由于技术因素，总会有各种各样的问题困扰着自己。所以我决定把一些让我很生气的坑记录下来，留作纪念和参考，不定时更新。

1.AVAudioRecorder的Duration问题

在进行Yinner的开发过程中，用到了系统原生的AVAudioRecorder类来进行真机录音，模拟器上调用`- (BOOL)recordForDuration:(NSTimeInterval) duration;`的时候完全没有问题，监听录音完成的代理也可以通过代理方法检测到。然而奇怪的是当使用真机的时候代理方法完全没有响应，通过断点最后定位到这一句代码。最后换了一种思路，利用NSNotificationCenter来控制和监听录音的始末动作。

2.AVAudioRecorder的真机测试无声问题

在进行Yinner的开发过程中，利用原生的AVAudioRecorder进行音频录制并且合成视频。然而，在模拟器上完全没有问题的代码，在进行真机测试的时候就发现变成了“无声电影”。经过查找，发现原因是没有激活真机的硬件音频会话层，官方解释如下：

>Checks to see if calling process has permission to record audio.  The 'response' block will be called immediately if permission has already been granted or denied.  Otherwise, it presents a dialog to notify the user and allow them to choose, and calls the block once the UI has been dismissed.  'granted' indicates whether permission has been granted.

因此只需要添加如下代码，激活会话层即可。
{% highlight ruby %}
	AVAudioSession *session = [AVAudioSession sharedInstance];
    NSError *sessionError;
    [session setCategory:AVAudioSessionCategoryPlayAndRecord error:&sessionError]；
    if(session == nil)
        NSLog(@"Error creating session: %@", [sessionError description]);
    else
        [session setActive:YES error:nil];
 {% endhighlight %}

3.UIGestureRecognizer重用的无效问题

这是最近在进行Yinner侧滑开发的过程中遇到的一个小问题。在侧滑菜单打开和关闭的时候要用到相同的UIGestureRecognizer对象和一个相同的方法- (void)onPanGuestureRecginizer:(UIPanGestureRecognizer *)pan。而在主界面侧滑缩放以后，需要给主界面添加一个蒙版遮罩，以突出侧滑菜单的重点。这样的情况就是，侧滑菜单在打开的时候主界面需要一个Pan手势，而在侧滑菜单缩放回去的时候主菜单上面的蒙版图层又需要另外一个Pan手势，并且这两个手势所调用的方法是同一个因此，我自作聪明，将这个手势对象设置为全局变量，在侧滑菜单出现后给蒙版也使用addGestureRecognizer方法添加同一个手势对象。这样所造成的问题是，当侧滑菜单消失后，主界面的蒙版也随之消失，而这一个手势对象只能添加在一个VIew中，因此，主界面原来的手势就消失了，导致莫名其妙的手势方法不进入的问题。真是蠢= =。

4.Navgation下的UITableView存在偏移的解决方法

最近在开发中遇到一个小问题，当一个UITableview嵌套在一个NavgationController上的时候，会发现，明明Tableview的frame以及其他的各项设置没有问题，但是显示的时候还是会存在TableView的向下偏移。后来发现，这个是ios7开始有的一个新的属性，目的是，如果视图中存在唯一一个scrollView，那么它会设置相应的内边距，这样可以占据整个视图。所以只需要把UITableView的automaticallyAdjustsScrollViewInsets属性设置为NO，就能解决这个问题。

5.iOS中两种退出触摸键盘的简单方法

比较常用的一种键盘失焦的方法是令UITextfiled对象调用resignFirstResponder。然而这个方法有一定的局限性，如果界面中存在多个UITextFiled输入框，那么你如果想要退出键盘你就需要多次调用上面的方法，这样会显得非常的繁琐。因此，可以的使用如下的方法。

第一种方法
{% highlight ruby %}
 [[[UIApplication
 sharedApplication] keyWindow] endEditing:YES];
 {% endhighlight %}

 第二种方法
 {% highlight ruby %}
 [[self findFirstResponderBeneathView:self] resignFirstResponder];
 
- (UIView*)findFirstResponderBeneathView:(UIView*)view 
{
    // Search recursively for first responder
    for ( UIView *childView in view.subviews ) {
        if ( [childView respondsToSelector:@selector(isFirstResponder)] && [childView isFirstResponder] ) 
            return childView;
        UIView *result = [self findFirstResponderBeneathView:childView];
        if ( result ) 
            return result;
    }
    return nil;
}
{% endhighlight %}
