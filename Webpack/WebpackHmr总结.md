# Webpack HMR介绍

## HMR基本原理介绍

HMR主要功能是会在应用程序运行过程中替换、添加或删除模块，而无需重新加载整个页面

思考：Webpack编译源码所产生的文件变化在编译时，替换模块实现在运行时，两者如何联系起来？

<img src="https://pic3.zhimg.com/80/v2-8bba4a7dac22f5f6c9fd8d4a3afc80a6_1440w.jpg" style="zoom:50%;" />

HMR工作流程分析：

1. 当过Webpack(Watchman)监听到项目中的文件/模块代码发生变化后，将变化通知Webpack中的构建工具(Packager)即HMR Plugin;
2. 然后经过HMR Plugin处理后，将结果发送到应用程序的运行时框架（HMR Runtime）;
3. 最后由HMR Runtime将这些发生变化的文件/模块更新(新增/删除或替换)到模块系统中。

其中，HMR Runtime是构建工具在编译时注入的，通过统一的Module ID将编译时的文件与运行时的模块对应起来，并且对外提供一系列API供应用层框架调用。

## HMR完整原理和源码分析

* Webpack-dev-server: 一个服务器插件，相当于express服务器，启动一个Web服务，只适用于开发环境；
* Webpack-dev-middleware：一个Webpack-dev-server的中间件，基本作用：通过watch mode，监听资源的变更，然后自动打包。
* Webpack-hot-middleware：结合Webpack-dev-middleware使用的中间件，它可以实现浏览器的无刷新更新，也就是HMR;

### 1.监听代码变化，重新编译打包

首先根据devServer配置，使用npm start将启动Webpack-dev-server启动本地服务器并进入Webpack的watch模式，然后初始化Webpack-dev-middleware，在Webpack-dev-middleware中对文件系统进行watch:

```javascript
// webpack-dev-server/lib/Server.js
// 1. initialize(): server初始化函数
// 2. setupApp(): 初始化express服务
// 3. setupDevMiddleware(): 初始化Webpack-dev-server

// webpack-dev-middleware/index.js
// 执行wdm函数开始进行监听
```

### 2. 保存编译结果

webpack-dev-middle有两种保存编译结果的方法分别是outputfile和writeToDisk。

Webpack 与 Webpack-dev-middleware 交互，Webpack-dev-middleware 调用 Webpack 的 API 对代码变化进行监控，并通知 Webpack 将重新编译的代码通过 JavaScript 对象**保存在内存中**。

### 3. 监控文件变化，刷新浏览器

与webpack-dev-middle不同的是，这里并不是监控代码变化重新编译打包。而是当我们在配置文件中配置了devServer.watchContentBase为true,Webpack-dev-server会监听配置文件夹中静态文件的变化，发生变化时，通知浏览器端对应用进行浏览器刷新，这与HMR不一样

### 4. 建立WS,同步编译阶段状态

在webpack-dev-server的浏览器端(client)和服务器端(webpack-dev-middleware)之间建立websocket长连接。

然后将webpack编译打包的各个阶段状态信息同步到浏览器端。

```javascript
// webpack-dev-server/lib/Server.js
// 1. listen: 监听express服务启用成功
// 2. createWebSocketServer: 建立websocket服务
// 3. setupHooks: 设置编译阶段的hooks函数，当监听到一次编译结束，就会调用sendStats方法
// getStats: 会获取最新打包文件的hash值
this.compiler.hooks.done.tap("webpack-dev-server", (stats) => {
      if (this.webSocketServer) {
        this.sendStats(this.webSocketServer.clients, this.getStats(stats));
      }

      this.stats = stats;
});
```

```javascript
// webpack-dev-server/lib/Server.js
sendStats {
    // 保存当前的hash
	this.currentHash = stats.hash;
    // 给客户端发送hash和ok
    this.sendMessage(clients, "hash", stats.hash);
    this.sendMessage(clients, "ok");
}
```

