---
layout:     post
title:      "关于一个下拉刷新BUG的一些思考"
subtitle:   ""
date:       2015-11-16 20:20
author:     "Maru"
header-img: "img/post-bg-01.jpg"
tags:
    - iOS
    - Objective-C
---

## 关于一个下拉刷新BUG的一些思考

>世界上没有什么事儿是一顿烧烤不能解决的。如果有,那就两顿。——亚洲气质舞王尼古拉斯 赵四

### 1.BUG表现
在最近项目中，关于网络请求那部分，还是毫无意外的用到了MJRefresh和AFNetworking。然而这个被玩烂的框架却被我用出了新的BUG：当你以正常人类的速度下拉刷新的时候（就是手指按住屏幕，往下移动，然后放开），刷新正常工作没有任何问题。然而，当你以极快的速度完成上述的动作的时候，奇迹发生了，Crash了！

![下载界面]({{ site.url }}/img/post-img/2015-11-16-0.gif)
![下载界面]({{ site.url }}/img/post-img/2015-11-16-1.gif)


###2.BUG分析
赶紧看看是什么问题

![下载界面]({{ site.url }}/img/post-img/2015-11-16-0.png)

![下载界面]({{ site.url }}/img/post-img/2015-11-16-1.png)

哦~数据源为空，所以在刷新的时候取值了一个空的数据源数组导致crash。等等，为空？怎么可能！我们先来看一下下拉刷新的方法：

{% highlight ruby %}

- (void)loadDataByAdd:(BOOL)isAdd {
    
    TSLog(@"刷新");

    if (!isAdd) {
        
        _pageIndex = 0;
    }else {
        
        _pageIndex++;
    }
    
    if (!isAdd) {
        
        [_tableData removeAllObjects];
    }
    
    NSString *url = [NSString stringWithFormat:@"%@/project/customProjectJSON!costDetailMobileList.action?", kHostUrl];

    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    formatter.dateFormat = @"yyyy年MM月";
    NSDate *date = [formatter dateFromString:displayDateString];
    formatter.dateFormat = @"yyyy-MM";
    
    NSDictionary *params = @{
                             @"userId" : kUserId,
                             @"date"   : [formatter stringFromDate:date],
                             @"pageIndex" : [NSString stringWithFormat:@"%li", _pageIndex],
                             @"pageCount" : [NSString stringWithFormat:@"10"]
                             };
    
    __weak typeof(_tableView) weakTableView = _tableView;
    
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.requestSerializer.timeoutInterval = 10;
    [manager POST:url parameters:params success:^(NSURLSessionDataTask *task, id responseObject) {
        
        TSLog(@"获取了数据");

        NSString *result = [responseObject objectForKey:@"success"];
        
        if ([result isEqualToString:@"0"]) {

            NSArray *array = [responseObject objectForKey:@"datas"];
            if (array) {
                for (NSDictionary *dict in array) {
                    XFJLModel *model = [XFJLModel objectWithKeyValues:dict];
                    [_tableData addObject:model];
                }
            }

        }else {

            _pageIndex = _pageIndex == 0 ? 0 : _pageIndex - 1;
            
        }

        [weakTableView.footer endRefreshing];
        [weakTableView.header endRefreshing];
        [weakTableView reloadData];


    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        
        _pageIndex = _pageIndex == 0 ? 0 : _pageIndex - 1;
        
        [weakTableView.footer endRefreshing];
        [weakTableView.header endRefreshing];
        [ZMAleart aleartwithVC:self addRemark:@"服务器连接失败,请检查网络"];
    }];
}


{% endhighlight %}

从上面的代码可以看出来，当isAdd为NO时，也就是下拉刷新的时候，在请求之前我们先移除了所有数据源的内容，然后再进行请求刷新。那么问题来了，按照这个顺序，为什么在<code>- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath</code>这个方法中拿到的数据源会是空呢？


### 3.BUG定位

是时候来一发DebugLog了，我分别在MJRefresh和TableView的刷新方法中标注了一些打印内容，现在让我们来看一下打印的结果：

![分类界面]({{ site.url }}/img/post-img/2015-11-16-2.png)

从图中我们可以看到，第一个框框是在没有发生bug的情况下的打印结果，一切顺序都是那么的完美，先清空数据（顺便提一嘴，图中打印刷新的时候是已经清空了数据源的数据的时候），然后请求回来数据，然后再重新加载。第二个框框是BUG发生时的打印结果，诡异的事情发生了，在还没有获取数据的时候竟然就开始更新了cell6和cell7， 而且并没有打印<code>- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section</code>中的内容（这里在图中对应数字17和27，是数据源的总数）,也就是说<code>- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath</code>并不是在reloadData的时候调用的，而是在刷新的时候，下方的TableviewCell移出屏幕又回到屏幕的时候调用的（从缓存池中加载cell）。

###4.BUG消除

由于个人水平的问题，我也不知道为什么相同的手势一快一慢会造成这样的执行顺序的问题，但是我知道的是，想要解决这个BUG必须防止在从缓存池中获取cell之前先清空的数据源。所以把<code>[_tableData removeAllObjects];</code>放在了AFNetworking的响应Block中，这样就没有问题了。
