## Vuex的基本概念

vuex采用了**集中管理**应用的所有组件的状态，这里vuex会将组件的共享状态抽取出来，以一个**全局单例模式**进行管理。

即我们所写的每个应用将**仅仅含有一个store实例**，store中便包含了我们所需的**唯一数据源（SSTO）也就是state**。state我们也可以称其为单一状态树，其是一个对象，在这个对象中包含了全部的应用层级状态。

但Vuex和单纯的全局对象也有以下两点的不同：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
2. 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地**提交 (commit) mutation**。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

<img src="https://vuex.vuejs.org/vuex.png" style="zoom: 67%;" />

从上图可以看出Vuex的催动视图变化的整体流程：

1. 我们在视图中发送Action（可以理解为对某一状态变化的描述）
2. 将Action提交到Mutation处理器中（对状态做加工的地方）
3. Mutation计算后，将新状态返回，并催动视图重新渲染

## Vuex的核心概念

### State

#### Vue组件中获取State状态

* 在计算属性中返回

```vue
computed: {
    count () {
    	return store.state.count
    }
}
```

* 根实例中注册 `store` 选项,Vue.use(Vuex)。这时候子组件就可以使用this.$store访问到，

```vue
computed: {
    count () {
    	return this.$store.state.count
    }
}
```

#### mapState辅助函数

当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性，让你少按几次键：

```vue
import { mapState } from 'vuex'
export default {
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,
    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',
    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 `mapState` 传一个字符串数组。

```vuex
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

#### mapState与局部计算属性混合

```
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```

### Getter

使用getter我们可以获取从State派生出来的新状态值。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

Getter可以接受state作为第一个参数，另一个getter作为第二个参数

```vue
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

#### 通过属性访问

```vue
store.getters.doneTodos
```

```vue
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

#### 通过方法访问

**getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果。**

```vue
getters: {
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

```
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

#### mapGetters辅助函数

mapGetters辅助函数会将store中的getter映射到局部计算属性

```vue
import { mapGetters } from 'vuex'
export default {
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
    ])
  }
}
```

使用对象形式，为getter属性另取名字

```vue
mapGetters({
  // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```

### Mutation





