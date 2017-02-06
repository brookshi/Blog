TypeScript和C#一样是微软搞出来的，而且都是大牛Anders Hejlsberg领导开发的，它们之间有很多共同点，现在尝试以C#程序员的角度来理解下TypeScript。
TypeScript一门是JavaScript的超集语言，除了支持最新的JS语法外，TypeScript还会增加一些其他好用的语法糖，最重要的是它在兼顾JavaScript灵活的基础上增加了强类型系统，这样更友好的支持开发大型系统。

现在来看下TypeScript基础类型：
### 数值
C#的数字类型有好几种：`int, long, float, double, byte`等，而TypeScript和JavaScript一样，所有的数字都是浮点数，都是用`number`表示，这样也省了很了事，少了C#里类似`long`转`int` overflow问题。

下面用不同进制方式显示数字20。

```ts
let num = 20;       // 10进制
let num = 0xa4;     // 16进制
let num = 0b10010;  // 2进制
let num = 0o24;     // 8进制
```
### 布尔
`boolean`，和C#的功能一样，不多说。

```ts
let isCheck: boolean = true;
```

### 枚举
`enum`，大家都知道javascript没有`enum`，这也是TypeScript为此作的补充。功能上和C#差不多：
1. 目的都是为数值提供一个友好的名字，增加代码可读性和可重构性
2. 默认情况下从0开始编号
3. 也可以手动赋值
4. 可以实现类似C# Flag特性
但也有一些细节不一样：
1. C#的枚举值`toString()`会返回枚举的文本值，而TypeScript是数值
2. TypeScript可以通过数值下标取得枚举字符串值

```ts
enum Action{
    add = 1,
    edit = 2,
    del = 4,
    all = add | edit | del
}

console.info(Action.add);  // 返回1
console.info(Action.add.toString());  // 返回1
console.info(Action[1]);  // 返回"add"
console.info(Action[3]);  // 返回undefined
console.info(Action.all); // 返回7
console.info(Action.all & Action.add) //返回1
```

上面的Action编译成JavaScript的结果：

```js
var Action;
(function (Action) {
    Action[Action["add"] = 1] = "add";
    Action[Action["edit"] = 2] = "edit";
    Action[Action["del"] = 4] = "del";
    Action[Action["all"] = 7] = "all";
})(Action || (Action = {}));
```

### 字符串
字符串也基本和C#一样，不过由于是JavaScript的超集，所以当然也支持单引号。
C#6.0里的模板字符串语法糖`$"this is {name}'s blog"`在TypeScript里也有类似的支持，当然，这也是ES6的规范。

```ts
let name: string = 'brook';
let note: string = `this is ${name}'s blog`;
```

### Symbol
这也是ES6的特性，用来当作唯一的标识，所有新建出来的Symbol都是不同的，不管传进去的值是否一样。
Symbol非常适合做唯一key。

```ts
let key1 = Symbol('key');
let key2 = Symbol('key');

console.info(key1 === key2); // return false
```

### any
这个和C#的`dynamic`很相似，可以代表任何东西且在上面调用方法或属性不会在编译时期报错，当然也本来就是JavaScript最基本的东西。

```ts
let test: any = 'test';
test = false;

test.test(); //编译时期不会有报错
let arr: any[] = ['test', false];
```

### void、null、undefined和never
`void`和C#的一样，表示没有任何东西。
`null`和`undefined`和JavaScript一样，分别就是它们自己的类型，个人觉得这两者功能有点重合，建议只使用`undefined`。
`never`是TypeScript引进的，个人觉得是一种语义上的类型，用来表示永远不会得到返回值，比如`while(true){}`或`throw new Error()`之类。

```ts
function test(): void{} //  void
let a: string = null; let b: null = null; // null有自己的类型，并且默认可以赋值给任何类型（除never之外），可用--strictNullChecks标记来限制这个功能
let a: string = undefined;  let b: undefined = undefined; // undefined， 同上
function error(): never{ // never
    throw new Error('error');
}
```

### 数组
有基本的数组: 

```ts
let arr: string[] = ['a', 'b', 'c'];
```
也有类似C#的泛型`List`

```ts
let list: Array<string> = ['a', 'b', 'c'];
```
数组功能没C#配合`linq`那么强大，不过配合其他一些库如`lodash`也可以很方便的进行各种操作。
数组还可以利用扩展操作符`...`来把数组解开再放入其他数组中。

```ts
let arr: number[] = [1, 2, 3];
let newArr: number[] = [...arr, 4, 5];
console.info(newArr); // 1, 2, 3, 4, 5
```

### 元组
C#也有个鸡肋的`Tuple`，不好用，不过新版的`Tuple`好像已经在C#7.0的计划当中。
下面这段代码是C#7.0的，真方便，不用再`new Tuple<>，item1, item2`之类的。

```ts
(string first, string middle, string last) LookupName(long id)
{
    return (first:'brook', middle:'', last:'shi');
}

var name = LookupName(id);
console.WriteLine(`first:${name.first}, middle:${name.middle}, last:${name.last}`);
```
TypeScript里的也不输给C#，不过叫法上是分开的，这里的元组只是对数组的处理，另外还有对象上的叫解构赋值，以后会写。

```ts
let tuple: [number, string] = [123, '456'];
let num = tuple[0]; //num
let str = tuple[1]: //string

