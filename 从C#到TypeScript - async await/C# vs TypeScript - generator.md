上篇讲了`Promise`，`Promise`的执行需要不停的调用`then`，虽然比callback要好些，但也显得累赘。所以ES6里添加了`Generator`来做流程控制，可以更直观的执行Promise，但终级方案还是ES7议案中的`async await`。
TypeScript在1.7版本开始支持`async await`编译到ES6，并在2.1版本开始支持编译到ES5和ES3，算是全面支持了。

当然`async await`本质上也还是`Generator`，可以算是`Generator`的语法糖。
所以这篇先来看下Generator.

## **Generator语法**
先来看个例子：

```ts
function* getAsync(id: string){
    yield 'id';
    yield id;
    return 'finish';
}

let p = getAsync('123');
console.info(p.next()); 
console.info(p.next());
console.info(p.next());
```
先看下和普通函数的区别，`function`后面多了一个`*`，变成了`function*`，函数体用到了`yield`，这个大家比较熟悉，C#也有，返回集合有时会用到。
在ES6里`yield`同样表示返回一个迭代器，所以用到的时候会用`next()`来顺序执行返回的迭代器函数。
上面代码返回的结果如下：

```json
{ value: 'id', done: false }
{ value: '123', done: false }
{ value: 'finish', done: true }
```
可以看到`next()`的结果是一个对象，`value`表示`yield`的结果，`done`表示是否真正执行完。
所以看到最后return了`finish`时`done`就变成true了，如果这时再继续执行`next()`得到的结果是`{ value: undefined, done: true }`.

## **Generator原理和使用**
`Generator`其实是ES6对协程的一种实现，即在函数执行过程中允许保存上下文同时暂停执行当前函数转而去执行其他代码，过段时间后达到条件时继续以上下文执行函数后面内容。
所谓协程其实可以看做是比线程更小的执行单位，一个线程可以有多个协程，协程也会有自己的调用栈，不过一个线程里同一时间只能有一个协程在执行。
而且线程是资源抢占式的，而协程则是合作式的，怎样执行是由协程自己决定。
由于JavaScript是单线程语言，本身就是一个不停循环的执行器，所以它的协程是比较简单的，线程和协程关系是 1:N。
同样是基于协程goroutine的go语言实现的是 M:N，要同时协调线程和协程，复杂得多。
在`Generator`中碰到`yield`时会暂停执行后面代码，碰到有`next()`时再继续执行下面部分。

当函数符合`Generator`语法时，直接执行时返回的不是一个确切的结果，而是一个函数迭代器，因此也可以用`for...of`来遍历，遍历时碰到结果`done`为true则停止。

```ts
function* getAsync(id: string){
    yield 'id';
    yield id;
    return 'finish';
}

let p = getAsync('123');
for(let id of p){
    console.info(id);
}
```
打印的结果是：
```
id
123
```
因为最后一个`finish`的`done`是true，所以`for...of`停止遍历，最后一个就不会打印出来。
`Generator`的`next()`是可以带参数的，

```ts
function* calc(num: number){
    let count = yield 1 + num;
    return count + 1;
}

let p = calc(2);
console.info(p.next().value); // 3
console.info(p.next().value); // NaN
//console.info(p.next(3).value); // 4
```
上面的代码第一个输出是`yield 1 + num`的结果，`yield 1`返回1，加上传进来的2，结果是3.
继续输出第二个，按正常想法，应该输出3，但是由于`yield 1`是上一轮计算的，这轮碰到上一轮的`yield`时返回的总是`undefined`。
这就导致`yield 1`返回`undefined`，undefined + num返回的是`NaN`，count + 1也还是NaN，所以输出是`NaN`。
注释掉第二个，使用第三个就可以返回预期的值，第三个把上一次的结果3用next(3)传进去，所以可以得到正确结果。
如果想一次调用所有，可以用这次方式来递归调用：

```ts
let curr = p.next();
while(!curr.done){
    console.info(curr.value);
    curr = p.next(curr.value);
}
console.info(curr.value);
```
`Generator`可以配合`Promise`来更直观的完成异步操作。

```ts
function delay(): Promise<void>{
    return new Promise<void>((resolve, reject)=>{setTimeout(()=>resolve(), 2000)});
}

function* run(){
    console.info('start');
    yield delay();
    console.info('finish');
}

let generator = run();
generator.next().value.then(()=>generator.next());
```
就`run`这个函数来看，从上到下执行是很好理解的，只是执行时需要不停的使用`then`。
好在TJ大神写了[CO模块](https://github.com/tj/co)，可以方便的执行这种函数，把`Generator`函数传给`co`即可。

```ts
co(run).then(()=>console.info('success'));
```
co的实现原理可以看下它的核心代码：

```js
function co(gen) {
  var ctx = this;
  var args = slice.call(arguments, 1);

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.apply(ctx, args);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);

    onFulfilled(); //最主要就是这个函数，递归执行next()和then()

    function onFulfilled(res) { 
      var ret;
      try {
        ret = gen.next(res); // next(), res是上一轮的结果
      } catch (e) {
        return reject(e);
      }
      next(ret); // 里面调用then，并再次调用onFulfilled()实现递归
      return null;
    }

    function onRejected(err) { // 处理失败的情况
      var ret;
      try {
        ret = gen.throw(err);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }

    function next(ret) {
      if (ret.done) return resolve(ret.value); // done是true的话表示完成，结束递归
      var value = toPromise.call(ctx, ret.value);
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
      return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
        + 'but the following object was passed: "' + String(ret.value) + '"'));
    }
  });
}
```
可以看到co的核心代码和我上面递归调用`Generator`函数的本质是一样的。

纵使有co这个库，但是使用起来还是略有不爽，下篇就轮到`async await`出场，前面这两篇都是为了更好的理解下一篇。