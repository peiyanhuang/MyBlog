---
layout: post
title:  React diff 浅析
date:   2018-05-01 19:58:00 +0800
categories: React
tag: React
---

* content
{:toc}

### 传统的 diff 算法

如何计算一颗树形转化为另一棵树形结构的最少操作。传统的 diff 算法通过循环递归来对节点进行依次比较还计算一棵树到另一棵树的最少操作，算法复杂度为 `O(n^3)`，其中 n 是树中节点的个数。算法不复杂，但在前端渲染场景下，性能消耗太大了。

### diff 算法

diff 算法有如下三个策略：

1. DOM 节点跨层级的移动操作发生频率很低，可以忽略不计；
2. 拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构；
3. 对于同一层级的一组子节点，可以通过唯一 id 进行区分。

基于以上策略，React 也分别 `tree diff`、`component diff`、`element diff` 进行算法优化。

#### 1. tree diff

基于策略一，React 对树进行分层比较，即两棵树只会对同一层次的节点进行比较，忽略DOM节点跨层级的移动操作。

React 只会对相同层级的 DOM 节点进行比较，即同一个父节点下的所有子节点。当发现节点已经不存在，则该节点及其子节点会被完全删除掉，不会用于进一步的比较。这样只需要对树进行一次遍历，便能完成整个 DOM 树的比较。由此一来，最直接的提升就是复杂度变为线型增长而不是原先的指数增长。

如果真的发生跨层级移动，例如某个 DOM 及其子节点移动到另一个 DOM 下时，React 是不会机智的判断出子树仅仅是发生了移动，而是会直接销毁，并重新创建这个子树，然后再挂在到目标 DOM 上。事实上，React 官方也是建议不要做跨层级的操作。

在开发组件时，保持稳定的 DOM 结构会有助于性能的提升。例如，可以通过 CSS 隐藏或显示某些节点，而不是真的移除或添加 DOM 节点。

