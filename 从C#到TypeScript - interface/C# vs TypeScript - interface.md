为了更好的抽象出行为和属性，TypeScript在ES6的基础上增加了接口`interface`。
C#也有interface，不过TypeScript的接口还不大一样，C#里的接口一般是为类服务，让类实现接口中定义的方法或属性。
TypeScript在C#基础上更进一步，由于JavaScript是门非常灵活的语言，TypeScript作为JavaScript的超集需要保持灵活性，
所以接口在TypeScript里可以脱离具体的类，单独作为类似契约的存在，接口里的属性也并非一定需要实现。

### 类的接口
这和C#的差不多，描述了公共的成员；不过实现接口语法有点类似于Java，用的是implements。

```ts
interface Selectable {
    isSelected: boolean;
}

class Control implements Selectable {
    isSelected : boolean;
}
```
同C#一样，接口可以多重继承其他接口，用的是extends。

```ts
interface Editable {}
interface Deleteable {}

interface Changeable extends Editable, Deleteable {}
```

### 接口的属性
接口的属性可以定义为`readonly`，这个和C#里只有`get`没有`set`的属性有点像，同样，实现接口的类也不一定需要`readonly`。

```ts
interface Selectable{
    readonly isSelected: boolean;
}

class Control implements Selectable{
    isSelected: boolean;
}

let s: Selectable = { isSelected : true };
s.isSelected = false; // 编译出错, readonly

let c: Control = { isSelected : true };
c.isSelected= false; // 没问题
```
另外，接口还支持可选属性，同C#的可空属性一样，用`?`表示，实现接口的类可以不用实现可选属性。

```ts
interface RequestConfig {
    url: string;
    body?: any;
}

class Request implements RequestConfig {
    url: string;
}
```

### 接口不需要类的支持
在C#里面，接口如果没有类来实现的话是没有什么意义的，但在TypeScript里不一样，接口可以单独使用。

```ts
interface RequestConfig {
    url: string;
    body?: any;
}

let config: RequestConfig = {url: 'www.google.com'};
```
这种经常用在函数的参数上面，用来描述具体的参数，把具体的参数放到接口里，方便操作，也方便重构。

```ts
function Request(config: RequestConfig){

}
```
接口除了描述属性外，还可以用来描述函数，不过一个接口只能描述一个函数，描述时定义好参数和返回值即可。
从实现上看有点类似于C#的`delegate`。

```ts
interface CheckLogin {
    (name: string, pwd: string): boolean;
}

let check: CheckLogin = function(name: string, pwd: string): boolean {
    return false;
}
```
另外，接口不可以用来描述可索引类型，就有点类似C#的`Dictionary`。
索引支持两种：`number`和`string`。

```ts
//定义一个Dict，key是string，value也是string
interface Dict {
    [key: string] : string;
}

let dict: Dict = { 'key1': 'value1', 'key2': 'value2'};
console.info(dict['key1']); // value1
console.info(dict['key']); // undefined
```

### 接口继承类
这在C#中很不可思议，接口居然还可以反过来继承类，不过对于JavaScript里来说，灵活方便很重要，所以TypeScript实现了这个功能来快速生成一个接口。
虽说在比较复杂的继承关系时可能会有用，不过个人认为有点鸡肋，因为复杂的继承通常会引入一些问题如紧耦合，牵一发而动全身，再加上这个，可能更让人摸不着头脑，不如用组合来得好。
接口继承类时会继承类中所有的成员，不管是`private`,`protected`还是`public`，只是不包括其实现。
不过继承了一个类不公开成员的接口只能被该类或该类的子类实现。

```ts
class User{
    name: string;
    protected pwd: string = "123";
}

class Admin extends User{
    constructor(n: string, p: string){
        super();
        this.name = n;
        this.pwd = p;
    }
}

interface UserConfig extends User{ //这里包含了name和private的pwd
}

let config: UserConfig = new Admin('brook', '123');
```

### 泛型
TypeScript是同C#一样支持泛型的，而且使用方面也差不多，在接口名后面加上`<T>`即可。

```ts
interface Testable<T> {
    field: T;
    (arg: T): T;
}
```
也支持泛型约束，关键字是`extends`。

```ts
interface Testable<T extends Object> {
    field: T;
    (arg: T): T;
}
```

TypeScript的接口对于C#程序员来说是有点奇怪了，不过用过之后还是发现非常符合JavaScript语言灵活的特性。