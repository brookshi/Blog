Hitchhiker 是一款开源的支持多人协作的 Restful Api 测试工具，支持Schedule, 数据对比，压力测试，支持上传脚本定制请求，可以轻松部署到本地，和你的team成员一起管理Api。

详细介绍请看： [http://doc.hitchhiker-api.com/cn/introduction.html](http://doc.hitchhiker-api.com/cn/introduction.html)

在线体验： [http://www.hitchhiker-api.com/](http://www.hitchhiker-api.com/)， 可以用 `try without login` 来免登录使用 （在线演示不支持压力测试和上传js，虚拟机单核的，撑不住）。

## 下面来看看这次的更新：

## request和setRequest

Script 增加属性request来得到请求的信息，包括 method, url, headers, body。
```javascript
const req = hitchhiker.request;
const {url, headers, method, body} = req;
```
另外增加了一个方法 setRequest(request)，这个方法是对请求进行修改。
request和setRequest配合着一起用就可以在请求发送前对其进行适当的编辑，比如增加一个签名，增加一个header之类。

```javascript
const crypto = hitchhiker.require('crypto-js');

const sign = crypto.HmacSHA1('test', 'asdf');

const req = hitchhiker.request;
url = `${url}?sign=${sign}`;
hitchhiker.setRequest({...hitchhiker.request, url});
```

当然，做得过份点，把GET请求变成POST请求也不是不行：
```javascript
let url = hitchhiker.request.url;

url = `${url.substr(0, url.lastIndexOf('/'))}/post?c=d`;

hitchhiker.setRequest({...hitchhiker.request, url, body: '{"name":"brook"}', method: 'POST'});
```

## Common Pre Request Script

之前有个Pre Request Script，是Request级别的，但一个Collection下往往有很多Request有几乎相同的操作，如果每个Request去写将会非常麻烦，维护也不方便。

一个典型的应用场景是Collection下面所有的Request的url都需要在发送前加一个动态hash值，把这些通用的事情放到Collection 级别来做就会非常方便。

## 配置 inviteMemberDirectly

Hitchhiker 增加了一个新配置：inviteMemberDirectly， 用于决定邀请成员时是否需要发邮件，还是直接拉到Project里来，默认是true。

背景是有些公司的server是不能访问外网的，也就用不了Hitchhiker提供的邮件功能，这时这个直接拉同事到Project里来的功能就非常有用了。

当然，Hitchhiker是支持外部邮件接口的，其实如果愿意的话自己在内网搭一个邮件服务器也不麻烦。

具体这些配置可以参考：[Configuration](http://doc.hitchhiker-api.com/cn/installation/configuration.html)

## Request Follow Redirect 和 Request Strict SSL

这两个都是Collection下面的属性。

Request Follow Redirect 用来设置这个Collection下面的请求是否在返回状态码为3xx时继续重定向到下一个页面，默认为false。

Request Strict SSL 用来设置这个Collection下面的请求在发送时是否需要做SSL证书的校验，因为有些公司用的自己做的证书，这些证书在严格SSL模式下会返回证书错误信息，不勾这个选项的话就会忽略这种错误，默认为false。

## 整理文档

把文档重新整理了一遍，使用gitbook来写和发布，不过gitbook貌似在国内经常被墙，所以在hitchhiker的网站上也放了一份，方便查阅。

文档地址：[http://doc.hitchhiker-api.com/cn/introduction.html](http://doc.hitchhiker-api.com/cn/introduction.html)

画了一个Script流程图：

![](https://raw.githubusercontent.com/brookshi/images/master/Hitchhiker/script/reuqest_wf.png)

## 后续计划

接下来的一个主要目标是让压力测试支持ES6和支持js库，不过因为压力点是用GO写的，用的otto的解释器，而otto只支持到ES5，需要在server做下转换，另外还要支持async/await，可能会有点麻烦。

Github: **[https://github.com/brookshi/Hitchhiker](https://github.com/brookshi/Hitchhiker)**， 觉得不错的话麻烦 **Star** 支持下，谢谢。