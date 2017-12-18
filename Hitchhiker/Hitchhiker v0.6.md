Hitchhiker 是一款开源的支持多人协作的 Restful Api 测试工具，支持Schedule, 数据对比，压力测试，支持上传脚本定制请求，可以轻松部署到本地，和你的team成员一起协作测试Api。

详细介绍请看： [http://doc.hitchhiker-api.com/cn/introduction.html](http://doc.hitchhiker-api.com/cn/introduction.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试和上传js，虚拟机单核的，撑不住）。

## 下面来看看这次的更新：

## 重写压力测试

Hitchhiker 之前的压力测试节点是一个基于Go的 [Hitchhiker-Node](https://github.com/brookshi/Hitchhiker-Node)，早期阶段Hitchhiker的脚本功能并不复杂，不支持上传js库，async/await，以及文件读取保存等，而Go的高并发非常有吸引力，做了下调研后，使用otto做为js解释器，是可以满足那时的脚本运行逻辑的，所以选用了Go做压力测试的节点，在早期是够用的。

后来Hitchhiker开始支持更多复杂的脚本功能，比如自定义js库，因为npm里的很多js库都基于Nodejs，而目前的Go以及otto满足不了这种需求，除非再加一个Node进程来执行脚本，然而这样又过于复杂，还不如直接使用Nodejs来写，所以综合考虑后还是使用Nodejs重写了压力测试点。

重写之后的压力测试是集成在Server一起的，也就是不用再部署其他的程序，而且支持现有所有的脚本功能。

### 两种方法的优劣：

Go的高并发以及goroutine使得写起这种压力程序时非常之轻松，性能也很有保障，缺点还是在于Hitchhiker的脚本是js，所以Go执行这些脚本比较费劲，也因此目前基于Go的压力点不支持Hitchiker脚本的高级特性。

Nodejs写这种压力测试程序就比较费劲，需要自己管理多进程，以及进程间通信，还没法精确控制1秒的请求数，也就是压力测试的参数QPS对Nodejs的压力点是没用的，不过好在Hitchhiker Server也是基于Nodejs的，所以可以重用请求处理的逻辑，而且Api的压力测试本质上是高IO的，所以Nodejs的性能也很不错。不过Nodejs的程序目前还不支持分布式，稍后会加上去，主体功能已经完成。

稍微比较了下两者的性能，在单机上基本旗鼓相当。

目前是以基于Nodejs的版本为默认的，也可以选用Go的，不过Go的暂时会停止维护，除非Go有了基于Node的js解释器，那时再考虑移回来。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/stresstest.gif)

## 重新整理请求流程

之前的请求流程有点乱，导致有些问题不容易发现，比如环境变量没应用到Test脚本里，所以在改这个bug时重构了下代码，把流程理清下：

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/script/reuqest_wf.png)

## response 展示图片

这个是有朋友在github上提出来的，之前我是想不到有人会用这个工具来请求图片，所以也没关注这块，不过有人使用，说明有需求（不止一人），所以实现了这个功能，如果response header有`image/*`的话就直接展示图片而不是图片内容（一片乱码）

## 修改Bug

1. global function 里的内容在切换模块后会消失

2. schedule里的请求返回是图片时，会造成JSON.parse失败，导致异常，改了图片只保存链接，不保存内容

3. 浏览器里压力测试的websocket有时会失败，加了重试

4. schedule的定时跑的记录会有1分钟左右的误差

5. 改请求的method时name会被重置

## 后续计划

短期内还是以增加测试新功能为主，比如curl生成请求，请求生成代码等，长期的一个是文档，一个是Mock，开始根据需求来决定下一个模块。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。