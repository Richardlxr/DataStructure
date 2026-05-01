### 朴素算法
#### 过程

可以每次找深度比较大的那个点，让它向上跳．显然在树上，这两个点最后一定会相遇，相遇的位置就是想要求的 LCA． 或者先向上调整深度较大的点，令他们深度相同，然后再共同向上跳转，最后也一定会相遇．

#### 性质

朴素算法预处理时需要 dfs 整棵树，时间复杂度为 𝑂(𝑛)![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "O(n)")，单次查询时间复杂度为 Θ(𝑛)![1](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "\Theta(n)")．如果树满足随机性质，则时间复杂度与这种随机树的期望高度有关．
```c


```