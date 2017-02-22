看看用TypeScript怎样使用常见的设计模式，顺便复习一下。

# 单例模式

### 特点：在程序的生命周期内只有一个全局的实例，并且不能再new出新的实例。

### 用处：在一些只需要一个对象存在的情况下，可以使用单例，比如Cache, ThreadPool等。

### 注意：线程安全，当然，这在单线程的JavaScript环境里是不存在的。

下面用TypeScript写一个Cache来看看单例模式：

```ts
class Cache{
    public static readonly Instance: Cache = new Cache();

    private _items: {[key: string]: string} = {};

    private Cache(){

    }

    set(key: string, value: string){
        _items[key] = value;
        console.log(`set cache with key: ${key}, value: ${value}`);
    }

    get(key: string): string{
        let value = _items[key];
        console.log(`get cache value: ${value} with key: ${key}`);
        return value;
    }
}

Cache.Instance.set('name', 'brook');
Cache.Instance.get('name');
```
输出：

```ts
set cache with key: 'name', value: 'brook'
get cache value: 'brook' with key: 'name'
```

很简单， 和C#基本一样， 设置一个全局只读的`Instance`并且把构造函数设为`private`，这样就确保了单例的特点。
可能有人有疑问：静态Instance在C#里能确保只有一个是CLR的功劳，这里每次访问Instance会不会重新new一个呢？
带着疑问来看看上面代码编译成JavaScript ES6的结果：

```js
class Cache {
    constructor() {
        this._items = {};
    }
    Cache() {
    }
    set(key, value) {
        this._items[key] = value;
        console.log(`set cache with key: '${key}', value: '${value}'`);
    }
    get(key) {
        let value = this._items[key];
        console.log(`get cache value: '${value}' with key: '${key}'`);
        return value;
    }
}
Cache.Instance = new Cache();
Cache.Instance.set('name', 'brook');
Cache.Instance.get('name');
```
可以看到TypeScript的静态实例Instance其实是直接加到了Cache本身上面，当然也就确保了不会再new出新的来。

# 建造者模式

### 特点：一步一步来构建一个复杂对象，可以用不同组合或顺序建造出不同意义的对象，通常使用者并不需要知道建造的细节，通常使用链式调用来构建对象。

### 用处：当对象像积木一样灵活，并且需要使用者来自己组装时可以采用此模式，好处是不需要知道细节，调用方法即可，常用来构建如Http请求、生成器等。

### 注意：和工厂模式的区别，工厂是生产产品，谁生产，怎样生产无所谓，而建造者重在组装产品，层级不一样。

下面用TypeScript写一个RequestBuilder来看看建造者模式：

```ts
enum HttpMethod{
    GET,
    POST,
}

class HttpRequest {}

class RequestBuilder{

    private _method: HttpMethod;

    private _headers: {[key: string]: string} = {};

    private _querys: {[key: string]: string} = {};

    private _body: string;

    setMethod(method: HttpMethod): RequestBuilder{
        this._method = method;
        return this;
    }

    setHeader(key: string, value: string): RequestBuilder{
        this._headers[key] = value;
        return this;
    }

    setQuery(key: string, value: string): RequestBuilder{
        this._querys[key] = value;
        return this;
    }

    setBody(body: string): RequestBuilder{
        this._body = body;
        return this;
    }

    build(): HttpRequest {
        // 根据上面信息生成HttpRequest
    }
}

let getRequest = new RequestBuilder()
                    .setMethod(HttpMethod.GET)
                    .setQuery('name', 'brook')
                    .build();

let postRequest = new RequestBuilder()
                    .setMethod(HttpMethod.POST)
                    .setHeader('ContentType', 'application/json')
                    .setBody('body')
                    .build();
```
上面`RequestBuilder`可以根据传进来的参数不同来构建出不同的`HttpReqeust`对象，这样使用者就可以按照自己需求来生成想要的对象。

这里有个问题是`RequestBuilder`需不需要抽象出来，个人觉得要看情况而定。
首先是保持简单，就是一个简单的构造功能给内部用也没必要抽象来增加代码复杂度，但如果业务上这个Builder是封装在一个库里面并且要对外提供服务，那还是需要一个抽象来隐藏细节，消除对实现的依赖。
并且如果业务上还需要不同的RequestBuilder，比如说`XmlRequestBuilder` `JsonRequestBuilder`之类，那就更需要一个抽象了。
