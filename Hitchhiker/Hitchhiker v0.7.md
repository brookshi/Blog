Hitchhiker 是一款开源的支持多人协作的 Restful Api 测试工具，支持Schedule, 数据对比，压力测试，支持脚本定制请求，可以轻松部署到本地，和你的team成员一起协作测试Api。

详细介绍请看： [http://doc.hitchhiker-api.com/cn/introduction.html](http://doc.hitchhiker-api.com/cn/introduction.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试和上传js库，虚拟机单核的，撑不住）。
 
## 下面来看看这次的更新：

## 可以以diff方式查看Schedule的对比结果

Hitchhiker的Schedule是支持不同环境的数据对比的，不过之前只是把两边的response和对比结果给出来，想要知道有哪些不同的话还需要借助其他diff工具来对比，比较麻烦。
这次加入了内置的对比工具，Schedule的结果不匹配时，会多出一个`view diff`的按钮，点击后会弹出对话框显示两边reponse的不同。

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/schedule/diff.png)

## 支持在脚本里写console.log(info, warn, error)来调试代码

测试工具里的脚本调试起来比较麻烦，因为脚本是在服务端跑的，所以使用console只会在服务端打印结果，浏览器端是看不到的，这次发布就添加了对console的支持，在脚本里写的打印信息会从服务端返回回来再在浏览器控制台里打印出来。
![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/script/console.PNG)

## Parameters可以做为一个变量存在，以便在运行时动态生成Parameters

之前Hitchhiker只支持在Parameters里的某个值使用变量，但有些时候Parameters需要从文件里读取出来构建，这时整个Parameters都需要做为一个变量存在来使用从文件里读取出来的数据，所以就加了这个功能。
其实也是一个外国友人提的feature，不过他希望实现的是在Parameters里面可以选择上传上来的文件并以此文件的内容来构建请求，不过考虑到Parameters不一定来自文件，可能以其他的方式动态构建出来的，所以以Parameters整体做为一个变量的形式来实现这个需求更灵活些，不过这个功能只能在Schedule里起作用。

## 支持自定义SMTP来发送邮件

Hitchhiker 会在邀请Project成员或跑Schedule后时发送邮件，用的是一个自己的邮箱系统，但是用户的服务器经常不能访问外网，所以Hitchhiker提供了两种自定义mail方式。之前有介绍过邮件接口的方式，现在多提供了一个SMTP方式，这样就不需要额外写接口了，使用起来也更方便。

## 支持以cURL来新建request

这个对于快速调试非常有用，在chrome的控制台Network里右键点击请求，选择copy as cUrl(bash)，再导入这里来就可以调试这个请求了。
![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/simple_tutorial/curl.png)

## 支持为request生成java, python, go, c#等语言的请求代码

这个功能对于开发还是比较友好的，支持目前流行的一些语言的代码生成，对于API工具来说算是标配了。
![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/generate_code.png)

## 其他小功能

1. 支持Swagger V2版本的API json文件导入。

2. 支持美化body

3. 支持xml response的美化

4. 去除body或脚本里使用变量时编辑框的语法错误提示

## 修改Bug

1. 新Collection的Common pre script保存不了

2. Schedule在勾上保存然后取消勾时会保存不了

3. 导入Postman json时出错，有header为null

4. 请求如果没响应时，请求返回的时间会为0

## 后续计划

短期内还是以继续增加测试新功能为主，比如Schedule的统计模式、一次运行所有Schedule、中断压力测试等。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。