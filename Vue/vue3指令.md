# Vue3指令

[官方文档](https://v3.cn.vuejs.org/api/directives.html#v-text)

## 基本介绍

| 指令名称                        | 基本介绍                                                     |
| ------------------------------- | ------------------------------------------------------------ |
| v-text                          | 更新元素的文本信息                                           |
| v-html                          | 更新元素的innerHTML                                          |
| v-show                          | 是否展示，切换元素的display值                                |
| v-if<br />v-else<br />v-else-if | 条件渲染，根据条件判断需要挂载那个元素                       |
| v-for                           | 循环遍历                                                     |
| v-on                            | 绑定事件监听器，缩写@                                        |
| v-bind                          | 动态绑定一个或多个attribute, 或一个组件prop到表达式， 缩写： |
| v-model                         | [v-model原理]()                                              |
| v-slot                          | 详情看我写的插槽介绍文章<br />[vue3插槽介绍]()               |
| v-pre                           | 跳过这个元素和它的子元素的编译过程                           |
| v-once                          | 只渲染元素和组件**一次**。<br />随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。 |
| v-memo                          | 记住一个模板的子树。元素和组件上都可以使用。<br />该指令接收一个固定长度的数组作为依赖值进行记忆比对。<br />如果数组中的每个值都和上次渲染的时候相同，则整个该子树的更新会被跳过。 |

## v-if 和 v-show

下面直接展示v-if 和 v-show编译完成后的代码

### v-if

```vue
<div v-if="type === 'A'">Type A</div>
<div v-else-if="type === 'B'">Type B</div>
<div v-else>Default Type</div>
```

```js
// 编译后
function genThisHTML(scopeData) {
  // scopeData 为 Vue 实例里绑定的 data 数据
  if (scopeData.type === "A") {
    return `<div>Type A</div>`;
  } else if (scopeData.type === "B") {
    return `<div>Type B</div>`;
  } else {
    return `<div>Default Type</div>`;
  }
}
```

条件渲染指令其实是将常见的 Javascript 语法， vue编译器在进行识别后，然后去匹配对应的执行逻辑。

### v-show

`v-show`和`v-if`不一样，`v-if`会在条件具备的时候才进行渲染，而`v-show`的逻辑是一定渲染，但在条件具备的时候才显示：

```javascript
function genVShowHTML(scopeData) {
  // scopeData 为 Vue 实例里绑定的 data 数据
  // 这里的 hide 类名具有样式 display: none;
  return `<div ${scopeData.isShow ? "" : 'class="hide"'}>Something</div>`;
}
```

### v-if和v-show应用

带有`v-show`的元素始终会被渲染并保留在 DOM 中。一般来说，`v-if`有更高的切换开销（因为要不停地重新渲染），而`v-show`有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用`v-show`较好；如果在运行时条件很少改变，则使用`v-if`较好。

> 以上的介绍来自basin的博客[vue-常用指令](https://godbasin.github.io/vue-ebook/vue-ebook/5.html#_5-1-%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A4)

## v-model原理

可以看我写的vue3消息通信里的内容

[vue3v-model原理](https://github.com/lin0606/note/blob/main/Vue/vue3%E9%80%9A%E4%BF%A1.md#%E6%8B%93%E5%B1%95v-model%E5%8E%9F%E7%90%86)

## 自定义指令

通常我们会在需要给某个元素添加简单的事件处理逻辑的时候，会使用到自定义指令。

### Vue3中指令的钩子函数

vue3为了更好地与组件的生命周期保持一致，对指令的钩子函数进行了更新，如下：

```javascript
const MyDirective = {
  created(el, binding, vnode, prevVnode) {}, // 新增
  beforeMount() {},
  mounted() {},
  beforeUpdate() {}, // 新增
  updated() {},
  beforeUnmount() {}, // 新增
  unmounted() {}
}
```

下面对其进行一一介绍

| 钩子函数      | 调用时机                                         |
| ------------- | ------------------------------------------------ |
| created       | 在元素的 attribute 或事件监听器被应用之前调用。  |
| beforeMount   | 指令绑定到元素后调用。只调用一次。               |
| mounted       | 元素插入父 DOM 后调用。                          |
| beforeUpdate  | 在元素本身被更新之前调用。                       |
| updated       | 一旦组件和子级被更新，就会调用这个钩子。         |
| beforeUnmount | 在元素被卸载之前调用。                           |
| unmounted     | 一旦指令被移除，就会调用这个钩子。也只调用一次。 |

### 示例

下面是一个自定义的长按指令

```javascript
export default (app) => {
    //自定义组件
    app.directive('longpress', {
        beforeMount: function (el, binding, vNode) {
            if (typeof binding.value !== 'function') {
                const compName = vNode.context.name;
                let warn = `[longpress:] provided expression '${binding.expression}' is not a function, but has to be `;
                if (compName) {
                    warn += `Found in component '${compName}' `;
                }
                console.warn(warn);
            }
            let pressTimer = null
            el.startEvent = e => {
                // 判断pc | mobile 是否单击
                if (e.type === 'click' && e.button !== 0) {
                    return;
                }
                if (pressTimer === null) {
                    pressTimer = setTimeout(() => {
                        handler();
                    }, 1000)
                }
            }
            el.cancelEvent = e => {
                if (pressTimer === null) {
                    clearTimeout(pressTimer)
                    pressTimer = null;
                }
            }
            const handler = e => {
                binding.value(e)
            }
            // 添加事件监听器
            el.addEventListener("mousedown", el.startEvent, true);
            el.addEventListener("touchstart", el.startEvent, true);
            // 取消计时器
            el.addEventListener("click", el.cancelEvent, true);
            el.addEventListener("mouseout", el.cancelEvent, true);
            el.addEventListener("touchend", el.cancelEvent, true);
            el.addEventListener("touchcancel", el.cancelEvent, true);
        },
        beforeUnmount: function (el) {
            // 解绑事件
            el.removeEventListener("mousedown", el.startEvent, true);
            el.removeEventListener("touchstart", el.startEvent, true);
            // 取消计时器
            el.removeEventListener("click", el.cancelEvent, true);
            el.removeEventListener("mouseout", el.cancelEvent, true);
            el.removeEventListener("touchend", el.cancelEvent, true);
            el.removeEventListener("touchcancel", el.cancelEvent, true);
        }
    })
}
```

使用该自定义的长按指令

```vue
<template>
  <div v-longpress="longPress">{{ text }}</div>
</template>

<script>
import { ref } from "vue";
export default {
  setup() {
    let text = ref("初始化");
    const longPress = () => {
      text.value = "长按";
    };
    return {
        text,
        longPress
    }
  },
};
</script>
```

在`main.js`中引入这个指令

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import directives from './directives/longpress.js'

const app = createApp(App)
directives(app)
app.mount('#app')
```

