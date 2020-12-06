# 二叉树、AVL树、红黑树

树是一种**非线性**的数据结构，相对于线性的数据结构(链表、数组)而言，**树的平均运行时间更短(往往与树相关的排序时间复杂度都不会高)**。树具有递归性质。

## 二叉树

### 基本性质

1. 有且仅有一个称之为根的结点
2. 二叉树的每个结点至多只有两颗子树（不存在度大于2的结点）
3. 二叉树的子树具有左右之分，次序不能随意颠倒

```java
public class TreeNode {  // 一个简单的二叉树结构

    int value = 0;
    TreeNode left = null;
    TreeNode right = null;
}
```

二叉树的结点可以定义为：**一个数据、两个指针(如果有节点就指向节点、没有节点就指向null)**

### 二叉树的遍历

**二叉树遍历有三种方式**

- 先序遍历
  - 先访问根节点，然后访问左节点，最后访问右节点(根->左->右)
- 中序遍历
  - 先访问左节点，然后访问根节点，最后访问右节点(左->根->右)
- 后序遍历
  - 先访问左节点，然后访问右节点，最后访问根节点(左->右->根)

 **如果被访问的结点有子结点，则先访问子结点（递归遍历），当被访问的结点没有子结点了，就递归返回。**

```java
public void preOrder(TreeNode root){  // 先序遍历
    if (root != null){
        System.out.print(root.value + " ");   // 中后序遍历只需要改变输出语句的位置即可
        preOrder(root.left);
        preOrder(root.right);
    }
}
```

若二叉树中各结点的值均不相同，任意一棵二叉树结点的先序、中序和后序遍历都是**唯一**的。

反过来，**由二叉树的先序和中序序列，或者中序和后序序列，都能唯一的确定一颗二叉树。**

```java
public class reConstruct {     // 重建二叉树
	// pre为先序序列， in为中序序列
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {   
        if (pre == null || in == null) {
            return null;
        }
        TreeNode root = recursive(pre, 0, pre.length-1, in, 0, in.length-1);
        return root;
    }

    public TreeNode recursive(int[] pre, int startPre, int endPre,
                              int[] in, int startIn, int endIn) {
        if (startPre > endPre || startIn > endIn) {
            return null;
        }
        TreeNode node = new TreeNode(pre[startPre]);   // 先序遍历的第一个元素为根节点
        for (int i = startIn; i <= endIn; i++) {   // 循环直到在中序遍历中找到根节点
            if (in[i] == pre[startPre]) {
                /**
                 *  对于先序遍历，左子树是根节点（startPre）后一位开始，数量是中序遍历的开始位置						 *  startIn到中序的根节点 i 的元素数量
                 *  对于中序遍历，左子树是从startIn开始，到 i 为止（即 i-1）
                 */
                node.left = recursive(pre, startPre+1, startPre-startIn+i, 
                                      in, startIn, i-1);
                /**
                 *  对于先序遍历，右子树是从左子树尾（即上文的endPre+1）开始，直到列表尾部（endPre）
                 *  对于中序遍历，右子树是从根节点（ i+1）开始，直到列表尾部
                 */
                node.right = recursive(pre, startPre-startIn+i+1, endPre, 
                                       in, i+1, endIn);
                break;
            }
        }
        return node;
    }
}
```

## 二叉搜索树(binary search tree)

二叉搜索树的性质：**当前根节点的左边全部比根节点小，当前根节点的右边全部比根节点大**。

![image-20200315204102801](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200315204102801.png)

通过这个性质，二叉搜索树可以很方便的查找一个数。在二叉树节点个数确定的情况下，**整颗树的高度越低，节点的查询复杂度越低**。存在两种极端情况：

- 如果二叉排序树是**平衡**的，则n个节点的二叉排序树的高度为**Log2n+1**，其查找效率为**O(Logn)**，近似于**折半查找**

- 如果二叉排序树**完全不平衡**，则其深度可达到**n**，查找效率为**O(n)**，退化为**顺序查找**。

对二叉搜索树使用**中序遍历**得到的序列是排好序的（从小到大）。

### 性能分析

