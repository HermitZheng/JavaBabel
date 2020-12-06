# 剑指Offer：11-19（Leetcode）

## 11. 旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

```
示例 1：
    输入：[3,4,5,1,2]
    输出：1
```

### 遍历

时间复杂度O(N)，空间复杂度O(1)

```java
        int temp = numbers[0];
        for (int i=1; i<len; i++) {
            if (numbers[i] >= temp) {
                temp = numbers[i];
            } else {
                return numbers[i];
            }
        }
		return numbers[0];			// 如果整个数组都是递增，那么数组的第一个位置就是最小值
```

### 二分查找

时间复杂度O(LogN)，空间复杂度O(1)

```java
        while (left < right) {
            int mid = (left + right) / 2;
            if (numbers[mid] > numbers[right]) {		// 说明[left, mid]是递增的，旋转点在[m+1, right]
                left = mid + 1;		// 注意：这里mid点不可能是最小值点（旋转点），所以可以m+1
            } else if (numbers[mid] < numbers[right]) {	// 说明[mid, right]是递增的，旋转点在[left, m]
                right = mid;		// 注意：这里mid点有可能是最小值点
            } else {
                right--;
            }
        }
        return numbers[left];
```



## 12. 矩阵中的路径

请设计一个函数，用来判断在一个矩阵中**是否存在一条包含某字符串所有字符的路径**。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径**不能再次进入该格子**。

例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[
    ["a","**b**","c","e"],
    ["s","**f**", "**c**","s"],
    ["a","d","**e**","e"]
]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，**路径不能再次进入这个格子**。

**示例 1：**

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

**示例 2：**

```
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false
```

**提示：**

- `1 <= board.length <= 200`
- `1 <= board[i].length <= 200`

### 递归DFS + 回溯（剪枝）

首先，遍历矩阵找到和字符串第一个字符相同的位置；然后使用DFS查找是否有一条路径存在所有字符，使用一个index来记录字符串word的检查位置。

在DFS的过程中，对于每一个搜索的位置，除了检查是否有**越界**之外，还要检查这个位置的字符是否和字符串中的相应位置的字符相同，以及**这个位置在这条路径中是否被访问过**，即设置一个visited数组来记录（也可以在原矩阵上进行记录），同时记得要进行**回溯**，因为这个visited记录只对这条路径有效。

当某次搜索时，发现index已经到头了`index == word.length()`，那么就可以判定这条路径存在所有的word字符，返回`true`。

- 深度优先搜索： 可以理解为暴力法遍历矩阵中所有字符串可能性。DFS 通过递归，先朝一个方向搜到底，再回溯至上个节点，沿另一个方向搜索，以此类推。
- 剪枝： 在搜索中，遇到**这条路不可能和目标字符串匹配成功的情况**（例如：此矩阵元素和目标字符不同、此元素已被访问），则应立即返回，称之为**可行性剪枝** 。

```java
    public boolean exist(char[][] board, String word) {
        char[] words = word.toCharArray();
        for (int i=0; i<board.length; i++) {
            for (int j=0; j<board[0].length; j++) {
                if (board[i][j] == words[0]) {	// 找到起始位置
                    if (dfs(board, i, j, words, 0)) return true;
                }
            }
        }
        return false;
    }
    public boolean dfs(char[][] board, int i, int j, char[] words, int index) {
        // board[i][j] > 256 这个判断是否visited的条件，已经包含在 board[i][j] != words[index++] 当中
        if (i < 0 || j < 0 || i >= board.length || j >= board[0].length || board[i][j] != words[index++]) {
            return false;
        }
        if (index >= words.length) return true; // 这条路径存在所有的word字符
        board[i][j] += 256;	// 记录这个位置已经被访问过了
        boolean res = dfs(board, i+1, j, words, index) || dfs(board, i, j+1, words, index) 
                        || dfs(board, i-1, j, words, index) || dfs(board, i, j-1, words, index);
        board[i][j] -= 256;	// 撤销访问记录，即回溯
        return res;
    }
```

**复杂度分析：**

