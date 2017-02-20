在C#里面如果想要不直接修改类或方法，但给类或方法添加一些额外的信息或功能，可以想到用`Attribute`，这是一个十分方便的功能装饰器。
用TypeScript同样也可以利用装饰器来给类、函数、属性以及参数添加附加功能，装饰器是ES7的一个提案，在TypeScript里已经有实现可用，不过需要在`tsconfig.json`里启用`experimentalDecorators`。

```ts
"compilerOptions": {
    ..., // other options
    "experimentalDecorators": true
}
```

## **装饰器介绍**
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

## **装饰器的实现**
装饰器根据实现可以分两种：
一种是不带括号，和属性一样，如`@Testable`。

```ts
function Testable(target: Function) { // 类、方法、属性、方法参数的参数各不相同
    //这里可以记录一些信息到target，或者针对target做一些处理，如seal
}
```
另外一种是带括号的，和函数一样，如`@Log('controller')`，实现函数里的参数就是括号里的参数，而且需要返回一个`function`。

```ts
function Log(name: string) { // name就是传进来的参数'controller'
    return function(target: Function) { // 类、方法、属性、方法参数的参数各不相同
        // 这里可以根据name和target来做一些处理
    }
}
```

## **类装饰器**
上面的`(target: Function)`其实就是类的装饰器参数，指向的是类的构造函数，如果想给类加一个简单的seal功能，可以这样做：

```ts
function sealed(target: Function) {
    Object.seal(target);
    Object.seal(target.prototype);
}

@sealed
class Test{
    
}

Test.prototype.test = ''; // 运行时出错，不能添加
```
上面的`sealed`就是类的装饰器，`target`指构造函数，类装饰器就这么一个参数。

## **方法装饰器**
方法装饰器的使用方法和类装饰器类似，只是参数不一样，方法装饰器有三个参数：
1. 如果装饰的是静态方法，则是类的构造函数，如果是实例方法则是类的原型。
2. 方法的名字。
3. 方法的`PropertyDescriptor`。
`PropertyDescriptor`即属性描述符，有
configurable  是否可以配置，如动态添加删除函数属性之类
writable      是否可写，可以用来设置只读属性
enumerable    是否可枚举，即是否能在`for...in`中能枚举到
value         对象或属性的值

有了这些参数就可以很好的给方法添加一些功能，比如下面实现类型WebApi里的Get的路由：

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

## **参数装饰器**
方法参数同样可以有装饰器，同样有三个参数，前两个参数和方法的一致，最后一个参数是所装饰的参数的位置。
能过参数装饰器可以给方法动态的检查或设置参数值，下面是检查参数是否为空，为空则抛出异常。

```ts
const CheckNullKey = Symbol();
const Router = Symbol();

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
                if (params[index] == CheckNullKey &&  // 找到CheckNull的参数并抛异常
                    (arguments[index] === undefined || arguments[index] === null)) {
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

new Controller().getContent(null); // error : Missing required argument.
```

## **属性装饰器**
用法同上，参数只有两个，和类装饰器的前两个一样，常用来标识属性的特性。

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
        return this._name;
    }

    set name(value: string) {
        this._name = value;
    }
}
```

## **多个装饰器的执行顺序**
一个声明可以添加多个装饰器，所以会有个执行先后顺序。
首先从上到下执行装饰器函数，然后再从下往上应用带括号的装饰器返回的函数。

```ts
function Test1(){
    console.info('eval test1');
    return function(target: any, name: string, descriptor: PropertyDescriptor){
        console.info('apply test1');
    }
}

function Test2(){
    console.info('eval test2');
    return function(target: any, name: string, descriptor: PropertyDescriptor){
        console.info('apply test2');
    }
}

class User1{

    @test1()
    @Test2()
    getName(){

    }
}
```
结果是：

```ts
eval test1
eval test2
apply test2
apply test1
```

总之，装饰器等于引入了天然的装饰模式，给类，方法等添加额外功能。不过装饰器目前还不算太稳定，但是由于确实方便，已经有成熟项目在使用了。