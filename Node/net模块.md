# net模块介绍

## 概念介绍

`net` 模块提供了异步的网络 API，用于创建基于流的 TCP 或 [IPC](http://nodejs.cn/api/net.html#ipc-support) 服务器 ([`net.createServer()`](http://nodejs.cn/api/net.html#netcreateserveroptions-connectionlistener)) 和客户端 ([`net.createConnection()`](http://nodejs.cn/api/net.html#netcreateconnection))。

这里IPC指的是进程间通信（Inter-Process Communication）

## 开启第一个TCP服务

建议阅读：[TCP协议介绍](https://github.com/lin0606/note/blob/main/Network/TCP.md)

nodejs中，我们可以使用net模块，很容易的建立一个tcp服务。

```javascript
import net from 'net'

const HOST = '127.0.0.1'
const PORT = 3000
const server = net.createServer()

server.listen(PORT, HOST)

server.on('listening', () => {
    console.log(`服务已开启于${HOST}:${PORT}`)
})
```

## net模块类

[官方文档](http://nodejs.cn/api/net.html#identifying-paths-for-ipc-connections)

net模块下有多个类。

* BlockList类：用于禁用对特定IP地址、IP范围或者子网的入站或出站访问的规则。
* Server类：用于创建TCP或者IPC服务
* Socket类：此类是TCP套接字或流式IPC端点的抽象。可以**由用户创建并直接与服务器交互**

下面主要针对Server类和Socket类进行简单总结：

## TCP事件

### TCP服务事件

* close事件：服务器关闭时触发
* connect事件：建立新连接时触发
* error事件：发生错误时触发
* listening事件：调用`server.listen()`后绑定服务器时触发

### TCP套接字连接事件

* close事件：一旦套接字完全关闭就触发
* connect事件：当成功建立套接字连接时触发
* data事件：接收到数据时触发
* drain事件：当写缓冲区变空时触发。可用于限制上传
* end事件：当套接字的另一端表示传输结束时触发，从而结束套接字的可读端。
* error事件：发送错误时触发，close事件将在其之后触发
* lookup事件：在解析主机名之后但在连接之前触发。不适用Unix套接字
* ready事件：当套接字准备好使用时触发。`connect`后立即触发。
* timeout事件：如果套接字因不活动而超时则触发。

## 示例代码

[参考源码地址](https://github.com/qufei1993/Nodejs-Roadmap/blob/master/docs/nodejs/net.md)

**服务端实现**

```javascript
import net from 'net'

const HOST = '127.0.0.1'
const PORT = 3000;
const server = net.createServer()

server.listen(PORT, HOST)

server.on('listening', () => {
    console.log(`服务已开启 ${HOST}:${PORT}`);
});

server.on('connection', socket => {
    socket.setNoDelay(true)
    socket.on('data', buffer => {
        const msg = buffer.toString()
        console.log(msg)

        socket.write(Buffer.from('你好 ' + msg))
    })
})

server.on('close', () => {
    console.log('Server Close')
})

server.on('error', err => {
    if (err.code === 'EADDRINUSE') {
        console.log('地址正被使用，重试中...');

        setTimeout(() => {
            server.close();
            server.listen(PORT, HOST);
        }, 1000);
    } else {
        console.error('服务器异常：', err);
    }
});
```

**客户端实现**

```javascript
import net from 'net'

const client = net.createConnection({
    host: '127.0.0.1',
    port: 3000
});

client.on('connect', () => {
    // 向服务器发送数据
    client.write('Nodejs 技术栈');

    setTimeout(() => {
        client.write('JavaScript ');
        client.write('TypeScript ');
        client.write('Python ');
        client.write('Java ');
        client.write('C ');
        client.write('PHP ');
        client.write('ASP.NET ');
    }, 1000);
})

client.on('data', buffer => {
    console.log(buffer.toString());
});

// 例如监听一个未开启的端口就会报 ECONNREFUSED 错误
client.on('error', err => {
    console.error('服务器异常：', err);
});

client.on('close', err => {
    console.log('客户端链接断开！', err);
});
```

先运行server，后运行client，我们期望的打印结果如下：

```
Nodejs 技术栈
JavaScript
TypeScript
Python
Java
C
PHP
ASP.NET
```

而最终服务端打印的结果如下：

```
Nodejs 技术栈
JavaScript
TypeScript Python Java C PHP ASP.NET
```

出现这种现在的原因主要是TCP粘包问题。

## TCP粘包问题

