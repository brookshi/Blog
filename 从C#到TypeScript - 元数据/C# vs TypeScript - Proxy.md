我们知道在C#中要实现代理功能需要自己来实现代理类，并且每个类需要不同的代理类，使用起来不方便，虽然借助一些AOP框架可以一定程度实现拦截，但毕竟框架级别的还是太重了。
现在ES6倒是出了个解决方案，Proxy是ES6推出的用于拦截操作的一层代理实现，TypeScript当然也同样支持，下面来看下Proxy是怎么用的。

## **Proxy使用**
Proxy本身是一个类，可以通过`new`来实例化一个代理。

```ts
let p = new Proxy(target, handle)
```
Proxy有两个参数：
`target`指所要代理的对象。
`handle`也是一个对象，对象里包含对`target`操作的拦截。
看个例子：

```ts
let obj = { name: 'brook' };
let p = new Proxy(obj, {
    get(target, property){
        return 'cnblogs';
    }
});

console.info(obj.name); // brook
console.info(p.name); // cnblogs
```
可以看到，`p`做为`obj`的代理，在handle里加了对目标对象的属性`get`操作进行拦截，所以第一次直接输出`obj`的name是'brook'，用代理`p`输出就变成'cnblogs'了。
因为handle里对获取属性操作进行了重新定义。
`get`函数同样有两个参数，`target`仍然是操作对象，另一个`property`则是要访问的属性的名字。

## **Proxy可拦截的操作**
- get(target, propKey, receiver)

- set(target, propKey, value, receiver)

- apply(target, object, args)

- defineProperty(target, propKey, propDesc)

- deleteProperty(target, propKey)

- has(target, propKey)

- ownKeys(target)

- construct(target, args)

- getPrototypeOf(target)

- setPrototypeOf(target, proto)

- getOwnPropertyDescriptor(target, propKey)

- isExtensible(target)

- preventExtensions(target)

看过上一篇Reflect的有没有很熟，没错，Reflect里的操作Proxy里都同样有一份，这样在做Proxy的时候，如果要回到原始的结果，直接调用Reflect对应的操作就好。
接下来挑几个重要的看看。

## **get**
**get(target, propKey, receiver)**
上面提到过`get`，不过没说第三个参数，其实`receiver`指的就是new出来的Proxy对象。

```ts
let obj = { name: 'brook' };
let p = new Proxy(obj, {
    get(target, property, receiver){
        console.info(receiver === p); // true
        return 'cnblogs'
    }
});
console.info(p.name);
```
再来个例子来看看`get`能做到什么程度，我们知道数组的索引不能为负数，现在我们通过Proxy来让数组来支持它：

```ts
let arr = ["b", "r", "o", "o", "k"];
let p = new Proxy(arr, {
    get(target, property){
        let index = Math.abs(Number(property));  // 取负数的绝对值
        return arr[index];
    }
});
console.info(arr[2]);  // 输出o
console.info(p[-2]);  //同样输出o
```

## **set**
**set(target, propKey, value, receiver)**
`set`用来拦截属性的赋值操作，比如`number`类型的数组，可以让它接受任何类型的值，当不是`number`的时候就给值`0`，当然这只是个不符合实际使用的功能演示：

```ts
let arr = new Array<number>();
let p = new Proxy(arr, {
    set(target, property, value, receiver){
        if(typeof value != 'number'){  // 不是number就设为0
            value = 0;
        }

        return Reflect.set(target, property, value);
    }
});

p[0] = 11;
p[1] = "brook";

console.info(arr[0]); // 11
console.info(arr[1]); // 0
```
现在前端MVVM很火，而用`set`就可以轻松做到设置属性值的同时更新Dom对象，实现绑定的效果。

## **apply**
**apply(target, object, args)**
这可以拦截函数的调用，第一个和第三个参数的意思很明确，分别指函数和函数的参数。
第二个参数是指上下文的`this`，this的不同会可能导致函数里变量值的不同。

```ts
class Test1{
    text: string = 'test1';

    func(){
        console.info('call test1 func')
        console.info(`I am brook in ${this.text}`);
    }
}

class Test2{
    text: string = 'test2';

    func(){
        console.info('call test2 func')
        console.info(`I am brook in ${this.text}`);
    }
}

let t1 = new Test1();
let t2 = new Test2();

let p = new Proxy(t1.func, {
    apply(target, thisArg, args){
        Reflect.apply(target, t2, args);
    }
});

p();

```
上面代码输出信息如下：

``` ts
call test1 func
I am brook in test2
```
也就是实际调用的还是Test1的func，所以第一条输出为`call test1 func`，虽然Proxy代理的是Test1的func，但实际执行时传的this是t2，所以函数里的this指向了Test2，取的也就是`test2`中要实现代理功能需要自己来实现代理类，并且每个类需要不同的代理类，使用起来不方便，虽然借助一些AOP框架可以一定程度实现拦截，但毕竟框架级别的还是太重了。

上面介绍了几个常用的，其他的意思也很明显就不多说了，Proxy的应用场景除了上面说过的MVVM外，还可以用在ORM中，把对象的行为映射到数据库中，还有数据访问的代理，总之想用到代理的可以考虑Proxy。
还有就是要记住Proxy不是透明代理，它有自己的this，使用时需要注意。
