虽然TypeScript里有了类，但做JavaScript的`function`也还在，这也是和C#的不同所在。
C#里函数不能脱离类工作，但TypeScript的`function`和JavaScript一样，可以单独工作。

### 函数类型
函数和C#一样可以有名字，也可以是匿名函数，匿名函数有两种写法：

```ts
function checkLogin(name: string, pwd: string): boolean{
    return true;
}

let checkLogin = (name: string, pwd: string) => {
    return false;
}

let checkLogin = function(name: string, pwd: string){
    return true;
}
```
前面文章写变量声明时有写变量类型`let str: string`，但上面其实都没有把函数的类型真正写出来，比如最后一个`let checkLogin`里并没有标明返回的类型。
如果要把函数做为参数或返回值的话如果没有明确类型的话使用会很不方便，没有智能提示，重构也不方便。
函数的返回类型其实就是由参数+返回类型构成。

```ts
let checkLogin: (name: string, pwd: string) => boolean = function(name: string, pwd: string){
    return true;
}
```
返回类型里的参数名不需要与真正的参数名一致，只需要类型一致即可。
当然，大部分情况下是不用写这么复杂的返回类型的，前面文章有说过类型推论，TypeScript会根据上下文推论出返回值的类型。

### 函数参数
TypeScript的参数和JavaScript的参数不太一样，调用JavaScript函数的参数可以多或少都可以，但TypeScript里函数需要确保传入参数的个数和定义的一致。
同C#里的函数参数可以有默认值一样，TypeScript也支持，并且还支持可空参数。
默认值只需要在参数后面写上`=`某值就可以，默认值参数可以在任意位置，不过在必须参数前面时，想用默认值的话需要传`undefined`。
可空参数和前面说的可空属性一样，参数名后加`?`号，可空参数必须是在最后面。

```ts
function checkLogin(name: string, pwd: string, isAdmin: boolean = false, email?: string){
    console.info(isAdmin);
}

checkLogin('brook'); // 编译不了
checkLogin('brook', '123456'); // false
checkLogin('brook', '123456', undefined); // false
checkLogin('brook', '123456', false); // false
checkLogin('brook', '123456', true, 'brook@email.com'); // true
```

### 剩余参数
JavaScript里的参数本身是个数组，可以是任意个数且都可以在函数体内用`arguments`来访问。
TypeScript同样可以通过剩余参数来支持。
剩余参数的格式是`...restParam: string[]`。

```ts
function checkLogin(name: string, pwd: string, ...others: string[]){
    console.info(others.join(' '));
}

checkLogin('brook', '123456', 'brook@email.com', `12800`); // brook@email.com 12800
```

### this
`this`在JavaScript里总是指向调用者，这点经常容易导致被坑，在ES6之前经常需要类似`var self = this`来把`this`保存下来。
ES6和TypeScript针对这点做了改进，使用箭头函数可以把创建函数时的`this`自动保存下来。

```ts
let permission = {
    name: 'brook',

    checkLogin: function() {
        return function() {
            console.info(this.name === 'brook');
        }
    }
};

permission.checkLogin()(); // 出错， this为undefined

let permission = {
    name: 'brook',

    checkLogin: function() {
        return () => {
            console.info(this.name === 'brook');
        }
    }
};

permission.checkLogin()(); // 通过，因为用了箭头函数
```
不过上面的`this`还是有个缺点，得到的`this`是`any`类型，这样重构起来会不方便，这时可以用`this`参数来改进：

```ts
interface Permission{
    name: string;
    checkLogin(this: Permission): ()=>void;
}

let permission: Permission = {
    name: 'brook',

    checkLogin: function(this: Permission) {
        return () => {
            console.info(this.name === 'brook'); //这里的this不是any，而是Permission
        }
    }
};

permission.checkLogin()();
```
这样也倒逼着定义好类型，发挥TypeScript强类型的优势。

### 泛型函数
同C#一样支持泛型函数，写法也差不多。

```ts
function deserialize<T>(content: string): T { }

function addItem<TKey, TValue>(key: TKey, value: TValue) { }
```
也支持泛型约束，C#用的是`where T: object`，而TypeScript用的是`extends Object`。

```ts
function deserialize<T extends Object>(content: string): T { }
```
泛型函数类型比普通函数类型在前面多了个`<T>`，比如上面`deserialize`。

```ts
let deserialize: <T>(content: string) => T;
```
但这样如果做为参数就略显复杂，可以用接口重构下：

```ts
function deserialize<T extends Object>(content: string): T { }

interface serializable<T> {
    (content: string): T;
}

function parse<T>(s: serializable<T>){ }

parse(deserialize);
```