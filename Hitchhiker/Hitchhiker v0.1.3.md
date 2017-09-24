Hitchhiker 是一款开源的 Restful Api 集成测试工具，你可以轻松部署到本地，和你的team成员一起管理Api。

详细介绍请看： [http://www.cnblogs.com/brookshi/p/7440663.html](http://www.cnblogs.com/brookshi/p/7440663.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用。

## 这次版本主要增加一个重磅功能 - 参数化请求：

## 参数化请求
什么是参数化请求，就是把一个Api里可变的点提取出来，参数化，这样就可以用一个Case覆盖到所有可变请求。

参考下图（比较大，可能会比较慢出来）：`parameters`就是用来构建参数化请求的，请求通常有很多参数，比如query string, body里的变化点等，这些参数可能会有不止一个值，每个都要覆盖的话需要写很多request。

举个例子：比如一个request有三个可变的参数`A`, `B`, `C`，每个参数又分别有3个值，A的`1，2，3`， B的`4，5，6`， C的`7，8，9`，这样随机组合下来会有`3*3*3=27`个request：
```
147 148 148 157 158 159 167 168 169
247 248 248 257 258 259 267 268 269
347 348 348 357 358 359 367 368 369
```
很麻烦有没有，如果再多两个参数呢，轻松过百了呀，想想都头大，但其实它们之间只是一点不同，何必要费这么大劲呢，参数化请求可以帮你做这个事，只需要把可变的参数写在parameter里面，Hitchhiker会自动构建出所有request。

`parameters`有两种组合方式，一种是所有组合`Many to Many`，另一种是一对一组合`One to One`，上面生个27个request的就是`ManytoMany`，如果用一对一组合的话就只有3个，分别是：`147, 258, 369`。

`Parameters`的格式是一个json对象，对象的下一层是变量以及它的值：数组。看个例子：
``` json
{
    "A": [1, 2, 3],
    "B": [3, 4, 5], 
    "C": [7, 8, 9]
}
```
使用的方式同变量一样，用`{{}}`包起来。

下图就展示了参数化请求的使用方式，可变的三个参数`name`， `pwd`， `age`。
`name`有两个值：`tom`和`jerry`， `pwd`有两个值：`123`和`456`，`age`也是两个值：`20`和`18`，使用`OnetoOne`时会生成两个请求：`name:tom, pwd:123, age:20`和`name:jerry, pwd:456, age:18`，一一对应的，可以分别请求，也可以一起请求。
如果选了`ManytoMany`就会有8个请求，这里就不一一列举出来。
参数化请求的request保存后左边对应的item里会显示出请求的真正个数，如图中的`8`。
参数化请求跑schedule一样没问题，会自动拆分开跑和显示。

大图：右键新标签打开图片
![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/parameters.gif)

## 处理对比数据
Hitchhiker的一个重要功能就是可以对比不同环境的返回数据，之前是直接对比response，但实际上往往想要对比的是其中一部分或去掉可变部分，考虑一种情况，返回的response里带有一个当前时间，也就表示每次返回的数据都是不同的，因为时间肯定不一样，这样就影响了对比结果，但是这个时间没什么对比意义，所以我们需要在对比前把它去掉，这时可以用这个功能了。

具体用法：在`test`里用js处理`responseObj`，然后用`$export$(data)`函数导出处理后的数据（data就是处理后的数据），然后跑`schedule`时就会用导出的数据进行对比了。

## 默认Headers
之前有加一个header收藏功能，方便使用一些常用的header，但是有些server会校验一些请求头，比如`Accept`,`UserAgent`等，这个是每个请求都需要带的，每个request都写也有些麻烦，现在可以配置一些默认header，这些header可以在根目录下的appconfig.json里配置，默认定义的是这些：
``` json
"defaultHeaders": [
    "Accept:*/*",
    "User-Agent:Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.113 Safari/537.36",
    "Cache-Control:no-cache"
]
```
可以根据需要自行修改。

## 后续计划
本来的计划是两周一版本，其中一周做小版本的新功能和改bug，另一周做大版本的压力测试。不过这次参数化请求比预想的要麻烦些，上面两周时间基本都花这上面了，压力测试这块就没进展，下两周除了改bug外就全力做压力测试这块，希望国庆过后能做到差不多。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。