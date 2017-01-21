用过WebApi或Asp.net MVC的都知道微软的路由设计得非常好，十分方便，也十分灵活。虽然个人看来是有的太灵活了，team内的不同开发很容易使用不同的路由方式而显得有点混乱。 不过这不是重点，我在做Node项目的时候就觉得不停的用`use(...)`来指定路由路径很烦人，所以用`Typescript`写了这个基于`Koa`和`Koa-router`的路由插件，可以简单实现一些类似WebApi的路由功能。

目标是和WebApi一样，加入的controller会自动加入路由，也可以手动指定路由。 可以定义是`GET`或`POST`等等。 可以方便的得到路由里的参数，比如`路径参数`，`Query参数`，`post body`等。

现在来看看效果：

### 第一步，先设置controllers的目录和url的固定前缀。

所有的controller都在这目录下，这样会根据物理路径自动算出路由。 url的固定前缀就是host和路由之间的，比如`localhost/api/v2/user/name`，`api/v2`就是这个固定前缀。
```ts
import { WebApiRouter } from '../lib/index';

app.use(new WebApiRouter().router('sample/controllers', 'api'));
```
### 第二步是controller都继承自`BaseController`。
```ts
export class TestController extends BaseController
{

}
```
### 第三步是把装饰器加到controller的接口上。
```ts
@POST('/user/:name')
postWithPathParam(@PathParam('name') name: string, @QueryParam('id') id: string, @BodyParam body: any) {
    console.info(`TestController - post with name: ${name}, body: ${JSON.stringify(body)}`);
    return 'ok';
}
```
`:name`是路径里的变量，比如 `/user/brook`, `:name`就是`brook`，可以在方法的参数里用`@PathParam`得到

用`@BodyParam`可以得到`Post`上来的`body`

用`@QueryParam`可以得到`url`里`?`后的参数

`@POST`里的参数是可选的，空的话会用这个controller的物理路径做为路由地址。

## 现在具体看看是怎么实现的

实现过程其实很简单，从上面的目标入手，首先得到controllers的物理路径，还要得到被装饰器装饰的方法以及它的参数。 装饰器的目的在于要得到是`Get`还是`Post`等，还要得到指定的`Path`，最后就是把`node request`里的数据放到方法的参数里。

核心代码：

### 得到物理路径
```ts
initRouterForControllers() {
    //找出指定目录下的所有继承自BaseController的.js文件
    let files = FileUtil.getFiles(this.controllerFolder);
    files.forEach(file => {
        let exportClass = require(file).default;
        if(this.isAvalidController(exportClass)){
            this.setRouterForClass(exportClass, file);
        }
    });
}
```
### 从物理路径转成路由
```ts
private buildControllerRouter(file: string){
    let relativeFile = Path.relative(Path.join(FileUtil.getApiDir(), this.controllerFolder), file);
    let controllerPath = '/' + relativeFile.replace(/\\/g, '/').replace('.js','').toLowerCase();
    if(controllerPath.endsWith('controller'))
        controllerPath = controllerPath.substring(0, controllerPath.length - 10);
    return controllerPath;
}
```
### 装饰器的实现

装饰器需要引入`reflect-metadata`库

先看看方法的装饰器，`@GET`,`@POST`之类的，实现方法是给装饰的方法加一个属性`Router`，`Router`是个`Symbol`，确保唯一。 然后分析装饰的功能存到这个属性中，比如`Method`，`Path`等。
```ts
export function GET(path?: string) {
    return (target: BaseController, name: string) => setMethodDecorator(target, name, 'GET', path);
} 

function setMethodDecorator(target: BaseController, name: string, method: string, path?: string){
    target[Router] = target[Router] || {};
    target[Router][name] = target[Router][name] || {};
    target[Router][name].method = method;
    target[Router][name].path = path;
}
```
另外还有参数的装饰器，用来给参数赋上`request`里的值，如`body`,`param`等。
```ts
export function BodyParam(target: BaseController, name: string, index: number) {
    setParamDecorator(target, name, index, { name: "", type: ParamType.Body });
}

function setParamDecorator(target: BaseController, name: string, index: number, value: {name: string, type: ParamType}) {
    let paramTypes = Reflect.getMetadata("design:paramtypes", target, name);
    target[Router] = target[Router] || {};
    target[Router][name] = target[Router][name] || {};
    target[Router][name].params = target[Router][name].params || [];
    target[Router][name].params[index] = { type: paramTypes[index], name: value.name, paramType: value.type };
}
```
这样装饰的数据就存到对象的属性上，后面构建路由时就可以用了。

### 绑定路由到`Koa-router`上

上面从物理路径得到了路由，但是是以装饰里的参数路径优先，所以先看看刚在存在原型里的`Router`属性里有没有`Path`，有的话就用这个作为路由，没有`Path`就用物理路由。
```ts
private setRouterForClass(exportClass: any, file: string) { 
    let controllerRouterPath = this.buildControllerRouter(file);
    let controller = new exportClass();
    for(let funcName in exportClass.prototype[Router]){
        let method = exportClass.prototype[Router][funcName].method.toLowerCase();
        let path = exportClass.prototype[Router][funcName].path;
        this.setRouterForFunction(method, controller, funcName,  path ? `/${this.urlPrefix}${path}` : `/${this.urlPrefix}${controllerRouterPath}/${funcName}`);
    }
}
```
绑定路由到`KoaRouter`上
```ts
private setRouterForFunction(method: string, controller: any, funcName: string, routerPath: string){
    this.koaRouter[method](routerPath, async (ctx, next) => { await this.execApi(ctx, next, controller, funcName) });
}

private async execApi(ctx: Koa.Context, next: Function, controller: any, funcName: string) : Promise<void> {
    try
    {
        ctx.body = await controller[funcName](...this.buildFuncParams(ctx, controller, controller[funcName]));
    }
    catch(err)
    {
        console.error(err);
        next(); 
    }
}
```
这样就完成了简单版的类似WebApi的路由，源码在[https://github.com/brookshi/webapi-router](https://github.com/brookshi/webapi-router)，欢迎大家Fork/Star，谢谢。