在C#里如果想只通过名字来生成类实例、获取属性或执行方法可以使用反射，反射是基于元数据，现在很多流行语言都支持元数据，以此来提供更多便利的功能。
ES6和TypeScript也有Reflect，不过因为JavaScript本身是解释型语言，很多操作如根据名字字符串获取属性，根据字符串执行函数这些原本就有支持，Reflect只是把这些操作归结到一起。
下面来通过例子来看下TS Reflect常见的用法。

## **Reflect Get/Set**
定义如下：

```ts
Reflect.get(target, name, receiver);
Reflect.set(target, name, value, receiver);
```
看上去也很好理解，和C#很类似：
target：操作的对象
name：名字字符串
value：要赋的值
receiver：这个比较怪，因为类里可以有getter/setter属性，这两种操作可以在代码块里使用`this`，如果要用Reflect操作的话，receiver就会代替这个`this`。
Reflect的操作即使是类的private变量也能获取到。

```ts
class Test{
    constructor(age: number){
        this.age = age;
    }

    private _age: number;

    get age(): number{
        return this._age;  // this 会被receiver代替
    }

    set age(value: number) {
        this._age = value; // this 会被receiver代替
    }
}

class Receiver{
    _age: number = 2;
}

let t = new Test(1);
let r = new Receiver();

console.info(Reflect.get(t, "_age")); // 1, 获取t的_age值
console.info(Reflect.get(t, "age")); // 1, 获取t的age值
console.info(Reflect.set(t, "age", 3)); // true, 成功设置age值为3
console.info(Reflect.get(t, "age")); // 3, 再次获取t的age值
console.info(Reflect.get(t, "age", r)); // 2, 表面上是t的age，但实际上获取的是r的age
console.info(Reflect.set(t, "age", 3, r)); // true, 表面上是设置t的age, 实际上是设置r的age值为3
console.info(Reflect.get(r, "_age")); // 3, 直接获取r的_age
```

## **apply**
上面是属性，还有方法，定义如下：

```ts
Reflect.apply(func, thisArg, args);
```
熟悉JS的朋友应该知道Function也有apply方法，`fn.apply(obj, args)`，可以说是同样的效果。
如果要通过函数名来调用函数，可以这样做：

```ts
class Test{
    add(a: number, b: number): number{
        return a + b;
    }
}

let t = new Test();
console.info(Reflect.apply(t["add"], t, [1, 2])); // 3, 虽然t["add"]可以直接执行，不过有时可能需要设置thisArg
```

## **define/delete property**
define相比之前就真是简单把Object替换成了Reflect，delete和`delete obj[name]`效果一样。

```ts
Reflect.defineProperty(target, propertyKey, attributes);
Reflect.deleteProperty(obj, name);
```
例子延用上面的对象t:

```ts
//define
Reflect.defineProperty(t, 'time', {
  value: Date.now()
});

console.info(t.time); // 一串数字

//delete
let d = {time: 111};
console.info(d.time); // 111
Reflect.deleteProperty(d, 'time'); // 成功的话返回true，否则返回false
console.info(d.time); // undefined
```
可以看到define的参数`attributes`是一个PropertyDescriptor对象，`value`就是值，其他还有`writable, enumerable, configurable`用来控制属性的权限。
对于delete，需要注意的是deleteProperty对class的属性是无效的。

## **has ownKeys**
`ownKeys`返回的是对象所有属性，包括不可枚举的，如Symbol之类。
`has`用来判断对象是否有某个属性或方法，包括原型链上的。

```ts
class Test{
    constructor(name: string){
        this.name = name;
    }

    name: string;
    flag: Symbol = Symbol('flag');

    getName(): string{
        return this.name;
    }
}

let obj = new Test('123');
console.info(Reflect.has(obj, 'name')); // true
console.info(Reflect.has(obj, 'flag')); // true
console.info(Reflect.has(obj, 'get')); // true
console.info(Reflect.has(obj, 'toString')); // true
for(let p of Reflect.ownKeys(obj)){
    console.info(p); // name, flag
}
```

## **其他**
- **Reflect.construct(target,args)**

    实例化对象除了new之外，还可以用这个，有时候很有用，比如ORM框架里join的字段就可以在设置表时把关联的类型传给字段，使用时用该类型就可以创建出实例。
- **Reflect.getPrototypeOf(target) 和 Reflect.setPrototypeOf(target, prototype)**

    分别用于获取和设置对象的原型
- **Reflect.getOwnPropertyDescriptor(target, name)**

    设置对象属性的描述对象，如`configurable, writable, enumerable`。
- **Reflect.isExtensible(target)**

    分别用于判断对象是否可扩展。
- **Reflect.preventExtensions(target)**

    让一个对象变为不可扩展

Reflect基本上就是把之前Object的方法和一些命令如`delete` `in`之类聚到一起，相信ES6之后用Reflect来做这些操作将成为主流。