# js基础

## 1、数据类型

### 1.1 类别

* **基本数据类型**

  undefined 、null(null不是对象)、string、boolean、number、Bigint、Symbol

* **引用数据类型**

  function、object、array、Data、正则表达式、Math

### 1.2 类型检测

1. typeof（返回字符串）

   ​	**优点：**简单

   ​	**特殊**：null转换是object

   ​	因为javascript中不同**对象**在底层都表示为二进制，javascript中会把二进制的**前三位为0的表示为object类型**，而null的二进制表示形式都是0，所以自然也就被识别为object类型了。

   ​	**注意点：**{}和[]转换是object，function转换是function

   ```js
   console.log(typeof [])//"object"
   console.log(typeof typeof typeof [])//string
   ```

2. instanceof （基于原型链的检测）

   **优点：**可以弥补typeof 对 对象类型检测的缺点

   **缺点：**只适用于对象的类型检测，基本数据类型无法检测

   ```javascript
   console.log(1 instanceof Number);//false  
   console.log(new Number(1) instanceof Number)//true
   
   ```

   > instanceof判断失真现象

   左边对象的原型链上，只有`null`对象。这时，`instanceof`判断会失真。

   ```js
   var obj = Object.create(null)
   typeof obj//Object
   obj instanceof Object//false
   ```

   左边对象的原型链上没有右边构造函数的prototype

   * 字面量方式创建的不能检测
   * 构造函数创建的可以检测

   - 检测的实例必须时对象数据类型的

   **手写instanceof**

   ```js
   function my(left,right){
   	if(typeof left !== object && left == null) return;
       let proto = Object.getPrototypeof(left)
       while(true){
           if(proto == null) return false;
   		if(proto == right.prototype)
           	return true;
           proto = Object.getPrototypeof(proto);
        }
   }
   ```

   ```js
   var arr = new Array();
   var arr = ['aa','bb','cc'];
   var obj = {
       a: 'aa',
       b: 'bb',
       c: 'cc'
   };
   console.log(arr instanceof Array); //true
   console.log(arr instanceof Object); //true
   ```

3. constructor

   **缺点：**无法检测出null, 和 undefined,会报错，因为其是无效的对象，没有constructor

   ```javascript
   var obj = new Object()
   obj.constructor === Object	// true
   var person = new Person()
   person.constructor === Person	//true
   person.constructor === Object //false
   
   {}.constructor === Object //ture
   javascript是由一个原始对象clone出来的，创建一个对象字面量的使用，它的构造函数会指向原始对象Object所有
   ```

4. Object.prototype.tostring.call()

   ```js
   /**
    * @desc 数据类型检测
    * @param obj 待检测的数据
    * @return {String} 类型字符串
    */
   function type(obj) {
     return typeof obj !== "object" ? typeof obj : Object.prototype.toString.call(obj).slice(8, -1).toLowerCase();
   }
   ```

### 1.3 类型转换

>  补一张类型转化的图片，网上找一个好的

#### 双等和三等

== 有类型转换，=== 无类型转换，严格模式

#### 双等的转化规则

1. 如果有一个操作数是boolean，则在比较之前先转换为数值

2. 如果一个是字符串，一个是数值，在比较之前先将字符串转换为数值

3. 如果一个是对象，另一个不是，调用对象的valueOf方法，得到基本类型值按照前面规则比较

   ![image-20210604190943470](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210604190943470.png)

4. null ==  undefined //true

5. NaN == NaN //false

   只要有一个为NaN，则返回false

6. 如果两个操作数都是对象，则比较他们是否为同一个对象。如果两个操作数都指向同一个对象，则返回true

#### 对象怎么转换为原始类型

1. 优先调用内置函数Symbol.toPrimitive()
2. 调用valueOf函数
3. 调用toString函数
4. 没有返回原始类型，会报错

## 2、作用域和作用域链

### 2.1 LSH和RSH

运行一段JavaScript代码，就需要**JS引擎**和**编译器**。

#### 编译

1. 词法/语法分析：编译器先将字符串打断成有意义的片段，称为token（记号）
2. 解析/语法分析：编译器将一个token的流（数组）转换为一个“抽象语法树”，表示程序的语法结构
3. 代码生成：编译器将上一步生成的抽象语法树转换为机器指令，等待引擎执行

