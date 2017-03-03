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
            let res = `<res>${req}</res>`;
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

<res><req2><req1>request</req1></req2></res> // 根据前面处理的request返回数据

<res2><res><req2><req1>request</req1></req2></res></res2> // SecondHttpHandler对返回数据的第一次处理

<res1><res2><res><req2><req1>request</req1></req2></res></res2></res1> // FirstHttpHandler对返回数据的第二次处理

finish
```
处理的顺序就是 12*21,中间*是真正取数据的，这就是管道处理最基本的代码，用到的就是职责链模式。

当然职责链的形成有很多方式，这里采用的是保存下一个的引用的方式来形成一个链表，还可以采用队列或栈方式保存所有handler，按顺序执行。

# 命令模式 Command

### 特点：把请求封装成命令对象，命令对象里包含有接收者，这样client只需要发送命令，接收者就可以做出相关响应或相反的响应。

### 用处：当需要发送者和接收者解耦时可以考虑命令模式，常用于事件响应，请求排除，undo/redo等。

### 注意：。

下面用TypeScript简单实现一个支持undo/redo的命令模式：
遥控器算是典型的命令模式，按个按钮命令电话做相关响应，假设遥控器有三种功能，开、关和换台。

建个Command、undo/redo以及控制接口：

```ts
interface Executable{
    execute();
}

interface UndoRedoable{
    undo(param: {});
    redo(param: {});
}

interface Controllable{

    channelNum: number;

    open();
    close();
    switch(channelNum: number);
}
```
建个undo/redo管理器：

```ts
class UndoRedoManager{

    private static memoList: Array<MemoItem> = [];

    static push(command: Command, param: {}){
        this.memoList.push({command, param});
    }

    static redo(){
        let memoItem = this.memoList[this.memoList.length - 1];
        memoItem.command.execute(memoItem.param);
    }

    static undo() {
        let memoItem = this.memoList.pop();
        memoItem.command.undo(memoItem.param);
    }
}
```

抽象个Command:

```ts
abstract class Command implements Executable, UndoRedoable{

    constructor(protected controller: Controllable) { }

    execute(param: {}){
        UndoRedoManager.push(this, param);
    }

    redo(){
        UndoRedoManager.redo();
    }

    undo(){
       UndoRedoManager.undo(); 
    }
}
```

接下来分别实现 开、关、换台命令：

```ts
class OpenCommand extends Command{

    execute(param: {}){
        super.execute(param);
        this.controller.open();
    }

    undo(param: {}){
        this.controller.close();
    }
}

class CloseCommand extends Command{

    execute(param: {}){
        super.execute(param);
        this.controller.close();
    }

    undo(param: {}){
        this.controller.open();
    }
}

class SwitchCommand extends Command{

    execute(param: {}){
        super.execute({lastChannelNum:}});
        this.controller.switch(param.channelNum);
    }

    undo(param: {}){
        this.controller.switch(param.channelNum);
    }
}
```
再来实现 电视：

```ts
class TV implements Controllable{
    channelNum: number = 0;

    open(){
        console.log('open tv');
    }

    close(){
        console.log('close tv');
    }

    switch(channelNum: number){
        this.channelNum = channelNum;
        console.log(`switch to channel ${this.channelNum}`);
    }
}
```