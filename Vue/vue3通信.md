# vue3中组件间的通信

## Props

**父组件**

```vue
<div>
	<child-cmt :msg="msg"></child-cmt>
<div>
```

**子组件接收消息**

```vue
export default {
  props: {
    msg: String,
  },
  setup(props) {
    console.log('props:', props);
}
```

## emit

当我们想要从子组件向父组件发送一个消息时，我们可以在父组件中去自定义一个事件，由子组件去触发这个事件并且传递消息。

**父组件**

```vue
<div>
	<child-cmt @myClick="getChildMsg"></child-cmt>
</div>

<script>
    export default {
        setup() {
            const getChildMsg = (msg) => {
              console.log(msg);
            };
    	}
    }
</script>
```

**子组件**

```vue
<template>
  <button @click="handleClick">发送消息给父组件</button>
</template>
<script>
export default {
    setup(props, { emit }) {
        const handleClick = () => {
            emit("myClick", "这是发送给父组件的信息");
        };
    }
}
</script>
```

## v-model

**父组件**

```vue
<div>
    <child-cmt
        v-model:key="key"
    ></child-cmt>
    <p>{{ key }}</p>
</div>
```

**子组件**

```vue
<template>
  <button @click="handleClick">发送消息给父组件</button>
</template>
<script>
export default {
    setup(props, { emit }) {
        const handleClick = () => {
            emit("update:key", "新的key")
        };
    }
}
</script>
```

### 拓展(v-model原理)

其实我们在父组件挂载到子组件的v-model的方法其实就相当于

```vue
 <div>
 	<child-cmt
        :key="key"
        @update:key = "(val) => {key = val}"
    ></child-cmt>
    <p>{{ key }}</p>
 </div>

```

这里

1） 我们给子组件传递了一个props。名为key

2） 我们监听子组件上的事件update:key, 在回调函数中，将从子组件回传的值保存在key中

## ref

使用ref我们可以在父组件中获取子组件的实例并且调用子组件的方法，为子组件传递消息

**父组件**

```vue
<div>
    <child-cmt
        ref="child"
    ></child-cmt>
    <button @click="sendMsg">父组件调用子组件方法</button>
</div>
<script>
export default {
    setup(props, { expose }) {
        const sendMsg = () => {
          child.value.childMethod('msg');
        };
        // 子组件向外暴露的方法
        expose({
            sendMsg
        });
        return {
          sendMsg,
        };
    }
}
</script>
```

**子组件**

```vue
<script>
export default {
    setup(props) {
        const childMethod = (msg) => {
            console.log('接收父组件传来的消息', msg)
        }
    }
}
</script>
```

## Provide/Inject

`provide`允许我们向当前组件的所有后代组件，传递一份数据，所有后代组件能够通过`inject`这个方法来决定是否接受这份数据。

主要的应用场景为当我们需要深传递一个消息时，使用其比较方便

**父组件**

```vue
<script>
    export default {
        setup() {
            provide("name", "funny")
        }
    }
</script>
```

**子组件**

```vue
<script>
    export default {
        setup() {
            const name = inject("name")
            console.log(name) //funny
        }
    }  
</script> 
```

## vuex

说到深传递，我们还可以使用vue的插件vuex，可以看我写的vuex的总结

[vuex总结](https://github.com/lin0606/note/blob/main/Vue/Vuex.md)

