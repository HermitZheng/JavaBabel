# 剑指Offer：3-10（Leetcode）

## 3. 数组中重复的数字

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的**所有数字都在 0～n-1 的范围内**。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```

**限制：**

```
2 <= n <= 100000
```

### 字典

需要**O(N)**的空间复杂度，遍历构建字典需要**O(N)**的时间复杂度，字典的查找为O(1)。

### 排序

原地快速排序需要**O(NLogN)**的时间复杂度，再进行一遍O(N)的遍历。

### 原地“哈希”（抽屉原理）

通过交换```nums[i]、nums[nums[i]]```使得**原数组的第```i```个位置存放```i```这个数**（所有数字都在 0～n-1 的范围内，因此不会越界）,当```nums[i] == nums[nums[i]]```时，则为重复。

```java
        for (int i=0; i<nums.length; i++) {     // 所有数字都在 0～n-1 的范围内
            while (nums[i] != i) {              // 要求数组中第i个位置的数为i
                int temp = nums[i];
                if (nums[i] == nums[temp]) {    // 两个位置的值相同
                    return temp;
                }
                nums[i] = nums[temp];           // 将下一个位置(temp)的值赋给i位置
                nums[temp] = temp;              // 实为交换
            }
        }
```



## 4. 二维数组中的查找

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例:	现有矩阵 matrix 如下：

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
给定 target = 5，返回 true。
给定 target = 20，返回 false。
```

限制：

0 <= n <= 1000

0 <= m <= 1000

### 遍历、DFS、BFS

直接从上到下从左到右遍历，时间复杂度O(N*M)，但是没用到递增的性质。

### 从右上角开始（或者左下角）

**对于每一个```(i, j)```的位置，```i+1```的值更大，而``j-1``的值更小**；或者说可以看做是一颗二叉搜索树。

```java
        while (i < rows && j >= 0) {
            if (matrix[i][j] == target) return true;
            if (matrix[i][j] > target) {		// 比目标大则左移
                j--;
            }
            else if (matrix[i][j] < target) {	// 比目标小则下移
                i++;
            }
        }
```



## 5. 替换空格

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

```
示例 1：

输入：s = "We are happy."
输出："We%20are%20happy."
```

使用**StringBuilder**

```java
        StringBuilder sb = new StringBuilder();
        for (int i=0; i<s.length(); i++) {
            if (s.charAt(i) == ' ') {
                sb.append("%20");
            } else {
                sb.append(s.charAt(i));
            }
        }
        return sb.toString();
```



## 6. 从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

**限制：**

```
0 <= 链表长度 <= 10000
```

### 栈、递归

时间复杂度O(N)，空间复杂度O(N)

### 翻转链表

时间复杂度O(N)，空间复杂度O(1)

```java
        ListNode temp = null, point = head, next;
        int count = 0;
        while (point != null) {
            count++;
            next = point.next;
            point.next = temp;
            temp = point;
            point = next;
        }
		// 简单来做，其实只要记个数，反着存res就好了
        int count = 0;
        ListNode temp = head;
        while (temp != null) {
            count++;
            temp = temp.next;
        }
        int[] res = new int[count];
        for (int i=count-1; i>=0; i--) {
            res[i] = head.val;
            head = head.next;
        }
```



## 7. 重建二叉树（前序+中序）

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都**不含重复的数字**。

 ```
例如，给出

前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
 ```


返回如下的二叉树：

    	3
       / \
      9  20
        /  \
       15   7
```
限制：
0 <= 节点个数 <= 5000
```

### 递归

利用前序和中序的性质对序列进行分割，例如：

```java
前序遍历 preorder = [3 | 9 | 20,15,7]
中序遍历 inorder = [9 | 3 | 15,20,7]
// 接下来
前序遍历 preorder = [20 | 15 | 7]
中序遍历 inorder = [15 | 20 | 7]

使用 Arrays.copyOfRange(arr, begin, end) 将左右的序列分别交给递归去做
```

```java
int j = 0;
TreeNode root = new TreeNode(preorder[0]);
while (inorder[j] != preorder[0]) {
    j++;
}
root.left = buildTree(Arrays.copyOfRange(preorder, 1, j+1), 														Arrays.copyOfRange(inorder, 0, j));
root.right = buildTree(Arrays.copyOfRange(preorder, j+1, len), 														Arrays.copyOfRange(inorder, j+1, len));
return root;
```

时间复杂度O(N)：对于每个结点都有创建，以及创建其左右子树的过程

空间复杂度O(N)：存储整棵树的结点

### 迭代（更快）

时间复杂度O(N)，空间复杂度O(N)

