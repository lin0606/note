

每个主要Vue生命周期事件被分成两个钩子，分别在事件之前和之后调用。Vue应用程序中有4个主要事件(8个主要钩子)。

- 创建 —— 在组件创建时执行
- 挂载 —— DOM被挂载时执行
- 更新 —— 当响应数据被修改时执行
- 销毁 —— 在元素被销毁之前立即运行

## 钩子函数

- **beforeCreate（初始化界面前）**

  由于**创建**的挂钩是用于初始化所有响应数据和事件的事物，因此`beforeCreate`无法访问组件的任何响应数据和事件。

  实例刚在内存中被创建出来，此时还没有初始化好data和methods属性

- **created（初始化界面后）**

  实例已经在内存中创建好，此时data和methods已经创建好，此时还没有开始编译模板

- **beforeMount（渲染dom前）**

  此时已经完成了模板的编译，但没有挂载到页面中

- **mounted（渲染dom后）**

  已经将编译好的模板挂载到页面指定的容器中显示

- **beforeUpdate（更新数据前）**

  数据更新时调用，发生在虚拟 DOM 打补丁之前。这里适合在更新之前访问现有的 DOM，比如手动移除已添加的事件监听器。

  状态更新之前执行此函数，此时data中的状态值是最新的（控制台上的数据已经更新），但是界面上的数据还是原来的，因为此时还**`没有开始重新渲染DOM节点`**。

- **updated（更新数据后）**

  由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。如果要相应状态改变，通常最好使用[计算属性](https://v3.cn.vuejs.org/api/options-data.html#computed)或[侦听器](https://v3.cn.vuejs.org/api/options-data.html#watch)取而代之。

  实例更新完之后调用此函数，此时data中的状态值和界面上显示的数据都已经完成了更新，界面已经被重新渲染好了。

  注意，`updated` **不会**保证所有的子组件也都被重新渲染完毕。如果你希望等待整个视图都渲染完毕，可以在 `updated` 内部使用 [vm.$nextTick](https://v3.cn.vuejs.org/api/instance-methods.html#nexttick)；

- **beforeDestroy（卸载组件前）**

  实例销毁之前调用，此时，`实例仍然完全可用`。

- **destroyed（卸载组件后）**

  Vue实例销毁后调用。调用后，Vue实例指示的所有东西都会**解绑定**，所有的事件监听器会被移除，所有的子实例也会被销毁。

- **errorCaptured** 

  当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 `false` 以阻止该错误继续向上传播。

## vue2生命周期(options API)

![Image text](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/2/20/1690b0a7158c1cbd~tplv-t2oaga2asx-watermark.awebp)

注意点：

**created**

​	初始化数据和方法。在处理读/写反应数据时，使用created方法。

​	例如：要进行API调用然后存储该值，则可以在此处进行操作。

## vue3生命周期（composition API）

与vue2相比，除了`beforecreate`和`created`(它们被`setup`方法本身所取代)，我们可以在`setup`方法中访问的API生命周期钩子有9个选项:

![image-20211027194339776](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20211027194339776.png)

- **onBeforemount**

  在组件DOM实际渲染安装之前调用。在这一步中，根元素还不存在。在选项API中，可以使用this.$el来访问。在组合API中，为了做到这一点，必须在根元素上使用`ref`。

  ```js
  // 选项 API
  export default {
     beforeMount() {
       console.log(this.$el)
     }
   }
  // 组合 API
  <template>
     <div ref='root'>
       Hello World
     </div>
  </template> 
  
  import { ref, onBeforeMount } from 'vue'
  
  export default {
     setup() {
        const root = ref(null) 
        onBeforeMount(() => {   
           console.log(root.value) 
        }) 
        return { 
           root
        }
      },
      beforeMount() {
        console.log(this.$el)
      }
   }
  ```

  

- **onmounted**

  在组件的第一次渲染后调用，该元素现在可用，允许直接DOM访问

与vue2相比改变的钩子函数：

- **onBeforeUnmount** （beforeDestory）

  在卸载组件实例之前调用。在这个阶段，实例仍然是完全正常的。

- **onUnmounted** (destoryed)

   卸载组件实例后调用。调用此钩子时，组件实例的所有指令都被解除绑定，所有事件侦听器都被移除，所有子组件实例被卸载。

- **onActivated**  

  被 `keep-alive` 缓存的组件激活时调用。

  keep-alive:**缓存组件内部状态，避免重新渲染**

- **onDeactivated** 

  被 `keep-alive` 缓存的组件停用时调用。

- **onErrorCaptured** 

  当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 `false` 以阻止该错误继续向上传播。



## 组件的生命周期

![Image text](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/21/164bbf610b61f3cd~tplv-t2oaga2asx-watermark.awebp)

在被`keep-alive`包含的组件/路由中，会多出两个生命周期的钩子:`activated` 与 `deactivated`。

**activated在组件第一次渲染时会被调用，之后在每次缓存组件被激活时调用**。

**activated调用时机：**

第一次进入缓存路由/组件，在`mounted`后面，`beforeRouteEnter`守卫传给 next 的回调函数之前调用：

```js
beforeMount => 如果是从别的路由/组件进来（组件销毁destroyed/或离开缓存deactivated)=>mounted => activated 进入缓存组件 => 执行beforeRouterEnter回调
```

**deactivated调用时机：**

使用了`keep-alive`就不会调用beforeDestroy和destroyed，因为组件没被销毁，被缓存起来了。

如果缓存了组件，在组件销毁的时候要做一些事情，可放在这个钩子里。

如果离开路由，会依次触发：

```js
组件内的离开当前路由钩子beforeRouterLeave => 路由前置守卫 beforeEach => 全局后置钩子 afterEach => deactivated 离开缓存组件 => activated 进入缓存组件
```



注意点：

1. **ajax请求最好放在created里面**，因为此时已经可以访问**this**了，请求到的数据就放在data里面
2. **关于dom的操作要放在mounted里面**，在mounted前面访问dom会是**undefined**
3. 每次离开/进入组件都要做一些事情，用什么钩子：

- 不缓存：

  进入的时候可以用`created`和`mounted`钩子，离开的时候用`beforeDestory`和`destroyed`钩子,`beforeDestory`可以访问`this`，`destroyed`不可以访问`this`。

- 缓存了组件：

  缓存了组件之后，再次进入组件不会触发`beforeCreate`、`created` 、`beforeMount`、 `mounted`，**如果你想每次进入组件都做一些事情的话，你可以放在`activated`进入缓存组件的钩子中**。

  同理：离开缓存组件的时候，`beforeDestroy`和`destroyed`并不会触发，可以使用`deactivated`离开缓存组件的钩子来代替。



























