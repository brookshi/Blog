在C#里面如果想要不直接修改类或函数，但给类或函数添加一些额外的信息或功能，可以想到用`Attribute`，这是一个十分方便的功能装饰器。
现在用TypeScript同样也可以利用装饰器来给类、函数、属性以及参数添加附加功能，装饰器是ES7的一个提案，在TypeScript里已经有实现可用，不过需要在`tsconfig.json`里启用`experimentalDecorators`。

```ts
"compilerOptions": {
    ..., // other options
    "experimentalDecorators": true
}
```

### 装饰器的介绍
TypeScript中装饰器可以应用到类、函数、属性及函数参数上，而且可以同时应用多个。
装饰器的写法是`@name()`，`()`可以不要，也可以在里面写一些参数。

```ts
@Testable
@Log('controller')
class Controller{

    @GET
    getContent(@QueryParam arg: string): string{
        return '';
    }
}
```

### 装饰器的实现
装饰器根据实现来说有两种：
一种是不带参数，如`@Testable`。

```ts
function Testable(target: Function) { // 类、函数、属性、函数参数的参数各不相同
    //这里可以记录一些信息到target，或者针对target做一些处理，如seal
}
```
另外一种是带参数，如`@Log('controller')`，这种实现函数里的参数就是括号里的参数，而且需要返回一个`function`。

```ts
function Log(name: string) { // name就是传进来的参数'controller'
    return function(target: Function) { // 类、函数、属性、函数参数的参数各不相同
        // 这里可以根据name和target来做一些处理
    }
}
```

### 类装饰器
上面的`(target: Function)`其实就是类的装饰器参数，指向的是类的构造函数，如果想给类加一个简单的seal功能，可以这样做：

```ts
function sealed(target: Function) {
    Object.seal(target);
    Object.seal(target.prototype);
}

@sealed
class Test22{
    
}

Test22.prototype.test = ''; // 运行时出错，不能添加
```