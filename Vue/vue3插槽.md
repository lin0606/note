# Vue3插槽

我们可以将`vue3`的插槽分为具名插槽和作用域插槽，`vue3`中为插槽引入了一个统一的语法`v-slot`，它取代了`slot`和`slot-scope`。

## Vue3插槽的基本使用

其实插槽就是为我们提供了一种方式去组合我们的组件

```vue
<!-- 我们自定义的TodoButton组件,b并为其设置了插槽 -->
<template>
    <button>
        <slot></slot>
    </button>
</template>
```

```vue
<!-- 调用TodoButton组件并为其传入插槽内容 -->
<todo-button>
    已做
</todo-button>
```

## 编译作用域

该插槽可以访问与模板其余部分相同的实例 property (即相同的“作用域”)。但插槽**不能**访问 `<todo-button>` 的作用域。

```vue
<todo-button action="delete">
  Clicking here will {{ action }} an item
  <!-- `action` 未被定义，因为它的内容是传递到 <todo-button>，而不是在<todo-button>里定义的。  -->
</todo-button>
```

## 具名插槽

我们可能会在我们的组件中定义多个插槽，我们在为插槽传入数据时要如何去判断为哪个插槽传入数据呢？

`slot`元素有一个name属性来为我们做判断

```vue
<!-- 我们自定义的BaseLayout组件 里面包含多个插槽 -->
<template>
    <div>
        <header>
            <slot name="header"></slot>
        </header>
        <main>
            <!-- 没有name的默认为default -->
            <slot></slot>
        </main>
        <footer>
            <slot name="footer"></slot>
        </footer>
    </div>
</template>
```

```vue
<!-- 调用BaseLayout组件 -->
<base-layout>
	<!-- #header相当于v-slot="header" -->
    <template #header>
    	<h1>我是头部</h1>
    </template>
    <template #default>
    <div>
    	我是body主体
    </div>
    </template>
    <template #footer>
    	<p>我是底部</p>
    </template>
</base-layout>
```

## 作用域插槽

有时让插槽内容能够访问子组件中才有的数据是很有用的。

看一个官方案例

```vue
<!-- 我们自定义的TodoList组件 -->
<template>
    <ul>
        <li v-for="( item, index ) in items">
            <slot :item="item" :index="index"></slot>
        </li>
    </ul>
</template>

<script>
import { reactive } from "@vue/reactivity";
export default {
    setup() {
        const items = reactive(['Feed a cat', 'Buy milk'])
        return {
            items
        }
    }
}
</script>
```

我们想把子组件里面的item，让父组件去调用，因为只有在 `<todo-list>` 组件中访问到item，所以要使 `item` 可用于父级提供的插槽内容，我们可以添加一个 `<slot>` 元素并将其作为一个 attribute。就是这条语句

```vue
<slot :item="item" :index="index"></slot>
```

这个时候我们便可以在父组件中去使用item和index了

```vue
<todo-list>
      <template #default="slotProps">
        <p>{{slotProps.item}}</p>
      </template>
</todo-list>
```

我们可以将其进行解构

```vue
<todo-list>
      <template #default="{ item }">
        <p>{{item}}</p>
      </template>
</todo-list>
```