#### 执行

LHS和RHS的 区别是：对变量的查询目的是**变量赋值**还是**查询**

**LSH**：变量在赋值操作的左侧，**变量赋值**

**RSH**：变量在赋值操作的右侧，**查询**（取到源值）

```js
function foo(a) { // LHS 查询
  console.log( a ); // RHS 查询
}

foo( 2 ); // RHS 查询
```

`LHS` 和 `RHS` 获取变量的位置就是 **作用域**

### 2.2 什么是作用域

> 程序中定义变量的区域，决定当前执行代码对变量的访问权限

两种作用域类型：

- 全局作用域：程序最外层作用域，一直存在，
- 函数作用域：函数作用域只有函数被定义时才会创建，包含在父级函数作用域/全局作用域内。

### 2.3 作用域链

> 当可执行代码内部访问变量时，会先查找本地作用域，如果找到目标变量即返回，否则会去父级作用域继续查找...一直找到全局作用域。我们把这种作用域的嵌套机制，称为 作用域链。

### 2.4 词法作用域

词法作用域是***JavaScript***使用的作用域类型

```js
var value = 1;

function foo() {
  console.log(value);
}

function bar() {
  var value = 2;
  foo();
}

bar();
//结果是1
```

foo访问本地作用域中没有的变量value。引擎为了拿到这个变量就要去 `foo` 的上层作用域查询，那么 `foo` 的上层作用域是什么呢？是它 **调用时** 所在的 bar 作用域？还是它 **定义时** 所在的全局作用域？JavaScript是词法作用域，所以foo上层作用域是它定义时所在的全局作用域。

> 词法作用域，函数定义时作用域已经确定了，和拿到哪里执行无关，因此词法作用域也被称为“静态作用域”。

”欺骗“词法作用域：

eval和with

### 2.5 块级作用域

`javascript` 不是原生支持块级作用域的，但es6提出了使用***let***和**const**代替var关键字，来创建块级作用域

创建作用域方法：

1. 定义函数，创建函数作用域

2. 使用let和const创建块级作用域

   let特点：

   - 创建块级作用域
   - 不存在变量提升
   - 存在暂时性死区
   - 不能重复定义变量
   - 全局作用域下声明变量不会挂载到顶层对象上

   const特性：

   - 同let

   - 一旦初始化赋值，后面不能被修改
   - 定义时必须初始化

3. try catch创建作用域，error仅存在catch子句中

4. eval欺骗词法作用域

5. with欺骗词法作用域



## 3、闭包

### 3.1 事先了解

#### 3.1.1 JS中的堆内存和栈内存

**堆内存**

1. 储存引用类型值(对象：键值对； 函数：代码字符串)
2. 当前堆内存释放销毁，那么这个引用类型就彻底没了
3. 堆内存的释放：当堆内存没有被任何的变量或者其他东西所占有，浏览器在空闲时会自动的进行内存回收，把所有不被占用的堆内存销毁（谷歌浏览器）。另外，``xxx= null``,通过空对象指针null可以让原始变量谁都不指向，所以先前原有被占用的堆内存现在就不被东西占用了，浏览器便会销毁它

**栈内存**

1. 提供一个供js代码自上向下执行的环境(代码都在栈中执行)
2. 由于**基本数据**类型值比较简单，他们都是直接在**栈内存**中开辟一个位置，把值直接储存进去
3. 当栈被销毁，存储的那些基本值也都跟着被销毁了

#### 3.1.2 JS作用域

- 词法作用域（js就是词法作用域）

  作用域是在函数创建的时候生成的，和拿到哪里执行无关

  举个例子：

  ```js
  var a = 1;
  function bar(){
  	console.log(a);
  }
  function foo(){
  	var a = 2;
      bar();//1
  }
  foo();
  ```

  打印1原因是bar函数在创建时生成的作用域中只有全局变量a = 1

  ```js
  var a = 1;
  function foo(){
      var a = 2;
      function bar(){
          console.log(a);
      }
      bar()
  }
  foo();
  ```

- 动态作用域

  ```js
  var a = 1;
  function bar(){
  	console.log(a);
  }
  function foo(){
  	var a = 2;
      bar();//1
  }
  foo();
  ```

  同样一个例子，如果是动态作用域，bar函数执行的时候，就不会向外找，而是会在调用他的函数作用域中查找，会输出2

