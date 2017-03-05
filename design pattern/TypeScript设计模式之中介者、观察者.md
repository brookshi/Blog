看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 中介者模式 Mediator

### 特点：为减少对象间的互相引用而引入的一个中介对象，用来来封装一系列对象的互相操作。

### 用处：当多个对象间需要互相引用且互相频繁操作时可以考虑中介者模式，如MVC里的Controller。

### 注意：中介者本身的复杂度。

下面用TypeScript简单实现一下中介模式：
现在滴滴打车其实就可以算是中介，就以这个例子吧，对象主要就是用户，车主和平台。

先定义用户, 车主和中介者接口：
用户的行为是叫车，车主是接送，中介者则需要维护用户和车主列表并且知道车的状态和提供叫车服务。

```ts
interface Client{
    getTaxi();
    pay();
} 

interface Car{
    isWorking: boolean;

    startWork();
    finishWork();
}

interface Mediator{

    registerClient(client: Client);
    registerCar(car: Car);

    getCar(): Car;
    pay(car: Car);
    updateCarStatus(car: Car);
}
```
接口定义好了就可以写实现了：
用户的实现，持有中介者的引用，用来注册，叫车和付款，本来是没必要存taxi的，只需要个id就可以了，具体是由中介去做匹配，不过这里为了简单就直接存对象了

```ts
class User implements Client{
    taxi: Car;

    constructor(private mediator: Mediator){
        this.mediator.registerClient(this);
    }

    getTaxi(){
        this.taxi = this.mediator.getCar();
        if(this.taxi){
            console.log('车来了');
        } else {
            console.log('没叫到车');
        }
    }

    pay(){
        this.mediator.pay(this.taxi);
        console.log('付款');
    }
}
```
车主的实现，同样需要持有中介者引用来领任务和报状态

```ts
class Taxi implements Car{
    isWorking: boolean = false;

    constructor(private mediator: Mediator){
        this.mediator.registerCar(this);
    }

    startWork(){
        console.log('有人叫车');
        this.isWorking = true;
        this.mediator.updateCarStatus(this);
    }

    finishWork(){
        console.log('送完这趟了');
        this.isWorking = false;
        this.mediator.updateCarStatus(this);
    }
}
```
中介的实现，中介的作用就是提供车子服务，这里为了简单没维护用户与车子的关系

```ts
class DiDi implements Mediator{
    private clientList: Array<Client> = [];
    private carList: Array<Car> = [];

    registerClient(client: Client){
        this.clientList.push(client);
    }

    registerCar(car: Car){
        this.carList.push(car);
    }

    getCar(): Car{
        let car = this.carList.find(o=>!o.isWorking);
        car.startWork();
        return car;
    }

    pay(car: Car){
        car.finishWork();
    }

    updateCarStatus(car: Car){
        console.log(`车子状态：${car.isWorking ? '工作' : '闲置'}`);
    }
}
```
跑一下看看：

```ts
let didi = new DiDi();
let taxi = new Taxi(didi);
let user = new User(didi);
user.getTaxi();
user.pay();

//结果
有人叫车
车子状态：工作
车来了
送完这趟了
车子状态：闲置
付款
```
这样，用户的目的只是叫车，对中介说声，中介派出车，用户不管是什么车，哪来的，把我送到目的车就可以了。
这就是中介者模式的作用，逻辑都在自己这里，用户不需要管车，车子也不用管用户，一个叫车，一个接单，互不干扰，突然想到了拉皮条。。。
当然也是因为这里聚集了各方面的逻辑，所以要注意中介者本身的复杂度，中介者本身也需要良好的设计和模式来提高代码的可读性和可维护性。

# 观察者模式 Observer

### 特点：为减少对象间的互相引用而引入的一个中介对象，用来来封装一系列对象的互相操作。

### 用处：当多个对象间需要互相引用且互相频繁操作时可以考虑中介者模式，如MVC里的Controller。

### 注意：中介者本身的复杂度。

下面用TypeScript简单实现一下中介模式：