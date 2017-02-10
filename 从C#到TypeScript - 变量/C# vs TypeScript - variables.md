TypeScript的变量声明和ES6差不多，相比之前主要是多了`let`和`const`

## **为什么不用var**
不管是TypeScript还是ES6都会兼容以前的javascript，所以在TypeScript里`var`也还是保留了。
虽然C#里也有`var`，但和JavaScript的可不一样，`var`在javascript里会有一些奇怪的表现，比如会置前，而且作用域是整个函数，可以不写`var`来声明变量，然后变量变成全局。
这些都可能会带了一些不容易注意到的问题。
比如经典的：

```js
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.info(i); }, 100);
}
```
结果并不是期望的0， 1， 2， 3...，跑出来的结果全是10，这是因为`var`出来的`i`的作用域是整个函数。
这就导致循环完成后`i`变成10，`setTimeout`内的函数才被执行，所以结果都是10了。
虽然可以用立即完成函数把重新构建一个作用域，但毕竟用起来麻烦，而且不符合人的思维，所以就有了`let`。

## **使用let声明变量**
`let`主要是对`var`的一个代替，用`let`更符合人思考的过程，这才和C#`var`的功能是差不多。
`let`的用法和`var`是一样的：

```ts
let str = 'string';
```
let的作用域是块级作用域，比如上面的循环，用`let`声明`i`的话就可心得到期望的值

```ts
for (let i = 0; i < 10; i++) {
    setTimeout(function() { console.info(i); }, 100);
}
```
这里得到的结果就是0, 1, 2....9。
所以建议还是抛弃`var`，选择`let`。

## **const**
C#也有const，意义上差不多，都是常量，不想变量被改变。

```ts
const str = 'string';
str = 'new string'; // 编译不了
```
一般情况下，主张确定不变的变量用`const`声明来增加代码健壮性和可读性。

## **解构**
所谓解构，就是把对象或数组里的成员分解出来。
比如数组：

```ts
let [first, second] = [1, '2', false];
console.info(`first: ${first}`);
```
这里就把数组的第一个成员和第二个成员分别用`first`和`second`解构出来，就省去了分别声明两个变量，并用下标取数组里的值来赋值了。
这也可以方便的提供一些功能，比如交换数组里的两个值，按以前的做法需要借助下中间变量，现在就不需要了：

```ts
let [first, second] = [second, first];
```

可以利用`...`扩展符号来解开数组，再并入其他数组。

```ts
let arr = [1, 2, 3];
let newArr = [...arr, 4, 5];
console.info(newArr); // 1, 2, 3, 4, 5
```

对象同样可以被解构：

```ts
class Test{
    str = "string";
    int = 1;
    bool = false;

    func(){
        console.info('func');
    }
}
let {str, bool, func} = new Test(); //名字必须和类里的保持一致
let {str: newStr} = new Test(); //通过这种方式可以把str改为newStr
console.info(`${str}, ${bool}`);
func();
```
`...`符号同样可以用于对象，不过只能解开可枚举的变量，所以函数不会解出来。
延用上面的class：

```ts
let {str, ...other} = new Test();
console.info(other.int); // 输出1
console.info(other.func()); // 编译错误，...符号不能解出函数
```
还可以加上默认值，当解出来的值是`undefined`时会用上默认值

```ts
class Test{
    empty;
    str = '';
}
let {empty='empty', str='str'} = new Test();
console.info(`${empty}, ${str}`); // 输出 empty, ，因为str有值，所以用原始的''
```

以上就是常用的变量声明及赋值的使用方法，不过基本都是ES6的标准语法，TypeScript本身并没有在上面多做什么。