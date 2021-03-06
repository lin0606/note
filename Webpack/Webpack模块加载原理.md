webpack是模块打包器，它会将我们的每一个文件都看作是一个模块，最终将这些模块打包为一个大的js文件。

无论我们使用CommonJs规范还是ES6模块规范，打包后的文件都统一使用webpack自定义的模块规范来管理、加载模块。下面我们来看一下webpack的模块加载原理。

## CommonJS规范

### 示例代码

首先我们需要自己随便写两份源文件，这里我分别定义为`index.js`和`lib.js`,其次使用CommonJS方式进行模块导出和导入，代码如下：

```javascript
// index.js
console.log("start require");
var lib = require("./lib");
console.log("end require", lib);
console.log(lib.minus(3, 2));
```

```javascript
// lib.js
exports.minus = function (a, b) {
  return a - b;
};
```

### webpack模块加载

这里我使用的是webpack4.0版本。最终打包下来的代码如下(这里只抽离出来了主体代码)：

```javascript
(function (modules) {
  // webpackBootstrap
  // The module cache
  // 模块缓存对象
  var installedModules = {};

  // The require function
  // 请求函数
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    // 检查该模块是否已经被缓存过了
    if (installedModules[moduleId]) {
      // 如果缓存命中则直接返回模块的exports对象
      return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    // 否则新建一个模块对象并纳入缓存对象中
    var module = (installedModules[moduleId] = {
      // 模块Id, 模块的标识： /*! ./lib */ "./src/lib.js"
      i: moduleId,
      // 模块是否已经加载
      l: false,
      // exports对象
      exports: {},
    });

    // Execute the module function
    // 使用call绑定到module.exports对象上来执行模块函数
    modules[moduleId].call(
      module.exports,
      // 绑定函数的参数
      module,
      module.exports,
      __webpack_require__
    );

    // Flag the module as loaded
    // 模块函数执行完毕后，将其标志位已加载完成
    module.l = true;

    // Return the exports of the module
    // 返回module.exports, 其内容取决于我们自己所写模块导出的内容
    return module.exports;
  }

  // Load entry module and return exports
  // 请求入口函数，开始执行并返回模块对象
  return __webpack_require__((__webpack_require__.s = "./src/index.js"));
})({
  // 立即执行函数的参数对象，也就是我们所写的代码
  "./src/index.js": function (module, exports, __webpack_require__) {
    eval("console.log(\"start require\");\r\nvar lib = __webpack_require__(/*! ./lib */ \"./src/lib.js\");\r\nconsole.log(\"end require\", lib);\r\nconsole.log(lib.minus(3, 2));\n\n//# sourceURL=webpack:///./src/index.js?");
  },

  "./src/lib.js": function (module, exports) {
    eval("exports.minus = function (a, b) {\r\n  return a - b;\r\n};\n\n//# sourceURL=webpack:///./src/lib.js?");
  },
});
```

对于上面这么长的代码，其实我们现在只需要关注一下几点：

1. 打包完成后的代码主体是一个立即执行函数，即:

   ```javascript
   (function (modules) {}({
   	"./src/index.js": function(module, exports, __webpack_require__){eval()},
       "./src/lib.js": function (module, exports) {(eval())}
   })
   ```

2. 模块缓存对象，即：

   ```javascript
   var installedModules = {};
   ```

3. webpack自定义的require()函数，即：

   ```javascript
   function __webpack_require__(moduleId) {}
   ```

4. 入口函数的执行位置, 即：

   ```javascript
   return __webpack_require__((__webpack_require__.s = "./src/index.js"));
   ```

下面我们就来看看打包完后的代码究竟做了什么：

1. 建立模块缓存对象
2. 定义模块加载函数`__webpack_require__`
3. 请求入口函数，并开始执行

其主要的核心便是`__webpack_require__(moduleId)`函数，参数moduleId便是文件路径，下面我们看一看该函数以及他都做了什么：

```javascript
function __webpack_require__(moduleId) {
    // Check if module is in cache
    // 检查该模块是否已经被缓存过了
    if (installedModules[moduleId]) {
        // 如果缓存命中则直接返回模块的exports对象
        return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    // 否则新建一个模块对象并纳入缓存对象中
    var module = (installedModules[moduleId] = {
        // 模块Id, 模块的标识： /*! ./lib */ "./src/lib.js"
        i: moduleId,
        // 模块是否已经加载
        l: false,
        // exports对象
        exports: {},
    });

    // Execute the module function
    // 使用call绑定到module.exports对象上来执行模块函数
    modules[moduleId].call(
        module.exports,
        // 绑定函数的参数
        module,
        module.exports,
        __webpack_require__
    );

    // Flag the module as loaded
    // 模块函数执行完毕后，将其标志位设置为已加载完成
    module.l = true;

    // Return the exports of the module
    // 返回module.exports, 其内容取决于我们自己所写模块导出的内容
    return module.exports;
}
```