#### 3.1.3 作用域的作用

1. 保护作用（作用域外访问不到作用域里面的变量）
2. 寻找变量，从内到外 ,直到全局作用域结束

#### 3.1.4 变量生存周期

- 全局变量：除非主动消除（脚本退出），否则是永久的


- 局部变量：在函数内声明的变量，函数调用完，退出执行栈

### 3.2 什么是闭包

#### 相关认知

1. 普遍：一个函数作用域可以访问另一个函数的作用域

2. 另一种认知：函数执行形成一个私有的作用域，保护里面的私有变量不受外界的干扰，这种保护机制称之为“闭包”。

#### 讲讲闭包

当一个函数进行创建，其会在堆内存中储存其字符串。函数在执行时，会为其开辟一块栈空间（执行上下文），初始化其变量，执行其函数体，当函数执行完成后，所形成的私有环境（栈内存或者说是**局部变量**）都会被自动释放掉，但有一些特殊情况下不被销毁：

1. 函数执行完成，当前所形成的栈内存中，某些东西被栈内存以外的变量占用了（如返回到外面函数对其变量进行了引用），此时栈内存不能释放（一旦释放，外面找不到原有的内容了）。**这也就是我们说的形成闭包啦**。
2. 全局栈内存（或者说是**全局变量**）只有在页面关闭的时候才会被释放掉。

### 3.3 闭包作用（变量私有化）

- **封装变量**，保护函数的**私有变量**不受外部的干扰，形成不销毁的栈内存

- 延长局部变量的寿命

  ```js
  function foo(){
      let a = 1;
  	let fun = function bar(){
  		console.log(a)
      }
      return fun;
  }
  let b = foo()
  b()
  ```

> a变量一直存在

### 3.4 闭包缺点

会造成**内存泄漏**

使用闭包，是因为在以后还会使用变量，把闭包放在闭包和全局作用域，对内存的影响是一致的，若在将来要回收变量，可手动将变量设为**null**。

闭包和内存泄漏有关在于，使用闭包时容易形成**循环引用**。如果闭包的作用域链中保存着DOM节点，会造成内存泄漏。在IE浏览器中，BOM和DOM中的对象是使用C++以COM对象的方式实现的，COM对象的垃圾收集机制采用的时引用计数方法。引用计数------>**循环引用**，要解决循环引用带来的内存泄漏问题，将循环引用中的变量设为NULL

### 3.5 闭包的应用场景

#### 还未完成。。。。。

- 单例模式
- 模拟私有属性
- 柯里化





- 访问函数内部的变量

- 实现setTimeout 传值

  ```js
  function fun(num){
  	return fun1(){
  		console.log(num)
      }
  }
  var f1 = fun(1)
  setTimeout(f1,1000)
  ```

- 定义私有属性

  ```js
  var count = (function(){
      var privatenum = 0
      function change(value){
  		privatenum += value
      }
      return {
          increment:function(){
  			change(1)
          },
          decrement:function(){
  			change(-1)
          },
          num:function(){
  			return privatenum
          }
      }
  })()
  
  count.num//访问到privatenum
  
  // 联系一下es6中的class
  ```

- 案例应用

  - **节流**

  当持续触发事件，保证一段时间内，只调用一次事件处理函数

  可以在**表单的提交**中使用到

  ```js
  function throttle(callback,delay){
      let timer = null;
  	return function(){
          if(!timer){
             timer = setTimeout(function(){
                	timer = null;
              	callback()
          	},delay) 
          }
      }
  }
  ```

  - **防抖**

  持续的触发事件，一定时间内没有触发该事件，事件处理函数才会执行一次，如果在设定的时间内又触发了一次，就会重新计时

  **输入框事件**中会用到

  ```js
  function debounce(callback,delay){
      let timer = null;
  	return function (args){
          clearTimeout(timer)
          timer = setTimeout(function(){
              callback(args)
          },delay)
      }
  }
  ```

  * 利用闭包判断类型

  ```js
  isType(type){
  	return function(target){
          return `[object ${type}]` === 					Object.prototype.toString.call(target)
      }
  }
  const isArray = isType('Array')
  console.log(isArray([]))
  ```

