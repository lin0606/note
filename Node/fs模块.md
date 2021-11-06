# fs模块介绍

<img src="C:\Users\ASUS\Desktop\note\images\node\fs1.png" style="zoom: 50%;" />

当们打开编辑器的终端敲入fs，你会发现在fs模块下密密麻麻的挂载了很多方法，这些方法大部分都分为同步和异步两个版本。带Sync为同步方法，不带为异步方法。

为了更好的了解node文件模块，我们先了解一下计算机中文件的相关知识，主要包括文件的权限位 mode、标识位 flag、文件描述符 fs。

## 文件相关知识

### 权限位

即我们可以对某个文件行驶的操作权限，文件权限表：

![](http://img.xiaogangzai.cn/modules_fs_01.jpg)

在上面表格中，操作系统主要针对三种用户类型进行权限分配，即文件所有者（自己）、文件所属组（家人）和其他用户（陌生人），文件操作权限又分为三种，读、写和执行，数字表示为八进制数，具备权限的八进制数分别为 `4`、`2`、`1`，不具备权限为 0。

### 标识位

Node.js中，标识位代表着对文件的操作方式，如可读、可写、即可读又可写等。

常用标识位详情表：

| 符号 | 含义                                                     |
| ---- | -------------------------------------------------------- |
| r    | 读取文件，如果文件不存在则抛出异常。                     |
| r+   | 读取并写入文件，如果文件不存在则抛出异常。               |
| rs   | 读取并写入文件，指示操作系统绕开本地文件系统缓存。       |
| w    | 写入文件，文件不存在会被创建，存在则清空后写入。         |
| w+   | 读取并写入文件，文件不存在则创建文件，存在则清空后写入。 |
| a    | 追加写入，文件不存在则创建文件                           |
| a+   | 读取并追加写入，不存在则创建。                           |

这里我们只要理解了 read write add 这些模式的意义，等到具体使用的时候，根据场景来设置这个 flag 就可以了。

### 描述符

> 操作系统会为每个打开的文件分配一个名为文件描述符的数值标识，文件操作使用这些文件描述符来识别与追踪每个特定的文件。

在 Node.js 中，每操作一个文件，文件描述符是递增的，文件描述符一般从 3 开始，因为前面有 0、1、2 三个比较特殊的描述符，分别代表 `process.stdin`（标准输入）、`process.stdout`（标准输出）和 `process.stderr`（错误输出）

## 文件操作

```javascript
import fs from 'fs'
import path from 'path'
const __dirname = path.resolve()
const filePath = path.join(__dirname, 'a.txt')
const filePath1 = path.join(__dirname, 'b.txt')
```

### 文件读取

```javascript
fs.readFile(filename,[encoding],[callback(error,data)]
```

总共有三个参数分别为：读取的文件名，文件字符编码，接收文件内容的回调函数

```javascript
fs.readFile(filePath, 'utf8', function(err, data) {
    console.log(data);
})
```

### 文件写入

```javascript
fs.writeFile(filename, data, [options], callback)
```

总共有四个参数分别为：文件名，写入内容，配置项，文件写入后的回调函数

对于options其是一个对象，里面主要有三个属性：

```
encoding {String | null} default='utf-8'
mode {Number} default=438(可读可写不可执行)
flag {String} default='w' 
```

```javascript
fs.writeFile(filePath, 'write success', function(err) {
    if (err) {
        throw err;
    }
    // 写入完成后读取数据做测试
    var data = fs.readFileSync(filePath, 'utf-8');
    console.log('new data -->', data);
})
```

### 文件追加

```
fs.appendFile(filename, data, [options], callback)
```

其参数和文件写入方法的参数一致，主要区别在于其options中flag的默认值为'a'

```javascript
fs.appendFile(filePath, '追加的新数据', function(err) {
    if (err) {
        throw err;
    }
    var data = fs.readFileSync(filePath, 'utf-8')
    console.log(data)
})
```

### 拷贝文件

```javascript
fs.copyFile(filenameA, filenameB，callback)
```

参数一为原始文件，参数二为目标文件

### 删除文件

```javascript
fs.unlink(filename, callback)
```

## 高级操作

对于大文件的操作，由于我们不能一次性的读取文件内容到缓存中，所以需要分块读取和写入，这时候便需要使用一些更为高阶的操作了：

一般我们要先用`fs.open`来打开文件，然后才可以用`fs.read`去读，或者用`fs.write`去写文件，最后，我们需要用`fs.close`去关掉文件。

### 文件打开

```javascript
fs.open(filePath, flag, [mode], function(err, fd) {
  console.log(fd)
}) 
```

参数一位文件路径，参数二为标识符 flag， 参数三为[mode] 是文件的权限（可选参数，默认值是 0666即可读可写不可执行）， 参数四为callback 回调函数

### 文件读取

```javascript
fs.read(fd, buffer, offset, length, position, callback)
```

六个参数

1. fd：文件描述符，通过 fs.open 拿到；
2. buffer：一个 Buffer 对象，`v8`引擎分配的一段内存，将读取到的内容写入到的 Buffer；
3. offset：整数，向 Buffer 缓存区写入的初始位置，以字节为单位；
4. length：整数，读取文件的长度；
5. position：整数，读取文件初始位置；文件大小以字节为单位
6. callback：回调函数，有三个参数 err（错误），bytesRead（实际读取的字节数），buffer（被写入的缓存区对象），读取执行完成后执行。

```javascript
fs.open('fileName', 'r', function (err, fd) {
  var buf = new Buffer(1024)
  var offset = 0
  var len = buf.length
  var pos = 101
  // 这里我定义了参数，文件打开后，会从第 100 个字节开始，读取其后的 1024 个字节的数据。读取完成后，回调方法中可以处理读取到的的缓冲的数据了
  fs.read(fd, buf, offset, len, pos, function(err, bytes, buffer) {
    console.log('读取了' + bytes + ' bytes')
    //数据已被填充到 buf 中
    console.log(buf.slice(0, bytes).toString())
  })
})
```

### 文件写入

```javascript
fs.write(fd, buffer, offset, length, position, callback)
```

六个参数

1. fd：文件描述符，使用`fs.open` 打开成功后返回的；
2. buffer：一个 Buffer 对象，`v8` 引擎分配的一段内存，存储将要写入文件的buffer数据；
3. offset：整数，从 Buffer 缓存区读取数据的初始位置，以字节为单位；
4. length：整数，读取 Buffer 数据的字节数；
5. position：整数，写入文件初始位置；
6. callback：写入操作执行完成后回调函数，有三个参数 err（错误），bytesWritten（实际写入的字节数），buffer（被读取的缓存区对象），写入完成后执行。

```javascript
fs.open('fileName1', 'a', function (err, fd) {
  var buf = new Buffer('I Love Juejin')
  var offset = 0
  var len = buf.length
  var pos = 100

  fs.write(fd, buf, offset, len, pos, function(err, bytes, buffer) {
    console.log('写入了 ' + bytes + ' bytes')
    //数据已被填充到 buf 中
    console.log(buf.slice(0, bytes).toString())
    fs.close(fd, function(err) {})
  })
})
```

write还有第二种用法，就是不直接写入 buffer 数据，而是写入字符串

```javascript
fs.open('fileName1', 'a', function (err, fd) {
  var data = 'I Love Juejin'
  // 第一个参数依然是文件描述符，第二个是写入的字符串，第三个是写入文件的位置，第四个是编码格式，最后一个是回调函数，回调函数第一个参数是异常，第二个是 指定多少字符数将被写入到文件，最后一个是返回的字符串
  fs.write(fd, data, 0, 'utf-8', function(err, written, string) {
    console.log(written)
    console.log(string)

    fs.close(fd, function(err) {})
  })
})
```

### 文件关闭

```javascript
fs.close(fd, callback)
```

第一个参数：fd 文件`open`时传递的`文件描述符`

第二个参数 callback 回调函数,回调函数有一个参数 err（错误），关闭文件后执行。

### 大文件拷贝

```javascript
import fs from "fs";
// buffer 的长度
const BUFFER_SIZE = 3;
// copy 方法
function copy(src, dest, size = 16 * 1024, callback) {
    // 打开源文件
    fs.open(src, 'r', (err, readFd) => {
        // 打开目标文件
        fs.open(dest, 'w', (err, writeFd) => {
            let buf = Buffer.alloc(size);
            let readed = 0; // 下次读取文件的位置
            let writed = 0; // 下次写入文件的位置
            (function next() {
                // 读取
                fs.read(readFd, buf, 0, size, readed, (err, bytesRead) => {
                    readed += bytesRead;
                    // 如果都不到内容关闭文件
                    if (!bytesRead) fs.close(readFd, err => console.log('关闭源文件'));
                    // 写入
                    fs.write(writeFd, buf, 0, bytesRead, writed, (err, bytesWritten) => {
                        // 如果没有内容了同步缓存，并关闭文件后执行回调
                        if (!bytesWritten) {
                            fs.fsync(writeFd, err => {
                                fs.close(writeFd, err => !err && callback());
                            });
                            return;
                        }
                        writed += bytesWritten;
                        // 继续读取、写入
                        next();
                    });
                });
            })();
        });
    });
}
// 拷贝文件内容并写入
copy('a.txt', '7.txt', BUFFER_SIZE, () => {
    fs.readFile('7.txt', 'utf8', (err, data) => {
        // 拷贝完读取 7.txt 的内容
        console.log(data); // 你好
    });
});
```

## 文件夹操作

1. fs.mkdir 创建目录

2. fs.rmdir 删除目录

3. fs.readdir 读取目录

## 参考

[程序员成长指北](http://www.inode.club/node/module_fs.html#%E6%96%87%E7%AB%A0%E6%A6%82%E8%A7%88)