
## **背景**
相信之前用过JavaScript的朋友都碰到过异步回调地狱(callback hell)，N多个回调的嵌套不仅让代码读起来十分困难，维护起来也很不方便。
其实C#在`Task`出现之前也是有类似场景的，Async Programming Mode时代，用`Action`和`Func`做回调也很流行，不过也是意识到太多的回调嵌套代码维护不易，微软引入了`Task`和Task-based Async Pattern。
虽然不知道是哪个语言最早有这个概念，但相信是C#把`async await`带到流行语言的舞台，接着其他语言也以不同的形式支持`async await`，如Python, Dart, Swift等。
JavaScript同样在ES6开始支持`Promise`和`Generator`，并在ES7中提出支持`async await`的议案。

这篇先来看看Promise：

## **Promise的特点**
`Promise`之于TypeScript，相当于`Task`之于C#，只有返回`Promise`的函数才能使用`async await`。
`Promise`其实就是一个可以获取异步结果，并封装了一些异步操作的对象。
有三个状态：
`pending`: 进行中
`resolved`: 成功
`rejected`: 失败
并且这三个状态只有两种转换：`pending`->`resolved`、`pending`->`rejected`，也就是不是成功就是失败，并没有多余的状态转换。
这两种转换都是由异步返回的结果给定的，成功取回数据就是`resolved`，取数据失败或出异常就是`rejected`。
也因此，这转换过后的结果就是固定的了，不可能在转换过后还会变回`pending`或其他状态。
`Promise`不能在任务进行中取消，只能等结果返回，这点上不如C#的`Task`，`Task`可以通过`CancelTaskToken`来取消任务。

## **Promise的使用**
可以直接new一个`Promise`对象，构造函数的参数是一个有两个参数的函数。
这两个参数一个是`resove`，用来在异步操作成功后调用，并把异步结果传出去，调用`resove`后状态就由`pending`->`resolved`。
另一个是`reject`，用来在失败或异常时调用，并把错误消息传出去，调用`reject`后状态由`pending`->`rejected`。

```ts
var promise = new Promise(function(resolve, reject) {
    
});
```
通常需要在成功或失败后做一些操作，这时需要`then`来做这个事，`then`可以有两个函数参数，第一个是成功后调用的，第二个是失败调用的，第二个是可选的。
另外，`then`返回的也是一个Promise，不过不是原来的那个，而是新new出来的，这样可以链式调用，`then`后面再接`then`。

```ts
// 函数参数用lambda表达式写更简洁
promise.then(success => {
    console.info(success);
}, error => {
    console.info(error);
}).then(()=>console.info('finish'));
```

## **嵌套的Promise**
在实际场景中，我们可能需要在一个异步操作后再接个异步操作，这样就会有`Promise`的嵌套操作。
下面的代码显示的是`Promise`的嵌套操作：
`p1`先打印"start"，延时两秒打印"p1"。
`p2`在`p1`完成后延时两秒打印"p2"。

```ts
function delay(): Promise<void>{
    return new Promise<void>((resolve, reject)=>{setTimeout(()=>resolve(), 2000)});
}

let p1 = new Promise((resolve, reject) => {
    console.info('start'); 
    delay().then(()=>{
        console.info('p1'); 
        resolve()
    });
});

let p2 = new Promise((resolve, reject) => {
    p1.then(()=>delay().then(()=>resolve()));
});

p2.then(()=>console.info('p2'));
```

## **异常处理**
上面提到`Promise`出错时把状态变为`rejected`并把错误消息传给`reject`函数，在`then`里面调用`reject`函数就可以显示异常。
不过这样写显得不是很友好，`Promise`还有个`catch`函数专门用来处理错误异常。
而且`Promise`的异常是冒泡传递的，最后面写一个`catch`就可以捕获到前面所有promise可能发生的异常，如果用`reject`就需要每个都写。
所以`reject`函数一般就不需要在`then`里面写，在后面跟个`catch`就可以了。

```ts
new Promise(function(resolve, reject) {
  throw new Error('error');
}).catch(function(error) {
  console.info(error); // Error: error
});
```
也如上面所说状态只有两种变化且一旦变化就固定下来，所以如果已经在`Promise`里执行了`resolve`，再throw异常是没用的，catch不到，因为状态已经变成`resolved`。

```ts
new Promise(function(resolve, reject) {
    resolve('success');
    throw new Error('error');
}).catch(function(error) {
    console.info(error); // 不会执行到这里
});
```
另外，`catch`里的代码也可能出异常，所以`catch`后面也还可以跟`catch`的议案。
```ts
new Promise(function(resolve, reject) {
    resolve('success');
    throw new Error('error');
}).catch(function(error) {
    console.info(error); // 不会执行到这里
});
```

## **finally 和 done**
异常的`try...catch`后面可以跟`finally`来执行必须要执行的代码，`Promise`同样有`finally`来执行最后的代码。
另外还有`done`在最后面来表示执行结束并抛出可能出现的异常，比如最后一个`catch`代码块里的异常。
```ts
let p = new Promise(function(resolve, reject) {
    x = 2;  // error， 没有声明x变量
    resolve('success');
}).catch(function(error) {
    console.info(error); 
}).finally(()=>{ // 总会执行这里
    console.info('finish');
    y = 2;  // error, 没有声明y变量
}).done(); 

try{
    p.then(()=>console.info('done'));
} catch (e){
    console.info(e); // 由于最后面的done，所以会把finally里的异常抛出来，如果没有done则不会执行到这里
}
```

## **并行执行Promise**
虽然JavaScript是单线程语言，但并不妨碍它执行一些IO并行操作，如不阻塞发出http request，然后等待callback。
`Promise`除了用`then`来顺序执行外，也同样可以不阻塞同时执行多个`Promise`然后等所有结果返回再进行后续操作。
C#的`Task`有个`WhenAll`的静态方法来做这个事，`Promise`则是用`all`方法达到同样目的。
`all`方法接受实现Iterator接口的对象，比如数组。
```ts
let p = Promise.all([p1, p2, p3]);
```
`all`返回的是一个新的`Promise`- p，p的状态是由p1, p2, p3同时决定的：
```ts
p.resolved = p1.resolve && p2.resolve && p3.resolve
p.rejected = p1.rejected || p2.rejected || p3.rejected
```
也就是说p的成功需要p1,p2,p3都成功，而只要p1, p2, p3里有任何一个失败则p失败并退出。

`Promise`还有一个方法`race`同样是并行执行多个`Promise`，不同于`all`的是它的成功状态和错误状态一样，只要有一个成功就成功，如同C# Task的`Any`方法。
```ts
let p = Promise.race([p1, p2, p3]);
```