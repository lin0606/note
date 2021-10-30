![image-20211026220005281](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20211026220005281.png)

### init（初始化）和$mount（挂载）

init会初始化生命周期，data，watch和methods等，对数据进行【**响应式化**】。通过object.defineProperty设置`getter`和`setter`来实现**响应式**和**依赖收集**。`$mount`会挂载组件。如果是运行时编译，即不存在render function但是存在template，需要进行编译。

### compile

compile经历三个阶段，得到render function

- parse

  用正则等方式解析template中的指令、class、style等，形成AST（抽象语法书）

- optimize

  主要标记静态节点（Vue在编译过程中的一处优化），`update`更新页面时，会有一个patch的过程，diff算法会跳过静态节点，减少了patch的过程，优化了patch的性能。

- generate

  是将AST转化成render function字符串的过程，得到结果是render的字符串和staticRenderFns字符串。

  经历三个阶段后，组件中存在渲染VNode需要的render function了。

### 响应式（render function）

render function被渲染时，会读取所需的值，触发**getter**函数进行**依赖收集**（让某个数据，比如说data知道有哪些地方依赖它，自己变化的时候需要通知他们），让所有用到这个data的组件（观察者watcher对象）存放到当前的**订阅者Dep**的subs中（**addSub**方法）。

在修改对象的值的时候，会触发对应的 **setter**，通知之前「**依赖收集**」得到的 Dep 中的每一个 Watcher，调用Dep中的**notify**方法，告诉它们自己的值改变了，需要重新渲染视图。这时候这些 Watcher 就会开始调用 `update` 来更新视图，当然这中间还有一个 **`patch`** 的过程以及使用**队列来异步更新**的策略。

#### 订阅者Dep

```js
class Dep{
    construtor(){
        //存放Watcher对象数组
		this.subs = [];
    }
    //在subs中添加一个Watcher对象
    addSub(sub){
		this.subs.push(sub);
    }
    //通知所有Watcher对象更新视图
    notify(){
		this.subs.forEach((sub)=>{
            sub.update();
        })
    }
}
```

#### 观察者Watcher

```js
class Watcher{
    constructor(){
        //在new一个watcher对象时将该对象赋值给Dep.target,在get中会用到
		Dep.target = this;
    }
    //更新视图
    update(){
        console.log('视图更新了~')
    }
}
Dep.target = null;
```

#### 依赖收集

```js
function defineReactive(obj,key,val){
	const dep = new Dep();
    Object.defineProperty(obj,key{
         enumerable:true,
         configurable:true,
         get: function reactiveGetter(){
        	//将Dep.target(将当前的Watcher对象存入dep的subs中)
			dep.addSub(Dep,target);
        	return val;
    	}，
        set: function reactiveSetter(newVal){
			if(newVal === val) return;
            dep.notify();
        }               
    });
}
class Vue {
    constructor(options){
		this._data = options.data;
        observer(this._data);
        //新建一个watcher观察者对象，这时候Dep.target会指向这个Watcher对象
        new Watcher();
        //为了触发test属性的get函数
        console.log('render',this._data.test);
    }
}
```

setter->Dep->watcher->patch->视图

#### patch过程（diff算法）

diff算法是通过**同层**的树节点进行比较，时间复杂度为O(n)。`patch` 的主要功能是比对两个 VNode 节点，将「差异」更新到视图上。

##### patch

**function patch(oldVnode, vnode, parentElm)**

三种情况：

1. 没有oldVnode，addVnode（添加新节点）
2. 没有Vnode（新节点），removeVnodes（删除旧节点）
3. oldVnode和Vnode都存在
   - 看他们是否满足sameVnode，满足再进行patchVnode
   - 不相同，remove老节点，add新节点

