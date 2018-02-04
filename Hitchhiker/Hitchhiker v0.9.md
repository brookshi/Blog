v0.9是Hitchhiker在2017农历年的最后一个版本，而起点正是刚过完2016农历年，农历2018即将到来，一年轮回，今天写点东西稍微回顾下hitchhiker的2017。

先还是说v0.9，这次版本发布主要带来一个新的辅助测试功能：免脚本的断言测试，这是一个携程的朋友提出来的需求。

之前Hitchhiker支持在test脚本里写 tests['assert'] = value 这样来断言，但很多QA其实并不会编程，或者会其他语言但对js不熟，这样断言写起来就不太方便，所以这次应朋友的需求加了这个功能：

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/assert.gif)

上面动图已经展示了功能和用法，具体就不多说了。

回头看下Hitchhiker的2017，一年过来，对这个项目来说结果还不错，大小版本发了14个，github上有了1k+的star，我也因此认识了一些朋友，对技术上有也不少提升，总体看对我来说是成功了。

[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/contributor.png)

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/2017.png)

起初，大概是2016年年中，我开始负责公司一个API项目，因为是金融公司，对数据准确性要求很高，所以产生想法，做一个工具来辅助这个API项目的测试，减少沟通成本以及QA做regression时的压力。后面准备了下，在2016年农历年后，也就是17年的3月份，正式开始编码实现功能。

由于不懂设计，所以UI上参考了比较熟悉的一个成名已久的测试工具：Postman，这也导致：即使后来除了UI外，实现了很多Postman没有的功能也还是摆脱不了Postman的影子，不少人一看跟Postman一样，觉得没有意义，在这点上算是一个败笔。不过也因为类Postman UI的易用性，让使用Hitchhiker的人很容易上手，这又是一大优势，算是两者抵消吧。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/collection.png)

当时，想要通过这个工具解决的问题只有2个：

1. 减少开发的沟通成本，原因是我们的API是面向用户的，依赖公司其他Team的众多API，我们写一个接口可能要调用公司好几个API才能整合出想要的数据，这就需要开发去和好几个team打交道，沟通成本很大。而如果要所有开发都做一遍同样的事情，浪费的时间可想而知。

2. 减少浪费QA人力做无聊的数据对比，这个算是自动化的一部分，上面说了，金融数据的准确性是非常关键的，我们的产品又是直面用户的，有问题第一个找到我们头上，所以QA在这方面也非常头痛，以往都是依赖人眼去对比线上和UAT两个版本的报表是否匹配，容易疏忽不说，时间有效的情况下，覆盖率也很难达到要求，且对QA来说，这类事情是最应该自动化的。

解决这2个问题的方案是：

1. 很多工具需要互相share，有更新就share的话也很麻烦。 Hitchhiker支持多人同时在线维护同一份API，支持实时更新，一个开发在完成沟通后，把依赖的API都整理在一起，写好case，其他开发就可以直接借鉴使用了，只花一个人的时间，成果所有开发共享。

2. 使用Schedule来实现Case的自动化运行，以及用脚本做断言来判断数据是否正确，但金融数据上经常有动态值，比如求上个月的回报，对今天来说，上个月是1月，但过一个月后，上个月就是2月了，数据很可能就不一样了，所以对这类动态值用断言方式很难解决，Hitchhiker支持在做自动化测试时对比不同环境的数据，我们以线上的数据为准的话就可以知道没上线环境的API运行是否正常了。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/schedule.png)

这两个功能在17年7月左右先后实现，我的API项目的接口测试也陆续加了进去，基本上满足了需求。

由于项目的API的并发量比较大，在服务器有限的情况下，需要尽量提前优化来提高吞吐，避免上线后出问题，所以需要在测试阶段给到服务器压力，然后在10月份时用Go语言为Hitchhiker实现了压力测试。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/stresstest.gif)

在0.5版本时用gitbook重写了文档： [Hitchhiker使用文档](http://doc.hitchhiker-api.com/cn/)

接下来的一个版本又大幅加强了脚本功能，支持require，支持上传脚本库和数据文件，标志着 NPM 里几十万的js库尽可以拿来用了。

不过可惜的是基于Go语言写的压力测试由于对js支持有限，不得不放弃，转而使用Node重写了一份压力测试的功能并在v0.6版本上线。

其实到这时为止，Hitchhiker已经满足我的API项目的需求了，但随着使用者越来越多，需求不断出现，后续的版本基本都在实现这些需求了：

v0.7：支持自定义smtp，为请求生成各种语言的code，schedule数据不同时的diff展示

v0.8: 自动化测试结果统计

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/schedule/statistics.png)

v0.9: 基于UI的断言测试

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/assert.PNG)

还有很多功能想要实现，文档，Mock，管理平台等等，将会在接下来的2018里陆续实现。

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试和上传js库，虚拟机单核的，撑不住）。