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