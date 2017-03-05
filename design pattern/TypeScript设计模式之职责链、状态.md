看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 职责链模式 Chain of Responsibility

### 特点：可以让一个请求被不同的对象处理多次，请求像经过管道一样， 一路上都可以被拦下处理。

### 用处：当请求需要被链式处理时，可以考虑职责链模式，比如事件的冒泡，WebApi的管道Handler等。

### 注意：链的实现。

WebApi的handler可能大家有用过，对发出去的请求和请求回来的数据都可以用自定义handler在发出前或最终回来前进行处理，非常方便，下面用TypeScript来简单实现一个HttpHandler:

先建立一个抽象Handler类，包含一个发送请求的sendReqeust以及用来链式处理的innerHandler：

```ts
abstract class HttpHandler{

    constructor(protected innerHandler: HttpHandler){}

    async sendRequest(req: string): Promise<string>{
        if(this.innerHandler){
            return await this.innerHandler.sendRequest(req);
        } else {
            let res = `response`;
            console.log(res);
            return res;
        }   
    }
}
```
实现第一个Handler类：

```ts
class FirstHttpHandler extends HttpHandler{
    
    async sendRequest(req: string): Promise<string>{

        req = `<req1>${req}</req1>`; // 把请求包一下
        console.log(req);

        let res = await super.sendRequest(req);

        res = `<res1>${res}</res1>`; // 把结果包一下
        console.log(res);

        return res;
    }
}
```
再实现第二个Handler类：

```ts
class SecondHttpHandler extends HttpHandler{
    
    async sendRequest(req: string): Promise<string>{

        req = `<req2>${req}</req2>`; // 把请求包一下
        console.log(req);

        let res = await super.sendRequest(req);

        res = `<res2>${res}</res2>`; // 把结果包一下
        console.log(res);

        return res;
    }
}
```
把两个HttpHandler连起来

```ts
let httpHandler = new FirstHttpHandler(new SecondHttpHandler(undefined));
console.log('start')
httpHandler.sendRequest('request').then(res=>console.log('finish'));
```
输出：

```ts
start

<req1>request</req1> // 发请求前先在FirstHttpHandler里处理request

<req2><req1>request</req1></req2> // 在SecondHttpHandler里再次处理request

response // 返回数据

<res2>response</res2> // SecondHttpHandler对返回数据的第一次处理

<res1><res2>response</res2></res1> // FirstHttpHandler对返回数据的第二次处理

finish
```
处理的顺序就是 12*21,中间*是真正取数据的，这就是管道处理最基本的代码，用到的就是职责链模式。

当然职责链的形成有很多方式，这里采用的是装饰手段，保存下一个的引用的方式来形成一个链表，还可以采用队列或栈方式保存所有handler，按顺序执行。

# 状态模式 State

### 特点：通过状态来改变对象的行为。

### 用处：当对象的行为取决于它的状态或者有很多if else之类的是由状态控制的时候可以考虑状态模式，如常见的状态机。

### 注意：状态是由谁来转换。

下面用TypeScript简单实现一下状态模式：
大家都玩过游戏，控制游戏的主角时鼠标左键可以是移动，遇到怪时点击左键是攻击，遇到NPC时是对话。
下面就以这点简单实现个状态模式：

角色和状态的接口，状态只需要处理当前状态需要做的事：
```ts
interface Role{
    name: string;

    click();
    changeState(state: State);
}

interface State{
    handle(role: Role);
}
```
角色的具体实现：
```ts
class Player implements Role{
    private state: State;
    constructor(public name: string){
    }

    click(){
        if(this.state){
            this.state.handle(this);
        }
    }

    changeState(state: State){
        this.state = state;
        console.log(`change to ${this.state.constructor.name}`);
    }
}
```
状态的具体实现，分为移动状态，攻击状态，对话状态：
```ts
class MoveState implements State{
    static readonly instance = new MoveState();

    handle(role: Role){
        console.log(`${role.name} is moving`);
    }
}

class AttackState implements State{
    static readonly instance = new AttackState();

    handle(role: Role){
        console.log(`${role.name} is attacking`);
    }
}

class TalkState implements State{
    static readonly instance = new TalkState();

    handle(role: Role){
        console.log(`${role.name} is talking`);
    }
}
```
使用：
```ts
let player = new Player('brook');

player.changeState(MoveState.instance);
player.click();

player.changeState(AttackState.instance);
player.click();

player.changeState(TalkState.instance);
player.click();

//输出：
change to MoveState
brook is moving

change to AttackState
brook is attacking

change to TalkState
brook is talking
```
这样随着状态的变化，点击左键做不同的事。
对于由谁来驱动状态变化可以根据实际情况来考虑，简单的话直接放角色里面就行，由角色自己决定自己的状态，复杂的话可以考虑用表来驱动状态机，通过表过实现状态的跳转。