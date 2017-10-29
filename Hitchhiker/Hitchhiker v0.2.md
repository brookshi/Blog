Hitchhiker 是一款开源的 Restful Api 集成测试工具，支持Schedule, 数据对比，压力测试，可以轻松部署到本地，和你的team成员一起管理Api。

详细介绍请看： [http://www.cnblogs.com/brookshi/p/7440663.html](http://www.cnblogs.com/brookshi/p/7440663.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试，虚拟机单核的，撑不住）。

## 这次发布的算是一个大版本，主要增加一个重磅功能 - 压力测试：

## 压力测试

双11快到了，经常会有整点秒杀的活动，秒杀就是一个典型的压力场景，所以建了一个简单的Case来表现这种场景，来展示Hitchhiker压力测试功能：
![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/stresstest.gif)

Hitchhiker使用一个基于Golang的分布式压力节点，这是一个单独的项目：[Hitchhiker-Node](https://github.com/brookshi/Hitchhiker-Node)。得益于Golang的交叉编译，轻松跨平台生成文件，所以只有一个可执行文件和一个配置文件，没有环境依赖，直接执行。

使用时在[release页面](https://github.com/brookshi/Hitchhiker-Node/releases/tag/v0.1)先选择对应平台的zip文件下载下来，解压后会有两个文件，一个可执行文件和一个配置文件config.json，打开配置文件，把`Address`的值从localhost改为部署Hitchhiker机器的ip，然后再执行Hitchhiker-Node文件，这样就弄好了一个压力点。

如果想压出很大的请求就可以考虑部署到多台机器上，Hitchhiker会自动根据机器的CPU核数来分配任务，当然，一般情况下直接部署到Hitchhiker同一台机器就够用了。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/stress_config.PNG)
压力测试用的也是`Collection`的`Request`，可以选择性的挑出合适的`Request`用来做Case，压力测试的参数有：
> - Repeat: 运行整套请求的次数
> - Concurrency: 并发个数
> - QPS: 1秒内限制单个节点请求的个数，默认为0，即没有限制
> - Timeout: 请求的超时时间设置，单位为秒，默认为0，即没有超时设置
> - Keeplive: 设置请求是否使用Keeplive

运行压力测试任务时会实时显示运行状态，包括节点的状态（闪烁表示正在工作），当前任务及任务的数量，下方有三个图表，分别表示 
1. 当前的运行进度，包括完成的数量及TPS
2. 各个`Request`的请求消耗时间，包括 DNS, Connect, Request, Min, Max 这五个
3. 请求失败的状态，包括 No Response, Server Error(500), Test失败 这三种情况

## 其他改动

1. 源码部署时支持改端口，之前固定用的8080，要改需要改js文件，现在只需在部署文件时改就好了。

2. 改正Schedule空跑时的异常。

## 后续计划

压力测试在国庆后总算做出来，后来又花了一些时间来测试，0.2这个版本算是告一段落。
接下来版本计划要改下，涉及新功能的都是大版本，bug是小版本。
下个模块功能是支持API文档，希望能是一个自定义的，所见即所得，支持导出常用格式的API文档系统。
小功能和bug会持续改进。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。