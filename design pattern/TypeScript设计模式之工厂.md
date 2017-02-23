看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 简单工厂模式

### 特点：把同类型产品对象的创建集中到一起，通过工厂来创建，添加新产品时只需加到工厂里即可，也就是把变化封装起来，同时还可以隐藏产品细节。

### 用处：要new多个同一类型对象时可以考虑使用简单工厂。

### 注意：对象需要继承自同一个接口。

下面用TypeScript写一个序列化工具Serializer来看看简单工厂模式：

```ts
enum SerializeType{
    Json,
    Xml,
}

interface Serializer{
    serialize<T>(obj: T): string;
}

class JsonSerializer implements Serializer{
    serialize<T>(obj: T): string{
        //json 序列化
        return '{}';
    }
}

class XmlSerializer implements Serializer{
    serialize<T>(obj: T): string{
        //xml序列化
        return '<xml/>';
    }
}

class SerializerFactory{

    static createSerializer(type: SerializeType): Serializer{
        switch(type){
            case SerializeType.Json:
                return new JsonSerializer();
            case SerializeType.Xml:
                return new XmlSerializer();
            default:
                throw Error('not support this type yet');
        }
    }
}
```
上面代码`SerializerFactory`工厂就是根据类型来创建不同的工厂，使用的时候只需要引入这个工厂和接口即可。
比如上篇的RequestBuilder需要加入方法`setJsonBody`和`setXmlBody`，可以用这个工厂来做：

```ts
setXmlBody<T>(obj: T){
    this._body = SerializerFactory.createSerializer(SerializeType.Xml).serialize(obj);
    return this;
}

setJsonBody<T>(obj: T){
    this._body = SerializerFactory.createSerializer(SerializeType.Json).serialize(obj);
    return this;
}
```
这样就把变化封装到了工厂中，如果以后要支持`protobuf`，只需要加个实现`Serializer`接口的`ProtobufSerializer`类就可以了。

# 工厂方法模式

### 特点：把工厂抽象出来，让子工厂来决定怎么生产产品, 每个产品都由自己的工厂生产。

### 用处：当产品对象需要进行不同的加工时可以考虑工厂方法。

### 注意：这不是所谓的简单工厂的升级版，两者有不同的应用场景。

下面用TypeScript写一个枪工厂来看看工厂方法模式：

```ts
interface Shootable{
    shoot();
}

abstract class Gun implements Shootable{ // 抽象产品 - 枪
    abstract shoot();
}

class AK47 extends Gun{ //具体产品 - AK47
    shoot(){
        console.log('ak47 shoot.');
    }
}

class M4A1 extends Gun{ //具体产品 - M4A1
    shoot(){
        console.log('m4a1 shoot.');
    }
}

abstract class GunFactory{ //抽象枪工厂
    abstract create(): Gun;
}

class AK47Factory extends GunFactory{ //Ak47工厂
    create(): Gun{
        let gun = new AK47();  // 生产Ak47
        console.log('produce ak47 gun.');
        this.clean(gun);       // 清理工作
        this.applyTungOil(gun);// Ak47是木头枪托，涂上桐油
        return gun;
    }

    private clean(gun: Gun){
        //清洗
        console.log('clean gun.');
    }

    private applyTungOil(gun: Gun){
        //涂上桐油
        console.log('apply tung oil.');
    }
}

class M4A1Factory extends GunFactory{ //M4A1工厂
    create(): Gun{
        let gun = new M4A1();   // 生产M4A1
        console.log('produce m4a1 gun.');
        this.clean(gun);        // 清理工作
        this.sprayPaint(gun);   // M4是全金属，喷上漆
        return gun;
    }

    private clean(gun: Gun){
        //清洗
        console.log('clean gun.');
    }

    private sprayPaint(gun: Gun){
        //喷漆
        console.log('spray paint.');
    }
}

let ak47 = new AK47Factory().create();
ak47.shoot();

let m4a1 = new M4A1Factory().create();
m4a1.shoot();

//output
produce ak47 gun.
clean gun.
apply tung oil.
ak47 shoot.

produce m4a1 gun.
clean gun.
spray paint.
m4a1 shoot.
```
可以看到Ak47和M4A1在生产出来后的处理不一样，Ak需要涂桐油，M4需要喷漆，用简单工厂就比较难做到，所以就每个产品都弄个工厂来封装各自己的生产过程。
另外的好处是当加入其他枪比如沙漠之鹰时，再加一个产品和产品工厂就好了，并不需要改变现有代码，算是做到了遵守开闭原则。

# 抽象工厂模式

### 特点：把工厂抽象出来，让子工厂来决定怎么生产产品, 每个产品都由自己的工厂生产。

### 用处：当产品对象需要进行不同的加工时可以考虑工厂方法。

### 注意：这不是所谓的简单工厂的升级版，两者有不同的应用场景。

下面用TypeScript写一个枪工厂来看看工厂方法模式：