### 5.客户端接收和发布消息

```javascript
// webpack-dev-server/client/index.js
const onSocketMessage = {
    // 更新hash
	hash: function hash(_hash) {
        status.previousHash = status.currentHash;
        status.currentHash = _hash;
    },
    
    // 监听到ok消息时调用reloadApp()函数,进行更新检查等操作 
    ok: function ok() {
        sendMessage("Ok");

        if (options.overlay) {
          hide();
        }

        reloadApp(options, status);
    },
}
```

由于客户端(Client)并不请求热更新代码，也不执行热更新模块，因此通过emit一个“webpackHotUpdate”消息，将工作转交会webpack。下面就来看一下reloadApp这个函数

```javascript
function reloadApp(_ref, status) {
    if (hot && allowToHot) {
    log.info("App hot update...");
    // 发出 webpackHotUpdate 消息
    // websocket 仅仅用于客户端和服务端进行通信。而真正做事情的活还是交回给了webpack。
    hotEmitter.emit("webpackHotUpdate", status.currentHash);

    if (typeof self !== "undefined" && self.window) {
      // broadcast update to window
      self.postMessage("webpackHotUpdate".concat(status.currentHash), "*");
    }
  }
}
```

### 6.传递hash到HMR

现在我们可以聚焦到webpack里面来看一看webpack监听到webpackHotUpdate消息后会干什么事

```javascript
// webpack/hot/dev-server.js
hotEmitter.on("webpackHotUpdate", function (currentHash) {
    lastHash = currentHash;
    if (!upToDate() && module.hot.status() === "idle") {
        log("info", "[HMR] Checking for updates on the server...");
        check();
    }
});
```

Webpack/hot/dev-server 监听浏览器端 `webpackHotUpdate` 消息，将新模块 hash 值传到客户端 HMR 核心中枢的 HotModuleReplacement.runtime ，并调用 `check` 方法检测更新，**判断是浏览器刷新还是模块热更新**。如果是浏览器刷新的话，则没有后续步骤咯~~

```javascript
var check = function check() {
    // 调用module.hot.check()
	module.hot
		.check(true)
		.then(function (updatedModules) {
        	// 如果不是热模块更新
			if (!updatedModules) {
				// 进行浏览器刷新,无后续
				window.location.reload();
				return;
			}

			if (!upToDate()) {
				check();
			}
		})
		.catch(function (err) {
			// ...
        	window.location.reload();
		});
};
```

至于module.hot.check,实际上通过HotModuleReplacementPlugin已经注入到我们chunk中了,也就是HMR Runtime, 所以后面就是它如何更新bundle.js

### 7. HMR Runtime中更新bundle.js

我们仔细看我们的打包后的文件的话，开启热更新之后生成的代码会比不开启多出很多东西（为了更加直观看到，可以将其输出到本地），这些就是帮助 `webpack` 在浏览器端去更新 `bundle.js` 的 `HMR Runtime` 代码。

来看打包后的代码中新增了一个 `createModuleHotObject`

```js
module.hot = createModuleHotObject(options.id, module);
```

实际上这个函数就是用来返回一个 hot 对象，所以调用 `module.hot.check` 的时候，实际上就是执行 `hotCheck` 函数

```javascript
function createModuleHotObject(moduleId, me) {
  var hot = {
    // Module API
    addDisposeHandler: function (callback) {
      hot._disposeHandlers.push(callback);
    },
    removeDisposeHandler: function (callback) {
      var idx = hot._disposeHandlers.indexOf(callback);
      if (idx >= 0) hot._disposeHandlers.splice(idx, 1);
    },
    // Management API
    check: hotCheck,
    apply: hotApply,
    status: function (l) {
      if (!l) return currentStatus;
      registeredStatusHandlers.push(l);
    },
    addStatusHandler: function (l) {
      registeredStatusHandlers.push(l);
    },
    removeStatusHandler: function (l) {
      var idx = registeredStatusHandlers.indexOf(l);
      if (idx >= 0) registeredStatusHandlers.splice(idx, 1);
    },
  };
  currentChildModule = undefined;
  return hot;
}
```

