Hitchhiker 是一款开源的支持多人协作的 Restful Api 测试工具，支持自动化测试, 数据对比，压力测试，支持脚本定制请求，可以轻松部署到本地，和你的team成员一起协作测试Api。

详细介绍请看： [http://doc.hitchhiker-api.com/cn/introduction.html](http://doc.hitchhiker-api.com/cn/introduction.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试和上传js库，虚拟机单核的，撑不住）。
 
## 下面来看看这次的更新：

## 自动化测试的统计视图

Schedule默认展示的视图是每次跑Collection的结果，这个表可以很方便看到每次测试的结果，有哪些成功，有哪些失败，失败的response，数据对比的结果等。 但是有时我们可能希望看到Collection下面每个请求在这一段时间内的运行状况，哪些request比较稳定，哪些会经常有问题，然后改进。

所以这次把自动化测试后每个请求的统计视图做出来了。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/schedule/statistics.png)

exculde depredated request选项, 默认是true，如果false的话会把曾经在这个Collection现在已经被删掉的记录也包含进来。

## 一次跑多个Schedule

有时做代码上做了更改之后想跑下这些测试，每个Schedule都点一下的话还是略显麻烦，现在给Schedule前面加了个checkbox，勾上的话会有一个Run Selected Schedules的按钮在上面显示出来，点这个按钮会一次跑所有勾上的Schedule，方便使用。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/schedule/runselect.png)

## 中断压力测试

因为可能在压力测试过程中服务端已经暴露出了问题，不需要再跑下去，这时可以停止当前压力测试。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/stress/stop.png)

## Step by step安装

Hitchhiker的部署一直是个头痛的问题，虽然支持docker很方便的部署，不过并不是所有人都会或者说愿意使用docker，毕竟很大一部分受众是测试，需要从他们角度来思考下，怎样简化部署。

这次先把包打好了，然后加了个setup的脚本在服务端运行，通过浏览器就可以完成一步一步部署了。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/setup.png)

## 其他小功能及bug fix

1. Schedule表某些列支持过滤。

2. Duplicate出来的environment的改动变影响到原始的environment

## 后续计划

短期内还是以继续增加测试新功能为主，比如基于UI的断言测试等。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。