## 垃圾回收机制

> 标记清除（常用）和引用计数(循环引用)

```js
var a = {num:1}
var b = a
a.x = a = {num:2}

console.log(a.x)//undefined
console.log(b.x)//{num:2}
```

![image-20210725110057246](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210725110057246.png)



![image-20210725111617735](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210725111617735.png)

## 4、this指向问题

### 4.1 this是什么

this就是一个指针，指向调用的函数

### 4.2 this的绑定规则

- 默认绑定
- 隐式绑定
- 显示绑定
- 硬绑定
- new绑定

##### 默认绑定

```js
function fn(){
	console.log(this.name);//linrong (浏览器环境中运行)
    //在node环境中运行，打印undefined，node中name不是挂载到全局对象上的
}
var name = "linrong";
fn();
```

上面的例子，在非严格模式下，this指向全局对象window，严格模式下，this指向undefined，会抛出错误

##### 隐式绑定

函数的调用是在某个对象上触发的，即调用位置上存在上下文对象。

就是XXX.fun()

```js
var obj ={
  foo: function () {
    console.log(this);
  }
};

obj.foo() // obj

特殊情况：
// 情况一
(obj.foo = obj.foo)() // window
// 情况二
(false || obj.foo)() // window
// 情况三
(1, obj.foo)() // window
```

```js
// 情况一
(obj.foo = function () {
  console.log(this);
})()
// 等同于
(function () {
  console.log(this);
})()

// 情况二
(false || function () {
  console.log(this);
})()

// 情况三
(1, function () {
  console.log(this);
})()
```

还有a.b.c()的调用模式，不管有多少层，在判断this的时候，我们只关注最后一层。

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person2 = {
    name: 'Christina',
    sayHi: sayHi
}
var person1 = {
    name: 'YvetteLau',
    friend: person2
    //friend:{
    //    name: 'Christina',
    //    sayHi: function sayHi(){
    //        console.log('Hello,', this.name);
    //    }
    //}

}
person1.friend.sayHi();
//this指向friend
```

隐式绑定容易造成**绑定丢失**

- 避免多层this

  内层的this不指向外部，而指向顶层对象

```js
var o = {
  f1: function () {
    console.log(this);//object
    var f2 = function () {
      console.log(this);//window
    }();
  }
}
o.f1()

//实际代码
var temp = function () {
	 console.log(this);
};
var o = {
  f1: function () {
    console.log(this);//object
    var f2 = temp();
  }
}
o.f1()

//解决方法,改变this指向（后面详细说）
var o = {
  f1: function () {
    console.log(this);//object
    var that = this;
    var f2 = function () {
      console.log(that);//obj
    }();
  }
}
o.f1()
```



- 避免回调函数中的this

这个例子有点不太懂

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person1 = {
    name: 'YvetteLau',
    sayHi: function(){
        setTimeout(function(){
            console.log('Hello,',this.name);//实际this指向window
        })
    }
}
var person2 = {
    name: 'Christina',
    sayHi: sayHi
}
var name='Wiliam';
person1.sayHi();//wiliam
setTimeout(person2.sayHi,100);//wiliam
setTimeout(function(){
    person2.sayHi();//Christina
},200);
```



javaScript环境中的setTimeout（）函数实际是：

```js
function setTimeout(fn,delay){
    fn();//可以理解为在全局调用
}
```



- 避免数组处理方法中的this

  数组的`map`和`foreach`方法，允许提供一个函数作为参数。这个函数内部不应该使用`this`。

```js
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);//this指向window
    });
  }
}

o.f()
// undefined a1
// undefined a2


//解决方法
//法一：that = this
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    var that = this;
    this.p.forEach(function (item) {
      console.log(that.v+' '+item);
    });
  }
}
o.f()
// hello a1
// hello a2

//法二：this当作foreach方法的第二个参数
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);
    }, this);
  }
}
o.f()
```

##### 显式绑定

通过call，apply，**bind**（**硬绑定**）的方式，显式的指定this所指向的对象。

当函数赋值给另一个值时，this指向会改变

