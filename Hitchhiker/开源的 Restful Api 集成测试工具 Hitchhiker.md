Hitchhiker 是一款开源的 Restful Api 集成测试工具，你可以在轻松部署到本地，和你的team成员一起管理Api。

先上图看看：

<img src='https://github.com/brookshi/Hitchhiker/raw/master/doc/images/collection.png' width='800'/>

<img src='https://github.com/brookshi/Hitchhiker/raw/master/doc/images/history.png' width='800'/>

<img src='https://github.com/brookshi/Hitchhiker/raw/master/doc/images/env.png' width='800'/>

<img src='https://github.com/brookshi/Hitchhiker/raw/master/doc/images/schedule.png' width='800'/>

### 简单介绍

背景是Team在开发一些Api，这些Api依赖于其他Team的Api，依赖的Api是比较底层且比较大的，用起来不太方便且没有详细文档。

在开发Api的过程中有一个问题让我比较在意，我们Team是我先研究那个依赖的Api，过程不太容易，需要找文档，找那个Team的人协作，找case 等，研究了一些后做了一些东西，后面隔了一段时间开始陆续有其他同事参与进来，每进来一个都去研究一下那个Api，包括我做了其他事情后再回来开发Api时又得找资料熟悉下，这个过程造成了很大程度的时间和经验的浪费。

所以我觉得应该有款工具能让Team的人一起协作开发Api，和Code一样，每个人研究的东西可以保存下来方便其他开发，这就是开发Hitchhiker的第一个引子。

后来，Api开始发布出去，为减少QA的工作量，需要做一个Api的自动化测试工具来保证数据准确性，希望能让测试环境的数据和生产上的数据作对比，这样保证新开发的Api不影响到旧的，测试专注于新功能就好，这是第二个引子。

Api的性能也是个关键的指标，在大规模使用前也需要对Api的性能做测试，所以性能测试是Hitchhiker下一个目标。

如果Api是公开的话，文档是必须的，试想如果我们依赖的Api文档好的话不仅我们这边容易，他们那边其实也省事不少，至少我们不用去频繁打扰他们了。不过写文档过程是比较痛苦的且更新很麻烦，但如果Api的case都已经有了的话，文档的主体其实就有了，然后对参数加些说明就可以了，QA熟悉的话都可以帮着做，所以一个所见即所得并且支持模板的文档也在计划中。

其实我们之前也是有用过一些测试工具，但不是很满意，就易用性来说，最好用的还是Postman，所以Hitchhiker的UI就是模仿它的，用过Postman的话会很容易上手。

### 能做什么

* Team协作开发Api

* Api历史修改记录及支持diff展示

* 支持多环境变量及运行时变量

* 支持Schedule及批量run

* 不同环境下的请求数据对比 (eg: stage vs product)

* 易部署 (支持 docker, windows, linux), 数据都存在自己这里，不会上传及丢失

* 会记往任何修改，不用怕没保存时session失效或系统重启

* 支持导入Postman v1 collections

* 性能测试 (开发中...)

* Api文档 (计划中...)

### 如何部署

首推使用 docker 部署，简单快捷，具体操作参考 [deploy with docker](https://github.com/brookshi/Hitchhiker/blob/master/doc/howtoinstall-docker-cn.md)

如果没有docker环境也可以使用源码部署，也很简单

linux 请参考 [deploy to linux](https://github.com/brookshi/Hitchhiker/blob/master/doc/howtoinstall-linux-cn.md)

windows 请参考 [deploy to win](https://github.com/brookshi/Hitchhiker/blob/master/doc/howtoinstall-win-cn.md)

### 如何使用

参考 [使用说明](https://github.com/brookshi/Hitchhiker/blob/master/doc/howtouse-cn.md)

### 用到的技术

前后端分离，前端采用 React + Redux + AntDesign，后端基于 Nodejs， 采用 Koajs + TypeORM + MySQL。

语言统一用的 Typescript。

测试的话，前端用Jest，覆盖了逻辑最多的 reducer，后端使用的就是本工具来测试自己，这对时间有限的我来说算是最有性价比的选择。

### 开源

可以访问 http://hitchhiker-api.com/ 来使用，点击 `try without login` 免注册登录，另外，为了免备案，服务器在海外的，所以速度上可能会有点慢，请谅解。

当然最好还是在本地局域网部署，用起来会比较爽。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。