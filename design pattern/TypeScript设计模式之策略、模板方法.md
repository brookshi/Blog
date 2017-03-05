看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 策略模式 Strategy

### 特点：用组合的方式调用一些算法或逻辑，并且可以根据状态不同而选用不同的算法或逻辑。

### 用处：对象需要运行时切换算法或逻辑可以考虑使用策略模式。

### 注意：策略的生成方式。

下面用TypeScript简单实现一个策略模式：
说起策略就想到策略类游戏，年龄大点的可能都玩过War3，人族对兽族时如果侦察到对方不着急升本，用常规万金油打法，那人族就可以出狗男女来一波流。
如果侦察到兽族跳科技并摆下两个兽栏，那对方可能是暴飞龙，人族就要家里补个塔防偷农民，然后出点火枪或二本龙鹰。

```ts
class Orc{
    private _shenKeJi = false;

    get shenKeJi(): boolean { // 这里简单用升科技来判断是用常规还是飞龙
        return this._shenKeJi;
    }

    set shenKeJi(value: boolean){
        this._shenKeJi = value;
    }
}

abstract class Stragety{
    abstract execute();
}

class RushStragety extends Stragety{
    execute(){
        console.log('升科技');
        console.log('出狗男女');
        console.log('一波流');
    }
}

class DefendStragety extends Stragety{
    execute(){
        console.log('补塔防飞龙');
        console.log('出火枪');
        console.log('升科技');
        console.log('出龙鹰');
    }
}

class Human{
    stragety: Stragety;

    checkOrc(orc: Orc){
        if(orc.shenKeJi){ //根据兽族情况来决定策略
            console.log('侦察到兽族是跳科技打法');
            this.stragety = new DefendStragety();
        } else {
            console.log('侦察到兽族是常规打法');
            this.stragety = new RushStragety();
        }
    }

    deal(){
        this.stragety && this.stragety.execute();
    }
}

let orc = new Orc();
let human = new Human();

orc.shenKeJi = false;
human.checkOrc(orc);
human.deal();

orc.shenKeJi = true;
human.checkOrc(orc);
human.deal();

//输出
侦察到兽族是常规打法
升科技
出狗男女
一波流

侦察到兽族是跳科技打法
补塔防飞龙
出火枪
升科技
出龙鹰
```
这样人族就可以根据兽族的状态改变来做出不同的应对策略，其实现在游戏的AI基本都是通过决策树来实现的，也算是策略模式，只是更复杂，通过各种不同的条件最终得到一个决策来做出反应。

另外，有人可能已经发现了，上面生成策略的地方是可以拿出来，用之前讲的工厂模式来做，因为实际应用时策略通常比较多，甚至可能同时需要多种相关策略，用工厂模式来生产策略就可以很好的隐藏细节，解除依赖。


# 模板方法模式 Template Method

### 特点：通过多态来实现在运行时使用不同的算法或逻辑，通常有一个整体架子，通过抽象方法或虚方法来把细节代码延迟到子类实现。

### 用处：当多个类似功能的类有很多相同结构或代码时，可以抽象出整体架子时可以考虑模板方法。

### 注意：与策略模式的异同：同样是细节部分交出去，不同在于策略是对象行为，采用的是组合的方式，而模板方法是类行为，采用的是继承。

下面用TypeScript简单实现一个模板方法模式：
比方说发送http请求的代码，需要向两台不同的server(A和B)发送请求，两台server除了url不同，回来的数据格式也不一样，但由于都是http请求，主体架子是一样的，所以可以用模板方法来实现下。

```ts
class ClassA{} // Server A 返回的数据结构
class ClassB{} // Server B 返回的数据结构

abstract class RequesterBase<T>{

    constructor(private url: string){

    }

    reqeustData(): T{
        this.sendReqeust();
        return this.handleResponse();
    }

    protected sendReqeust(){
        console.log(`send request, url: ${this.url}`);
    }

    protected abstract handleResponse(): T; // 不同的server返回的数据交由子类去实现
}

class RequesterForServerA extends RequesterBase<ClassA>{
    protected handleResponse(): ClassA{
        console.log('handle response for Server A');
        return null;
    }
}

class RequesterForServerB extends RequesterBase<ClassB>{
    protected handleResponse(): ClassB{
        console.log('handle response for Server B');
        return null;
    }
}

let requesterA: RequesterBase<ClassA> = new RequesterForServerA('server A');
let requesterB: RequesterBase<ClassB> = new RequesterForServerB('server B');

requesterA.reqeustData();
requesterB.reqeustData();

//输出
send request, url: server A
handle response for Server A

send request, url: server B
handle response for Server B
```
这里可以看到主体功能由基类RequesterBase实现，两个子类则实现解析数据这些细节，这样就达到了消除重复代码的目的。
如果还有个ServerC的request发送部分也不一样，也没关系，TypeScript天生虚函数，在子类直接实现reqeustData即可，多态的作用下，运行时还是会调用到子类上。