Hitchhiker 是一款开源的 Restful Api 集成测试工具，支持Schedule, 数据对比，压力测试，可以轻松部署到本地，和你的team成员一起管理Api。

详细介绍请看： [http://www.cnblogs.com/brookshi/p/7440663.html](http://www.cnblogs.com/brookshi/p/7440663.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试，虚拟机单核的，撑不住）。

## 这次发布主要增加一个增强协作的功能 - 自动同步更新：

## 自动同步更新

我们写code时通常会用git或svn等工具来协同工作，但是Api case也用这种方式的话就显得有点麻烦了，一个接口的属性毕竟就那个几个，没必要修改前fetch & rebase，修改后还要push，Api的协作应该更简单，相信很多人用过Atlassian的wiki，我们在编辑文档的时候常常会收到提醒：某某更改了此文档，是否合并 之类，API的协作也应该这样，简单方便，所以就有这次的更新：

默认每30s会同步一次，有三种表现：
1. 本地没有修改的API，这时数据会自动更新。
2. 本地编辑过的，也就是tab上显示上红点的，这时如果别人更改了API，数据同步后tab里仍会保持编辑的数据，但是会提示些API有人更改过，可以view changes来看是被谁改了些什么，然后决定是否覆盖或放弃本地内容。
3. 远程上面被删除的，同步会提示此API已经被删除掉了，也就是说再在上面更改已经没有意义，可以关掉此API了。

下面的图片展示了同步过程：
1. 首先有两个人在同时维护，左边一个(chrome)，右边一个(firefox)，可以看到左边建立了一个Collection和一个request，右边马上得到了更新。
2. 然后左边更改了url，在后面加上?a=A，同时右边也做了更改，在url后面加上了?b=B并保存，这时左边得到了case被改的提示，view changes看了更改的内容，选择了覆盖，所以右边的也同步成?a=A了。
3. 左边把case删掉，右边得到case被删的提示。

图中的时间间隔设为了5秒，所以会比较快
![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/sync.gif)


## 其他改动

1. Url Query支持中文

## 后续计划

下个版本的目标是 pre request script以及项目folder，实现初始变量数据源以及在脚本中保存或打开文件的功能，可以借此来实现[动态参数输入源](https://github.com/brookshi/Hitchhiker/issues/29)

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。