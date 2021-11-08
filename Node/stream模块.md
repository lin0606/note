# Stream（流）介绍

在[buffer](https://github.com/lin0606/note/blob/main/Node/buffer%E6%A8%A1%E5%9D%97.md)模块中我们也提到过流的概念。如果我们将buffer比作公交车的话，stream就好比我们人，而我们的公路就好似运输的管道(pipe)。

## 流的类别

流里面也有不同的分工，在Node.js中主要分为以下四种：

* Readable Stream(可读流)：用来提供数据，外部来源的数据会被存储到内部Buffer数组内缓存起来。
* Writable Stream(可写流)：消费数据，从可读流里拿到buffer数据进行处理消耗，将其写入目标对象
* Duplex Streams(双工流)：即是Readable也是Writable，如Tcp socket
* Transform Streams(转换流)：也是双工流，主要负责处理和加工流经它的数据。

## 使用流实现mp3拷贝

我们可以想一想用文件的方式实现copy该怎么做?

```javascript
const source = fs.readFileSync('music.mp3')
fs.writeFileSync('music_copy.mp3', source)
```

当然也可以用fs.copyFile来实现，只不过这样太过简单粗暴。我们要知道数据在流动过程中，应该有精确的传输状态的，我们可以使用Stream去监听状态的变化，从而实现更精细的控制。

**注：**Stream是EventEmitter的实例

我们来用Stream实现一下音乐文件拷贝：

```javascript
import fs from 'fs'
import path from 'path'
const __dirname = path.resolve()
const pathName1 = path.join(__dirname, 'music.mp3')
const pathName2 = path.join(__dirname, 'music_copy.mp3')
const rs = fs.createReadStream(pathName1)
const ws = fs.createWriteStream(pathName2)

let n = 0
rs.on('data', (chunk) => {
    n++
    console.log(chunk.byteLength)
    // 缓冲数据如果没被写入，则暂停
    if (ws.write(chunk) === false) {
        rs.pause()
        console.log('暂停获取...')
    }
}).on('end', () => {
    console.log(`传输结束，共收到${n}个Buffer块`)
    ws.end()
}).on('close', () => {
    console.log('传输关闭')
}).on('error', (e) => {
    console.log('传输出错' + e)
})

ws.on('drain', () => {
    console.log('数据被消耗了，继续启动读数据')
    rs.resume()
})
```

打印结果类似这样：

```
65536
暂停获取...
数据被消耗了，继续启动读数据
65536
暂停获取...
数据被消耗了，继续启动读数据
38815
暂停获取...
数据被消耗了，继续启动读数据
传输结束，共收到73个Buffer块
传输关闭
```

这里，我们一共读取了73块缓冲，我们每次都是读取64kb除了最后一次，我们在数据还没被写入时暂停读取，在数据被消耗完后恢复读取，这样我们就可以对数据传输时的流速加以控制。

## 数据管道PIPE

我们再来看一下使用pipe来进行音乐拷贝

```javascript
fs.createReadStream('music.mp3').pipe(fs.createWriteStream('music.mp3'))
```

一行代码搞定。这里pipe的左边和右边都是流，左边读出的数据经过pipe输送给目标流，目标流可以进行处理，继续向下不断的pipe，从而形成一个pipe链条。

pipe还有一个特点便是，pipe会自动控制数据的读取速度，来帮助数据以一种比较合理的速度，源源不断的输送给目的地。

## 实现MP4转MP3

```javascript
import fs from 'fs'
import http from 'http'
import request from 'request'
import ChildProcess from 'child_process'
import EventEmitter from 'events'
import path from 'path'

const __dirname = path.resolve()
const spawn = ChildProcess.spawn
const mp3Args = ['-i', 'pipe:0', '-f', 'mp3', '-ac', '2', '-ab', '128k', '-acodec', 'libmp3lame', 'pipe:1']
const mp4Args = ['-i', 'pipe:0', '-c', 'copy', '-bsf:a', 'aac_adtstoasc', 'pipe:1']

class VideoTool extends EventEmitter {
    constructor(url, filename) {
        super()
        this.url = url
        this.filename = filename
    }

    mp3() {
        // 创建 FFMPEG 进程
        this.ffmpeg = spawn('ffmpeg', mp3Args)

        // 拿到 Stream 流
        http.get(this.url, (res) => {
            res.pipe(this.ffmpeg.stdin)
        })

        // 把拿到的流 pipe 到文件中
        this.ffmpeg.stdout.pipe(fs.createWriteStream(this.filename))

        this.ffmpeg.on('exit', () => {
            console.log('Finished:', this.filename)
        })
    }

    mp4() {
        let stream = fs.createWriteStream(this.filename)
        request
            .get(this.url, {
                headers: {
                    'Content-Type': 'video/mpeg4',
                    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36'
                }
            })
            .pipe(stream)
            .on('open', () => {
                console.log('start download')
            })
            .on('close', () => {
                console.log('download finished')
            })
    }
}

const video = 'http://vt1.doubanio.com/201810291353/4d7bcf6af730df6d9b4da321aa6d7faa/view/movie/M/402380210.mp4'
const m1 = new VideoTool(video, 'audio.mp3')
const m2 = new VideoTool(video, __dirname + '/video.mp4')

m1.mp3()
m2.mp4()
```

