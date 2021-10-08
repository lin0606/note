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

```js
computed: {
    count () {
    	return store.state.count
    }
}
```

* 根实例中注册 `store` 选项,Vue.use(Vuex)。这时候子组件就可以使用this.$store访问到，

```js
computed: {
    count () {
    	return this.$store.state.count
    }
}
```

#### mapState辅助函数

当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性，让你少按几次键：

```js
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

```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

#### mapState与局部计算属性混合

```js
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

```js
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

```js
store.getters.doneTodos
```

```js
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

#### 通过方法访问

**getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果。**

```js
getters: {
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

```js
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

#### mapGetters辅助函数

mapGetters辅助函数会将store中的getter映射到局部计算属性

```js
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

```js
mapGetters({
  // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```

### Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation

```js
const store = new Vuex.Store({
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

```js
store.commit('increment')
```

我们可以将mutations想象为一个事件管理器，使用commit去催动类型为increment事件的触发。

store.commit()有两个参数，第一个即为事件类型，第二个参数我们可以传递一个payload(载荷),即触发事件时所携带的额外参数

```js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

```js
store.commit('increment', {
  amount: 10
})
```

我们也可以以对象的方式进行提交

```js
store.commit({
  type: 'increment',
  amount: 10
})
```

#### Mutation 需遵守 Vue 的响应规则

由于vuex中store的状态都是响应式的，所以在我们使用mutation为state添加新属性时，要满足以下规则

* 使用 `Vue.set(obj, 'newProp', 123)`

* 新对象替代老对象

  ```js
  state.obj = { ...state.obj, newProp: 123 }
  ```

另外最好提前在你的 store 中初始化好所有所需属性

#### Mutation 必须是同步函数

在 mutation 中混合异步调用会导致你的程序很难调试。例如，当你调用了两个包含异步回调的 mutation 来改变状态，你怎么知道什么时候回调和哪个先回调呢

#### 提交Mutation

* `this.$store.commit('xxx')`

* 使用 `mapMutations` 辅助函数将组件中的 methods 映射为 `store.commit` 调用（需要在根节点注入 `store`）

  ```js
  import { mapMutations } from 'vuex'
  export default {
    // ...
    methods: {
      ...mapMutations([
        'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
  
        // `mapMutations` 也支持载荷：
        'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
      ]),
      ...mapMutations({
        add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
      })
    }
  }
  ```

上面刚说到Mutaition必须是同步函数，但是如果我们需要在store里进行异步调用，该如何处理呢？这时候我们就需要使用Action了

### Action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

Action 函数**接受一个与 store 实例具有相同方法和属性的 context 对象**，因此你可以调用 `context.commit` 提交一个 mutation，或者通过 `context.state` 和 `context.getters` 来获取 state 和 getters。但要注意的是**context不是store实例本身**

Action 通过 `store.dispatch` 方法触发

```js
store.dispatch('increment')
```

#### 分发方式

Actions 支持同样的载荷方式和对象方式进行分发：

```js
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

#### 在组件中分发

* `this.$store.dispatch('xxx')`

* 使用 `mapActions` 辅助函数将组件的 methods 映射为 `store.dispatch` 调用

  ```js
  import { mapActions } from 'vuex'
  
  export default {
    methods: {
      ...mapActions([
        'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
  
        // `mapActions` 也支持载荷：
        'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
      ]),
      ...mapActions({
        add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
      })
    }
  }
  ```

#### 组合多个 action

我们需要知道 `store.dispatch` 可以处理**被触发的 action 的函数返回的 Promise**，并且 `store.dispatch` 仍旧返回 Promise

```js
store.dispatch('actionA').then(() => {
  // ...
})
```

在另一个action中我们也可以这样

```js
actions: {
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```

使用`async`, `await`

```js
actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```

#### 注意

> 一个 `store.dispatch` 在不同模块中可以触发多个 action 函数。在这种情况下，只有当所有触发函数完成后，返回的 Promise 才会执行。

### Module

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。

为了解决上述问题，Vuex允许我们将store分割成模块（module）。每个模块拥有自己的state,mutation,action,getter,同时也可以嵌套子模块。

```js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

#### 模块的局部状态

对于模块内部的 mutation 和 getter，接收的第一个参数是**模块的局部状态对象**。

同样，对于模块内部的 action，局部状态通过 `context.state` 暴露出来，根节点状态则为 `context.rootState`

而对于模块内部的 getter，根节点状态会作为第三个参数暴露出来：

```js
const moduleA = {
  state: { count: 0 },
  mutations: {
    increment (state) {
      // 这里的 `state` 对象是模块的局部状态
      state.count++
    }
  },
  
   actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  },

  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
}
```

#### 命名空间

默认情况下，模块内部的 action、mutation 和 getter 是注册在**全局命名空间**的，可以通过添加 `namespaced: true` 的方式使其成为带命名空间的模块。

当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。

```js
this.$store.dispatch("moduleA/incrementIfOddOnRootSum");
```

#### 带命名空间的模块访问全局内容

如果你希望使用全局 state 和 getter，`rootState` 和 `rootGetters` 会作为第三和第四参数传入 getter，也会通过 `context` 对象的属性传入 action。

若需要在全局命名空间内分发 action 或提交 mutation，将 `{ root: true }` 作为第三参数传给 `dispatch` 或 `commit` 即可。

```js
modules: {
  foo: {
    namespaced: true,
    getters: {
      // 你可以使用 getter 的第四个参数来调用 `rootGetters`
      someGetter (state, getters, rootState, rootGetters) {},
    },
    actions: {
      // 在这个模块中， dispatch 和 commit 也被局部化了
      // 他们可以接受 `root` 属性以访问根 dispatch 或 commit
      someAction ({ dispatch, commit, getters, rootGetters, rootState }) {
        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
    }
  }
}
```

#### 在带命名空间的模块注册全局 action

```js
{
  actions: {
    someOtherAction ({dispatch}) {
      dispatch('someAction')
    }
  },
  modules: {
    foo: {
      namespaced: true,

      actions: {
        someAction: {
          // 这块设置为true, 将其设置为全局的
          root: true,
          handler (namespacedContext, payload) { ... } // -> 'someAction'
        }
      }
    }
  }
}
```

#### 如何使用辅助函数

* 我们可以给辅助函数的第一个参数传入命名空间的字符串

  ```
  computed: {
    ...mapState('some/nested/module', {
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    ...mapActions('some/nested/module', [
      'foo', // -> this.foo()
      'bar' // -> this.bar()
    ])
  }
  ```

* 通过使用 `createNamespacedHelpers` 创建基于某个命名空间辅助函数

  ```js
  import { createNamespacedHelpers } from 'vuex'
  
  const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
  
  export default {
    computed: {
      // 在 `some/nested/module` 中查找
      ...mapState({
        a: state => state.a,
        b: state => state.b
      })
    },
    methods: {
      // 在 `some/nested/module` 中查找
      ...mapActions([
        'foo',
        'bar'
      ])
    }
  }
  ```

### 官方示例

[购物车案例](https://github.com/vuejs/vuex/tree/dev/examples/shopping-cart)

## 剖析Vuex

