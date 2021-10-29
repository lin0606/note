# Vue3Diff

## 为什么需要diff

我们用Javascript直接操作DOM的计算成本很高。虽然用Javascript执行更新很快，但是找到所需要的的DOM节点并用Javascript更新他们的成本却很高。所以我们需要批量处理调用，并一次性更新DOM。批次更新的关键便是虚拟DOM(Virtual DOM)，我们也可以称之为DOM的副本。

虚拟DOM是轻量级的Javascript对象，由渲染函数创建。它包含三个参数：

* 元素
* 具有数据、prop、attr等的对象
* 子级数组

在更新列表项，我们可以借助[响应性](https://github.com/lin0606/note/blob/main/Vue/vue3MVVM.md)在Javascript中进行。我们将更新应用至虚拟DOM中，然后在它们和实际DOM之间执行diff。只有这样，我们才能对已更新的内容进行更新。虚拟DOM允许我们对UI进行高效的更新。



