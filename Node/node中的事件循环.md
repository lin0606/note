### 三大关键阶段

1. timer ：执行`定时器回调`的阶段。检查定时器，如果到了时间，就执行回调。这些定时器就是setTimeout，setInterval。
2. poll: `轮询`阶段。在node代码中有异步操作，比如文件I/O,网络I/O等，当异步操作完后，会通知js主线程，怎么通知呢？就是通过'data','connect'等事件使得事件循环到达`poll`阶段。
   - 如果存在定时器，且定时器有时间了，拿出来执行，eventLoop回到timer阶段。
   - 如果没有定时器，会查看回调函数对列。（执行I/O回调）
     - 如果poll对列不为空，拿出对列中的方法依次执行。
     - 如果poll对列为空看，检查是否有setImmediate回调，有进入`check`阶段；没有继续等待，等待callback函数加入对列，加入后立刻执行。一段时间后自动进入check阶段
3. check：执行setImmediate的回调。

### 完善

当第 1 阶段结束后，可能并不会立即等待到异步事件的响应，这时候 nodejs 会进入到 `I/O异常的回调阶段`。比如说 TCP 连接遇到ECONNREFUSED，就会在这个时候执行回调。

并且在 check 阶段结束后还会进入到 `关闭事件的回调阶段`。如果一个 socket 或句柄（handle）被突然关闭，例如 socket.destroy()， 'close' 事件的回调就会在这个阶段执行。

梳理一下，nodejs 的 eventLoop 分为下面的几个阶段:

1. timer 阶段
2. I/O 异常回调阶段
3. 空闲、预备状态(第2阶段结束，poll 未触发之前)
4. poll 阶段
5. check 阶段
6. 关闭事件的回调阶段

### setTimeout和setImmediate

#### 情况一

```js
setTimeout(function timeout () {
  console.log('timeout');
},0);
setImmediate(function immediate () {
  console.log('immediate');
});
```

- 对于以上代码来说，setTimeout 可能执行在前，也可能执行在后。

- 首先 setTimeout(fn, 0) === setTimeout(fn, 1)，这是由源码决定的 进入事件循环也是需要成本的，如果在准备时候花费了大于 1ms 的时间，那么在 timer 阶段就会直接执行 setTimeout 回调

- 如果准备时间花费小于 1ms，那么就是 setImmediate 回调先执行了

#### 情况二

在异步i/o callback内部调用时，总是先执行setImmediate，再执行setTimeout

```
const fs = require('fs')
fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('timeout');
    }, 0)
    setImmediate(() => {
        console.log('immediate')
    })
})
// immediate
// timeout
```

### process.nextTick

看到这么一个说法：可以吧process.nextTick理解为一个永远排在微任务队头的微任务。一个宏任务完成，就执行微任务，这时nextTick有队头微任务的特性，所以总是会优先于其他微任务先被执行。