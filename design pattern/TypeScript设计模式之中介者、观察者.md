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

### 特点：定义了对象间的一对多关系，当对象状态改变时，其他订阅了这个对象的对象就会收到通知。

### 用处：当一个对象状态的改变时需要其他对象也做出响应时可以考虑观察者模式，如网络聊天里的群。

### 注意：与中介者的区别。

下面用TypeScript简单实现一下观察者模式：
就以上面说的群聊天为例，群里的每个人都是注册到群里的对象，任何一个人发了信息其他人都能收到。

先定义群和群用户的接口：
群需要知道有哪些用户注册进来了，并且在有人发消息时去通知所有注册的人。
用户则需要发送消息和接收消息。

```ts
interface Observer{
    name: string;

    sendMsg(msg: string);
    receiveMsg(sender: Observer, msg: string);
}

interface Subject{
    register(observer: Observer);
    unregister(observer: Observer);
    sendMsg(sender: Observer, msg: string);
}
```
实现用户和群，用户在发消息时需要往群里发，群收到消息后通知所有注册的人
```ts
class User implements Observer{
    constructor(public name: string, private subject: Subject){
        this.subject.register(this);
    }

    sendMsg(msg: string){
        console.log(`${this.name} 发送 ${msg}`);
        this.subject.sendMsg(this, msg);
    }

    receiveMsg(sender: Observer, msg: string){
        console.log(`${this.name} 收到来自${sender.name}的消息： ${msg} `);
    }
}

class Group implements Subject{
    private userList: Array<Observer> = [];

    register(observer: Observer){
        this.userList.push(observer);
    }

    unregister(observer: Observer){
        var index = this.userList.indexOf(observer);
        if (index > -1) {
            this.userList.splice(index, 1);
        }
    }

    sendMsg(sender: Observer, msg: string){
        console.log(`群收到${sender.name}发信息：${msg}，通知所有人`);
        this.notify(sender, msg);
    }

    private notify(sender: Observer, msg: string){
        this.userList.forEach(user=>user.receiveMsg(sender, msg));
    }
}
```
写段代码测试一下：
```ts
let group = new Group();
let user1 = new User1('jim', group);
let user2 = new User1('brook', group);
let user3 = new User1('lucy', group);

user1.sendMsg('hello');
user3.sendMsg('well done!');
//结果：
jim 发送 hello
群收到jim发信息：hello，通知所有人
jim 收到来自jim的消息： hello 
brook 收到来自jim的消息： hello 
lucy 收到来自jim的消息： hello 

lucy 发送 well done!
群收到lucy发信息：well done!，通知所有人
jim 收到来自lucy的消息： well done! 
brook 收到来自lucy的消息： well done! 
lucy 收到来自lucy的消息： well done! 
```
只有要人发消息，所有注册的人都会收到，跟广播一样。
其实观察者模式可以做得更通用，类似一个消息中心，所有注册的对象按照一定协议实现匹配事件的方法来获取通知，消息中心不需要知道是什么类型的对象注册了，只要实现这个方法，那相关事件有通知时这个方法就会被调到，这样基本没有耦合度，有兴趣的朋友可以参考我之前写的一个win10开源库：[LLQNotify](https://github.com/brookshi/LLQNotifier)，就是用这种方式实现的。

另外，与中介者模式的区别在于：虽然都是注册回复，但观察者是分发性的，注册的人都能收到，而且中介者则是单一的，使用者发个请求，中介者回一个，使用者不需要知道到底是谁回的，中介隐藏了对象之间的交互。