- 使用前序遍历的第一个元素创建根节点。
- 创建一个栈，将根节点压入栈内。
- 初始化中序遍历下标为 0。
- 遍历前序遍历的每个元素，判断其上一个元素（即栈顶元素）是否等于中序遍历下标指向的元素。
  - 若上一个元素不等于中序遍历下标指向的元素，则将当前元素作为其上一个元素的左子节点，并将当前元素压入栈内。
  - 若上一个元素等于中序遍历下标指向的元素，则从**栈内弹出一个元素（相当于回溯）**，同时令中序遍历下标指向下一个元素，之后继续判断栈顶元素是否等于中序遍历下标指向的元素
    - 若相等则重复该操作，直至栈为空或者元素不相等。然后令当前元素为最后一个相等元素的右节点。
      遍历结束，返回根节点。

```java
        Stack<TreeNode> stack = new Stack<>();
        TreeNode root = new TreeNode(preorder[0]);
        stack.push(root);
        int inorder_index = 0;

        for (int i = 1; i < len; i++) {
            TreeNode node = stack.peek();
            if (node.val != inorder[inorder_index]) {
                node.left = new TreeNode(preorder[i]);
                stack.push(node.left);
            } else {
                // 回溯找到上一个具有右子树的结点
                while (!stack.isEmpty() && stack.peek().val == inorder[inorder_index]) {
                    node = stack.pop();		// 保存pop
                    inorder_index++;
                }
                node.right = new TreeNode(preorder[i]);
                stack.push(node.right);
            }
        }
```



## 9. 用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在**队列尾部插入整数**和在**队列头部删除整数**的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

```java
    Stack<Integer> in;		// 入栈时in.push()
    Stack<Integer> out;		// 出栈时out.push(in.pop())，再out.pop()
    int size;				// 记录“队列”中的元素个数
```

```java
	public void appendTail(int value) {	// 入队
        in.push(value);
        size++;
    }
    
    public int deleteHead() {			// 出队
        if (size == 0) return -1;		// 通过size判断比isEmpty更快
        if (out.isEmpty()) {
            while (!in.isEmpty()) {
                out.push(in.pop());
            }
        }
        size--;
        return out.pop();
    }
```

插入和删除的时间复杂度都为O(1)，空间复杂度为O(N)



## 10-1. 斐波那契数列

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```

斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

**答案需要取模 1e9+7（1000000007）**，如计算初始结果为：1000000008，请返回 1。

```
示例 1：
    输入：n = 2
    输出：1
```

**提示：**

- `0 <= n <= 100`

### 递归

```java
return fib(n-1) + fib(n-2)
```

### 记忆化递归

使用一个额外的数组来记录n以前的所有结果，空间复杂度O(N)：

```java
    int[] mem = new int[101];
```

```java
    // 递归
    public int fib(int n) {
        if (n <= 1) return n;
        if (mem[n] != 0) return mem[n];
        int res = (fib(n-1) + fib(n-2)) % 1000000007;
        mem[n] = res;
        return res;
    }
```

### 动态规划

时间复杂度O(N) ，优化后只使用标记变量空间复杂度O(1)

```java
    dp[0] = 0;
    dp[1] = 1;
    for (int i=2; i<=n; i++) {
        dp[i] = (dp[i-2] + dp[i-1]) % 1000000007;
    }

    // 不使用数组，节省空间
    int dp_1 = 1, dp_2 = 0, dp = 0;
    for (int i=2; i<=n; i++) {
        dp = (dp_1 + dp_2) % 1000000007;
        dp_2 = dp_1;
        dp_1 = dp;
    }
```



## 10-2 青蛙跳台阶问题

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

```
示例 1：
    输入：n = 2
    输出：2
```

**提示：**

- `0 <= n <= 100`

### 记忆化递归

可以使用HashMap来做缓存（也可以用数组）

```java
    private Map<Integer, Integer> map = new HashMap<>();
    public int numWays(int n) {
        if (n == 0) return 1;
        if (n == 1) return 1;
        if (map.get(n) == null) {
            map.put(n, numWays(n-1) + numWays(n-2));
        }
        return map.get(n) % 1000000007;
    }
```

### 动态规划

时间复杂度O(N)，空间复杂度O(1)

```java
    public int numWays(int n) {
        if(n == 0) return 1;
        if(n <= 2) return n;
        int dp_1 = 2;	// 命名规则为dp[i-1] -> dp_1
        int dp_2 = 1, dp = 0;
        for (int i=3; i<=n; i++) {
            dp = (dp_1 + dp_2) % 1000000007;
            dp_2 = dp_1;
            dp_1 = dp;
        }
        return dp;
    }
```

