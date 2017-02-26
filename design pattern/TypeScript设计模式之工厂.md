看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 简单工厂模式 Simple Factory

### 特点：把同类型产品对象的创建集中到一起，通过工厂来创建，添加新产品时只需加到工厂里即可，也就是把变化封装起来，同时还可以隐藏产品细节。

### 用处：要new多个同一类型对象时可以考虑使用简单工厂。

### 注意：对象需要继承自同一个接口。

下面用TypeScript写一个枪工厂来看看简单工厂模式：

```ts
enum GunType{
    AK,
    M4A1,
}

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

class GunFactory{

    static createGun(type: GunType): Gun{
        switch(type){
            case GunType.AK:
                return new AK47();
            case GunType.M4A1:
                return new M4A1();
            default:
                throw Error('not support this gun yet');
        }
    }
}

GunFactory.createGun(GunType.AK).shoot();
GunFactory.createGun(GunType.M4A1).shoot();

//输出
ak47 shoot.
m4a1 shoot.
```
上面代码`GunFactory`工厂就是根据类型来创建不同的产品，使用的时候只需要引入这个工厂和接口即可。
这样就把变化封装到了工厂中，如果以后要支持狙击枪，只需要加个实现`Gun`接口的`Sniper`类就可以了。

# 工厂方法模式 Factory Method

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
缺点也明显，增加一个产品就需要多加两个类，增加了代码复杂性。

# 抽象工厂模式 Abstract Factory

### 特点：同样隐藏了具体产品的生产，不过生产的是多种类产品。

### 用处：当需要生产的是一个产品族，并且产品之间或多或少有关联时可以考虑抽象工厂方法。

### 注意：和工厂方法的区别，工厂方法是一个产品， 而抽象工厂是产品族，线和面的区别。

继续用枪，外加子弹，用TypeScript写一个抽象枪工厂来看看抽象工厂模式：

```ts
interface Shootable{
    shoot();
}

abstract class Gun implements Shootable{ // 抽象产品 - 枪
    private _bullet: Bullet;

    addBullet(bullet: Bullet){
        this._bullet = bullet;
    }

    abstract shoot();
}

class AK47 extends Gun{ //具体产品 - AK47

    shoot(){
        console.log(`ak47 shoot with ${this._bullet}.`);
    }
}

class M4A1 extends Gun{ //具体产品 - M4A1
    
    shoot(){
        console.log(`m4a1 shoot with ${this._bullet}.`);
    }
}

abstract class Bullet{ // 抽象子弹
    abstract name: string;
}

class AkBullet{ // AK 子弹
    name: string = 'ak bullet';
}

class M4Bullet{ // m4a1 子弹
    name: string = 'm4a1 bullet';
}

abstract class ArmFactory{ //抽象军工厂
    abstract createGun(): Gun;
    abstract createBullet(): Bullet;
}

class AK47Factory extends ArmFactory{
    createGun(): Gun{
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

    createBullet(): Bullet{
        return new AkBullet();
    }
}

class M4A1Factory extends ArmFactory{ //M4A1工厂
    createGun(): Gun{
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

    createBullet(): Bullet{
        return new M4Bullet();
    }
}

//使用
function shoot(gun: Gun, bullet: Bullet) // 使用生产的枪和子弹
{
    gun.addBullet(bullet);
    gun.shoot();
}

let akFactory = new AK47Factory();
shoot(akFactory.createGun(), akFactory.createBullet());

let m4a1Factory = new M4A1Factory();
shoot(m4a1Factory.createGun(), m4a1Factory.createBullet());

//输出
produce ak47 gun.
clean gun.
apply tung oil.
add bullet: ak bullet
ak47 shoot with ak bullet.

produce m4a1 gun.
clean gun.
spray paint.
add bullet: m4a1 bullet
m4a1 shoot with m4a1 bullet.
```
工厂除了生产枪外还生产子弹，子弹和枪算是一个产品族，使用者接触到的只有抽象工厂和抽象产品，隐藏了具体实现细节。
在大的框架下面有很多小项目时用抽象工厂配合如动态对象生成之类的技术就可以很容易实现灵活的架构。