```js
function patch (oldVnode, vnode, parentElm) {
    if (!oldVnode) {
        addVnodes(parentElm, null, vnode, 0, vnode.length - 1);
    } else if (!vnode) {
        removeVnodes(parentElm, oldVnode, 0, oldVnode.length - 1);
    } else {
        if (sameVnode(oldVNode, vnode)) {
            patchVnode(oldVNode, vnode);
        } else {
            removeVnodes(parentElm, oldVnode, 0, oldVnode.length - 1);
            addVnodes(parentElm, null, vnode, 0, vnode.length - 1);
        }
    }
}
```



##### sameVnode

当key，tag，isComment（是否为注释节点），data同时定义（或不定义），input标签的type相同

```js
function sameVnode () {
    return (
        a.key === b.key &&
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        (!!a.data) === (!!b.data) &&
        sameInputType(a, b)
    )
}

function sameInputType (a, b) {
    if (a.tag !== 'input') return true
    let i
    const typeA = (i = a.data) && (i = i.attrs) && i.type
    const typeB = (i = b.data) && (i = i.attrs) && i.type
    return typeA === typeB
}
```



##### patchVnode

1. 在新老节点相同情况下，直接return

<font color="red">？？？？？？？？？patchVnode是在sameVnode满足触发的，为什么还要比较</font>

	2. 当新老节点是静态的（isStatic），并且`key`相同时，只要将 `componentInstance` 与 `elm` 从老 VNode 节点“拿过来”即可。
 	3. 当新节点是文本节点时，直接用 `setTextContent` 来设置 text，这里的 `nodeOps` 是一个适配层，根据不同平台提供不同的操作平台 DOM 的方法，实现跨平台。
 	4. 当新节点是非文本节点时，有四种情况：
     - 新子节点存在，如果老节点是文本节点，先将节点文本清除，将`ch`批量插入到节点elm下
     - 老子节点存在，将老节点通过removeVnodes清除
     - 当只有老节点且为文本节点时，清除节点文本内容
     - 当oldCh 与 ch都存在，且不同时，使用**updateChildren**函数来更新子节点

```js
function patchVnode (oldVnode, vnode) {
    if (oldVnode === vnode) {
        return;
    }

    if (vnode.isStatic && oldVnode.isStatic && vnode.key === oldVnode.key) {
        vnode.elm = oldVnode.elm;
        vnode.componentInstance = oldVnode.componentInstance;
        return;
    }

    const elm = vnode.elm = oldVnode.elm;
    const oldCh = oldVnode.children;
    const ch = vnode.children;

    if (vnode.text) {
        nodeOps.setTextContent(elm, vnode.text);
    } else {
        if (oldCh && ch && (oldCh !== ch)) {
            updateChildren(elm, oldCh, ch);
        } else if (ch) {
            if (oldVnode.text) nodeOps.setTextContent(elm, '');
            addVnodes(elm, null, ch, 0, ch.length - 1);
        } else if (oldCh) {
            removeVnodes(elm, oldCh, 0, oldCh.length - 1)
        } else if (oldVnode.text) {
            nodeOps.setTextContent(elm, '')
        }
    }
}
```



##### updateChildren

![image-20211030154034551](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20211030154034551.png)

定义 `oldStartIdx`、`newStartIdx`、`oldEndIdx` 以及 `newEndIdx` 分别是新老两个 VNode 的两边的索引，同时 `oldStartVnode`、`newStartVnode`、`oldEndVnode` 以及 `newEndVnode` 分别指向这几个索引对应的 VNode 节点。

while循环，oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx

1. 当oldStartVnode和oldEndVnode节点不存在时，start idx++  end idx--向中间靠拢，

