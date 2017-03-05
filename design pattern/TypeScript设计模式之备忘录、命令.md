看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 备忘录模式 Memento

### 特点：通过保存对象之前的状态来使对象可以恢复到之前的样子。

### 用处：当对象需要保存/加载某一时刻的状态时可以考虑备忘录模式，如游戏的save/load。

### 注意：状态过大产生的开销。

备忘录应该经常可以看到，游戏的save/load，photoshop的历史记录，windows的还原点都是这个模式的应用。
使用时也要注意保存的状态过大时产生的开销，保存在硬盘上的还好，如果是运行时保存在内存上的，比如一些复杂对象的undo/redo操作，保存每一个状态都是很大的内存开销，这时就需要做些限制，比方设置一个历史记录栈的最大值来限定内存的使用。

备忘录的例子和下面的命令模式一起写，实现一个支持undo/redo的操作。

# 命令模式 Command

### 特点：把请求封装成命令对象，命令对象里包含有接收者，这样client只需要发送命令，接收者就可以做出相关响应或相反的响应。

### 用处：当需要发送者和接收者解耦时可以考虑命令模式，常用于事件响应，请求排除，undo/redo等。

### 注意：命令数量爆炸，需要集中维护。

下面用TypeScript简单实现一个命令模式和备忘录模式的undo/redo：
遥控器算是典型的命令模式，按个按钮就可以命令电视做相关响应，假设遥控器有三种功能，开、关和换台。

建个Command、undo/redo、备忘录以及控制接口：

```ts
interface Executable{
    execute(param: any);
}

interface UndoRedoable{
    undo(currParam: any, lastParam: any);
    redo(param: any);
}

class MemoItem{
    command: Command;
    param: any;
}
interface Memento{
    currPos: number;  

    set(item: MemoItem);
    get(): MemoItem; 
    getNext(): MemoItem; //找到下一个做redo
    findLastWithSameType(memoItem: MemoItem): MemoItem; // 找出上个同类型的command，得到参数，以这个参数来做undo操作，回到之前的状态
}

interface Controllable{
    channelNum: number;

    open();
    close();
    switchTo(channelNum: number); //换台
}
```
实现备忘录

```ts
class History implements Memento{
    private memoList: Array<MemoItem> = []; // 记住所有command
    static defaultMemoItem: MemoItem = { command: undefined, param: {channelNum: 0} }; // undo到第一步时前面没有command了，返回一个默认command

    currPos: number = 0;  // 当前undo/redo到了哪一个

    get currIndex(): number{  // currPos是从后往前的顺序， currIndex是正向顺序
        return this.memoList.length - this.currPos - 1;
    }

    set(item: MemoItem){  
        if(this.currPos != 0){  // 不是0的话表示已经undo过，往上叠加push前先删除后面的
            this.memoList.splice(this.currIndex + 1);
            this.currPos = 0; // 重置currPos
        }
        this.memoList.push(item);
    }

    get(): MemoItem{
        if(this.currIndex < this.memoList.length){
            return this.memoList[this.currIndex];
        }
        return History.defaultMemoItem;
    }

    getNext(): MemoItem{
        if(this.currIndex + 1 < this.memoList.length){ 
            return this.memoList[this.currIndex+1];
        }
        return History.defaultMemoItem;
    }

    findLastWithSameType(memoItem: MemoItem): MemoItem{// 找出上个同类型的command，得到参数，以这个参数来做undo操作，回到之前的状态
        for(let i = this.currIndex - 1; i >= 0; i--){
            if(memoItem.constructor.name === this.memoList[i].constructor.name){
                return this.memoList[i];
            }
        }

        return History.defaultMemoItem;
    } 
}
```
undo/redo可以由个专门的管理器来管理，建个undo/redo管理器：
管理器要做的事有
1. 使用备忘录按顺序记住所有command
2. undo/redo操作，并记住undo/redo到了哪一步
3. 当undo/redo到了某一步时，再次有新的command，则在移除这步之后的command后再加新的command

```ts
class UndoRedoManager{

    static readonly instance: UndoRedoManager = new UndoRedoManager();

    private history: Memento = new History();

    push(command: Command, param: any){  // command执行时应该push进来
        this.history.set({command, param});
    }

    redo(){
        if(this.history.currPos == 0){ // 表示没undo过，当然redo也没必要了
            return;
        }
        let memoItem = this.history.getNext(); // 取出上次undo过的下一个command并执行
        this.history.currPos--;
        memoItem.command.redo(memoItem.param);
    }

    undo() {
        let memoItem = this.history.get();
        if(memoItem === History.defaultMemoItem){
            return;
        }
        let lastMemoItem = this.history.findLastWithSameType(memoItem); 
        this.history.currPos++;
        memoItem.command.undo(memoItem.param, lastMemoItem.param);
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

本来想写简单一点，不知不觉就写多了，undo/redo还是偏复杂了一些，而且这还只是最基本的架子，很多东西不严谨，有兴趣的朋友可以自己研究下，建议只针对用户常用的部分做undo/redo，保持系统的简单。

命令模式的优点已经清楚了，缺点也比较明显，一个操作就是一个命令，项目大的话命令会非常多，也是个麻烦的点。