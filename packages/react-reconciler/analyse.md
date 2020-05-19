react fiber 版本的 diff 算法是从ReactFiberBeginWork.js文件的reconcileChildren方法开始的；

`reconcileChildren`只是个入口，用于分辨这个节点的更新是新增(mountChildFibers) 还是更新(reconcileChildFibers)


diff 算法的核心api 都是在 ReactChildFiber.js 中定义的，新增(mountChildFibers) 和 更新(reconcileChildFibers) 来源于一个函数ChildReconciler->reconcileChildFibers，加了个闭包；所以两个api的入参是一样的，当给 ChildReconciler 输入true时表示更新，否则为新增；

入参：
```js
/**
 * returnFiber: workInprogressTree，是即将 Diff 的这层的父节点。
 * currentFirstChild: oldTree 是当前层的第一个 Fiber 节点。
 * newChild：是即将更新的 vdom 节点(可能是 TextNode、可能是 ReactElement，可能是数组)，不是 Fiber 节点
 * expirationTime：和diff没有关系
**/
function reconcileChildFibers(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    newChild: any,
    expirationTime: ExpirationTime,
) {

}
```
![newChild 单节点样子](https://doddle.oss-cn-beijing.aliyuncs.com/oldNotes/20200516230533.png)

![三个输入的样子](https://doddle.oss-cn-beijing.aliyuncs.com/oldNotes/20200516230734.png)

![newChild 节点为数组的样子](https://doddle.oss-cn-beijing.aliyuncs.com/oldNotes/20200517102002.png)

### 重点研究reconcileChildrenArray
这个方法就是diff中最复杂的，列表diff，存在有key和无key列表

**<font color="yellow">数组的diff 过程,从大步骤来讲，包含了四步：**</font> 
 - 先两个数组（老的是链表，新的是数组）直接对比相同位置(index)的节点，依赖`updateSlot`方法，若key相同且新旧节点的type一致，则复用节点，否则返回null，数组直接提前结束；
 - 第一步结束后，判断是否新节点都已找到可复用节点，是的话，完美结局；删除剩余的老节点，旅行结束
 - 新节点还没有遍历完，判断老节点是否已到底。到底说明能找的遍历节点都找完；那剩余还没找到的新节点都需要做新增
 - 如果前两步都不是，那就属于数组对比复用提前结束的情况；需要遍历剩余老节点链表，新建一个Map, 键为节点的key或index，值为节点Fiber; 遍历剩余未找到复用的新节点，看Map中是否有可复用的节点，有的话就复用，没有就新增，依赖`updateFromMap`方法 

 >上面只提到了节点的复用，但没有说复用后的节点是插入还是移动，还是原地不动。每个节点被创建时都会调用placeChild，如果老位置在当前已放置位置前，则需要移动复用节点，否则就可以保持不动，但需要把已放置位置更新，最新值为老位置；新增节点一律都是新增，且不占用已放置位置




