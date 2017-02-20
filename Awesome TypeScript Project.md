TypeScript出来有段时间了，也冒出了很多用TypeScript开发的优秀开源项目，下面我搜寻了一些基于TypeScript项目，给大家分享一下：

[https://github.com/brookshi/awesome-typescript-projects](https://github.com/brookshi/awesome-typescript-projects)

## **TypeScript**
可能有人奇怪这里说的是基于TypeScript的开源项目，为什么TypeScript本身也在这里。
其实TypeScript语言本身就是用TypeScript编写的，即self-hosting，使用上一稳定版本的编译器来编译本次版本。
TypeScript就不做太多介绍了，不熟悉的同学可以参考我之前写的 C#到TypeScript系列。

## **IDE - VSCode**
基于TypeScript + Nodejs + Electron开发的IDE. Github上star: 2万+

VSCode大家应该都知道，同样是微软开发，没使用过的建议试用下，相对于Visual Studio的笨重，VSCode非常轻，占用内存少，打开项目速度很快，而且跨平台，非常适合用来做前端或Nodejs开发。
支持的特性也很多，通过扩展能支持非常多的语言，比如C#, GO, C++等，最近发的包还原生支持Markdown语法，我的文章都是用VSCode写的。
打开大文件真的是秒开，之前用Notepad++打开大文件还有点迟顿，格式化成Json更是直接卡死，VSCode则完全没问题。

![](https://cloud.githubusercontent.com/assets/11839736/16642200/6624dde0-43bd-11e6-8595-c81885ba0dc2.png)

其他基于TypeScript的IDE还有 在线IDE monaco-edit，游戏开发IDE superpowers等，有兴趣的同学可以去[awesome typescript projects](https://github.com/brookshi/awesome-typescript-projects)了解下。

## **Framework - Angular2**
基于TypeScript + RxJS + ZoneJS的Framework. Github上star: 2万+

大名鼎鼎的前端三剑客之一，背后的老爹Google确保了Angular的质量，Angular从Angular2开始采用TypeScript来开发，强类型对框架的稳定性提供不少支持。
微软Azure的页面就是用Angular写的，下面这个也是Angular2的一个dashboard应用。

![](https://camo.githubusercontent.com/2dc499a9333ca5534cba0593956e68d43c7f3f92/687474703a2f2f692e696d6775722e636f6d2f514b39417a486a2e6a7067)

在Angular2上衍生了不少优秀的框架或库，如 angular-seed，material2, ui-router等。

其他还有很多诸如 ionic，NativeScript，AtomicGameEngine的优秀框架都是用TypeScript开发的，国内的白鹭引擎(egret)同样基于TypeScript。

## **UI - ant-design**
基于TypeScript + React的UI界面库. Github上star: 1万+

ant-design是由国内阿里旗下的蚂蚁金服的团队用TypeScript开发的一款企业级React UI库，已经应用到金服和其他阿里旗下产品当中。
ant-design的UI看起来非常美观，而且不显累赘，文档也非常完整，重要的是文档是中文版的，相信非常适合国内开发使用。

![](https://github.com/brookshi/awesome-typescript-projects/raw/master/images/antdesign/case.png)

ant-design也推出了mobile版ant-design-mobile，这样不管是web端还是移动端都可以有同一套UI设定。

![](https://github.com/brookshi/awesome-typescript-projects/raw/master/images/antdesign/mobilecase.png)

同样基于TypeScript的UI库还有不少，如Angular的material2，和ant-design有一拼的blueprint都是其中皎皎者。

## **library - ui-router**
基于TypeScript + Angular的UI router库. Github上star: 1万+

ui-router的目的是提供一个管理UI跳转的库，基于状态机维护了一个层级的状态树，这个库对于单页应用来说非常有用。
现在应用页面非常多，如果没有一个管理中心的话，不停的跳转后状态很容易乱掉，这个库就是用来解决这个问题。

下图最底下的那条就是页面的路由，在微软的Azure上也有用到。
![](https://raw.githubusercontent.com/brookshi/awesome-typescript-projects/master/images/ui-router.png)

## **library - RxJS**
这个库现在出到5代，之前是用JavaScript开发，5代开始采用TypeScript开发。 Github上star: 5千+

当然第四代是很出名的，Github已经有超过1万的star。
这个库算是响应式编程库家庭中的一员，其他还有RxJava，Rx.NET，RxGO等。

RxJS是基于流的概念，提供了一系列神奇的函数工具集，使用它们可以合并、创建、过滤这些流。 
一个流或者多个流可以作为另一个流的输入。比如你可以合并多个流，或者从很多流中选出你需要的，还可以将值从一个流映射到另一个流。
这种方式对于事件的处理会非常方便，具体可以去github上查看相关文档。

![](https://raw.githubusercontent.com/Reactive-Extensions/RxJS/master/doc/designguidelines/images/throttleWithTimeout.png)

## **tool - tslint**
做JavaScript开发的有ESLint来规范代码，而TypeScript则可以用TSLint。 Github上star: 1千+

开发一个项目往往有好几个甚至十几人，不同的人不同的代码风格，这时就需要一款工具来规范一下代码，来提高代码质量和可维护性。
基本上上面写的项目都有用到这款工具，可见其流行程度。

## **总结**
上面从IDE, Framework, UI, 库，工具等方面分别介绍了一些TypeScript的流行开源项目，其它还有很多有潜力的项目如Nodejs的ORM框架：TypeORM等，大家可以去[awesome typescript projects](https://github.com/brookshi/awesome-typescript-projects)翻翻。
这些都说明TypeScript已经非常成熟，稳定了，而且上面项目有一些是从JavaScript转过来的重新打造的项目，由此可以说明TypeScript确实可以给项目带来实实在在的好处。
TypeScript的发展还在继续，做为微软走向开源的一个标志性项目，有理由相信TypeScript将来会有很好的发展。