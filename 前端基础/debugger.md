# Debugger

在我们接手一个项目，需要对这个项目的某个功能点进行深入了解的时候或者当我们在进行bugfix的时候，debugger可以大大加快我们的开发效率。

## alert

在我们alert的位置，会在浏览器中弹出一个消息框，我们可以在alert函数中传入任何我们想要得到的信息。

## console.log

console.log或许是我们常用的到方法，他的优点是我们可以直观的获取我们想要得到的值，但是当我们的项目过大，函数调用栈过深，它可能就是我们的最佳选择了。

## 如何使用浏览器进行debugger

```javascript
class Animal {
    constructor(name, active) {
        this.name = name;
        this.active = active;
    }
    getBehavior() {
        debugger;
        console.log(`${this.name}正在${this.active}`)
    }
}

class Dog extends Animal {
    constructor(color) {
        super('dog', '看门');
        this.color = color;
    }
    getInfo() {
        this.getBehavior()
        console.log(`这只狗是${this.color}的`);
    }
}

const dog = new Dog('白色')

dog.getInfo();
```

1. 打开浏览器的控制台，点击sources这一栏, 按下ctrl+p选择自己需要调试代码所在的文件

   <img src="C:\Users\ASUS\Desktop\funny的blog\images\前端基础\debugger1.png" alt="image-20210923140049891" style="zoom:67%;" />

2. 打断点，比如我现在打在了Animal类中getBehavior函数中

   <img src="C:\Users\ASUS\Desktop\funny的blog\images\前端基础\debugger2.png" alt="image-20210923140504150" style="zoom:50%;" />

3. 开始调试

   当我们在次刷新页面时，会发现函数的调用会跑到我们的断点地方后停止。这时候我们就可以使用浏览器工具进行调试了，如下图：

   <img src="C:\Users\ASUS\Desktop\funny的blog\images\前端基础\debugger3.png" alt="image-20210923140906333" style="zoom:50%;" />

* Breakpoint：我们所有的断点都可以在这块查看

* call stack：是我们当前断点所到函数的调用栈，我们可以查看当前函数是怎么被调到的。如上图我们可以得知，getBehavior是被getInfo函数调用的。

* Scope是当前函数的作用域，我们可以查看一些我们想得到的value值

  在我们进行调试的时候，浏览器主要提供了我们一下的工具，从蓝色三角号开始一次进行说明

  ![image-20210923141511956](C:\Users\ASUS\Desktop\funny的blog\images\前端基础\debugger4.png)

  1. 继续执行，即从断点处开始继续向下执行直到遇到下一个断点的位置
  2. 向下执行一条语句，如果下条语句是一个函数，不会进入到函数里面
  3. 向下执行一条语句，和第二个圆弧图标不同的是，如果下条语句是一个函数，它会进入到函数里面
  4. 跳出当前的函数

  一般只使用前面的四个，后来就是第六个图标它可以禁用所有的断点。

## debugger语句

如果我们不想在浏览器中打断点，我们可以使用debugger语句在我们的项目代码中进行断点

![image-20210923142839542](C:\Users\ASUS\Desktop\funny的blog\images\前端基础\debugger5.png)

它的效果和浏览器打断点的效果一样，也需要借助浏览器控制台进行调试。

## 对于调试的一些心得体会

有了在补充...