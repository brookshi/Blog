看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 策略模式

### 特点：用组合的方式调用一些算法或逻辑，并且可以根据状态不同而选用不同的算法或逻辑。

### 用处：对象需要运行时切换算法或逻辑可以考虑使用策略模式。

### 注意：。

下面用TypeScript简单实现一个策略模式：
说起策略就想到策略类游戏，年龄大点的可能都玩过War3，人族对兽族时如果侦察到对方不着急升本，用常规万金油打法，那人族就可以出狗男女来一波流。
如果侦察到兽族跳科技并摆下两个兽栏，那对方可能是暴飞龙，人族就要家里补个塔防偷农民，然后出点火枪或二本龙鹰。

```ts
class Orc{
    shenKeJi: boolean; // 这里简单用升科技来判断是用常规还是飞龙
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
            this.stragety = new DefendStragety();
        } else {
            this.stragety = new RushStragety();
        }
    }

    deal(){
        this.stragety && this.stragety.execute();
    }
}

let orc = new Orc();
let human = new Human();

human.checkOrc(orc);
human.deal();

orc.shenKeJi = true;
console.log('兽族升科技了');

human.checkOrc(orc);
human.deal();

//输出
升科技
出狗男女
一波流

兽族升科技了

补塔防飞龙
出火枪
升科技
出龙鹰
```
这样人族就可以根据兽族的状态改变来做出不同的策略，其实现在游戏的AI基本都是通过决策树来实现的，也算是策略模式，只是更复杂，通过各种不同的条件最终到达一个决策来做出反应。

其实上面生成策略的地方是可以用上篇讲的工厂模式来做，因为实际应用时策略通常比较多，甚至可能同时需要多种相关策略，用工厂模式来生产策略就可以很好的隐藏细节，解除依赖。