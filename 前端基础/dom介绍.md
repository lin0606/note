# DOM

## Dom简介

文档对象模型 (DOM) 是HTML和XML文档的编程接口。它会将web页面和脚本或程序语言连接起来。

API (web 或 XML 页面) = DOM + JS (脚本语言)

## Dom的表现

![img](https://developer.mozilla.org/en-US/docs/Web/API/Document_object_model/Using_the_W3C_DOM_Level_1_Core/using_the_w3c_dom_level_1_core-doctree.jpg)

## Dom核心接口

- `document.getElementById(id)`
- `document.getElementsByTagName(name)`
- `document.createElement(name)`
- `parentNode.appendChild(node)`
- `element.innerHTML`
- `element.style`
- `element.setAttribute()`
- `element.getAttribute()`
- `element.addEventListener()`
- `window.onload`
- `window.scrollTo(x-coord, y-coord)`

## Dom 事件流

**DOM事件流（`event flow`）存在三个阶段：事件捕获阶段、处于目标阶段、事件冒泡阶段。**

* 捕获阶段：一开始从文档的根节点流向目标对象；
* 目标阶段：然后在目标对象上触发；
* 冒泡阶段：之后再回溯到文档的根节点。

### 事件捕获

当一个事件发生在具有父元素的元素上

- 浏览器检查元素的最外层祖先`html`，是否在捕获阶段中注册了一个`onclick`事件处理程序，如果是，则运行它。
- 然后，它移动到`html`中单击元素的下一个祖先元素，并执行相同的操作，然后是单击元素再下一个祖先元素，依此类推，直到到达实际点击的元素。

### 事件冒泡

- 浏览器检查实际点击的元素是否在冒泡阶段中注册了一个`onclick`事件处理程序，如果是，则运行它
- 然后它移动到下一个直接的祖先元素，并做同样的事情，然后是下一个，等等，直到它到达`html`元素。
- 我们可以通过 `stopPropagation()`，让事件不在冒泡链上进一步的扩大

### 事件委托

冒泡还允许我们利用事件委托——这个概念依赖于这样一个事实,如果你想要在大量子元素中单击任何一个都可以运行一段代码，您可以将事件监听器设置在其父节点上，并让子节点上发生的事件冒泡到父节点上，而不是每个子节点单独设置事件监听器。

# shadow dom

## 简介

shadow DOM它可以将一个隐藏的、独立的 DOM 附加到一个元素上。
以一个有着默认播放控制按钮的 <video>元素为例。你所能看到的只是一个 <video>标签，实际上，在它的 Shadow DOM 中，包含来一系列的按钮和其他控制器。

## 使用

1. 操作shadow DOM里的元素其实和普通DOM元素的操作一样，例如添加子节点、设置属性，以及为节点添加自己的样式。可以在设置<style>元素，给整个 Shadow DOM添加样式。

2. 在挂载shadow Dom前我们需要指定一个dom节点，作为shadow Dom的宿主元素，我们称之为shadow host

3. 开始设置我们的shadow节点。使用 Element.attachShadow()方法来将一个 shadow root 附加到任何一个元素上。它接受一个配置对象作为参数，该对象有一个 mode属性，值可以是open或者closed。

  ```
  let shadow = elementRef.attachShadow({mode: 'open'});
  let shadow = elementRef.attachShadow({mode: 'closed'});
  ```

  true表示可以通过页面内的 JavaScript 方法来获取 Shadow DOM。

3. 需要注意的一点是：如果shadow host下面有其他普通元素, 在添加了shadow Root后，其他普通元素就不会显示了。![img](https://mdn.mozillademos.org/files/15788/shadow-dom.png)

# 虚拟dom

虚拟`DOM`其实是用来模拟真实`DOM`的中间产物，主要包括以下功能：

**1. 用`JS`对象模拟`DOM`树，简化`DOM`对象。**

简单来说，就是用一个对象模拟，保留主要的一些`DOM`属性，其他的则去掉。

**2. 使用虚拟`DOM`，结合操作`DOM`的接口，来生成真实`DOM`。**

使用假`DOM`生成真`DOM`，同时保持真实`DOM`对象的引用，以便3步骤的执行。

**3. 更新`DOM`时，比较两棵虚拟`DOM`树的差异，局部更新真实`DOM`。**

这个就比较有意思，可以根据数据的变化，来最小化地移动、替换、删除原有的`DOM`元素。

结合使用以上功能，便能在复杂应用中更好地维护了。



