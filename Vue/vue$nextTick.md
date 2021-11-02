# Vue3 nextTick介绍

## 官网介绍

将回调推迟到下一个 DOM 更新周期之后执行。在更改了一些数据以等待 DOM 更新后立即使用它。

```javascript
import { createApp, nextTick } from 'vue'

const app = createApp({
  setup() {
    const message = ref('Hello!')
    const changeMessage = async newMessage => {
      message.value = newMessage
      await nextTick()
      console.log('Now DOM is updated')
    }
  }
})
```

## 什么是事件循环（EventLoop）

这块可以去看一下我的另一篇文章[浏览器的事件循环](https://github.com/lin0606/note/blob/main/%E5%89%8D%E7%AB%AF%E5%9F%BA%E7%A1%80/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF.md)

## 原理解析

对于我个人理解，为什么我们所写的回调会在dom更新周期之后执行？是由于这个过程发生了两次nextTick(),

1. 第一次是在我们进行数据改变的时候，vue会将改变数据的所有依赖函数放入一个schedulerQueue队列中，并用一个函数封装这个队列，用于执行；最后将这个函数推送到cb队列中（这里我们可以理解为微任务队列）
2. 第二次便是我们自己手动所写的，这个函数也会进入微任务队列中进行执行，但其永远都在第一次nextTick之后，这是依赖已经触发，新的Dom节点已经被渲染了出来。

## Vue2源码分析

```javascript
update () {
    if (this.lazy) {
        this.dirty = true
    } else if (this.sync) {
        /*同步则执行run直接渲染视图*/
        this.run()
    } else {
        /*主要看这里：异步推送到观察者队列中，下一个tick时调用。*/
        queueWatcher(this)
    }
}
```

首先看一下update函数，其是调度者接口，当我们的依赖数据发生改变时进行回调，我们主要明白其调用了queueWatcher函数，下面我们再看一看queueWatcher函数做了什么？

```javascript
export function queueWatcher (watcher: Watcher) {
  // 获取watcher的id
  const id = watcher.id
  // 检验id是否存在，已经存在则直接跳过，不存在则标记哈希表has，用于下次检验
  if (has[id] == null) {
    has[id] = true
    // 队列不是在刷新时，则将watcher进行push, 否则进行拼接
    if (!flushing) {
      queue.push(watcher)
    } else {
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    
    // 使用nextTick进行调度，传入清空watcher队列的函数flushSchedulerQueue
    if (!waiting) {
      waiting = true
      nextTick(flushSchedulerQueue)
    }
  }
}
```

从上面代码我们可以看出，其主要作用为将一个观察者对象push进观察者队列，在队列中已经存在相同的id则该观察者对象将被跳过。最后调度nextTick函数，并为其传入清空watcher队列的函数。

下面我们在来看看nextTick的核心代码：

```javascript
const callbacks = []
let pending = false

function flushCallbacks() {
    pending = false
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
        copies[i]()
    }
}

// 用于清空任务队列的函数
let timerFunc
timerFunc = () => {
    p.then(flushCallbacks)
}

// nextTick 主函数, 会将cb放入callbacks任务队列中
export function nextTick(cb?: Function, ctx?: Object) {
    callbacks.push(() => {
        cb.call(ctx)
    })
    if (!pending) {
        pending = true
        timerFunc()
    }
}
```

首先nextTick会将他的cb参数进行入队即callbacks, 后来调用timerFunc调用flushCallbacks清空执行callbacks队列。注意flushCallbacks是使用promise.then执行的，就是说其会放入微任务队列中，如果我们再次调用nextTick，会将里面的函数再进行微任务入队，知道没有微任务，才进行下一次循环。

最后我们来看一张图，整体了解一下nextTick的执行过程：

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/11/19/16e83067c347ff13~tplv-t2oaga2asx-watermark.awebp" alt="img" style="zoom:50%;" />