Hitchhiker 是一款开源的 Restful Api 测试工具，支持Schedule, 数据对比，压力测试，支持上传脚本定制请求，可以轻松部署到本地，和你的team成员一起管理Api。

详细介绍请看： [http://www.cnblogs.com/brookshi/p/7440663.html](http://www.cnblogs.com/brookshi/p/7440663.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试和上传js，虚拟机单核的，撑不住）。

## 下面来看看这次的更新：

## Pre Request Script

这个算是之前就想实现的，拖了会，不过也是有朋友在github里的issue里提出，正好促使我完成这个功能。
在Pre Request Script里写的脚本会在请求发送前执行，这就使得可以在请求发送前处理一些事情，比如生成一个md5给请求使用，或者读取文件内容，再或者在请求前先请求一个数据，把这个数据做为变量给现在的请求使用，可以做的事有很多，发挥的余地很大。

现在在脚本里可以使用的方法有：
``` javascript
require             // 这个做js的都懂，有了这个就有无限可能，内置了'lodash', 'request', 'cypro-js'等库，重要的是支持上传js库
readFile            // 读取文件
readFileByReader    // 使用自定义的方法读取文件，比如读取excel
saveFile            // 保存文件
removeFile          // 删除文件
setEnvVariable      // 设置环境变量
getEnvVariable      // 获取环境变量
removeEnvVariable   // 删除环境变量
environment         // 获取当前环境的名字
```
当然上面的函数同样可以在Test中使用，下面这些只在Test里支持：
``` javascript
responseBody
responseObj
responseHeaders
responseTime
responseCode.code
responseCode.name 
```
![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/pre_request_script.PNG)

## 项目文件夹

对每个项目来说都有一个`data`文件夹和一个`lib`文件夹。
`data`文件夹用于上传一些测试所需要的数据，可以是任何格式，只要你能读取。
`lib`文件夹则用于上传一些js库，需要先压缩成zip格式，上传后会自动解压。
然后在脚本里就可以通过 `readFile` 读取 `data`文件夹下的文件，或者通过 `saveFile`保存文件到这个文件夹。
同样可以在脚本通过`require`来引用上传的js库，然后使用它。

除了项目文件夹外其实还有一个全局的文件夹，这个文件夹可以放一些全局的js库或数据，比如已经内置了一些常用的js库：`uuid`，`lodash`等。

## schedule支持以小时或分钟为单位

这个算是呼声比较高的，之前只是做到按天来跑schedule，后来收到不少这方面的需求，所以增加了以小时或分钟为单位的schedule。


## 支持自定义邮件发送接口

这个也算是刚需了，因为很多公司会过滤一些来源不明的邮件，所以 Hitchhiker发出的邮件很可能会收不到，现在增加了一个自定义的邮件接口，Hitchhiker会把数据post到这个接口上，就可以使用公司的邮箱来接发邮箱了。


## 开放schedule的run now接口以便其他程序调用

有朋友表示想在Jenkins里调用Schedule的Run接口，这是个好方法，所以开放了这个接口出来，方便其他程序调用。

## Bug fix

* schedule的顺序执行无效
* sync有时会覆盖用户已经更改的数据
* sync时环境变量编辑对应框里的内容会被清掉

## 后续计划

现在的Pre Request Script和文件夹系统在压力测试下是不支持的，这个得想个办法支持起来，另外一个是文档，现在文档有点乱，得整理下。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。