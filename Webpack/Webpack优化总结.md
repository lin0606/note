webpack的优化基本分为对构建过程的优化，以及对构建结果的优化

## 构建过程优化

### 优化resolve配置

1. alias：创建import或require的别名，用来简化模块引用
2. extensions：设置模块引用扩展名
3. modules：告诉webpack解析模块时应该搜索的目录

```javascript
const config = {
    resolve: {
        alias: {
            "@": resolve("src"),
            "~": resolve("src"),
            components: resolve("src/components"),
        },
        extensions: ["ts", "js", "..."],
        modules: [resolve("src"), "node_modules", resolve("loader")],
    },
}
```

### 范围缩小

这里指缩小loader的作用文件的范围，我们可以设置include和exclude这两个配置项

* include：符合条件的模块进行解析
* exclude：排除符合条件的模块，不解析

其中exclude的优先级更高，在我们项目搭建中更推荐使用include

```javascript
const config = {
  	module: { 
        noParse: /jquery|lodash/,
        rules: [
            {
                test: /\.js$/i,
                include: resolve('src'),
                exclude: /node_modules/,
                use: [
                    'babel-loader',
                ]
            },
        ]
      }
};
```

### 利用缓存

1. 开启babel缓存

   ```javascript
   {
   	loader: 'babel-loader',
       options: {
       	cacheDirectory: true // 启用缓存
       }
   },
   ```

2. 开启其他loader缓存

   我们可以使用cache-loader，

   其会缓存性能开销较大的loader的处理结果

### 多进程配置

thread-loader

配置在thread-loader之后的loader都会在一个单独的worker池中运行。

```javascript
{
    test: /\.js[x]?$/,
    use: [	
        {
            // 开启多线程
            loader: "thread-loader",
            options: {
                worker: 3,
            },
        },
     ]
}
```

### externals和noParse

* externals

  `externals` 配置选项提供了「**从输出的 bundle 中排除依赖**」的方法。

* noParse

  对于我们不需要解析依赖的第三方大型类库等，可以通过这个字段进行配置，以提高构建速度。

  使用 noParse 进行忽略的模块文件中不会解析 `import`、`require` 等语法。

## 构建结果优化

### css压缩

安装[optimize-css-assets-webpack-plugin](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Foptimize-css-assets-webpack-plugin)插件

注意对于webpack5，要使用 [css-minimizer-webpack-plugin](https://github.com/webpack-contrib/css-minimizer-webpack-plugin)

进行配置：

```
optimization: {
    minimize: true,
	minimizer: [new CssMinimizerPlugin()],
},
```

### js压缩

webpack已经内置了[terser-webpack-plugin](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fterser-webpack-plugin) 插件，所以我们对其进行直接引用后使用

```
optimization: {
    minimize: true,
	minimizer: [new TerserPlugin({})],
},
```

### 清除无用css

[purgecss-webpack-plugin](https://link.juejin.cn/?target=https%3A%2F%2Fwww.purgecss.cn%2Fplugins%2Fwebpack.html%23%E7%94%A8%E6%B3%95) 会单独提取 CSS 并清除用不到的 CSS

```javascript
// ...
const PurgecssWebpackPlugin = require('purgecss-webpack-plugin')
const glob = require('glob'); // 文件匹配模式
// ...

function resolve(dir){
  return path.join(__dirname, dir);
}

const PATHS = {
  src: resolve('src')
}

const config = {
  plugins:[ // 配置插件
    // ...
    new PurgecssPlugin({
      paths: glob.sync(`${PATHS.src}/**/*`, {nodir: true})
    }),
  ]
}

```

### 开启tree-shaking

Tree-shaking 作用是剔除没有使用的代码，以降低包的体积。

webpack 默认支持，需要在 .bablerc 里面设置 `model：false`，即可在生产环境下默认开启

```
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "model": false,
                "useBuiltIns": "usage",
                "corejs": 3
            }
        ],
        "@babel/preset-react"
    ]
}
```

### Scope Hoisting

Scope Hoisting 即作用域提升，原理是将多个模块放在同一个作用域下，并重命名防止命名冲突，**通过这种方式可以减少函数声明和内存开销**。

- webpack 默认支持，在生产环境下默认开启
- 只支持 es6 代码

## 学习资料：

[学习代码](https://github.com/funny741643/webpack-learn/tree/main/webpack-optimize)

[透过分析 webpack 面试题，构建 webpack5.x 知识体系](https://juejin.cn/post/7023242274876162084#heading-22)

[带你深度解锁Webpack系列(优化篇)](https://juejin.cn/post/6844904093463347208)