```js
function sayHi(){
    console.log('Hello,', this.name);
}
var person = {
    name: 'YvetteLau',
    sayHi: sayHi
}
var name = 'Wiliam';
var Hi = function(fn) {
    fn();
}
Hi.call(person, person.sayHi); 
//可以理解为person.sayHi函数赋给fn，fn执行在全局作用域下
```

非严格模式下，会将原始类型的值包装成对象

严格模式下，原始类型的值就是原始值

```js
var doSth = function(name){
    console.log(this);
    console.log(name);
}
doSth.call(2, '若川'); // Number{2}, '若川'
var doSth2 = function(name){
    'use strict';
    console.log(this);
    console.log(name);
}
doSth2.call(2, '若川'); // 2, '若川'
```

##### new绑定

构造函数的this指向他的实例对象

> new关键字：
>
> 1、创建一个新对象
>
> 2、this指向新对象，执行构造函数（返回结果为result）
>
> 3、新对象的_ _proto_ _ 指向构造函数的prototype
>
> 4、判断：若result是一个对象，返回这个对象，否则返回新创建的对象

![image-20210728210305471](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210728210305471.png)

##### 绑定优先级

new绑定 > 硬绑定>显示绑定 > 隐式绑定 > 默认绑定

```js
function foo(someing){
	this.a = someing;
}
var obj1 = {};

var bar = foo.bind(obj1);//bar接收返回一个新的函数，this指向obj1
bar(1)//obj1.a = 1
console.log(obj1.a);//1

var baz = new bar(2);//bar的this指向baz
console.log(obj1.a);
console.log(baz.a);//2
```

##### 绑定例外

如果我们将null或者是undefined作为this的绑定对象传入call、apply或者是bind,这些值在调用时会被忽略，实际应用的是默认绑定规则。

#### 4.2.2 箭头函数

特点：

- 箭头函数没有自己的this，不能用call，apply和bind方法，**它的this继承的是外层代码块的this**
- 不能当做构造函数，不可以使用new命令，否则会报错
- 不可以使用arguments对象（在函数体内不存在），如果要用，可以用rest参数代替
- 不可以用yield命令，所以箭头函数不能用作Generator函数

例子：

```js
var obj = {
    hi: function(){
        console.log(this);
        return ()=>{
            console.log(this);
        }
    },
    sayHi: function(){
        return function() {
            console.log(this);
            return ()=>{
                console.log(this);
            }
        }
    },
    say: ()=>{
        console.log(this);
    }
}
let hi = obj.hi();  //输出obj对象
hi();               //输出obj对象
let sayHi = obj.sayHi();
let fun1 = sayHi(); //输出window
fun1();             //输出window
obj.say();          //输出window
```



```js
var obj = {
    hi: function(){
        console.log(this);
        return ()=>{
            console.log(this);
        }
    },
    sayHi: function(){
        return function() {
            console.log(this);
            return ()=>{
                console.log(this);
            }
        }
    },
    say: ()=>{
        console.log(this);
    }
}
let sayHi = obj.sayHi();
let fun1 = sayHi(); //window
fun1();             //window

let fun2 = sayHi.bind(obj)();//obj
fun2();                      //obj

```

### 4.3 一道题

```js
var number = 5;
var obj = {
    number: 3,
    fn: (function () {
        var number;
        this.number *= 2;
        number = number * 2;
        number = 3;
        return function () {
            var num = this.number;
            this.number *= 2;
            console.log(num);
            number *= 3;
            console.log(number);
        }
    })()
}
var myFun = obj.fn;

myFun.call(null);

obj.fn();

console.log(window.number);


//解析：
var myFun = obj.fn;
/*
var myFun = function() {
            var num = this.number;
            this.number *= 2;
            console.log(num);
            number *= 3;
            console.log(number);
        };
此时立即执行函数已经执行了，立即执行函数的this指向window，所以执行立即函数中的代码，在函数的作用域里面创建变量number;随着立即执行函数执行的结束，内部变量会自动销毁
this.number中this指向window，window.number=10;
函数作用域中的number是undefined，所以number= =NAN；下一行中number赋值为3；
*/
myFun.call(null);
/*
myFun中this用了默认绑定，指向window；
num = window.number ,所以num为10;
window.number*=2,所以window.number=20;
这里number*=3，number未在函数中定义，会向它的父作用域中查找，number=3，最后number*3=9
打印10,9
*/

obj.fn();
/*
这里this指向obj，num=obj.number,num=3;
obj.number*=2,ojb.number=6;
number=9,*3后=27；
打印3,27
*/

console.log(window.number);//20

//this是动态的，要去看在谁的环境里执行的函数
```