2. 接下来这一块，是将 `oldStartIdx`、`newStartIdx`、`oldEndIdx` 以及 `newEndIdx` 两两比对的过程，一共会出现 2*2=4 种情况。

   - oldStartVnode于newStartVnode相同，两个的idx++
   - oldEndVnode与newEndVnode相同，两个idx--
   - oldStartVnode与newEndVnode相同，oldStartVnode插到oldEndVnode后面，oldstartIdx++,newEndIdx--
   - oldEndVnode与newStartVnode相同，oldEndVnode插到oldStartVnode前面，oldEndIdx--,newStartIdx++

   以上四种情况不符合时：

   `createKeyToOldIdx` 函数产生 `key` 与 `index` 索引对应的一个 map 表。根据key值，从oldKeyToIdx中获取相同的key节点的索引idxInOld，然后找到相同节点。

   - 如果没有找到相同的节点，则通过 `createElm` 创建一个新节点，并将 `newStartIdx` 向后移动一位。
   - 如果不符合 `sameVnode`，只能创建一个新节点插入到 `parentElm` 的子节点中，`newStartIdx` 往后移动一位。

当 `while` 循环结束以后，如果 `oldStartIdx > oldEndIdx`，说明老节点比对完了，但是新节点还有多的，需要将新节点插入到真实 DOM 中去，调用 `addVnodes` 将这些节点插入即可。

```js
function updateChildren (parentElm, oldCh, newCh) {
    let oldStartIdx = 0;
    let newStartIdx = 0;
    let oldEndIdx = oldCh.length - 1;
    let oldStartVnode = oldCh[0];
    let oldEndVnode = oldCh[oldEndIdx];
    let newEndIdx = newCh.length - 1;
    let newStartVnode = newCh[0];
    let newEndVnode = newCh[newEndIdx];
    let oldKeyToIdx, idxInOld, elmToMove, refElm;

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (!oldStartVnode) {
            oldStartVnode = oldCh[++oldStartIdx];
        } else if (!oldEndVnode) {
            oldEndVnode = oldCh[--oldEndIdx];
        } else if (sameVnode(oldStartVnode, newStartVnode)) {//四种情况
            patchVnode(oldStartVnode, newStartVnode);
            oldStartVnode = oldCh[++oldStartIdx];
            newStartVnode = newCh[++newStartIdx];
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode);
            oldEndVnode = oldCh[--oldEndIdx];
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldStartVnode, newEndVnode)) {
            patchVnode(oldStartVnode, newEndVnode);
            nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm));
            oldStartVnode = oldCh[++oldStartIdx];
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldEndVnode, newStartVnode)) {
            patchVnode(oldEndVnode, newStartVnode);
            nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
            oldEndVnode = oldCh[--oldEndIdx];
            newStartVnode = newCh[++newStartIdx];
        } else {//情况之外
            let elmToMove = oldCh[idxInOld];
            if (!oldKeyToIdx) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx);
            idxInOld = newStartVnode.key ? oldKeyToIdx[newStartVnode.key] : null;
            if (!idxInOld) {
                createElm(newStartVnode, parentElm);
                newStartVnode = newCh[++newStartIdx];
            } else {
                elmToMove = oldCh[idxInOld];
                if (sameVnode(elmToMove, newStartVnode)) {
                    patchVnode(elmToMove, newStartVnode);
                    oldCh[idxInOld] = undefined;
                    nodeOps.insertBefore(parentElm, newStartVnode.elm, oldStartVnode.elm);
                    newStartVnode = newCh[++newStartIdx];
                } else {
                    createElm(newStartVnode, parentElm);
                    newStartVnode = newCh[++newStartIdx];
                }
            }
        }
    }

    //结束循环
    if (oldStartIdx > oldEndIdx) {
        refElm = (newCh[newEndIdx + 1]) ? newCh[newEndIdx + 1].elm : null;
        addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx);
    } else if (newStartIdx > newEndIdx) {
        removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
    }
}
```



### Virtual DOM

render function会被转化成VNode节点。Virtual DOM 就是一颗以JavaScript对象（VNode节点）作为基础的树,用对象属性来描述节点，实际上它只是一层对真实DOM的抽象。最终可以通过**一系列操作**<font color="red">（这里的操作是patch？）</font>使这棵树映射到真实环境上。由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等。

### 批量异步更新和nextTick