由以上查询复杂度、构造复杂度和删除复杂度的分析可知，三种操作的时间复杂度皆为 ![O(log_2 n)](https://math.jianshu.com/math?formula=O(log_2%20n))~![O(n)](https://math.jianshu.com/math?formula=O(n))。下面分析线性结构的三种操作复杂度，以二分法为例：

- 查询复杂度，时间复杂度为 ![O(log_2 n)](https://math.jianshu.com/math?formula=O(log_2%20n))，优于二叉搜索树；
- 元素的插入操作包括两个步骤，查询和插入。查询的复杂度已知，插入后调整元素位置的复杂度为 ![O(n)](https://math.jianshu.com/math?formula=O(n))，即单个元素的构造复杂度为：![O(n)](https://math.jianshu.com/math?formula=O(n))
- 删除操作也包括两个步骤，查询和删除，查询的复杂度已知，删除后调整元素位置的复杂度为 ![O(n)](https://math.jianshu.com/math?formula=O(n))，即单个元素的删除复杂度为：![O(n)](https://math.jianshu.com/math?formula=O(n))

由此可知，二叉搜索树相对于线性结构，在构造复杂度和删除复杂度方面占优；在查询复杂度方面，二叉搜索树可能存在类似于斜树，每层上只有一个节点的情况，该情况下查询复杂度不占优势。

## AVL树

AVL树，是一种平衡(balanced)的二叉搜索树(binary search tree)。

```java
public class AVLNode {  // 简单的AVL结构
    
    int value;
    int depth;   // 深度，这里计算每个结点的深度，通过深度的比较可得出是否平衡
    AVLNode parent;
    AVLNode left;
    AVLNode right;
}
```

它具有以下两个性质：

- AVL树是一棵二叉搜索树，拥有二叉搜索树的全部基本性质
- 每个节点的左右子节点的**高度之差**的绝对值最多为1，即平衡因子为范围为[-1,1]。

它通过一定的机制能保证二叉搜索树的平衡，而平衡的二叉搜索树的查询效率更高。

### 平衡因子

**定义：**某节点的左子树与右子树的高度(深度)差即为该节点的平衡因子（BF,Balance Factor），平衡二叉树中不存在平衡因子大于 1 的节点。在一棵平衡二叉树中，**节点的平衡因子只能取 0 、1 或者 -1 ，分别对应着左右子树等高，左子树比较高，右子树比较高。**

### 旋转

AVL树的出现就是为了解决平衡性问题，在每一次插入数值之后，树的平衡性都可能被破坏，它的核心内容就是平衡处理机制，即所谓的旋转。一共有四种形式的旋转：右单旋、左单旋、左右双旋和右左双旋。

| 插入方式 | 描述                                       | 旋转方式 |
| -------- | ------------------------------------------ | -------- |
| LL       | 在左子树根节点的左子树上插入节点而破坏平衡 | 右旋转   |
| RR       | 在右子树根节点的右子树上插入节点而破坏平衡 | 左旋转   |
| LR       | 在左子树根节点的右子树上插入节点而破坏平衡 | 左右双旋 |
| RL       | 在右子树根节点的左子树上插入节点而破坏平衡 | 右左双旋 |

以下的AVL树，当3被插入时破坏了平衡，插入方式是**LL**插入，因此要找到离新插入结点最近的不平衡的树进行调整，也就是7。7结点的左子树高度为2，右子树为空，高度为0，因而不平衡，需要进行右旋转。

![image-20200315213900297](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200315213900297.png)



以下的AVL树，当6被插入时破坏了平衡，插入方式是**LR**插入，离新插入结点最近的不平衡的树是7。

![image-20200315214503869](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200315214503869.png)

显然，经过旋转的子树依旧要满足二叉搜索树的左小右大性质。我们可以这样理解：以上图为例，将5称为旋转结点的话，经过**左旋**后的旋转结点必然在**左分支**，此时6经过**逆时针**旋转成为了旋转结点5的父节点；同样的，7作为旋转结点经过了**右旋**后成为了6的**右分支**，此时6经过**顺时针**旋转成为了旋转结点7的父节点。

https://www.cs.usfca.edu/~galles/visualization/AVLtree.html（一个演示AVL旋转的页面）

这是一个数据结构可视化工具，可以动态地看到AVL是如何完成旋转平衡的。



## 红黑树

红黑树本质上是一种二叉查找树，但它在二叉查找树的基础上额外添加了一个标记（颜色），同时具有一定的规则。这些规则使红黑树保证了一种平衡，插入、删除、查找的**最坏时间复杂度都为 O(logn)。**

在 Java 集合框架中，很多部分(HashMap, TreeMap, TreeSet 等)都有红黑树的应用。

https://www.cs.usfca.edu/~galles/visualization/RedBlack.html（一个演示红黑树自平衡的页面）

![image-20200326233557138](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200326233557138.png)

### 特性

1. 每个节点要么是红色，要么是黑色；
2. **根节点永远是黑色的；**
3. 所有的叶节点都是黑色的（注意这里说叶子节点其实是 NIL 节点）；
4. **每个红色节点的两个子节点一定都是黑色** (从每个叶子到根的所有路径上不能有两个连续的红色节点)
5. **从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；**

红黑树使用两种方法来对节点做自平衡：

- **旋转**：顺时针旋转和逆时针旋转（左旋和右旋）
- **反色**：交换一个节点的红黑颜色



