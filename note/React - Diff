# React Diff 算法
React Diff 算法就是在某一组件更新时，将该组件更新前和更新后虚拟DOM树做对比，尽可能多的复用其中的树节点，从而达到高效的更新组件的过程。

传统的 Diff 算法最优的解决方案也需要 O(n<sup>3</sup>) 的算法复杂度，开销太大。所以 React 在其中做了取舍，并提出了一套 O(n) 算法复杂度的启发式算法，它遵循下面几个规则：
- 该算法不会尝试匹配不同组件类型的子树。如果你发现你在两种不同类型的组件中切换，但输出非常相似的内容，建议把它们改成同一类型。实际上这种情况往往不会发生。
- 提出Key的概念，Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 Math.random() 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失。

React Diff 算法在此基础上大致上分为 `单节点Diff` 和 `多节点Diff` 两类，下面我们分别讨论
> 你可以在[这里](https://zh-hans.reactjs.org/docs/reconciliation.html#motivation)看到 React Diff 算法的设计动机

## 单节点 Diff
单节点的情况十分简单，基于上文提到的两个规则，我们只需判断更新前后的两个节点 `key` 和 `type` 是否一致即可确定该节点是否可复用。

## 多节点 Diff
多节点 Diff 时整体逻辑会经历两轮遍历：

第一轮遍历
第一轮遍历步骤如下：

let i = 0，遍历newChildren，将newChildren[i]与oldFiber比较，判断DOM节点是否可复用。

如果可复用，i++，继续比较newChildren[i]与oldFiber.sibling，可以复用则继续遍历。

如果不可复用，分两种情况：

key不同导致不可复用，立即跳出整个遍历，第一轮遍历结束。

key相同type不同导致不可复用，会将oldFiber标记为DELETION，并继续遍历

如果newChildren遍历完（即i === newChildren.length - 1）或者oldFiber遍历完（即oldFiber.sibling === null），跳出遍历，第一轮遍历结束。
你可以从这里 (opens new window)看到这轮遍历的源码

当遍历结束后，会有两种结果：

#步骤3跳出的遍历
此时newChildren没有遍历完，oldFiber也没有遍历完。

#步骤4跳出的遍历
可能newChildren遍历完，或oldFiber遍历完，或他们同时遍历完。

第二轮遍历
对于第一轮遍历的结果，我们分别讨论：

#newChildren与oldFiber同时遍历完
那就是最理想的情况：只需在第一轮遍历进行组件更新 (opens new window)。此时Diff结束。

#newChildren没遍历完，oldFiber遍历完
已有的DOM节点都复用了，这时还有新加入的节点，意味着本次更新有新节点插入，我们只需要遍历剩下的newChildren为生成的workInProgress fiber依次标记Placement。



#newChildren遍历完，oldFiber没遍历完
意味着本次更新比之前的节点数量少，有节点被删除了。所以需要遍历剩下的oldFiber，依次标记Deletion。



#newChildren与oldFiber都没遍历完
这意味着有节点在这次更新中改变了位置。

这是Diff算法最精髓也是最难懂的部分。我们接下来会重点讲解。

处理移动的节点
由于有节点改变了位置，所以不能再用位置索引i对比前后的节点，那么如何才能将同一个节点在两次更新中对应上呢？

我们需要使用key。

为了快速的找到key对应的oldFiber，我们将所有还未处理的oldFiber存入以key为key，oldFiber为value的Map中。

const existingChildren = mapRemainingChildren(returnFiber, oldFiber);


接下来遍历剩余的newChildren，通过newChildren[i].key就能在existingChildren中找到key相同的oldFiber。

#标记节点是否移动
既然我们的目标是寻找移动的节点，那么我们需要明确：节点是否移动是以什么为参照物？

我们的参照物是：最后一个可复用的节点在oldFiber中的位置索引（用变量lastPlacedIndex表示）。

由于本次更新中节点是按newChildren的顺序排列。在遍历newChildren过程中，每个遍历到的可复用节点一定是当前遍历到的所有可复用节点中最靠右的那个，即一定在lastPlacedIndex对应的可复用的节点在本次更新中位置的后面。

那么我们只需要比较遍历到的可复用节点在上次更新时是否也在lastPlacedIndex对应的oldFiber后面，就能知道两次更新中这两个节点的相对位置改变没有。

我们用变量oldIndex表示遍历到的可复用节点在oldFiber中的位置索引。如果oldIndex < lastPlacedIndex，代表本次更新该节点需要向右移动。

lastPlacedIndex初始为0，每遍历一个可复用的节点，如果oldIndex >= lastPlacedIndex，则lastPlacedIndex = oldIndex。
