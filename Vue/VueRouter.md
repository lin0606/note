# Vue-router

[vue-router官方文档](https://next.router.vuejs.org/zh/guide/)

## 基本原理

对于vue-router有两种模式，一种是hash模式，一种是history,这两种模式的实现是基于Location API和History API这些浏览器提供的API能力封装的。

## Hash模式

Hash模式的实现主要依赖于Location对象的**hash属性**和**hashchange**事件。

### Location对象

**`Location `**接口表示其链接到的对象的位置（URL）。所做的修改反映在与之相关的对象上。 [`Document`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document) 和 [`Window`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window) 接口都有这样一个链接的Location，分别通过 [`Document.location`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/location)和[`Window.location`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/location) 访问。

下面我使用一个例子，对location对象进行描述

**https://www.example.com/zh-CN/docs?name=vue#location**

| 属性     | 描述                            | 返回结果                                     |
| -------- | ------------------------------- | -------------------------------------------- |
| hash     | 设置或返回从‘#’开头的url        | #location                                    |
| href     | 设置或返回整个URL               | www.example.com/zh-CN/docs?name=vue#location |
| protocol | 设置或返回当前URL的协议         | https:                                       |
| host     | 设置或返回当前URL的域名和端口号 | www.example.com，端口默认为80                |
| hostname | 设置或返回当前 URL 的域名       | www.example.com                              |
| port     | 设置或返回当前URL的端口号       | 默认为80                                     |
| pathname | 设置或返回当前URL的路径         | /zh-CN/docs                                  |
| search   | 设置或返回当前URL的参数         | ?name=vue                                    |
| origin   | 获取页面来源的域名的标准形式    | https://www.example.com                      |

### hashChange

当一个页面的hash发生了改变，便会触发hashchange事件，也就因此，我们可以通过window.onhashchange

监听hash的变化，来进行页面的更新处理

### 实现步骤

1. 设置监听器，监听`popstate`或者`hashchange`事件。[popstate介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/popstate_event)
2. 通过 hash（location.href.hash）获取当前的路由位置。
3.  根据当前匹配路径，判断后加载对应模块。

### Vue-router简单实现

```javascript
// histroy父类，hash模式和h5模式都会继承自这一个父类
export default class History {
  // router 是路由对象 也就是 VUe-Router的一个实例
  constructor(router) {
    // 赋值给自己的 router
    this.router = router
    // 默认的的当前路径为 /
    this.current = createRoute(null, '/')
    // cb 一个回调函数，它的作用就是修改 响应式路由的值 ，对应的视图然后就刷新
    this.cb = null
  }
  // 通过 listen来修改 cb的值
  listen(cb) {
    this.cb = cb
  }
  // 将要跳转的链接
  transitionTo(path, onComplete) {
    // this.current保存path和matched, 即路径和路径对应的路由规则或可以说所对应的组件
    this.current = this.router.matcher.match(path)
    // cb 修改响应式路由的值， 可以让页面响应式的刷新
    this.cb && this.cb(this.current)
    // Vue-router初始化时候调用，用于设置hash监听器
    onComplete && onComplete()
  }
}
```

```javascript
// 导入 base中的 History
import History from './base'
// 继承了 History
export default class HashHistory extends History {
  constructor(router) {
    super(router)
    // 确保第一次访问的时候路由加上 #/
    ensuerSlash()
  }
  // 监听URL的改变并设置当前的current和响应式的route,进行页面响应式的刷新
  setUpListener() {
    window.addEventListener('hashchange', () => {
      this.transitionTo(this.getCurrentLocation())
    })
  }
  // 获取当前的URL的hash
  getCurrentLocation() {
    // 为了兼容火狐， 不建议使用window.loaction.hash
    let href = window.location.href
    const index = href.indexOf('#')
    // 当没有 #的时候 直接返回 空字符串
    if (index < 0) return ''
    // 获取 #后面的地址
    href = href.slice(index + 1)
    return href
  }
}

// 确保第一次加上 #/
function ensuerSlash() {
  if (window.location.hash) {
    return
  }
  // 如果没有hash值 只要给 hash 加上一个 / 它会自动加上 /#/
  window.location.hash = '/'
}
```

## history模式

history路由模式，主要依赖history对象，该对象是挂载到window.history属性上的。history对象提供了操作浏览器绘画历史的接口，使用window.history我们可以实现以下与路由相关的重要能力：

1. 在history中跳转

使用`window.history.back()`、`window.history.forward()`和`window.history.go()`方法来完成在用户历史记录中向后和向前的跳转。

2. 添加和修改历史记录中的条目

HTML5 引入了`history.pushState()`和`history.replaceState()`方法，它们分别可以添加和修改历史记录条目。这两个 API 都会操作浏览器的历史栈，而不会引起页面的刷新。区别在于，`pushState()`会增加一条新的历史记录，而`replaceState()`则会替换当前的历史记录：

3. 监听popstate事件

当同一个页面在历史记录间切换时，就会产生`popstate`事件，我们可以通过监听`window.onpopstate`来获取事件对象。

上面介绍的`history.pushState()`或者`history.replaceState()`调用不会触发`popstate`事件，`popstate`事件只会在浏览器某些行为下触发, 比如点击后退、前进按钮(或者在 JavaScript 中调用`history.back()`、`history.forward()`、`history.go()`方法)。所以我们可以结合`popstate`事件、`pushState()`和`replaceState()`来完成完整的路由监听和修改能力。