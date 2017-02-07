
## **背景**
相信之前用过JavaScript的朋友都碰到过异步回调地狱(callback hell)，N多个回调的嵌套不仅让代码读起来十分困难，维护起来也很不方便。
其实C#在`Task`出现之前也是有类似场景的，Async Programming Mode时代，用`Action`和`Func`做回调也很流行，不过也是意识到太多的回调嵌套代码维护不易，微软引入了`Task`和Task-based Async Pattern。
虽然不知道是哪个语言最早有这个概念，但相信是C#把`async await`带来流行语言的舞台，接着其他语言也以不同的形式支持`async await`，如Python, Dart, Swift等。
JavaScript同样在ES6开始支持`Promise`和`Generator`，并在ES7中提出支持`async await`的议案。
TypeScript在1.7版本开始支持`async await`编译到ES6，并在2.1版本开始支持编译到ES5和ES3，算是全面支持了。

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