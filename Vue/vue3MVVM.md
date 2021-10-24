# Vue3MVVM

## 知原由

对于`vue2`来说，其已经使用了`Object.defineProperty()`实现了数据相应式，而`vue3`为何要使用`proxy`对其进行核心改造呢？其是主要有两点：

首先，`proxy`本质上是某个对象进行劫持，所以它不仅仅可以监听对象某个属性值的变化，还可以监听对象属性的新增和删除（也就是说它不用像`vue2`一样需要额外判断劫持的数据类型是否是数组，进入对数组原型的方法进行重写）。而`object.defineProperty()`是给对象的某个已存在的属性添加对应的`getter`和`setter`,所以只能监听这个属性值的变化，而不能去监听对象属性的新增和删除。

其次，就是性能的问题，这块需要注意的是`proxy`在性能上要比`object.defineProperty`差的。在性能方面的优化其实是体现在把嵌套层级较深的对象变成响应式的场景。在 `Vue 2` 的实现中，在组件初始化阶段把数据变成响应式时，遇到子属性仍然是对象的情况，会递归执行 `Object.defineProperty` 定义子对象的响应式；而在 `Vue 3` 的实现中，只有在对象属性被访问的时候才会判断子属性的类型来决定要不要递归执行 `reactive`，这其实是一种延时定义子对象响应式的实现，在性能上会有一定的提升。

## 简易实现

```javascript
// 单例模式，存储目前的副作用
let currentEffect = null

class Dep {
    constructor() {
        // 依赖收集器
        this.effects = new Set()
    }
    // 收集依赖， 对应target中对应key收集自己副作用
    depend() {
        if (currentEffect) {
            this.effects.add(currentEffect)
        }
    }
    // 触发对应的target中对应key 所对应的所有副作用
    notice() {
        this.effects.forEach(effect => {
            effect()
        })
    }
}

// targetMap中，target可以映射到一个依赖类集合的Map上，我们可以称之为depsMap
// depsMap中， 每个key(之前对象上的属性)可以对应一个依赖类 dep
let targetMap = new Map()

// 获取目标dep
function getDep(target, key) {
    let depsMap = targetMap.get(target)
    if (!depsMap) {
        depsMap = new Map()
        targetMap.set(target, depsMap)
    }
    let dep = depsMap.get(key)
    if (!dep) {
        dep = new Dep()
        depsMap.set(key, dep)
    }
    return dep
}

function reactive(raw) {
    return new Proxy(raw, {
        // get时 对prop进行依赖收集
        get(target, key) {
            let dep = getDep(target, key)
            // 收集依赖
            dep.depend()
            return Reflect.get(target, key)
        },
        // set时 进行依赖触发
        set(target, key, value) {
            let dep = getDep(target, key)
            let result = Reflect.set(target, key, value)
            // 依赖触发
            dep.notice()
            return result
        }
    })
}

// 设置当前的副作用
function watchEffect(effect) {
    currentEffect = effect
    // 进行副作用的收集
    effect()
    currentEffect = null
}

const user = reactive({
    age: 1
})

let b,c;
watchEffect(() => {
    b = user.age + 1
    console.log(b)
})
watchEffect(() => {
    c = user.age + 3
    console.log(c)
})
user.age = 20
```

对于简易实现主要分为四部分

* 依赖收集的类（`Dep`）
* 获取目标target中目标key所对应的依赖类实例(`getDep`)
* 将目标对象变为响应式对象的函数`reactive`
* 副作用观察函数`watchEffect`

上述代码的主要运行流程为：将我们的目标对象设置为响应式的（即进行proxy代理，在get的时候，进行依赖收集，在set的时候，进行依赖触发）。使用`watchEffect`函数，将其参数callback函数（即副作用函数）设置到target对象对应key的依赖集中。

设置effect函数到依赖集中的主要流程为：

1. 调用proxy的get
2. 调用`getDep`函数，获取对应key的依赖集，若当前没有，则创建新的。
3. 将单例副作用函数`currentEffect`添加到依赖集中

最后我们在使用`user.age = 20`,即set的时候，触发proxy的set，进行对应依赖的触发。

