Hitchhiker 是一款开源的支持多人协作的 Restful Api 测试工具，支持自动化测试, 数据对比，压力测试，支持脚本定制请求，可以轻松部署到本地，和你的team成员一起协作测试Api。

详细介绍请看： [http://doc.hitchhiker-api.com/cn/introduction.html](http://doc.hitchhiker-api.com/cn/introduction.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试和上传js库，虚拟机单核的，撑不住）。
 
## 更新：

## 中文版

这个是自从Hitchhiker第一次Release以来每次更新都会有人问到, 就我个人来说其实觉得意义不大, 但是用户的需求第一, 所以花了一些时间让Hitchhiker支持了多语言。

无论是安装包还是docker，安装后默认都还是英文，需要中文的朋友可以加上环境变量 HITCHHIKER_APP_LANG，值为zh表示中文，en表示英文。 

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/chinese.png)

## 其他小功能及bug fix

1. 需要的话自动给url加上http://

2. 导入postman时form转成body

3. 处理form data里包含特殊符号时的异常

4. 改进email格式校验

## 后续计划

下个版本应该是增加form data和query string的友好支持，为下下个版本的文档做准备。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。