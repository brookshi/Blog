在C#里面如果想要不直接修改类或函数，但给类或方法添加一些额外的信息或功能，可以想到用`Attribute`，这是一个十分方便的功能装饰器。
现在用TypeScript同样也可以利用装饰器来给类、函数、属性以及参数添加附加功能，装饰器是ES7的一个提案，在TypeScript里已经有实现可用，不过需要在`tsconfig.json`里启用`experimentalDecorators`。

```ts
"compilerOptions": {
    ..., // other options
    "experimentalDecorators": true
}
```

### 装饰器的介绍
TypeScript中装饰器可以应用到类、方法、属性及函数参数上，而且可以同时应用多个。
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
function Testable(target: Function) { // 类、方法、属性、方法参数的参数各不相同
    //这里可以记录一些信息到target，或者针对target做一些处理，如seal
}
```
另外一种是带参数，如`@Log('controller')`，这种实现函数里的参数就是括号里的参数，而且需要返回一个`function`。

```ts
function Log(name: string) { // name就是传进来的参数'controller'
    return function(target: Function) { // 类、方法、属性、方法参数的参数各不相同
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
上面的`sealed`就是类的装饰器，`target`指构造函数，类装饰器就这么一个参数。

### 方法装饰器
方法装饰器的使用方法和类装饰器类似，只是参数不一样，方法装饰器有三个参数：
1. 如果装饰的是静态方法，则是类的构造函数，如果是实例方法则是类的原型。
2. 方法的名字。
3. 方法的`PropertyDescriptor`。
`PropertyDescriptor`即属性描述符，有
configurable  是否可以配置，如动态添加删除函数属性之类
writable      是否可写，可以用来设置只读属性
enumerable    是否可枚举，即是否能在`for...in`中能枚举到
value         对象或属性的值

有了这些参数就可以很好的给方法添加一些功能。

```ts
const Router = Symbol(); // 唯一key,用来存装饰器的信息

function GET(path?: string) { // GET带了个可选参数
    return (target: any, name: string) => setMethodDecorator(target, name, 'GET', path);
}

//把method和path存起来，路由查找的时候就可以用了
function setMethodDecorator(target: any, name: string, method: string, path?: string){
    target[Router] = target[Router] || {};
    target[Router][name] = target[Router][name] || {};
    target[Router][name].method = method;
    target[Router][name].path = path;
}

// 通过PropertyDescriptor来设置enumerable
function Enumerable(enumerable: boolean) { 
    return (target: any, name: string, descriptor: PropertyDescriptor) => {
        descriptor.enumerable = enumerable;
    };
}

class Controller{

    @GET
    @Enumerable(true)
    getContent(arg: string): string{
        return '';
    }
}
```

### 参数装饰器
方法参数同样可以有装饰器，同样有三个参数，前两个参数和方法的一致，最后一个参数是所装饰的参数的位置。
能过参数装饰器可以给方法动态的检查或设置参数值，下面是检查参数是否为空，为空则抛出异常。

```ts
const CheckNullKey = Symbol();

// 把CheckNull装饰的参数存起来
function CheckNull(target: any, name: string, index: number) {
    target[Router] = target[Router] || {};
    target[Router][name] = target[Router][name] || {};
    target[Router][name].params = target[Router][name].params || [];
    target[Router][name].params[index] = CheckNullKey;
}

// 找出CheckNull的参数，并检查参数值，为空则抛异常，否则继续执行方法
function Check(target: any, name: string, descriptor: PropertyDescriptor) {
    let method = descriptor.value;
    descriptor.value = function () {
        let params = target[Router][name].params;
        if (params) {
            for (let index = 0; index < params.length; index++) {
                if (params[index] == CheckNullKey && 
                    (arguments[parameterIndex] === undefined || arguments[parameterIndex] === null)) {
                    throw new Error("Missing required argument.");
                }
            }
        }

        return method.apply(this, arguments);
    }
}

class Controller{

    @Check
    getContent(@CheckNull id: string): string{
        console.info(id);
        return id;
    }
}
```

### 属性装饰器
用法同上，参数只有两个，同类装饰器的前两个一样，常用来标识属性的特性。

```ts
function Column(target: any, name: string) {
    //把name存起来，这个column仅仅是标识出来对应数据库中的列，常用在ORM框架中
}

class Table{

    @Column  
    name: string;
}
```
另外还有属性访问器的装饰器，和方法基本一样，同样的三个参数，不过同个属性的`get`和`set`只能有一个有，而且必须是先声明的那个。

```ts
class User {
    private _name: string;

    @Enumerable(true)
    get name(){
        return _name;
    }

    set name(value: string) {
        _name = value;
    }
}
```

### 多个装饰器的执行顺序