[Fiber](https://peiyanhuang.github.io/MyBlog/2018/05/15/virtual-dom/#5-%E6%B8%B2%E6%9F%93%E6%9B%B4%E6%96%B0)

#### 2. component diff

1. 如果是同类型组件，则按照原策略继续比较virtual DOM树；

2. 如果不是，则将该组件判断为 dirty component，然后整个 unmount 这个组件下的子节点对其进行替换；

3. 对于同类型组件，virtual DOM 可能并没有发生任何变化，这时我们可以通过 shouldCompoenentUpdate 钩子来告诉该组件是否进行 diff，从而提高大量的性能。

#### 3. element diff

* 无 key diff

比如同层有 A、B、C、D 四个节点，更新后包含 B、A、D、C，此时会比较新旧集合，比较 A != B，则创建并插入 B 至新集合，删除就集合 A，以此类推，创建并插入 A、D 和 C，删除 B、C、D。

* 有 key diff

所有同一层级的子节点，他们都可以通过 key 来区分。如图：

![image]({{ '/images/react/diff-1.png' | prepend: site.baseurl }})

1. nextChildren 的第一个元素是 B ，在 prevChildren 中的位置是 1（_mountIndex），nextIndex 和 lastIndex 都是 0。节点移动的前提是mountIndex < lastIndex，因此 B 不需要移动。lastIndex 更新为 _mountIndex 和 lastIndex 中较大的：1 。nextIndex 自增：1;
2. nextChildren 的第二个元素是 A ，中 prevChildren 中的位置是 0（_mountIndex），nextIndex 和 lastIndex 都是 1。这时_mountIndex < lastIndex，因此将 A 移动到 lastPlacedNode (B)的后面 。lastIndex 更新为 _mountIndex 和 lastIndex 中较大的：1 。nextIndex 自增：2；
3. nextChildren 的第三个元素是 D ，中 prevChildren 中的位置是 3（_mountIndex），nextIndex 是 2 ，lastIndex 是 1。这时不满足_mountIndex < lastIndex，因此 D 不需要移动。lastIndex 更新为 _mountIndex 和 lastIndex 中较大的：3 。nextIndex 自增：3；
4. nextChildren 的第四个元素是 C ，中 prevChildren 中的位置是 2（_mountIndex），nextIndex 是 3 ，lastIndex 是 3。这时_mountIndex < lastIndex，因此将 C 移动到 lastPlacedNode (D)的后面。循环结束。

### 4.Fiber Diff 更新

如果让我设计一个Diff算法，我首先想到的方案是：

先判断当前节点的更新属于哪种情况

1. 如果是新增，执行新增逻辑
2. 如果是删除，执行删除逻辑
3. 如果是更新，执行更新逻辑

按这个方案，其实有个隐含的前提——不同操作的优先级是相同的，但是React团队发现，在日常开发中，相较于新增和删除，更新组件发生的频率更高。所以Diff会优先判断当前节点是否属于更新。

在我们做数组相关的算法题时，经常使用双指针从数组头和尾同时遍历以提高效率，但是这里却不行。

虽然本次更新的 JSX 对象 `newChildren` 为数组形式，但是和 `newChildren` 中每个组件进行比较的是 `current fiber`，同级的 `Fiber` 节点是由 `sibling` 指针链接形成的单链表，即不支持双指针遍历。即 `newChildren[0]` 与 `fiber`比较，`newChildren[1]` 与 `fiber.sibling` 比较。所以无法使用双指针优化。

基于以上原因，Diff算法的整体逻辑会经历两轮遍历：

1. 第一轮遍历：处理`更新`的节点。
2. 第二轮遍历：处理剩下的不属于`更新`的节点。

#### 第一轮遍历

第一轮遍历步骤如下：

1. `let i = 0`，遍历 `newChildren`，`将newChildren[i]` 与 `oldFiber` 比较，判断DOM节点是否可复用。
2. 如果可复用，`i++`，继续比较 `newChildren[i]` 与 `oldFiber.sibling`，可以复用则继续遍历。
3. 如果不可复用，分两种情况：
 - `key`不同导致不可复用，立即跳出整个遍历，第一轮遍历结束。
 - `key`相同`type`不同导致不可复用，会将 `oldFiber` 标记为 `DELETION`，并继续遍历
4. 如果 `newChildren` 遍历完（即`i === newChildren.length - 1`）或者 `oldFiber` 遍历完（即 `oldFiber.sibling === null`），跳出遍历，第一轮遍历结束。

当遍历结束后，会有两种结果：

- 步骤3跳出的遍历: 此时 `newChildren` 没有遍历完，`oldFiber` 也没有遍历完。
- 步骤4跳出的遍历: 可能 `newChildren` 遍历完，或 `oldFiber` 遍历完，或他们同时遍历完。

#### 第二轮遍历

1. `newChildren` 与 `oldFiber` 同时遍历完，那就是最理想的情况：只需在第一轮遍历进行组件更新。此时 Diff 结束
2. `newChildren` 没遍历完，`oldFiber` 遍历完：已有的DOM节点都复用了，这时还有新加入的节点，意味着本次更新有新节点插入，我们只需要遍历剩下的 `newChildren` 为生成的 `workInProgress fiber` 依次标记 `Placement`
3. `newChildren` 遍历完，`oldFiber` 没遍历完：意味着本次更新比之前的节点数量少，有节点被删除了。所以需要遍历剩下的 `oldFiber`，依次标记 `Deletion`
4. `newChildren` 与 `oldFiber` 都没遍历完：这意味着有节点在这次更新中改变了位置，是接下来的重点

我们需要使用`key`，为了快速的找到`key`对应的`oldFiber`，我们将所有还未处理的`oldFiber`存入以`key`为`key`，`oldFiber`为`value`的`Map`中。

```js
const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
```

接下来遍历剩余的 `newChildren`，通过 `newChildren[i].key` 就能在 `existingChildren` 中找到 `key` 相同的 `oldFiber`。

最后一个可复用的节点在 `oldFiber` 中的位置索引（用变量`lastPlacedIndex`表示）。

由于本次更新中节点是按 `newChildren` 的顺序排列。在遍历 `newChildren` 过程中，每个遍历到的可复用节点一定是当前遍历到的所有可复用节点中最靠右的那个，即一定在 `lastPlacedIndex` 对应的可复用的节点在本次更新中位置的后面。

那么我们只需要比较遍历到的可复用节点在上次更新时是否也在 `lastPlacedIndex` 对应的 `oldFiber` 后面，就能知道两次更新中这两个节点的相对位置改变没有。

我们用变量 `oldIndex` 表示遍历到的可复用节点在 `oldFiber` 中的位置索引。如果 `oldIndex < lastPlacedIndex`，代表本次更新该节点需要向右移动。

`lastPlacedIndex`初始为`0`，每遍历一个可复用的节点，如果`oldIndex >= lastPlacedIndex`，则`lastPlacedIndex = oldIndex`。
