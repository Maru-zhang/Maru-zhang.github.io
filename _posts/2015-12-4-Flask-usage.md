---
layout:     post
title:      "如何用Flask快速搭建你APP的接口"
subtitle:   ""
date:       2015-12-04 16:30
author:     "Maru"
header-img: "img/post-bg-01.jpg"
tags:
    - Python
---


### (一)前言

最近在做自己的全栈项目的时候，免不了要自己做一个接口。作为一名菜鸟的后端码农，刚开始用的是MyEclipse 10 + Servlet来构建自己的接口，虽然这个IDE已经给我提供了极大的便利（[原来的作法](http://maru-zhang.tk/2015/11/21/Full-Stack-2/)），但是当我知道Flask的时候我就知道：我已经回不去了。

### (二)万年不变

#### 1.简介

在Python浩如烟海的框架中，有两款著名的Web框架，一个是[Django](https://github.com/django/django)，另一个就是[Flask](https://github.com/mitsuhiko/flask)。正如Flask自身的介绍一样：“A microframework”，相比较于Django它更加的轻量与灵活，更加适合于个人定制。类似于接口之类的比较迷你的需求，Flask更加适合，并且完全胜任。

#### 2.安装
安装是最没有意思的步骤了，网上教程一大堆，所以博主就不再赘述了。小伙伴们可以看[这里](http://dormousehole.readthedocs.org/en/latest/installation.html#virtualenv)。


### (三)简单初识

现在，你的已经安装完Flask了，下面让我们来初步使用一下它。

万物万事初始于“Hello world！”。

    from flask import Flaskfrom

    app = Flask(__name__)

    @app.route('/')
    def start():
      return 'Hello world！'

    if __name__ == '__main__': 
      app.run()

这就是一个最小的Flask程序，是不是简单到爆炸？route装饰器直接指定了URL，后面的方法就是URL对应的方法（这里是start方法），直接一个return就返回了响应的内容。就是这么简单，就是这么粗暴。

> 访问127.0.0.1:5000我们就可以看到响应结果了

### (四)构建接口

#### 1.目标

在原来的服务端的功能中主要有三大功能：

* 根据页码和影片类型参数，响应一个视频列表JSON数据；
* 根据页码，响应一个新闻列表JSON数据；
* 根据关键词，响应一个相应的数据列表的JSON数据；

由于其实这三个功能从实现逻辑上来讲大同小异，所以本篇文章只以第一个功能作为例子进行讲解。

我们先来看一下数据库中的数据:

![屏幕快照 2015-12-04 下午2.11.08.png](http://upload-images.jianshu.io/upload_images/791315-ecdf75e3dddf26f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里需要解释一下的是leve1和leve2，leve1是视频的一层分类，包括动漫、纪录片，学习等，而leve2则是在leve1之下的细分。

博主计划的SC交互过程是这样的：客户机发送一个POST请求，请求中包含index，leve1和leve2，参数。如果leve2不为空，那么根据leve2和index来查询数据库，然后根据固定的json返回格式返回给客户端，就像这样：

![屏幕快照 2015-12-04 下午2.21.32.png](http://upload-images.jianshu.io/upload_images/791315-5728e42e80f2e25e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.构建

首先，我们需要获取客户机POST请求中的参数，在Flask中我们可以轻松而又优雅的获取到：

     index = request.args.get('index')

当然我们也是需要考虑一个情况，那就是如果客户机没有传递这个参数那怎么办？Flask也已经替我们考虑了这个情况，当发生诸如此类的情况时，Flask会抛出一个KeyError的异常，我们只需要try...except处理，返回一个表达错误的JSON就可以啦！

OK！在写SQL语句之前我们还需要一个步骤，由于APP需要分页功能，所以我们还需要计算页码，在外部我们先定义一个单请求返回数据数（一个请求返回多少条数据）：

    pageCount = 20

现在让我们来计算我们需要从哪一个数据开始查询数据库：

    pageIndex = int(pageCount) * int(index)

> 注意：这里如果不进行类型的转换的话会发生一个错误，当index=0的时候是没有问题的，pageIndex=0，但是当index=1时，pageIndex=111111111111，那就不对了。

现在我们来编写SQL语句，当leve2是空字符串的时候：

    sql = 'select title,url from FreeVideo where leve1="%s" limit %s,%s' % (leve1,pageIndex,pageCount)

当leve2不是空字符串的时候：

    sql = 'select title,url from FreeVideo where leve2="%s" limit %s,%s' % (leve2,pageIndex,pageCount)

接下来复用之前爬虫用的mysql类来进行数据库的连接和查询[mysql的代码](http://maru-zhang.tk/2015/10/26/Full-Stack-1/)：

    db = mysql.Mysql()
    db.queryData(sql=sql)

获取结果集进行遍历，并且放入响应结果数组中:

    result = []
    for item in  result_mysql:
      ji = {"title": item[0],"url":item[1]}
      result.append(ji)

由于返回的JSON数据都包括三个字段：1.datas，这个字段包含返回的数据。2.msg：包含服务器返回的提示信息。3.success：包含操作结果成功还是失败（BOOL类型）。因此我定义了一下基础返回响应的函数：

    def getBaseReturnValue(data,msg,code):
      json_data = json.dumps({'datas':data,'msg':msg,'success':code},ensure_ascii=False,encoding='gb2312')
    return json_data

> 注意：由于返回的数据中包含中文的数据，因此在运用json模块中的dumps方法时必须加上encoding参数，否则服务器会报错！


终于，最后一步，根据响应数据的长度决定返回的数据：

    if len(result) == 0:
      return getBaseReturnValue(data=result,msg="没有更多数据!",code=False)
    else:
      return getBaseReturnValue(data=result,msg='OK',code=True)

### (五)结语

我们来看一下最后的APP使用接口的效果：


![Achievement.gif](http://upload-images.jianshu.io/upload_images/791315-e684886f0a8c568b.gif?imageMogr2/auto-orient/strip)


![SearchAchievement.gif](http://upload-images.jianshu.io/upload_images/791315-0cfef31a2f84a5f8.gif?imageMogr2/auto-orient/strip)

最后的最后，github地址附上，所有的代码都在这里！[iCCUT-Server-Flask](https://github.com/Maru-zhang/iCCUT-Server-Flask)