tuple[3] = '789'; //可以，越界后会以联合类型来判断，后面会讲联合类型
tuple[4] = true; //不行
```
这一篇主要就讲这些基本类型，下一篇会讲TypeScript的高级类型。


上一篇讲了基础类型，基本上用基础类型足够开发了，不过如果要更高效的开发，还是要看下高级类型，这篇和C#共同点并不多，只是延用这个主题。

### 联合类型
可以从字面上进行理解：其实就是多个类型联合在一起，用`|`符号隔开。
如： `string | number`， 表示希望这个类型既可以是`string`，又可以是`number`。
联合类型的字段只能调用这些类型共同拥有的方法，除非类型推论系统自动判断出真正的类型。

```ts
//这里sn就是一个联合类型的字段，由于类型推论推断出sn肯定是string，所以sn可以调用string的所有方法
let sn: string | number = 'string, number';
//这里就推断不出具体的类型，只能调用toString, toValue了
function snFunc(): string | number{
    return 'string, number';
}
```
联合类型不光是可以联合基本类型，也可以是用户自定义的`class, interace`等。

### 交叉类型
有`|`就有`&`，交叉类型就是用`&`符号隔开，表示把多个类型合在一起，新类型包含所有类型的功能。
一些语言如Python有`mixins`功能，用这个就很容易做到，主要是类似多重继承，不过个人不是用喜欢这个，明显违反了单一原则。
下面这段代码就显示了`mixins`结合两个类的功能，看起来是不是有点不大合理，目前的趋势也是组合优先，用组合同样也可以做到这些。

```ts
class Person {

    talk(): string {
        return `一个人`;
    }
}

class Dog {

    bark(): string {
        return '汪汪汪';
    }
}

function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    for (let func of Object.getOwnPropertyNames(Object.getPrototypeOf(first))) {
        (<any>result)[func] = (<any>first)[func];
    }
    for (let func of Object.getOwnPropertyNames(Object.getPrototypeOf(second))) {
        (<any>result)[func] = (<any>second)[func];
    }
    return result;
}

let personDog = extend(new Person(), new Dog());
console.info(personDog.talk());
console.info(personDog.bark());
```

### 类型转换
C#里常用的类型转换一个是前面圆括号加类型，一个是`as`。
TypeScript和C#一样，只不是圆括号改成尖括号。

```ts
let test: any = '123';
let str1: string = <string>test;
let str2: string = test as string;
```

### 类型保护
联合类型返回的是多个类型的其中一个，但是用的时候并不知道是哪个，需要一个一个判断，这显得很麻烦。

```ts
function get(): number | string{
    return 'test';
}
let test = get();
var len = test.length; //编译不了，不知道test到底是number还是string
let str = '';

if((<string>test).sub){
    // string
} else {
    // number
}
```
除了通过是否有string特有的方法来判断是否是string，也可以用类似C#的`typeof`来得到它的类型，而且重要的是会提供类型保护机制，
即在`typeof`作用域里会知道这个变量的类型。

```ts
function get(): number | string{
    return 'test';
}
let test = get();
if(typeof test === 'string'){
    console.info(test.length); // 这里由于typeof确定了test类型是string，所以作用域内可以直接取length，而不用<string>转一次
}
```
`typeof`比较是有限制的，自己创建的类返回的都是`object`，这时会用到`instanceof`，并且`instanceof`同样会提供类型保护机制。
另外还有类型断言可以提供类似的功能，不过不如上面的来得方便。

```ts
function get(): number | string{
    return 'test';
}
let test = get();
function isStr(p : number | string): p is string{
    return (<string>p).sub !== 'undefined';
}

if(isStr(test)) {
    console.info(test.length);
} else {
    console.info(test + 1);
}
```
上面`p is string`就是断言参数`p`是`string`类型，从而在`isStr`后可以自动得到`test`的类型，并且在`else`里也知道是`number`类型。

这点上比C#来得好，一般C#做法可能是用`as`操作符转过来，然后判断是否为空，如果类型多操作起来也很复杂。

### 类型别名
类型别名即可以为现有类型取一个新名字。

```ts
type newstring = string;
let str: newstring = 'aaa';
console.info(str.length);
```
在C#中也可以用`using strList = System.Generic.List`做个别名，不过还是不一样，C#的是可以实例化的。
TypeScript别名不是新建一个类型，而是现有类型的一个引用。
给现在类型起别名意义不大，倒是可以配合联合类型或交叉类型做成一些可读的或比较新颖的类型。
别名也支持泛型，现在就有一个用别名创建了一个Tree类型，不过也只是别名，不能实例化，只能是看的，这点不如接口实在。

```ts
class Chicken{}
class Duck{}
type Fowl = Chicken | Duck;

type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

### 字符串字面量类型
TypeScript可以让string成为一个类型，比如`let strType = 'string type'`。
这个可以用在方法参数中，用来限制参数的输入。

```ts
function test(param1: 'test1' | 'test2' | 'test3'){

}
test('test'); // 编译不了，参数只能是test1, test2或test3
```

### 可辨识联合
综合上面的字符串字面量类型、联合类型、类型保护、类型别名可以创建一个可辨识联合的模式。
必须要在自定义的多个类中有相同的字段，这个字段用的是字符串字面量类型并且把这些类型联合起来。

```ts
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}

interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;

// 这里就可以用可辨识联合
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

### 类型推论
TypeScript可以根据赋值或上下文推论出变量的类型，所以有时可以不用明确标明变量或函数返回值的类型。

```ts
let x = 123; // 这里能推论出x是number，就不用写成这样: let x: number = 123;

function get(){ 
    return [1, 2, 3];
}
let arr = get(); // 这里也能推论出arr是number[];

function get(){
    return [1, '2', 3];
}
let arr = get(); // 这里能推论出arr是(number | string)[];
```
不过个人觉得除了一些很明显的`let x = 123`之类可以不写，其他的最好还是写上类型，增加代码可读性。

以上就是TypeScript的类型了，比较灵活也比较难，可能要在实际项目中用用就会比较好掌握。