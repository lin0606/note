# Nodejs事件循环

[官网文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

## 什么是事件循环

事件循环允许Node.js执行非阻塞I/O操作，尽管Javascript是单线程的，只要有可能就将操作卸载到系统内核。

由于大多数现代操作系统内核都是多线程的，所以它们可以处理在后台执行的多个操作。当其中一个操作完成时，内核会告诉Node.js，以便适当的回调可以添加到轮询队列（poll queue）中，最终执行。

## 事件循环介绍

当Node.js启动时，它初始化事件循环，处理提供的输入脚本，它可能进行异步API调用，调度计时器，或调用process.nextTick()，然后开始处理事件循环。

下面我们看一下事件循环的各个阶段：

## 事件循环各个阶段

![image (1).png](https://s0.lgstatic.com/i/image6/M00/13/1F/CioPOWBB0_iAYF-EAACboqFVHbQ092.png)

对于上图的每个阶段都有一个要执行的FIFO回调队列。虽然每个阶段都有其独特的方式，但通常情况下，当事件循环进入给定的阶段时，它将执行该阶段特定的任何操作，然后在该阶段的队列中执行回调，直到队列耗尽或执行了最大数量的回调。当队列耗尽或达到回调限制时，事件循环将进入下一阶段，以此类推。

由于这些操作中的任何一个都可能安排更多的操作，并且在轮询阶段处理的新事件将由内核排队，因此轮询事件可以在处理轮询事件时排队。因此，长时间运行的回调会允许轮询阶段运行的时间比计时器的阈值长得多。

从上图可以看到这一流程包含6各阶段，每个阶段代表的含义如下所示:

1. timers: 本阶段执行已经被setTimeout()和setInterval()调度的回调函数，简单理解就是由这两个函数启动的回调函数
2. pending callbacks: 执行推迟到下一次循环迭代的I/O回调，本阶段主要执行某些系统操作(如TCP错误类型)的回调函数
3. idle、prepare: 仅系统内部使用（知道有这个即可）
4. poll: 检索新的I/O事件，执行与I/O相关的回调，Node.js将在适当的时候在此阻塞。这也是最复杂的一个阶段，所有的事件循环以及回调处理都在这个阶段执行。
5. check: setImmediate()回调函数在这个执行，setImmediate并不是立马执行，而是当事件循环poll中没有新的事件处理时就执行该部分
6. close callbacks:  some close callbacks,比如：socket.on('close',...)

对于上述的各个阶段，我们主要关注三个阶段即可：timer, poll, check。下面分别对这三个阶段做一个介绍：

### timer

计时器指定的是执行所提供的回调的阈值，而不是人们希望它执行的确切时间。计时器回调将在经过指定的时间后尽可能早地运行;然而，操作系统调度或其他回调的运行可能会延迟它们。

> Technically, the [**poll** phase](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#poll) controls when timers are executed.

我们来看一个例子：

```javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

这段代码中setTimeout里面最终的打印结果是：

```
105ms have passed since I was scheduled
```

当事件循环进入轮询阶段时，它有一个空队列(fs.readFile()还没有完成)，因此它将等待剩余的ms数，直到最快的计时器达到阈值。当它等待95ms时，fs.readFile()完成读取文件，它的回调(需要10ms完成)被添加到轮询队列并执行。当回调结束时，队列中不再有回调，因此事件循环将看到最快计时器的阈值已经达到，然后回调到计时器阶段来执行计时器的回调。在本例中，您将看到正在调度的计时器和正在执行的回调之间的总延迟是105毫秒。

### poll

poll阶段有两个主要功能：

1. 计算I/O阻塞或轮询的时间
2. 处理轮询队列里的事件

当事件循环进入Poll阶段并且此时没有定时器调度，有以下两种情况会发生：

1. 轮询队列不为空：事件循环将循环遍历同步执行它们的回调队列，直到队列耗尽或达到依赖于系统的硬限制。
2. 轮询队列为空：
   * 如果setImmediate()调度了脚本，则事件循环将结束轮询阶段并继续到检查阶段执行这些调度的脚本。
   * 如果setImmediate()没有调度脚本，则事件循环将等待回调添加到队列中，然后立即执行它们。

一旦轮询队列为空，事件循环将检查达到时间阈值的计时器。如果一个或多个计时器已准备就绪，则事件循环将返回到计时器阶段，以执行那些计时器的回调。

### check

此阶段允许用户在轮询阶段完成后立即执行回调。如果轮询阶段变为空闲，并且脚本已经用setImmediate()排队，事件循环可能会继续到检查阶段而不是等待。

setimmediation()实际上是一个特殊的计时器，它在事件循环的单独阶段中运行。它使用libuv API来安排在轮询阶段完成后执行的回调。

通常，在执行代码时，事件循环最终将进入轮询阶段，在此阶段它将等待传入的连接、请求等。但是，如果用setimmediation()安排了回调，并且轮询阶段变为空闲，则它将结束并继续检查阶段，而不是等待轮询事件。

## setTimeout和setImmediate

上文的Poll阶段有提到过，当轮询队列为空时，如果有定时器到达阙值时且已准备就绪，则事件循环将返回到计时器阶段，以执行那些计时器的回调。但是我们也说过，在轮询队列为空时，事件循环也可能进入到下一个阶段，check,从而执行setImmediate里的回调。

在这一块我们来主要讨论一下setTimeout和setImmediate的执行时机

> `setImmediate()` and `setTimeout()` are similar, but behave in different ways depending on when they are called.

从官方原话我们看出，其两个的行为方式地不同取决于调用它们的时间。

* setImmediate()：用于在当前轮询阶段完成后执行脚本
* setTimeout()： 超过最小阈值(以毫秒为单位)后运行脚本。

计时器的执行顺序将根据调用它们的上下文而变化。如果从主模块中调用这两个函数，则定时将受到进程性能的限制(这可能会受到运行在机器上的其他应用程序的影响)。

例如，如果我们运行以下脚本，它不在一个I/O周期内(即主模块)，两个计时器的执行顺序是不确定的，因为它受进程性能的限制:

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```

```
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

然而, 如果我们在一个I/O周期调用setTimeout和setImmediate, immediate回调总是先执行的:

```javascript
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

```
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

与setTimeout()相比，使用setImmediate()的主要优点是，如果在I/O周期内调度，setImmediate()将始终在任何计时器之前执行，而与存在多少计时器无关。

## process.nextTick()

您可能已经注意到process.nextTick()没有显示在图中，尽管它是异步API的一部分。这是因为**process.nextTick()在技术上不是事件循环的一部分**。相反，nextTickQueue将在当前操作完成后处理，而不管事件循环的当前阶段是什么。在这里，操作被定义为从底层C/C++处理程序的转换，并处理需要执行的JavaScript。回顾我们的图表，在给定阶段的任何时候调用process.nextTick()，传递给process.nextTick()的所有回调都将在事件循环继续**之前**被解析。这可能会造成一些糟糕的情况，因为它允许您通过执行递归process.nextTick()调用来“饿死”您的I/O，从而阻止事件循环到达轮询阶段。

### 为何需要process.nextTick()

这里我们举个栗子:

我们要做的是将一个错误返回给用户，但需在用户代码的其余部分执行之后。

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(callback,
                            new TypeError('argument should be string'));
}
```

这里通过使用process.nextTick()，我们保证apiCall()总是在**用户代码的其余部分之后**和**允许事件循环进行之前**运行它的回调。

为了实现这一点，JS调用堆栈允许被展开，然后立即执行提供的回调，该回调允许用户对process.nextTick()进行递归调用，且不会触发`RangeError: Maximum call stack size exceeded from v8`错误。

## 例子

```javascript
 fs.readFile('./test.txt', { encoding: 'utf-8' }, (err, data) => {
    if (err) throw err;
    console.log('read file success');
    setTimeout(() => {
        console.log('in timeout')
    }, 0)
    
    setImmediate(() => {
        console.log('in immediate')
    })
});

setTimeout(() => {
    console.log('setTimeout');
}, 0);

Promise.resolve().then(() => {
    console.log('Promise callback');
});

process.nextTick(() => {
    console.log('nextTick callback');
});

console.log('end');
```

打印结果

```
start
end
nextTick callback
Promise callback
setTimeout
read file success
in immediate
in timeout
```

1. 输入脚本开始执行，打印start和end
2. 检测到process.nextTick，其总会在微任务之前优先执行，打印nextTick callback
3. 执行微任务，打印Promise callback
4. 此时poll队列为空，检测到达阙值的计时器，打印setTimeout
5. 再次进入到Poll阶段，执行文件读取回调函数，打印read file success
6. 由于在一个I/O周期内同时有setTimeout和setImmediate被调度，所以执行总会先执行setImmediate，最终打印in immediate和 in timeout

**注：**上述打印结果是在node12版本下执行的，而我在node16版本下测试的结果是，具体原因还没查到

```
start
end
Promise callback
nextTick callback
setTimeout
read file success
in immediate
in timeout
```

