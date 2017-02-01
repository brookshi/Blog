在ES6之前Javascript的类都是用function定义的，ES6把类关键字正式加进来，虽说其实也还是function，不过代码可读性上好了不少。
TypeScript同样支持class，并且和C#也非常相似，下面来看看：

### 类
同C#一样，由构造函数，属性，方法组成，属性和方法有三个级别的访问权限：`private, protected, public`，比C#少个`internal`。
不过不同的是C#类的成员默认是`private`，而TypeScript默认是`public`。
在类里面所有成员都必须用`this`来访问。

```ts
class User{
    constructor(name: string, pwd: string){
        this.name = name;
        this.pwd = pwd;
    }

    name: string;
    private pwd: string;

    checkLogin(): boolean{
        return this.name === 'brook' && this.pwd === '123';
    }
}

let u: User = new User('brook', '123');
console.info(u.checkLogin()); // true

u.name = 'test';
console.info(u.checkLogin()); // false
```

### 参数属性
上面的`User`类有两个成员，而且都是从构造函数赋值的，也就是其实构造函数的参数就是类的成员，这就是参数属性。
类里面的那两个属性其实可以不用写，只要在构造函数的参数上加上操作限定符，TypeScript就会自动为参数生成属性，来重构下上面的`User`。

```ts
class User{
    constructor(public name: string, private pwd: string){ }

    checkLogin(): boolean{
        return this.name === 'brook' && this.pwd === '123';
    }
}
```

### getter/setter
同样，也有存取器，不过语法和C#不太一样，写起来稍麻烦些。
只有`get`的时候也就变成只读属性了。

```ts
class User{
    private _name: string;

    get name(): string{
        return this._name;
    }

    set name(name: string){
        this._name = name;
    }
}
```

### 静态属性和方法
上面说的都是实例成员，TypeScript也支持静态成员，不用实例化，而是通过类名来访问。

```ts
class User{
    static permission = 'user';

    static setPermission(p: string){
        User.permission = p;
    }
}

console.info(User.permission); // user

User.setPermission('admin');
console.info(User.permission); // admin
```
也同时支持`static`和`readonly`，不过`static`要放在前面，这样可以实现单例模式。

```ts
class User{
    
    static readonly instance = new User();

    private constructor(){}

    checkLogin(name: string, pwd: string): boolean{
        return name === 'brook' && pwd === '123';
    }
}

console.info(User.instance.checkLogin('brook', '123'));
```

### 抽象类
这点和C#一样，都可以用抽象类来把有共同行为抽象出来，关键字都是`abstract`。
不能实例化，可以包含实现，`abstract`标识的方法，继承类必须实现。
但没有`virtual`关键字，不过和Java一样，可以认为是天生虚函数，也不需要`override`，直接覆盖也能支持多态。
继承类里要调用父类的函数需要用`super`关键字。

```ts
abstract class User{
    name: string;
    pwd: string;

    abstract checkLogin(): boolean;
    
    checkName(): boolean {
        return this.name.indexOf('.') < 0;
    }
}

class Admin extends User {
    checkLogin(): boolean {  // 必须实现的抽象方法
        return this.checkName();  
    }

    checkName(): boolean {  // 这里会把User里的checkName覆盖掉
        return true && super.checkName();
    }
}

let user: User;
user = new Admin(); 
user.name = 'brook.shi';
console.info(user.checkLogin()); // 同样有多态，checkLogin里调用的是Admin的checkName
```
另外，继承时还需要注意，如果派生类里有构造函数，则构造函数必须要调用父类的构造函数：`super()`。

### 兼容性
TypeScript里的类是有兼容性的，这点和C#很不一样，TypeScript认为：只有成员的类型是兼容的，那它们的类型也是兼容的。
不过成员的兼容对于`public`很宽容，完全不想干的类如果全是public的成员，并且一致就认为是兼容的。
但对于`private`和`protected`则只有是继承的才会被认可为兼容。

```ts
class Test1{
    name: string;
    pwd: string;

    checkName(): boolean{
        return true;
    }
}


class Test2{
    name: string;
    pwd: string;

    checkName(): boolean{
        return false;
    }
}

let t: Test1 = new Test2();
console.info(t.checkName());  // false
```
如果给上面的`Test`和`Test2`各加一个`private email: string;`，结果是编译不了，因为它们并不是继承关系，也就兼容不了。

### 泛型
同接口一样支持泛型，用法也一样，可以参考接口泛型。

```ts
interface Testable<T> {
    field: T;
    method(arg: T): T;
}

class Test<T> implements Testable<T>{
    field: T;
    method(arg: T): T{
        console.info(`arg is ${typeof arg}`);
        return null;
    }
}

let test11 = new Test<string>();
test11.method('method');  // arg is string
test11.method(123); // error, 123 is not string
```

总的来说，TypeScript的类和C#或Java可以说十分相似，除了兼容性基本上没有什么新的东西。