## webpack基础

[基础配置代码](https://github.com/funny741643/webpack-learn/tree/main/webpack-base)

[webpack参考阅读](https://juejin.cn/post/7023242274876162084)

### 环境区分

对于本地开发与线上部署，我们肯定有不同的代码打包需求：

**本地环境**，我们需要构建速度更快，可以打印debug信息，hot reload, sourcemap进行问题定位等。

**生产环境**，我们需要更小的包体积，会进行代码压缩，tree-shaking，代码分割，图片压缩等。

我们可以使用cross-env做环境区分，对不同环境做不同的配置：

1. 安装

   ```
   npm install cross-env -D
   ```

2. 配置启动命令

   ```json
   "scripts": {
       "dev": "cross-env NODE_ENV=dev webpack serve --mode development", 
       "test": "cross-env NODE_ENV=test webpack --mode production",
       "build": "cross-env NODE_ENV=prod webpack --mode production"
   },
   ```

3. 在webpack配置文件中获取环境变量

   ```javascript
   // 打印环境变量
   console.log('process.env.NODE_ENV=', process.env.NODE_ENV)
   const config = {}
   module.exports = (env, argv) => {
       // 打印 mode(模式) 值
     	console.log('argv.mode=',argv.mode)
     	// 这里可以通过不同的模式修改 config 配置
     	return config;
   }
   ```

### loader与plugin

#### loader

什么是 loader？它是一个转换器，用于对源代码进行转换。例如使用 `babel-loader` 可以将 ES6 代码转换为 ES5 代码；`sass-loader` 将 sass 代码转换为 css 代码。

其实 loader 并不复杂，很容易就能写一个 loader。下面就是一个简单的 loader，它的作用是将代码中的 `var` 关键词替换为 `const`：

```javascript
module.exports = function (source) {
    return source.replace(/var/g, 'const') // var a = 1; 将被转换为 const a = 1;
}
```

#### plugin

webpack 在整个编译周期中会触发很多不同的事件，plugin 可以监听这些事件，并且可以调用 webpack 的 API 对输出资源进行处理。

这是它和 loader 的不同之处，loader 一般只能对源文件代码进行转换，而 plugin 可以做得更多。plugin 在整个编译周期中都可以被调用，只要监听事件。

例如 `html-webpack-plugin` 插件在编译完成后自动的将资源 URL 插入到 html 文件。

### sourceMap配置差异

| devtool                      | build | rebuild       | 显示代码 | SourceMap 文件 | 描述         |
| ---------------------------- | ----- | ------------- | -------- | -------------- | ------------ |
| (none)                       | 很快  | 很快          | 无       | 无             | 无法定位错误 |
| eval                         | 快    | 很快（cache） | 编译后   | 无             | 定位到文件   |
| source-map                   | 很慢  | 很慢          | 源代码   | 有             | 定位到行列   |
| eval-source-map              | 很慢  | 一般（cache） | 编译后   | 有（dataUrl）  | 定位到行列   |
| eval-cheap-source-map        | 一般  | 快（cache）   | 编译后   | 有（dataUrl）  | 定位到行     |
| eval-cheap-module-source-map | 慢    | 快（cache）   | 源代码   | 有（dataUrl）  | 定位到行     |
| inline-source-map            | 很慢  | 很慢          | 源代码   | 有（dataUrl）  | 定位到行列   |
| hidden-source-map            | 很慢  | 很慢          | 源代码   | 有             | 无法定位错误 |
| nosource-source-map          | 很慢  | 很慢          | 源代码   | 无             | 定位到文件   |

对照校验规则`^(inline-|hidden-|eval-)?(cheap-(module-)?)?source-map$`分析关键字：

| __关键字__ | __描述__                                            |
| ---------- | --------------------------------------------------- |
| inline     | 代码内通过dataUrl形式引入SourceMap                  |
| hidden     | 生成SourceMap文件，但不使用                         |
| eval       | eval(...)形式执行代码，通过dataUrl形式引入SourceMap |
| nosources  | 不生成SourceMap                                     |
| cheap      | 只需要定位到行信息，不需要列信息                    |
| module     | 展示源代码中的错误位置                              |

### 文件相关

#### 资源模块

webpack5 新增资源模块(asset module)，允许使用资源文件（字体，图标等）而无需配置额外的 loader。

资源模块支持以下四个配置：

1. `asset/resource` 将资源分割为单独的文件，并导出 url，类似之前的 file-loader 的功能
2. `asset/inline` 将资源导出为 dataUrl 的形式，类似之前的 url-loader 的小于 limit 参数时功能
3. `asset/source` 将资源导出为源码（source code）. 类似的 raw-loader 功能
4. `asset` 会根据文件大小来选择使用哪种类型，当文件小于 8 KB（默认） 的时候会使用 asset/inline，否则会使用 asset/resource

#### 文件hash值

| __占位符__  | __解释__                   |
| ----------- | -------------------------- |
| ext         | 文件后缀名                 |
| name        | 文件名                     |
| path        | 文件相对路径               |
| folder      | 文件所在文件夹             |
| hash        | 每次构建生成的唯一 hash 值 |
| chunkhash   | 根据 chunk 生成 hash 值    |
| contenthash | 根据文件内容生成hash 值    |

对于三种hash的区别：

* hash：任何一个文件改动，整个项目的构建hash值都会改变
* chunkhash：文件的改动只会影响其他所在chunk的hash值
* contenthash：每个文件都有单独的hash值，文件的改动只会影响自身的hash值。



