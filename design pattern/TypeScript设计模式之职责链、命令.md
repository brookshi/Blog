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

response // 根据前面处理的request返回数据

<res2>response</res2> // SecondHttpHandler对返回数据的第一次处理

<res1><res2>response</res2></res1> // FirstHttpHandler对返回数据的第二次处理

finish
```
处理的顺序就是 12*21,中间*是真正取数据的，这就是管道处理最基本的代码，用到的就是职责链模式。

当然职责链的形成有很多方式，这里采用的是装饰手段，保存下一个的引用的方式来形成一个链表，还可以采用队列或栈方式保存所有handler，按顺序执行。

# 命令模式 Command

### 特点：把请求封装成命令对象，命令对象里包含有接收者，这样client只需要发送命令，接收者就可以做出相关响应或相反的响应。

### 用处：当需要发送者和接收者解耦时可以考虑命令模式，常用于事件响应，请求排除，undo/redo等。

### 注意：。

下面用TypeScript简单实现一个支持undo/redo的命令模式：
遥控器算是典型的命令模式，按个按钮命令电视做相关响应，假设遥控器有三种功能，开、关和换台。

建个Command、undo/redo以及控制接口：

```ts
interface Executable{
    execute(param: any);
}

interface UndoRedoable{
    undo(currParam: any, lastParam: any);
    redo(param: any);
}

interface Controllable{
    channelNum: number;

    open();
    close();
    switchTo(channelNum: number); //换台
}
```
undo/redo可以由个专门的管理器来管理，建个undo/redo管理器：
管理器要做的事有
1. 按顺序记住所有command
2. undo/redo操作，并记住undo/redo到了哪一步
3. 当undo/redo到了某一步时，再次有新的command，则在移除这步之后的command后再加新的command

```ts
class MemoItem{
    command: Command;
    param: any;
}

class UndoRedoManager{

    static readonly instance: UndoRedoManager = new UndoRedoManager();

    private memoList: Array<MemoItem> = []; // 记住所有command
    private currPos: number = 0;  // 当前undo/redo到了哪一个
    private defaultMemoItem: MemoItem = { command: undefined, param: {channelNum: 0} }; // undo到第一步时前面没有command了，返回一个默认command

    private get currIndex(): number{  // currPos是从后往前的顺序， currIndex是正向顺序
        return this.memoList.length - this.currPos - 1;
    }

    push(command: Command, param: any){  // command执行时应该push进来
        if(this.currPos != 0){  // 不是0的话表示已经undo过，往上叠加push前先删除后面的
            this.memoList.splice(this.currIndex + 1);
            this.currPos = 0; // 重置currPos
        }
        this.memoList.push({command, param});
    }

    redo(){
        if(this.currPos == 0){ // 表示没undo过，当然redo也没必要了
            return;
        }
        let memoItem = this.memoList[this.currIndex + 1]; // 取出上次undo过的下一个command并执行
        this.currPos--;
        memoItem.command.redo(memoItem.param);
    }

    undo() {
        if(this.currIndex < 1){ 
            return;
        }

        let memoItem = this.memoList[this.currIndex];
        let lastMemoItem = this.findLastWithSameType(memoItem); // 找出上个同类型的command，得到参数，以这个参数来做undo操作，回到之前的状态
        this.currPos++;
        memoItem.command.undo(memoItem.param, lastMemoItem.param);
    }

    private findLastWithSameType(memoItem: MemoItem): MemoItem{
        for(let i = this.currIndex - 1; i >= 0; i--){
            if(memoItem.constructor.name === this.memoList[i].constructor.name){
                return this.memoList[i];
            }
        }

        return this.defaultMemoItem;
    } 
}
```
抽象个Command, Command需要做到执行命令、撤消上次所做的操作及重做, 这里就可以用上面的UndoRedoManager:

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

接下来分别实现具体的 开、关、换台命令：

```ts
class OpenCommand extends Command{

    execute(param: any){
        super.execute(param);
        this.tv.open();
    }

    undo(currParam: any, lastParam: any){
        this.tv.close();
    }
}

class CloseCommand extends Command{

    execute(param: any){
        super.execute(param);
        this.tv.close();
    }

    undo(currParam: any, lastParam: any){
        this.tv.open();
    }
}

class SwitchCommand extends Command{

    execute(param: any){
        super.execute(param);
        this.tv.switchTo(param.channelNum);
    }

    undo(currParam: any, lastParam: any){
        this.tv.switchTo(lastParam.channelNum);
    }
}
```
最后来实现 电视和遥控器，遥控器通常只有一个开关按钮，要么开要么关，另外遥控器可以撤消到上次选的频道，也可以取消撤消，重新回到当前的：
电视只需要做具体的事就可以了，遥控器也不需要知道命令是谁在执行，只管发命令就好，这就是命令模式的好处。

```ts
class TV implements Controllable{

    open(){
        console.log('open');
    }

    close(){
        console.log('close');
    }

    switchTo(channelNum: number){
        console.log(`switch to channel: ${channelNum}`);
    }
}

class Controller {

    isOn: boolean = false;

    constructor(private openCmd: Command, private closeCmd: Command, private switchCmd: Command){
    }

    onOff(){
        if(this.isOn){
            this.isOn = false;
            this.closeCmd.execute(null);
        } else {
            this.isOn = true;
            this.openCmd.execute(null);
        }
    }

    switchTo(channelNum: number){
        this.switchCmd.execute({channelNum: channelNum});
    }

    undo(){
        UndoRedoManager.instance.undo(); //只需要调用UndoRedoManager做undo/redo就可以了，不需要管具体的细节
    }

    redo(){
        UndoRedoManager.instance.redo();
    }
}
```
来看看成果：
先定个执行顺序，
打开电视 -> 3频道 -> 4频道 -> 7频道 -> 撤消 -> 撤消 -> 重做 -> 11频道 -> 12频道 -> 撤消 -> 撤消 -> 关电视

预期结果：
open -> 3 -> 4 -> 7 -> 4 -> 3 -> 4 -> 11 -> 12 -> 11 -> 4 -> close
从11回到4是因为在push 11频道时的command是4，也就是7已经被删掉了。

看看具体执行结果：

```ts
let tv = new TV();
let controller = new Controller(new OpenCommand(tv), new CloseCommand(tv), new SwitchCommand(tv));

controller.onOff();
controller.switchTo(3);
controller.switchTo(4);
controller.switchTo(7);
controller.undo();
controller.undo();
controller.redo();
controller.switchTo(11);
controller.switchTo(12);
controller.undo();
controller.undo();
controller.onOff();
```
执行结果：
open
switch to channel: 3
switch to channel: 4
switch to channel: 7
switch to channel: 4
switch to channel: 3
switch to channel: 4
switch to channel: 11
switch to channel: 12
switch to channel: 11
switch to channel: 4
close

完全一样，没问题。

本来想写简单一点，不知不觉就写多了，undo/redo还是偏复杂了一些，而且这还只是最基本的架子，很多东西不严谨，有兴趣的朋友可以自己研究下。

命令模式的优点已经清楚了，缺点也比较明显，一个操作就是一个命令，项目大的话命令会非常多，也是个麻烦的点。