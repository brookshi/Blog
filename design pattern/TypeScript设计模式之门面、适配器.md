看看用TypeScript怎样实现常见的设计模式，顺便复习一下。

# 适配器模式 Adapter

### 特点：把类或接口转换成另一个接口以便系统调用。

### 用处：当系统需要引入多个功能类并且这些功能的接口不统一时可以考虑用适配器模式把它们转成统一的接口，现实中的例子很多，比如充电器接口适配器。

### 注意：分为对象适配器和类适配器。

适配器模式的目的主要在于解决接口兼容性。

下面用TypeScript简单实现一下适配器模式：
假定现在项目已经在用一个画图接口Graph以及它的实现Canvas2D：

```ts
interface Graph{
    drawLine();

    drawPie();
}

class Canvas2D implements Graph{
    drawLine(){
        console.log('draw 2d line');
    }

    drawPie(){
        console.log('draw 2d pie');
    }
}
```
项目升级需要提高UI美观，引入3D画图库Canvas3D，两者接口不一样：

```ts
class Canvas3D{
    draw3DLine(){
        console.log('draw 3d line');
    }

    draw3DPie(){
        console.log('draw 3d pie');
    }
}
```
项目是依赖接口Graph的，如果要直接加上3d功能就需要改接口，这个代价比较大，这时适配器派上用场：

```ts
class Canvas3DAdapter implements Graph{
    private canvas3D: Canvas3D = new Canvas3D();

    drawLine(){
        this.canvas3D.draw3DLine();
    }

    drawPie(){
        this.canvas3D.draw3DPie();
    }
}

let canvas2D: Graph = new Canvas2D();
canvas2D.drawLine();
canvas2D.drawPie();

let canvas3D: Graph = new Canvas3DAdapter();
canvas3D.drawLine();
canvas3D.drawPie();

//输出
draw 2d line
draw 2d pie

draw 3d line
draw 3d pie
```
这样，使用时用Canvas3DAdapter就可以了，项目还是只依赖Graph这个接口就可以画出3D图。
在Canvas3DAdapter里引入了Canvas3D对象，可以看出这是对象上的行为适配，所以叫对象适配器。
另外还有一种叫类适配器，使用多重继承来使新的适配类继承原来接口并且拥有两个类的功能，在TypeScript里虽然不能用多重继承，但是可以用mixins方式强行加起来，这里就不写例子了。

# 外观模式 Facade

### 特点：给子系统定义一个统一的接口来方便外面调用，并且可以减少对子系统的直接依赖。

### 用处：当系统实现一个功能需要调用其他库或第三方库的很多功能时，需要有个统一调用维护的地方，这时可以考虑外观模式。

### 注意：和适配器的区别。

外观模式的目的主要在于简化调用，只需要一个简单的接口就可以解除对其他类的依赖。

下面用TypeScript简单实现一下外观模式：
假定现在项目的需求是实现一个简单图表的功能来画出近年来收入曲线图和收入来源配比图，引入一个第三方绘图库。

```ts
//第三方绘图库
class Axis{
    draw(); // 画坐标轴
}

class Line{
    draw(); // 画曲线
}

class FanShape{
    draw(angle: number); // 画扇形
}
```
项目没必要和第三方的库紧耦合，所以按需求抽象出一个接口Graph：

```ts
// 项目接口
interface Graph{ // 只需要两种图表， 线图和饼图
    drawLineChart();

    drawPieChart();
}
```
再用第三方库里的画图功能实现这个接口：

```ts
class Chart implements Graph{ // 实现接口
    drawLineChart(){
        new Axis().draw();
        new Line().draw();
    }

    drawPieChart(){
        new FanShape().draw(90);
        new FanShape().draw(180);
        new FanShape().draw(90);
    }
}
```
这样项目只需要通过Graph接口来画图表就好了，而不用知道具体的细节。

与适配器相同的点是同样是一种封装处理，不同的是适配器已有一个接口，而用这个接口不能使用另外一个系统，这时需要把那个系统做个适配来匹配现有接口，重点在于兼容接口,解决冲突。
而外观则是封装现有系统来对外提供一种简单的使用方式，重点在于简化调用。