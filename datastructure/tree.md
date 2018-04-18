## 名词定义
- tree
- edge
- node(value/data) parentNode childNode leaves
- height of the tree
- depth of a node

## 二叉树
### node属性
- value
- left child
- right child
### 插入
- 如果当前的结点没有left child，我们就创建一个新结点，然后将其设置为当前结点的left child
- 如果已经有了左结点，我们就创建一个新结点，并将其放在当前left child的位置。然后再将原左结点值为新left child的left child
- 右边同理
### 深度优先遍历DFS
- 前序遍历
  - 输出节点的值
  - 进入其左结点并输出。当且仅当它拥有左结点。
  - 进入右结点并输出之。当且仅当它拥有右结点
- 中序遍历 左结点优先，之后是中间，最后是右结点
- 后序遍历 左结点优先，之后是右结点，根结点的最后
- 广度优先遍历BFS
