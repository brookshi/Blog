
# 解释器模式 Interpreter

### 特点：使用给定语法来解释一段内容。

### 用处：管理类系统经常会定义一些搜索语句格式来使用户方便搜索库里的内容，这时就可以考虑用解释器来翻译执行这些语句。

### 注意：适合相对简单的语法。

解释器模式通过把一段表达式拆开成很多个，分为不同的解析类，一个一个的去解析并执行，这过程中经常会用Context来保存解析过程的信息。
这种解释器的优点在于各种表达式的解析相对独立，要加入新的规则也不会影响现有的解析。缺点也很明显，一个表达式一个类，复杂语法或复合语法的话表达式数量就非常多，并且表达式之间也很难真正独立。

下面用TypeScript写一个简单正则表达式的解释器：
要解释的表达式有：{}, [], \d, ^, $这几种。

先建立一个Expression接口，所有解释器都实现这个接口：
```ts
interface Expression{
    interpret(context: Context);
}
```
可以看到接口里用到了一个Context，这个用来保存解析时的一些数据和进度，包含：
pattern: 整个表达式
currentPatternIndex: 当前正在验证的表达式的位置
lastExpression: 上一个表达式，用于{}解析
text: 需要验证的文本
currentTextIndex: 当前验证到text里的哪个字符的位置
isMatch: 是否匹配成功

```ts
class Context{
    constructor(public pattern: string, public text: string){

    }

    currentTextIndex: number = 0;
    get currentText(): string{
        return this.text[this.currentTextIndex];
    }

    currentPatternIndex: number = 0;
    lastExpression: string;
    get currentPatternExpression(): string{
        return this.pattern[this.currentPatternIndex];
    }

    isMatch: boolean;
}
```
现在分别给那些符号写解析类：
```ts
// 开始符号：^
class BeginExpression implements Expression{
    interpret(context: Context){
        context.isMatch = context.currentPatternIndex === 0;
        context.currentPatternIndex++;
    }
}

// 结束符号：$
class EndExpression implements Expression{
    interpret(context: Context){
        context.isMatch = context.currentTextIndex === context.text.length;
        context.currentPatternIndex++;
    }
}

// 反斜杠：\d，只支持\d
class BslashExpression implements Expression{
    interpret(context: Context){
        if(context.pattern[context.currentPatternIndex + 1] !== 'd'){
            throw new Error('only support \\d');
        }

        let target = context.currentText;
        context.lastExpression = '\\d';
        context.isMatch = Number.isInteger(Number.parseInt(target));
        context.currentPatternIndex+=2;
        context.currentTextIndex++;
    }
}

// 中括号：[]
class BracketExpression implements Expression{
    interpret(context: Context){
        let prevIndex = context.currentPatternIndex;
        while(context.pattern[++context.currentPatternIndex] !== ']'){
            if(context.currentPatternIndex+1 === context.pattern.length){
                throw new Error(`miss symbol ]`);
            }
        }
        let expression = context.pattern.substr(prevIndex+1, context.currentPatternIndex - prevIndex - 1);
        let target = context.currentText;

        context.lastExpression = `[${expression}]`;
        context.isMatch = [...expression].indexOf(target) > -1;
        context.currentPatternIndex++;
        context.currentTextIndex++;
    }
}

// 大括号：{}
class BraceExpression implements Expression{
    interpret(context: Context){
        let endIndex = context.currentPatternIndex;
        while(context.pattern[++endIndex] !== '}'){
            if(i+1 === context.pattern.length){
                throw new Error(`miss symbol }`);
            }
        }
        let expression = context.pattern.substr(context.currentPatternIndex + 1, endIndex - context.currentPatternIndex - 1);
        let num = Number.parseInt(expression);
        if(!Number.isInteger(num)){
            throw new Error('{} only support number');
        }
        let newExpression = '';
        for(let i=1;i<num;i++){
            newExpression += context.lastExpression;
        }
        context.pattern = context.pattern.substr(0, context.currentPatternIndex) + 
                                     newExpression + 
                                     context.pattern.substr(endIndex+1);
    }
}

// 普通字符
class StringExpression implements Expression{
    interpret(context: Context){
        context.lastExpression = context.currentPatternExpression;
        context.isMatch = context.currentPatternExpression === context.currentText;
        context.currentPatternIndex++;
        context.currentTextIndex++;
    }
}
```
有了这些解释器，现在解析表达式就很轻松了：
```ts
class Regex{
    mapping: {[key:string]: Expression} = {
                                         '^': new BeginExpression(),
                                         '$': new EndExpression(),
                                         '{': new BraceExpression(),
                                         '[': new BracketExpression(),
                                         '\\':new BslashExpression(),
                                        }; // 这是一个表达式-解释器的映射表
    stringExp: Expression = new StringExpression();        

    constructor(private pattern: string){

    }

    IsMatch(text: string): boolean{
        let context = new Context(this.pattern, text);
        
        for(context.currentPatternIndex=0;context.currentPatternIndex<context.pattern.length;){
            let symbol = this.mapping[context.currentPatternExpression];
            symbol ? symbol.interpret(context) : this.stringExp.interpret(context); //通过找到对应的解释器来解释匹配文本
            if(!context.isMatch){
                break;
            }
        }
        return context.isMatch;
    }
}
```
写个手机号码验证的正则表达式测试一下：
```ts
let pattern = '/^1[34578]\d{9}$/';
let regex = new Regex(pattern);

let text = '13712345678';
console.log(`match ${text}: ${regex.IsMatch(text)}`); // 正常手机号：成功

text = '1371234567p';
console.log(`match ${text}: ${regex.IsMatch(text)}`); // 手机号里有字母：失败

text = '137123456789';
console.log(`match ${text}: ${regex.IsMatch(text)}`); // 多了一位：失败

text = '1371234567';
console.log(`match ${text}: ${regex.IsMatch(text)}`); // 少了一位：失败
```
结果符合预期，可以看到用解释器把表达分开解释的好处很明显，各个解释器互不干扰，主体部分调用这些解释器分别进行解释就可以了，非常方便。
当然这也只是处理简单的语法，如果语法很复杂就需要考虑引入分析引擎或编译器了。