其中就有 `hotCheck` 中调用了 `__webpack_require__.hmrM`

```js
function hotCheck(applyOnUpdate) {
  setStatus("check");
    return __webpack_require__.hmrM().then(function (update) {
  }
}
```

#### `__webpack_require__.hmrM`——加载.hot-update.json

来看 `__webpack_require__.hmrM`, 其中 `__webpack_require__.p` 指的是我们本地服务的域名，类似 `http://0.0.0.0:9528` ， 另外 `__webpack_require__.hmrF` 去获取 `.hot-update.json` 文件的地址，就是我们之前提到的重要文件之一

```js
__webpack_require__.hmrM = () => {
  if (typeof fetch === "undefined") throw new Error("No browser support: need fetch API");
  return fetch(__webpack_require__.p + __webpack_require__.hmrF()).then((response) => {
    if(response.status === 404) return; // no update available
    if(!response.ok) throw new Error("Failed to fetch update manifest " + response.statusText);
    return response.json();
  });
};
复制代码
/* webpack/runtime/get update manifest filename */
(() => {
  __webpack_require__.hmrF = () => ("main." + __webpack_require__.h() + ".hot-update.json");
})();
```

### 8. 加载要更新的模块

下面来看如何加载我们要更新的模块的，可以看到打包出来的代码中有 `loadUpdateChunk`

```js
function loadUpdateChunk(chunkId) {
  return new Promise((resolve, reject) => {
    var url = __webpack_require__.p + __webpack_require__.hu(chunkId);
    // create error before stack unwound to get useful stacktrace later
    var error = new Error();
    var loadingEnded = (event) => {
      // ...加载后的处理
    };
    __webpack_require__.l(url, loadingEnded);
  });
}
```

再来看 `__webpack_require__.l`，主要通过类似 `JSONP` 的方式进行，因为`JSONP`获取的代码可以直接执行。

```js
__webpack_require__.l = (url, done, key, chunkId) => {
  // ...
  if (!script) {
    script = document.createElement("script");

    script.charset = "utf-8";
    script.timeout = 120;
    if (__webpack_require__.nc) {
      script.setAttribute("nonce", __webpack_require__.nc);
    }
    script.setAttribute("data-webpack", dataWebpackPrefix + key);
    script.src = url;
  }
  // ...
  needAttach && document.head.appendChild(script);
};
```

我们的 HMR Runtime 中就是全局定义了webpackHotUpdate函数

```javascript
// webpackHotUpdate + 项目名
self["webpackHotUpdatelearn_hot_reload"] = (chunkId, moreModules, runtime) => {
  for(var moduleId in moreModules) {
    if(__webpack_require__.o(moreModules, moduleId)) {
      currentUpdate[moduleId] = moreModules[moduleId];
      if(currentUpdatedModulesList) currentUpdatedModulesList.push(moduleId);
    }
  }
  if(runtime) currentUpdateRuntime.push(runtime);
  if(waitingUpdateResolves[chunkId]) {
    waitingUpdateResolves[chunkId]();
    waitingUpdateResolves[chunkId] = undefined;
  }
};
```

所以，客户端接受到服务器端推动的消息后，如果需要热更新，**浏览器发起 http 请求去服务器端获取新的模块资源解析并局部刷新页面**。

整体流程图如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b41ea608505419391d6cfed3b997a08~tplv-k3u1fbpfcp-watermark.awebp)

## 学习参考

[【Webpack 进阶】聊聊 Webpack 热更新以及原理](https://juejin.cn/post/6939678015823544350)

[了不起的 Webpack HMR 学习指南](https://zhuanlan.zhihu.com/p/148815633)