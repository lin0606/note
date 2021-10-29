# 什么是NodeJs

## 概念

1. Node.js是Javascript的运行时环境。
2. Node.js是构建在V8引擎之上的，这也就是Node.js可以运行javascript的原因。
3. Node.js是轻量且高效的，每个函数都是同步的，而I/O操作时异步的。所有由Javascript编写的函数的I/O操作最终都将由libuv事件循环处理库来处理，隐藏了非阻塞I/O的具体细节，简化了并发编程模型。

## Node.js架构图

![](C:\Users\ASUS\Desktop\note\images\node\node1.png)

上图中，Chrome V8引擎解释并执行Javascript代码（这就是浏览器能执行Javascript的原因），libuv由事件循环和线程池组成，负责所有I/O任务的分发与执行。

在解决并发问题，Node.js主要采取了异步的方案，由事件循环来接受处理，而执行任务则由线程池中的具体线程进行执行。

我们之所以说Node.js是单线程的，是因为它在接受任务的时候是单线程的，无须切换进程/线程，非常高效，但它执行具体任务的时候是多线程的。

Node.js Bindings层做的事就是将Chrome V8引擎暴露的C/C++接口转换成Javascript API，并且结合这些API编写Node.js标准库，所有这些API被统称为Node.js SDK。

## Node.js是单线程的吗？

其实只有Js执行线程是单线程，I/O执行是在其他线程之上的。

nodeJs会从Js代码通过node-bindings调用到C/C++代码，然后通过C/C++代码封装一个叫“请求对象”的东西给libuv，这个请求对象里面无非就是需要执行的功能+回调之类的东西，给libuv执行以及执行完实现回调。

总结，一个异步I/O的大致流程如下：

1. 发起I/O调用

用户通过Javascript代码调用Node核心模块，将参数和回调函数传入到核心模块；

Node核心模块会将传入的参数和回调函数封装成一个请求对象；

将这个请求对象推入到I/O线程池等待执行

Javascript**发起**的异步调用结束，Javascript线程继续执行后续操作。

2. 执行回调

I/O操作完成后，会取出之前封装在请求对象中的回调函数，执行这个回调函数，以完成Javascript回调的目的。

从这里，我们可以看到，我们其实对Node.js的单线程一直有误会。事实上，它的单线程指的是自身Javascript运行环境的单线程，Node.js并没有给Javascript执行时创建新线程的能力，最终的实际操作，还是通过libuv以及事件循环来执行的。这就是为什么Javascript一个单线程语言，能在Node.js里面实现异步操作的原因，两者并不冲突。

## Node.js中的并发

上文提到了Node.js的线程池，其默认大小为4，也就是说。同时能有4个线程去做文件i/o的工作，剩下的请求会被挂起等待直到线程池有空闲。所以nodejs对于并发数，是有限制的。

线程池的大小可以通过 UV_THREADPOOL_SIZE 这个环境变量来改变 或者在nodejs代码中通过 process.env.UV_THREADPOOL_SIZE来重新设置。