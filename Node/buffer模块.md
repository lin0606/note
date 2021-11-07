# Buffer模块介绍

<img src="C:\Users\ASUS\Desktop\note\images\node\buffer1.png" style="zoom:60%;" />

## 什么是二进制

我们知道在计算机的世界里只有0和1，无论是文字，图片，视频都不例外，而我们在网上的交流其实也就是0,1的交换。

二进制数据是使用0和1两个数码来表示的数据，为了存储或展示一些数据，计算机需要将这些数据转换为二进制来表示。例如我们要存储66这个数字，计算机会先将其转换为01000010,，再进行存储。我们看一下转换规则：

| 128  | 64   | 32   | 16   | 8    | 4    | 2    | 1    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0    | 1    | 0    | 0    | 0    | 0    | 1    | 0    |

再比如我们要对M操作，`javascript`里通过`'M'.charCodeAt()`取到对应的ASCII码后再转为二进制表示

## Buffer出现的意义

Node.js用Buffer来处理二进制流数据或与之进行交互。

Buffer用于读取或者操作二进制数据流，做为Node.js API的一部分使用时无需require,其用于操作网络协议，数据库，图片和文件I/O等一些需要大量二进制数据的场景。

## Stream简单介绍

上文说到，Buffer用于操作二进制数据流，这里的流指的是什么？什么是流？

流，英文Stream是对输入输出设备的抽象，这里的设备可以是文件，网络，内存等。

比如当我们打开一个网络视频时，由于视频文件过于庞大，我们的计算机不可以一次性的将视频数据全部载入到内存，然后供我们观看，先不说电脑带宽，光服务器内存可能都吃不消。计算机所采用的策略为，我们可以不断的把数据拿出来放到管道里面，然后让其像水一样不间断的流走，直到数据全部抵达目的地。像这样的机制，在计算机里，就是通过Stream(流)来实现数据的动态读写，再通过管道，让流即可以被生产也可以被消费。

## 什么是Buffer

上面说了那么多，我们再来看看什么是Buffer。通过对流的讲解我们知道数据是从一端流向另一端，那么他们是如何流动的？

伴随着时间的推移，每一个过程都会有一个最小或最大数据量。如果数据到达的速度比进程消耗的速度快，那么少数早到达的数据会处于等待区等候被处理。反之，如果数据到达的速度比进程消耗的数据慢，那么早先到达的数据需要等待一定量的数据到达之后才能被处理。

这里的等待区就指的缓冲区（Buffer），它是计算机中的一个小物理单位，通常位于计算机的 RAM 中。

## Buffer的使用

### 创建Buffer

**Buffer.from()**

```
Buffer.from(array)
BUffer.from(string,[encoding])
Buffer.from(buffer)
```

这里说一下`Buffer.from(array)`。下面是官方文档对API的说明，也就是说，每个array的元素对应1个字节（8位），取值从0到255。

> Allocates a new Buffer using an array of octets.

```javascript
var buff = Buffer.from([62])
// <Buffer 3e>
// buff[0] === parseInt('3e', 16) === 62
```

```javascript
var buff = Buffer.from([062])
// <Buffer 32>
// buff[0] === parseInt(62, 8) === parseInt(32, 16) === 50
```

```javascript
var buff = Buffer.from([0x62])
// <Buffer 62>
// buff[0] === parseInt(62, 16) === 98
```

当数组元素超出1个字节

```javascript
var buff = Buffer.from([256])
// <Buffer 00>
```

**Buffer.alloc**

返回一个已初始化的Buffer,可以保证新创建的Buffer永远不会包含旧数据

```javascript
const bAlloc1 = Buffer.alloc(10); // 创建一个大小为 10 个字节的缓冲区

console.log(bAlloc1); // <Buffer 00 00 00 00 00 00 00 00 00 00>
```

**Buffer.allocUnsafe**

创建一个大小为 size 字节的新的未初始化的 Buffer，由于 Buffer 是未初始化的，因此分配的内存片段可能包含敏感的旧数据。在 Buffer 内容可读情况下，则可能会泄露它的旧数据，这个是不安全的，使用时要谨慎。

```javascript
const bAllocUnsafe1 = Buffer.allocUnsafe(10);

console.log(bAllocUnsafe1); // <Buffer 49 ae c9 cd 49 1d 00 00 11 4f>
```

### Buffer 字符编码

通过使用字符编码，可实现 Buffer 实例与 JavaScript 字符串之间的相互转换，目前所支持的字符编码如下所示：

- 'ascii' - 仅适用于 7 位 ASCII 数据。此编码速度很快，如果设置则会剥离高位。
- 'utf8' - 多字节编码的 Unicode 字符。许多网页和其他文档格式都使用 UTF-8。
- 'utf16le' - 2 或 4 个字节，小端序编码的 Unicode 字符。支持代理对（U+10000 至 U+10FFFF）。
- 'ucs2' - 'utf16le' 的别名。
- 'base64' - Base64 编码。当从字符串创建 Buffer 时，此编码也会正确地接受 RFC 4648 第 5 节中指定的 “URL 和文件名安全字母”。
- 'latin1' - 一种将 Buffer 编码成单字节编码字符串的方法（由 RFC 1345 中的 IANA 定义，第 63 页，作为 Latin-1 的补充块和 C0/C1 控制码）。
- 'binary' - 'latin1' 的别名。
- 'hex' - 将每个字节编码成两个十六进制的字符。

```javascript
const buf = Buffer.from('hello world', 'ascii');
console.log(buf.toString('hex')); // 68656c6c6f20776f726c64
```

### 其他方法

* buffer比较：`Buffer.equals()`，`Buffer.compare()`
* buffer连接：`Buffer.concat()`
* buffer拷贝：`Buffer.copy()`
* buffer查找：`Buffer.indexOf()`
* buffer写入：`Buffer.write()`
* buffer填充：`Buffer.fill()`
* 转为`json`:      `Buffer.toJson()`
* 遍历：           `Buffer.values()`、`Buffer.keys()`、`Buffer.entries()`
* 截取：           `Buffer.slice()`

## Buffer的内存分配

由于 Buffer 需要处理的是大量的二进制数据，假如用一点就向系统去申请，则会造成频繁的向系统申请内存调用，所以 Buffer 所占用的内存**不再由 V8 分配**，而是在 Node.js 的 **C++ 层面完成申请**，在 **JavaScript 中进行内存分配**。因此，这部分内存我们称之为**堆外内存**。

1. 在初次加载时就会初始化 1 个 **8KB 的内存空间**，buffer.js 源码有体现
2. 根据申请的内存大小分为 **小 Buffer 对象** 和 **大 Buffer 对象**
3. 小 Buffer 情况，会继续判断这个 slab 空间是否足够
   - 如果空间足够就去使用剩余空间同时更新 slab 分配状态，偏移量会增加
   - 如果空间不足，slab 空间不足，就会去创建一个新的 slab 空间用来分配
4. 大 Buffer 情况，则会直接走 createUnsafeBuffer(size) 函数
5. 不论是小 Buffer 对象还是大 Buffer 对象，内存分配是在 C++ 层面完成，内存管理在 JavaScript 层面，最终还是可以被 V8 的垃圾回收标记所回收。