1. 检查请求模块是否已经被缓存过了，若命中缓存则直接返回缓存过的模块对象中的exports属性。
2. 若未命中缓存，则新建一个模块对象并纳入缓存对象中。其中模块对象包括：
   * moduleId：即模块标识也就是我们的文件路径
   * loaded：模块是否已加载完成的标志
   * exports：模块导出的内容，默认是一个对象。我们可以使用`module.exports=...`对其进行修改
3.  使用call将模块函数绑定到`module.exports`对象上来执行
4. 模块函数执行完成后，设置loaded为true
5. 返回`module.exports`

下面我们看一下打包后的`index.js`代码：

```javascript
console.log("start require");
var lib = __webpack_require__(/*! ./lib */ "./src/lib.js");
console.log("end require", lib);
console.log(lib.minus(3, 2));
//# sourceURL=webpack:///./src/index.js?
```

将打包后的模块代码和原模块的代码进行对比，可以发现仅有一个地方发生了变化，那就是 `require` 变成了 `__webpack_require__`。

从刚才的分析可知，`__webpack_require__()` 加载模块后，会先执行模块对应的函数，然后返回该模块的 `exports` 对象。所以入口模块能通过 `__webpack_require__()` 引入 `lib.js`中的minus函数并执行。

到目前为止可以发现 webpack 自定义的模块规范完美适配 CommonJS 规范。

## ES6 module

我们将代码改为使用es6 模块规范进行导入和导出，再看看其与CommonJs进行打包后代码的不同。

```javascript
// index.js
console.log("start require");
import { minus } from "./lib.js";
console.log("end require", minus);
console.log(minus(3, 2));
```

```javascript
// lib.js
export const minus = function (a, b) {
    return a - b;
};
```

打包完成后发现，使用 ES6 module 规范打包后的代码和使用 CommonJS 规范打包后的代码绝大部分都是一样的。一样的地方是指 webpack 自定义模块规范的代码一样，唯一不同的是上面两个文件打包后的代码不同。

```javascript
{
    "./src/index.js": function (module, __webpack_exports__, __webpack_require__) {
        "use strict";
        eval(
            '__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _lib_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./lib.js */ "./src/lib.js");\nconsole.log("start require");\r\n\r\nconsole.log("end require", _lib_js__WEBPACK_IMPORTED_MODULE_0__["minus"]);\r\nconsole.log(Object(_lib_js__WEBPACK_IMPORTED_MODULE_0__["minus"])(3, 2));\r\n\n\n//# sourceURL=webpack:///./src/index.js?'
        );
    },

    "./src/lib.js": function (module, __webpack_exports__, __webpack_require__) {
        "use strict";
        eval(
            '__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "minus", function() { return minus; });\nconst minus = function (a, b) {\r\n  return a - b;\r\n};\n\n//# sourceURL=webpack:///./src/lib.js?'
        );
    },
}
```

可以看到函数所传的第二个参数为`__webpack_exports_`,而commonjs里为`exports`。将执行代码的格式处理一下：

```javascript
// index.js
__webpack_require__.r(__webpack_exports__);
var _lib_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/lib.js");
console.log("start require");
console.log("end require", _lib_js__WEBPACK_IMPORTED_MODULE_0__["minus"])
console.log(Object(_lib_js__WEBPACK_IMPORTED_MODULE_0__["minus"])(3, 2))
//# sourceURL=webpack:///./src/index.js?
```

```javascript
// lib.js
__webpack_require__.r(__webpack_exports__);
__webpack_require__.d(__webpack_exports__, "minus", function() { return minus; });
const minus = function (a, b) { return a - b; };
//# sourceURL=webpack:///./src/lib.js?
```

我们可以发现，每个模块的开头都会执行`__webpack_require__.r(__webpack_exports__);`这条语句，对于lib.js其还多出了`__webpack_require__.d()`函数。下面我们就分别看看这两个函数：

### __webpack_require__.d()

```javascript
// define getter function for harmony exports
__webpack_require__.d = function (exports, name, getter) {
    if (!__webpack_require__.o(exports, name)) {
        Object.defineProperty(exports, name, {
            enumerable: true,
            get: getter,
        });
    }
};
```

可以看出该函数是为`__webpack_exports__`定义其所导出的变量。在打包后的lib.js代码中我们可以看到其调用：

```javascript
__webpack_require__.d(__webpack_exports__, "minus", function() { return minus; });
```

我们可以将其看作为

```javascript
__webpack_exports__[minus] = function() { return minus };
```

如果我们使用export default 进行导出，打包后的代码会是这样：

```javascript
__webpack_require__.d(__webpack_exports__, "default", function() { return minus; });
```

### __webpack_require__.r()

```javascript
// define __esModule on exports
__webpack_require__.r = function (exports) {
    if (typeof Symbol !== "undefined" && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, {
            value: "Module",
        });
    }
    Object.defineProperty(exports, "__esModule", { value: true });
};
```

`__webpack_require__.r()` 函数的作用是给 `__webpack_exports__` 添加一个 `__esModule` 为 `true` 的属性，表示这是一个 ES6 module。添加该属性的主要作用在于**处理混合使用 ES6 module 和 CommonJS 的情况。**