## 5、原型，原型链，继承

### 5.1 原型

**定义：**原型就是指函数的prototype属性所引用的对象，这个对象就是原型

**作用：**原型的作用就是为了实现对象之间的数据共享，es5获取对象原型的标准方法是Object.getPrototypeOf()。

判断原型和实例的继承关系方法：

方法一：用**instanceof**操作符 （实例   instanceof  原型）

方法二：用  **原型.prototype.isprototypeOf(实例)**    来判断

### 5.2 原型链

![img](https://img2018.cnblogs.com/blog/850375/201907/850375-20190708153139577-2105652554.png)



### 5.3 六大继承

#### 原型链继承

```js
function SuperType(){
	this.property = true;
}
SuperType.prototype.getSuperValue = function(){
	return this.property;
}
function SubType() {
    this.subproperty = false;
}
//这里是关键，创建SuperType的实例，并将该实例赋值给SubType.prototype
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function() {
    return this.subproperty;
}
var instance = new SubType();
console.log(instance.getSuperValue()); // true
```

**缺点：**

​			1、多个实例对引用类型的操作会被篡改。

​			2、子类型在实例化时不能向父类型的构造函数传参。

#### 借用构造函数继承

使用父类的构造函数来增强子类**实例**，等同于复制父类的实例给子类（不使用原型）

```js
function  SuperType(){
    this.color=["red","green","blue"];
}
function  SubType(){
    //继承自SuperType
    SuperType.call(this);
}
var instance1 = new SubType();
instance1.color.push("black");
alert(instance1.color);//"red,green,blue,black"

var instance2 = new SubType();
alert(instance2.color);//"red,green,blue"

//在创建子类实例时，会调用SuperType构造函数，子类实例会将SuperType构造函数中的属性复制一份。就是新的SubType对象上运行了SuperType()函数中的所有初始化代码。
```

相比于原型链，借用构造函数的优点是：可以在子类型构造函数中向父类构造函数传参。

**缺点：**

- 只能继承父类的**实例**属性和方法，不能继承原型属性/方法
- 无法实现复用，每个子类都有父类实例函数的副本，影响性能



构造函数继承

（子类完全继承父类）

1. 在子类的构造函数中，调用父类的构造函数   fun.call(this)：用fun对象代替this对象
2. 让子类的原型对象指向父类的原型对象

```js
function son(){
	father.call(this)
}
son.prototype = Object.create(father.prototype);//赋值，深浅拷贝的区别
//如果这里是直接赋值的话，son.prototype改变，father原型对象也会改变
son.prototype.constructor = son;
```

（子类继承父类的单个方法）

```js
classson.prototype.fun = function (){
	classfather.prototype.fun.call(this);
}
```

(多重继承)

通过方法实现一个对象继承多个对象

```js
function fun1(){
    this.hello = "hello";
}
function fun2(){
	this.world = "world";
}
function son(){
    fun1.call(this);
    fun2.call(this);
}
//继承fun1
son.prototype = Object.create(fun1.prototype);
//在原型链上加上fun2
Object.assign(son.prototype,fun2.prototype);
//指定构造函数
son.prototype.constructor=son;
```

#### 组合继承

通过原型链实现对原型上属性和方法的继承

通过借用构造函数实现实例的属性的继承（将父类的属性赋给子类）

```js
function SuperType(name) {
    this.name = name;
    this.color = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
    alert(this.name);
}
function SubType(name, age) {
    //继承属性
    //第二次调用SuperType()
    SuperType.call(this, name);
    this.age = age;
}

//继承方法
//构建原型链
//第一次调用SuperType()
SubType.prototype = new SuperType();
//重写SubType.prototype的constructor属性，指向自己的构造函数SubType
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
    alert(this.age);
}

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); //"red, blue, green, black"
instance1.sayName(); //"Nicholas";
instance1.sayAge(); //29

var instance2 = new SubType("Greg", 27);
alert(instance2.color); //"red, blue, green"
instance2.sayName(); //"Greg";
instance2.sayAge(); //27
```

缺点：

- 第一次调用`SuperType()`：给`SubType.prototype`写入两个属性name，color。
- 第二次调用`SuperType()`：给`instance1`写入两个属性name，color。

实例对象`instance1`上的两个属性就屏蔽了其原型对象SubType.prototype的两个同名属性。所以，组合模式的缺点就是在使用子类创建实例对象时，其原型中会存在两份相同的属性/方法。

#### 原型式继承

利用一个空对象作为中介，将某个对象直接赋值给空对象构造函数的原型。

```javascript
function object(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}
```

object()对传入其中的对象执行了一次浅复制，将构造函数F的原型直接指向传入的对象。

```javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friend); //"Shelby, Court, Van, Rob, Barbie"
```

**缺点：**

- 原型链继承多个实例的引用类型属性指向相同，存在篡改的可能。
- 无法传递参数。

另外，ES5中存在Object.create()的方法，能够代替上面的object方法。



- 寄生式继承

核心：在原型式继承的基础上，增强对象，返回构造函数

```javascript
function createAnother(original) {
    var clone = object(original); //通过调用object()函数创建一个新对象
    clone.sayHi = function() {
        alert("hi");
    }
    return clone; //返回这个对象
}
```

函数的主要作用是为构造函数新增属性和方法，以**增强函数**

```javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = createAnother(person);
anotherPerson.sayHi(); //"hi"
```

**缺点（同原型式继承）：**

- 原型链继承多个实例的引用类型属性指向相同，存在篡改的可能。
- 无法传递参数

#### 寄生组合继承

结合借用构造函数传递参数和寄生模式实现继承

```javascript
function inheritPrototype(subType, superType){
  var prototype = Object.create(superType.prototype); // 创建对象，创建父类原型的一个副本
  prototype.constructor = subType;                    // 增强对象，弥补因重写原型而失去的默认的constructor 属性
  subType.prototype = prototype;                      // 指定对象，将新创建的对象赋值给子类的原型
}

// 父类初始化实例属性和原型属性
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
};

// 借用构造函数传递增强子类实例属性（支持传参和避免篡改）
function SubType(name, age){
  SuperType.call(this, name);
  this.age = age;
}

// 将父类原型指向子类
inheritPrototype(SubType, SuperType);

// 新增子类原型属性
SubType.prototype.sayAge = function(){
  alert(this.age);
}

var instance1 = new SubType("xyc", 23);
var instance2 = new SubType("lxy", 23);

instance1.colors.push("2"); // ["red", "blue", "green", "2"]
instance1.colors.push("3"); // ["red", "blue", "green", "3"]
```

这个例子的高效率体现在它只调用了一次SuperType构造函数，并且因此避免了在SubType.prototype上创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用instanceof和isPrototypeOf()。

#### 多重继承

通过方法实现一个对象继承多个对象

```js
function fun1(){
    this.hello = "hello";
}
function fun2(){
	this.world = "world";
}
function son(){
    fun1.call(this);
    fun2.call(this);
}
//继承fun1
son.prototype = Object.create(fun1.prototype);
//在原型链上加上fun2
Object.assign(son.prototype,fun2.prototype);
//指定构造函数
son.prototype.constructor=son;
```

#### es6类继承extends



## 6、跨域（还未完成）

### 6.1 同源策略

具备相同的域名，相同的端口，相同的协议，如果不满足其中一个就叫做跨域

哪些请求**遵循同源策略**

XMLHttpRequest 和 Fetch api

### 6.2 解决跨域问题

![image-20210804182458843](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210804182458843.png)

1、修改响应头

CORS(跨域资源共享)









# js进阶

### 1、类数组

是一个对象类型，属性从0开始，一次为0,1,2……最后还有callee和length属性

常见类数组有：

- 用getElementsByTagName/ClassName()获得的HTMLCollection
- 用querySelector获得的nodeList

**类数组转换为数组方法：**

1. Array.prototype.slilce.call()
2. Array.from()
3. 扩展运算符  let args = [...arguments]`
4. concat+apply   Array.prototype.concat.apply([], arguments);
5. 在创建一个数组，用for循环遍历arguments，push进去

### 2、数组扁平化

将多层级数组转化为一级数组（即提取嵌套数组元素最终合并为一个数组）

```js
let ary = [1,[2,[3,[4,5]]],6];
```

- flat(参数) 参数表示深度

```js
console.log(arr.flat(Infinity))
```

- reduce方法

```js
function flatten(ary){
    return ary.reduce((result,item)=>{
		return result.concat(Array.isArray(item)?flatten(item):item);
    },[])
}
```

- 扩展运算符

```js
while(ary.some(Array.isArray)){
    ary = [].concat(...ary)
}
```

- 数组转成字符串，再将字符串转成数组

```js
 function flatten(ary) {
            return ary.join(',').split(',').map(item=>
                parseInt(item)
            )
        }
        console.log(flatten(ary))

//ary.join(',')--->1,2,3,4,5,6
//ary.join(',').split()---> ["1", "2", "3", "4", "5", "6"]

```

- replace+JSON

```js
let str = JSON.stringify(ary).replace(/\[|\]/g,'');
console.log(JSON.parse('[' + str + ']'));
```

### 3、手写map

**了解map**

参数:接受两个参数，一个是回调函数，一个是回调函数的this值(可选)。

其中，回调函数被默认传入三个值，依次为当前元素、当前索引、整个数组。

```js
let nums = [1, 2, 3];
let obj = {val: 5};
let newNums = nums.map(function(item,index,array) {
  return item + index + array[index] + this.val; 
  //对第一个元素，1 + 0 + 1 + 5 = 7
  //对第二个元素，2 + 1 + 2 + 5 = 10
  //对第三个元素，3 + 2 + 3 + 5 = 13
}, obj);
console.log(newNums);//[7, 10, 13]
```



```js
Array.prototype.newmap = function(fn,thisAry){
	if(object.prototype.toString.call(fn) !== '[Object Function]' ){
        throw('Error in params')
    }
    let resAry = [];
    let curAry = this;
    for(let i = 0;i < curAry.length;i++){
       resAry =  fn.call(thisAry,curAry[i],i,arr)
    }
    return resAry;
}
```



# es6

## 1、let、var、const

1. let特点：

   - 创建块级作用域
   - 不存在变量提升
   - 存在暂时性死区
   - 不能重复定义变量 
   - 全局作用域下声明变量不会挂载到顶层对象上

   ```js
   let a = 1;
   console.log(this.a)//undefined
   console.log(window.a)//undefined
   ```

   

   const特性：

   - 同let

   - 一旦初始化赋值，后面不能被修改(基本数据类型不能改变)
   - 定义时必须初始化

2. try catch创建作用域，error仅存在catch子句中

3. eval欺骗词法作用域

4. with欺骗词法作用域

## 2、解构赋值

### 数组解构赋值

- 模式匹配(支持数组嵌套)

- 不完全解构

**指定默认值**

```js
let [f = true] = []
console.log(f);//true
let [a,b="y"] = ["a"]
console.log(b);//y

let [x = 1] = [null]
let [y = 2] = [undefined]
console.log(x);//null
console.log(y);//2
//es6内部使用 === 判断是否是undefined，如果是undefined 默认值才会生效
```

默认值可以引用解构赋值的其他变量，但是变量必须提前声明了

```js
let [x=1,y=x] = []
console.log(x)//1
console.log(y)//1
//是从左往右赋值的
let [x=y,y=1]=[]//报错
//这里y还未定义
```

**惰性求值**

```js
function f(){
	return '2'
}
let [x= f()] = ['1'];
console.log(x)//1
```

### 对象的解构赋值 

对象的解构和数组不一样，和顺序无关

对象解构赋值的时候，对象等号右边也得是对象

```js
let person = {
    name:"chirs",
    age:33
}
let {age,name} = person;
console.log(name);//chirs
console.log(age);//33

let {sex} = person;
//let sex = person.sex
console.log(sex);//undefined

//从person中获取name，赋值给name1
let {name:name1} = person;
console.log(name1);//
```

```js
const obj1 = {}
const obj2 = {name:'lang'}
Object.setPrototypeOf(obj1,obj2);
//obj1.__proto__ = obj2
const {name} = obj1;
console.log(name);//lang
```

## 3、map和set












