上两篇分别说了`Promise`和`Generator`，基础已经打好，现在可以开始讲`async await`了。
`async await`是ES7的议案，TypeScript在1.7版本开始支持`async await`编译到ES6，并在2.1版本支持编译到ES5和ES3，算是全面支持了。

## **async await 用法**
和C#里的十分相似，看个例子：

```ts
function delay(): Promise<void>{
    return new Promise<void>((resolve, reject)=>{setTimeout(()=>resolve(), 2000)});
}

async function run(){
    console.info('start');
    await delay();
    console.info('finish');
}

run();
console.info('run');
```
上面代码执行的结果是执行完`run()`后立即返回一个`Promise`，遇到`await`跳出函数，继续往下走，所以先输出`start`，再紧接着输出`run`，过了2秒后再输出`finish`。
可以看到`run`函数，function前面多了个`async`（如果是class里的方法，则是在函数名前），delay()前面多了个`await`，表示的意思很明显，就是在两者之间等待2秒。
`run`函数返回的也是一个`Promise`对象，后面可以接`then`来做后续操作。
`await`必须要在`async`块中，await的对象可以是`Promise`对象也可以不是，不是的话会自动转为已经resolved的Promise对象。
另外，`await`在代码块中是按顺序执行的，前面wait完后再会走下一步，如果需要并行执行，可以和`Promise`一样，用`Promise.all`或`Promise.race`来达到目的。

```ts
async function run1(){
    await delay();
    console.info('run1');
}
async function run2(){
    await delay();
    console.info('run2');
}
async function run3(){
    await delay();
    console.info('run3');
}
Promise.all([run1(), run2(), run3()]);
```
上面代码会在两秒后几乎同时输出run1, run2, run3。

## **async返回Promise状态**
一个async函数中可以有N个await，async函数返回的Promise则是由函数里所有await一起决定，只有所有await的状态都resolved之后，async函数才算真正完成，返回的Promise的状态也变为resolved。
当然如果中间return了或者出了异常还是会中断的。

```ts
async function run(){
    console.info('start');
    await delay();
    console.info('time 1');
    await delay();
    console.info('time 2');
    return;
    //下面当然就不会执行了
    await delay();
    console.info('time 3');
}
```
`run`的状态在time 2输出后return就转为`resolved`了。
当这里出异常时，async函数会中断并把异常返回到`Promise`里的`reject`函数。

```ts
async function run(){
    await Promise.reject('error'); // 这里出异常
    console.info('continue'); // 不会执行到这里
    await delay();
}
```

## **异常处理**
之前有提到`Promise`的异常可以在后面用`catch`来捕获，`async await`也一样。
向上面的例子，可能有需要把整个函数即使出异常也要执行完，就可以这样做：

```ts
async function run(){
    await Promise.reject('error').catch(e=>console.info(e));
    console.info('continue'); // 继续往下执行
    await delay();
}

let g = run(); //这里的g也是成功的，因为异常已经被处理掉
```
如果是多个异常需要处理，可以用`try...catch`

```ts
async function run(){
    try{
        await Promise.reject('error1');
        await Promise.reject('error2');
    } catch(e){
        console.info(e);
    }
    console.info('continue'); // 继续往下执行
    await delay();
}
```

## **async await原理**
前篇有说过`async await`其实是`Generator`的语法糖。
除了`*`换成`async`， `yield`换成`await`之外，最主要是`async await`内置了执行器，不用像`Generator`用那样`next()`一直往下执行。
其实也就是`async await`内部做了co模块做的事。

先来看看async await在TypeScript翻译后的结果：

```ts
async function run(){
    await delay();
    console.info('run');
}
//翻译成
function run() {
    return __awaiter(this, void 0, void 0, function* () {
        yield delay();
        console.info('run');
    });
}
```
可以注意到其实还是用`__await()`包了一个`Generator`函数，`__await()`的实现其实和上篇的co模块的实现基本一致：

```ts
var __awaiter = (this && this.__awaiter) ||
function(thisArg, _arguments, P, generator) {
	return new(P || (P = Promise))(function(resolve, reject) {
		function fulfilled(value) { // 也是fulfilled，resolved的别名
			try {
				step(generator.next(value)); // 关键还是这个step，里面递归调用fulfilled
			} catch (e) {
				reject(e);
			}
		}

		function rejected(value) {
			try {
				step(generator["throw"](value));
			} catch (e) {
				reject(e);
			}
		}

		function step(result) {
			result.done ? resolve(result.value) : new P(function(resolve) { //P是Promise的类型别名
				resolve(result.value);
			}).then(fulfilled, rejected); // 没有done的话继续fulfilled
		}
		step((generator = generator.apply(thisArg, _arguments)).next());
	});
};
```