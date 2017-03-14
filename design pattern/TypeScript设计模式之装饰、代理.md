看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 装饰模式 Decorator

### 特点：在不改变接口的情况下，装饰器通过组合方式引用对象，并由此在保持对象原有功能的基础上给对象加上新功能。

### 用处：当需要不影响现有类并增加新的功能时，可以考虑装饰模式，它可以动态透明的给对象增加功能。

### 注意：与继承的优劣。

下面用TypeScript简单实现一下装饰模式：
现在有一辆小轿车，加速到100km/h需要10秒：

```ts
interface Movable{
    accelerate();
}

class Car implements Movable{
    accelerate(){
        console.log('加速到100km/h需要10秒');
    }
}
```
现在想改装下，提高加速度，加个涡轮增压器。

```ts
class TurboCharger{
    use(){
        console.log('使用涡轮增压');
    }
}

class RefittedCar implements Movable{
    construct(private car: Car){
    }

    turboCharger = new TurboCharger();

    accelerate(){
        this.car.accelerate();
        this.turboCharger.use();
        console.log('加速到100km/h需要5秒'); 
    }
}

let refitterCar: Movable = new RefittedCar(new Car());

refitterCar.accelerate();

//输出：
加速到100km/h需要10秒
使用涡轮增压
加速到100km/h需要5秒
```
这样改装后的车用的还是原来的接口，但新对象却可以在保持原对象的功能基础上添加新的加速功能。
代码实现的特点主要还是在于使用同样接口，并在新对象里有对原对象的引用，这样才能使用原对象的功能。

上面的功能其实通过继承也很容易做到：

```ts
class TurboCharger{
    use(){
        console.log('使用涡轮增压');
    }
}

class RefittedCar extends Car{
    turboCharger = new TurboCharger();

    accelerate(){
        super.accelerate();
        this.turboCharger.use();
        console.log('加速到100km/h需要5秒'); 
    }
}

let refitterCar: Movable = new RefittedCar();
refitterCar.accelerate();

//输出同样是：
加速到100km/h需要10秒
使用涡轮增压
加速到100km/h需要5秒
```
用哪个好呢？对比看下各自的优缺点：（装饰器用的是组合）
继承的优点是 直观，容易理解，缺点是继承链太长的话将很难维护，并且耦合度较高，相对死板，不够灵活。
组合的优点是 灵活，但如果装饰器本身也比较多且复杂时，代码的复杂度也会增加不少。

就我个人来说在这种场景下还是用组合比较舒服，不是很喜欢在已经使用的类上继承出子类。
接上面的例子，有个皮卡也要增加加速功能，继承的话又得再实现一个皮卡子类，而用装饰模式的话使用时传进来就好了，本身不需要做任何更改。

当然，如果是从新设计，改下接口的话可能会选继承，毕竟直观很多，以后的需求有改动时再重构就好了，代码保持简单，怎么简单怎么来，所谓组合优于继承也要根据实际情况来定夺。

# 代理模式 Proxy

### 特点：在对象前面加了一层，外部需要通过这层代理来操作原对象，代理可以加一些控制来过滤或简化操作。

### 用处：当对象不希望被外部直接访问时可以考虑代理模式，典型的有远程代理、保护代理、透明代理、虚拟代理等。

### 注意：与外观、装饰器模式的区别。

远程代理：在Visual Studio里很容易用到，Web Reference，直接把web service当本地引用使用。
保护代理：通常用来对一个对象做权限控制。
虚拟代理：做web页面时对图像经常使用虚拟代理，看不到的图像先用个统一的图像替代，页面滑到哪就加载到哪，省资源。

下面用TypeScript简单实现一下远程代理模式：
数据接口：

```ts
interface DataService{
    getData(): string | Promise<string>;
}
```
在server端的远程服务：

```ts
class RemoteService implements DataService{
    getData(): string{
        return 'get remote data';
    }
}
```
假设一个Reqeuster类，可以取server端数据：

```ts
class Requester{
    Request(): Promise<string>{
        return Promise.resolve(new RemoteService().getData()); //这里本来应该从网络取，现在只是演示一下
    }
}
```
本地代理：

```ts
class DataProxy implements DataService{

    async getData(): string{
        return await new Requester().Request();
    }
}

function isPromise(p: string | Promise<string>): p is Promise<string> { //用来判断是否是promise
    return (<Promise<string>>p).then !== undefined;
}

let dataService: DataService = new DataProxy();
let data = dataService.getData();

if(isPromise(data)){
    data.then(o=>console.log(o));
} else {
    console.log(data);
}

//输出
get remote data
```
DataProxy和远程的RemoteService使用同样的接口，client像调用本地功能一样通过DataProxy使用远程功能。

代理模式与外观模式的差别在于：
代理和被代理使用同一抽象，并且代理着重于访问控制。
外观则着重于简化原本复杂的操作，并在此基础上提取新抽象。

代理模式与装饰器模式的差别在于：
代理的目的一般不是为了增加新功能而在于访问控制，同时代理通常是自己来创建被代理对象。
装饰器则着重于增加新功能，且被装饰对象通常是作为引用传给装饰器的。