- *M*,*N* 分别为矩阵行列大小， *K* 为字符串 `word` 长度。

- 时间复杂度O(3^k^MN)：

  - 对于每一次DFS的搜索过程中矩阵的某个位置，都有**除了上一个位置**以外的**三个位置**可以搜索，因此方案数的复杂度为O(3^k^)。
  - 对于整个矩阵，一共需要搜索*MN*次来找到所有可能的起点，因此时间复杂度为O(MN)

- 空间复杂度O(K)：

  只在递归时使用了栈空间，而当一整条路径搜索结束后，相应的栈空间将会被释放。最坏情况下*K* == *MN*，即O(MN)。



## 13. 机器人的运动范围

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也**不能进入行坐标和列坐标的数位之和大于k的格子**。

例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

**示例 1：**

```
输入：m = 2, n = 3, k = 1
输出：3
```

**示例 2：**

```
输入：m = 3, n = 1, k = 0
输出：1
```

**提示：**

- `1 <= n,m <= 100`	**注意：**这个条件即表示，**坐标最多只会是二位数**，进而`sum = i/10 + i%10`
- `0 <= k <= 20`

**（图片来自于本题的官方题解）**

通过图片可以观察到，在每一个小分区内，都存在着一条“划分线”将是否大于*K*的区域一分为二，左上角都小于等于*K*，而右下角都大于*K*。同时，也存在着一下区域虽然计算值小于等于*K*，但是与起始位置**并不连通**，如下图的右下角位置，即机器人无法到达。因此我们可以归纳出一些有用的规律：

- 使用**BFS或DFS**即可遍历到所有的连通区域
- 对于每一个位置，只需要向**右下**继续搜索即可

![image-20200703232114324](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200703232114324.png)

### BFS

使用**队列**来进行广度优先搜索。

```java
    public int movingCount(int m, int n, int k) {
        int[] dx = {1, 0}, dy = {0, 1};     // 右下的两个方向
        Deque<int[]> queue = new LinkedList<>();
        boolean[][] visited = new boolean[m][n];    // 记录该位置是否已访问，防止重复计数
        visited[0][0] = true;
        queue.offer(new int[] {0, 0});
        int res = 1;    // 已包括起始点(0, 0)

        while(!queue.isEmpty()){
            int[] pos = queue.poll();
            int x=pos[0], y=pos[1];
            for(int i=0; i<2; i++) {    // 向两个方向搜索
                int nextx = x+dx[i], nexty = y+dy[i];   
                // 没有越界，未曾访问，且满足数位和小于等于K
                if(nextx < m && nexty < n && !visited[nextx][nexty] && calculate(nextx, nexty) <= k) {   
                    res++;
                    visited[nextx][nexty] = true;
                    queue.offer(new int[] {nextx, nexty});
                }
            }
        }
        return res;
    }
```

- 时间复杂度O(MN)，最坏情况下机器人需要遍历所有的格子。

- 空间复杂度O(MN)，需要一个`visited[][]`来记录是否已访问。

### DFS

使用**递归**来进行深度优先搜索。（在Leetcode中，这个DFS比上面的BFS快许多）

```java
    public int movingCount(int m, int n, int k) {
        boolean[][] visited = new boolean[m][n];
        return dfs(m, n, 0, 0, k, visited);
    }

    private int dfs(int m, int n, int i, int j, int k,boolean[][] visited) {
        if (i >= m || j >= n || visited[i][j] || calculate(i, j) > k) {	// 不满足要求
            return 0;
        }
        visited[i][j] = true;
        // 向右下两个方向递归搜索，同时加上当前位置的一个计数
        return 1 + dfs(m, n, i+1, j, k, visited) + dfs(m, n, i, j+1, k, visited);
    }
```

时间复杂度O(MN)，空间复杂度O(MN)，皆同上。



## 14-1. 剪绳子

给你一根长度为 n 的绳子，请把绳子剪成**整数**长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m-1] 。请问 k[0]*k[1]*...*k[m-1] 可能的**最大乘积**是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。(2 <= n <= 58)

**示例 1：**

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1
```

**示例 2:**

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```

