# Events模块介绍

<img src="C:\Users\ASUS\Desktop\note\images\node\event1.png" style="zoom:60%;" />

Events，我们也可以称为发布/订阅模式，Node.js中大多数模块都依赖于此，如Net,Http,Fs,Stream等。

事件驱动是Node.js的核心，当我们触发一个事件，我们可以通过设置回调来接收处理，如`fs.readFile(path, callback)`

## EventEmitter的基本用法

```javascript
import { EventEmitter } from 'event'
const emitter = new EventEmitter();

// 设置监听器
emitter.on("起床", function(time) {
    console.log(`早上 ${time} 开始起床，新的一天加油！`)
});

// 触发监听器
emitter.emit("起床", "6:00");
```

对于一个事件实例上有如下的属性和方法：

*   addListener(event, listener): 向事件队列后面再增加一个监听器
*   emit(event, \[arg1\], \[arg2\], \[…\]): 向事件队列触发一个事件，同时可以对该事件传过去更多的数据
*   listeners(event): 返回事件队列中特定的事件监听对象
*   on(event, listener): 针对一个特定的事件注册监听器，该监听器就是一个回调函数
*   once(event, listener): 与 on 一样，只不过它只会执行一次，只生效一次
*   removeAllListeners(\[event\]): 移除所有指定事件的监听器，不指定的话，移除所有监听器，也就是清空事件队列
*   removeListener(event, listener): 只移除特定事件监听器
*   setMaxListeners(n): 设置监听器数组的最大数量，默认是 10，超过 10 个会收到 Node 的警告

## 定制自己的events

一日计划，在不同的时间点触发相应的事件，处理相应的事件

```javascript
import { EventEmitter } from 'events'

const oneDayPlanRun = {
    "6:00": function() {
        console.log('现在是早上六点，你还需要继续睡一会')
    },
    "7:00": function() {
        console.log('现在是早上七点，起床洗漱吃早餐')
    }
}

class OneDayPlan extends EventEmitter {}

let oneDayPlan = new OneDayPlan()

oneDayPlan.on("6:00", function() {
    oneDayPlanRun["6:00"]();
})

oneDayPlan.on("7:00", function() {
    oneDayPlanRun["7:00"]();
})

async function sleep(s) {
    return new Promise(function(resolve) {
        setTimeout(function() {
            resolve(1)
        }, s)
    })
}

async function doMain() {
    oneDayPlan.emit("6:00");
    await sleep(2000);
    oneDayPlan.emit("7:00");
}

doMain();
```

## 同步异步问题

> EventEmitter 会按照监听器注册的顺序同步地调用所有监听器。 所以必须确保事件的排序正确，且避免竞态条件。

```javascript
import { EventEmitter } from 'events'
const emitter = new EventEmitter()

emitter.on('test',function(){
    console.log(111)
});
emitter.emit('test');
console.log(222)

// 输出
// 111
// 222
```

可以使用 setImmediate() 或 process.nextTick() 切换到异步模式：

```javascript
import { EventEmitter } from 'events'
const emitter = new EventEmitter()

emitter.on('test',function(){
    setImmediate(() => {
        console.log(111);
    });
});
emitter.emit('test');
console.log(222)

// 输出
// 222
// 111
```

## 错误处理

我们应始终为'error'事件注册监听器，防止程序崩溃导致Node进程自动退出

```javascript
import { EventEmitter } from 'events'
const emitter = new EventEmitter()

emitter.on('error', function(err) {
    console.error(err);
})

emitter.emit('error', new Error('This is a error'));
```

