### leetcode

#### hash相关

##### q1 两数之和

```
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例:
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

- 暴力解法

  ```java
  class Solution {
      public int[] twoSum(int[] nums, int target) {
          for(int i = 0; i< nums.length; i++){
              for(int j = i + 1; j < nums.length; j++){
                  if(nums[i] + nums[j] == target){
                      return new int[] {i, j};
                  }
              }            
          }
          return null;
      }
  }
  ```

- 使用hash

  构建hashset，key为数组元素的值，value为数组元素的索引，遍历时判断hashset中是否存在`target-nums[i]`的键。

  但使用hashset的问题是数组中存在重复元素时，hashset无法存在重复的key。对于`[3,3]`这类数组，这段只想通过一次遍历完成的代码无法解决。

  ```java
  public int[] twoSum(int[] nums, int target) {
  	if (nums == null && nums.length < 2)
  		return null;
  	Map<Integer, Integer> map = new HashMap<>();
      for (int i = 0; i < nums.length; i++) {
          map.put(nums[i], i);
          int another = target - nums[i];
          if (map.containsKey(another)) {
              int j = map.get(another);
              if(j != i)
                  return new int[]{j, i};
          }
      }
      return null;
  }
  ```

  二次遍历可以完成，虽然重复的key在第一次遍历时会更新value，但第二次从头遍历时i和j是不相同的，因此可以找到。

  ```java
  public int[] twoSum2(int[] nums, int target) {
          if (nums == null && nums.length < 2)
              return null;
          Map<Integer, Integer> map = new HashMap<>();
          for (int i = 0; i < nums.length; i++) {
              map.put(nums[i], i);
          }
          for (int i = 0; i < nums.length; i++) {
              int another = target - nums[i];
              if (map.containsKey(another)) {
                  int j = map.get(another);
                  if (j != i)
                      return new int[]{i, j};
              }
          }
          return null;
      }
  ```

  但其实一次遍历是能完成任务的，关键在于执行顺序，先判断后添加（对同样的key是更新），这样才能保证i和j不相同。

  ```java
  public int[] twoSum(int[] nums, int target) {
  	if (nums == null && nums.length < 2)
  		return null;
  	Map<Integer, Integer> map = new HashMap<>();
      for (int i = 0; i < nums.length; i++) {
          int another = target - nums[i];
          if (map.containsKey(another)) {
              int j = map.get(another);
              if(j != i)
                  return new int[]{j, i};
          }
      	map.put(nums[i], i);
      }
      return null;
  }
  ```

- 边界情况：
  - 数组为null或长度为1
  - target为其中一个元素的2倍
  - target为两个相同元素的和

#### 链表相关

##### q2 两数相加

```
给出两个非空的链表用来表示两个非负的整数。其中，它们各自的位数是按照逆序的方式存储的，并且它们的每个节点只能存储一位数字。
如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

- 原始解法

  ```java
  /**
   * Definition for singly-linked list.
   * public class ListNode {
   *     int val;
   *     ListNode next;
   *     ListNode(int x) { val = x; }
   * }
   */
  class Solution {
      public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
          
          int carry = 0;
          ListNode lastNode = null;
          ListNode cur = null;
          while (l1 != null && l2 != null) {
              int val = l1.val + l2.val + carry;
              if (val >= 10) {
                  val -= 10;
                  carry = 1;
              } else {
                  carry = 0;
              }
              cur = new ListNode(val);
              cur.next = lastNode;
              lastNode = cur;
              l1 = l1.next;
              l2 = l2.next;
          }
          
          while(l1 != null) {
              int val = l1.val + carry;
              if (val >= 10) {
                  val -= 10;
                  carry = 1;
              } else {
                  carry = 0;
              }
              cur = new ListNode(val);
              cur.next = lastNode;
              lastNode = cur;
              l1 = l1.next;
          }
  
          while(l2 != null) {
              int val = l2.val + carry;
              if (val >= 10) {
                  val -= 10;
                  carry = 1;
              } else {
                  carry = 0;
              }
              cur = new ListNode(val);
              cur.next = lastNode;
              lastNode = cur;
              l2 = l2.next;
          }
  
          if (carry != 0) {
              cur = new ListNode(carry);
              cur.next = lastNode;
          }
          return reverse(cur);
      }
  
      public ListNode reverse(ListNode h) {
          if (h.next == null)
              return h;
          ListNode t1 = h;
          ListNode t2 = h.next;
          ListNode t3 = h.next.next;
          while (t2 != null) {
              t2.next = t1;
              t1 = t2;
              t2 = t3;
              if (t3 != null)
                  t3 = t3.next;
          }
          h.next = null;
          return t1;
      }
  }
  ```

  按位相加，记录进位，每次创建新的节点指向上一次创建的节点，但这样最后需要逆转一下链表。

- 优化代码：

  ```java
  public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
      ListNode dummyHead = new ListNode(0);
      ListNode p = l1, q = l2, curr = dummyHead;
      int carry = 0;
      while (p != null || q != null) {
          int x = (p != null) ? p.val : 0;
          int y = (q != null) ? q.val : 0;
          int sum = carry + x + y;
          carry = sum / 10;
          curr.next = new ListNode(sum % 10);
          curr = curr.next;
          if (p != null) p = p.next;
          if (q != null) q = q.next;
      }
      if (carry > 0) {
          curr.next = new ListNode(carry);
      }
      return dummyHead.next;
  }
  
  ```

  使用哑节点，这样在开始就有一个节点能指向新创建的节点，从而不需要逆转链表。

  另外两条链表的处理是可以写在一个循环里的，需要进行判断。

- 边界情况：
  - 一个链表比另一个长。
  - 加到最高位时出现进位，如11+89。

##### q19 删除链表的倒数第N个节点

```
给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。

示例：
给定一个链表: 1->2->3->4->5, 和 n = 2.
当删除了倒数第二个节点后，链表变为 1->2->3->5.

说明：给定的 n 保证是有效的。

进阶：你能尝试使用一趟扫描实现吗？
```

- 原始解法

  ```java
  /**
   * Definition for singly-linked list.
   * public class ListNode {
   *     int val;
   *     ListNode next;
   *     ListNode(int x) { val = x; }
   * }
   */
  class Solution {
      public ListNode removeNthFromEnd(ListNode head, int n) {
          ListNode h = head;
          int size = 0;
          for (; h!=null; h=h.next, size++);
          //if(n <= 0 || n > size) return null;
          h = head;
          if (size == n) return head.next;
          for (int i=0; i<size-n-1; i++)
              h = h.next;
          h.next = h.next.next;
          return head;
      }
  }
  ```

  先遍历一遍得到链表长度，然后根据n找到要删除的节点的前一个节点。

  边界情况：删除第一个节点。但也可以使用哑节点来处理该情况。

- 一次遍历解法，双指针

  ```java
  class Solution {
      public ListNode removeNthFromEnd(ListNode head, int n) {//注意给定的n是有效的
          ListNode dummy = new ListNode(0);
          dummy.next = head;
          ListNode h1 = dummy, h2 = dummy;
          for (int i=0; i<n+1; i++, h2=h2.next);
          for (; h2!=null; h1=h1.next, h2=h2.next);
          h1.next = h1.next.next;
          return dummy.next;
      }
  }
  ```

  采用哑节点处理第一个节点的删除。

  双指针，两个指针初始时均指向哑节点，指针2先移动n+1个位置，然后开始同时移动两个指针，直到指针2为空，此时指针1指向的就是要删除的节点的前一个节点。

##### q61 旋转链表

```java
给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。

示例 1:

输入: 1->2->3->4->5->NULL, k = 2
输出: 4->5->1->2->3->NULL
解释:
向右旋转 1 步: 5->1->2->3->4->NULL
向右旋转 2 步: 4->5->1->2->3->NULL


示例 2:

输入: 0->1->2->NULL, k = 4
输出: 2->0->1->NULL
解释:
向右旋转 1 步: 2->0->1->NULL
向右旋转 2 步: 1->2->0->NULL
向右旋转 3 步: 0->1->2->NULL
向右旋转 4 步: 2->0->1->NULL
```

- 原始解法

  ```java
  /**
   * Definition for singly-linked list.
   * public class ListNode {
   *     int val;
   *     ListNode next;
   *     ListNode(int x) { val = x; }
   * }
   */
  class Solution {
      public ListNode rotateRight(ListNode head, int k) {
          if(head==null) return null;
          ListNode l = head;
          int len = 0;
          while(l.next != null) {
              len++;
              l = l.next;
          }
          len++;
          l.next = head;
          l = head;
          for(int i=0; i<len-k%len-1; i++){
              l = l.next;
          }
          head = l.next;
          l.next = null;
          return head;
      }
  }
  ```

  思路为先记录链表长度，并且将最后一个节点指向头节点形成一个环。然后类似于删除倒数第N个节点，找到倒数第k个节点的前一个节点，把头节点指向它的下一个节点，而它指向null即设为尾节点即可。

  因为输入的k可能超过链表的长度，因此在得知链表长度之前无法使用双指针，那双指针也就没有必要了。

##### q138 复制带随机指针的链表

```
给定一个链表，每个节点包含一个额外增加的随机指针，该指针可以指向链表中的任何节点或空节点。
要求返回这个链表的 深拷贝。 
我们用一个由 n 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 [val, random_index] 表示：
	val：一个表示 Node.val 的整数。
	random_index：随机指针指向的节点索引（范围从 0 到 n-1）；如果不指向任何节点，则为  null 。
	
示例 1：
输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]

示例 2：
输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]

示例 3：
输入：head = [[3,null],[3,0],[3,null]]
输出：[[3,null],[3,0],[3,null]]

示例 4：
输入：head = []
输出：[]
解释：给定的链表为空（空指针），因此返回 null。

提示：
	-10000 <= Node.val <= 10000
	Node.random 为空（null）或指向链表中的节点。
	节点数目不超过 1000 。
```

- DFS解法

  ```java
  /*
  // Definition for a Node.
  class Node {
      int val;
      Node next;
      Node random;
  
      public Node(int val) {
          this.val = val;
          this.next = null;
          this.random = null;
      }
  }
  */
  
  class Solution {
      Map<Node, Node> visited = new HashMap<>();
      public Node copyRandomList(Node head) {
          if(head == null) return null;
          if(visited.containsKey(head)) return visited.get(head);
          Node node = new Node(head.val);
          visited.put(head, node);
          node.next = copyRandomList(head.next);
          node.random = copyRandomList(head.random);
          return node; 
      }
  }
  ```

  思路：因为带有随机指针，所以类似于图，可以考虑使用DFS来遍历图。需要一个散列表来记录访问情况，例如`Map<Node, Boolean>`，但这道题需要复制链表，因此可以将value设置成Node。对原链表遍历时如果访问过，那么直接返回该节点；如果没访问过，那就创建新节点，并将它设置为当前节点的值。这里进行节点连接的操作与二叉树的操作相同。

  ```java
  class Solution {
      public Node copyRandomList(Node head) {
          if(head == null) return null;
          Map<Node, Node> visited = new HashMap<>();
          Node h = head;
          while(h!=null) {
              visited.put(h, new Node(h.val));
              h = h.next;
          }
          h = head;
          Node node = visited.get(h);
          while(h != null){
              node.next = visited.get(h.next);
              node.random = visited.get(h.random);
              node = node.next;
              h = h.next;
          }
          return visited.get(head); 
      }
  }
  ```

  思路：不使用DFS，但关键仍然是散列表。先用散列表记录节点对应情况，然后按顺序遍历原链表，在此过程中完成新链表的连接关系。

##### q206 反转链表

```
反转一个单链表。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

进阶:
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？
```

- 迭代

  ```java
  /**
   * Definition for singly-linked list.
   * public class ListNode {
   *     int val;
   *     ListNode next;
   *     ListNode(int x) { val = x; }
   * }
   */
  class Solution {
      public ListNode reverseList(ListNode head) {
          if (head == null || head.next == null) return head;
          ListNode l1 = head;
          ListNode l2 = head.next;
          ListNode l3 = head.next.next;
  
          while(l2 != null) {
              l2.next = l1;
              l1 = l2;
              l2 = l3;
              if(l3 != null)
                  l3 = l3.next;
          }
          head.next = null;
          return l1;
      }
  }
  ```
  
  更优雅的方法
  
```java
  public ListNode reverseList(ListNode head) {
      ListNode prev = null;
      ListNode curr = head;
      while (curr != null) {
          ListNode nextTemp = curr.next;
          curr.next = prev;
          prev = curr;
          curr = nextTemp;
      }
      return prev;
  }
```

- 递归

  ```java
  class Solution {
      public ListNode reverseList(ListNode head) {
          if (head == null || head.next == null) return head;
          ListNode p = reverseList(head.next);
          head.next.next = head;
          head.next = null;
          return p;
      }
  }
  ```

  关键在于首先沿着链表延伸到最后一个节点，然后沿着递归树往回爬时更新连接，这个过程其实并没有用到返回的节点，因此最终返回的就是最后一个节点，也就是逆转后的头节点。

#### 动态规划：求最值

基本解题套路见[labuladong](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/dong-tai-gui-hua-xiang-jie-jin-jie)。

```
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```

##### q322 零钱兑换

```
给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。

示例 1:
输入: coins = [1, 2, 5], amount = 11
输出: 3 
解释: 11 = 5 + 5 + 1

示例 2:
输入: coins = [2], amount = 3
输出: -1

说明:
你可以认为每种硬币的数量是无限的。
```

- 暴力解法：

  ```java
  	public int coinChange(int[] coins, int amount) {
          if(amount == 0) return 0;
          if(amount < 0) return -1;
  
          int res = Integer.MAX_VALUE;
          for (int coin : coins) {
              if (amount - coin < 0) continue;
              res = Math.min(res, 1 + coinChange(coins, amount - coin));
          }
          return res == Integer.MAX_VALUE ? -1 : res;
      }
  ```

  这段代码的错误之处在于，递归树最底层的子问题无解（amount=1，coin=2），返回-1，但是高层的子问题都满足`amount - coin > 0`，这样会使最底层的无解结果在高层进行累加，导致错误结果。

  正确的逻辑应该是最底层的子问题无解时，高层的子问题必然无解，因此递归返回时每一层的子问题都应当明确这个信息，即

  ```java
  	public int coinChange(int[] coins, int amount) {
          if(amount == 0) return 0;
          if(amount < 0) return -1;
  
          int res = Integer.MAX_VALUE;
          for (int coin : coins) {
              int subproblem = coinChange(coins, amount - coin);
              if (subproblem == -1) continue;//只要最底层子问题无解，高层子问题都不会再进行求解
              res = Math.min(res, 1 + subproblem);
          }
          return res == Integer.MAX_VALUE ? -1 : res;
      }
  ```

- 使用记忆集优化

  ```java
  	public int coinChangeM(int[] coins, int amount, int[] mem) {
          if (amount == 0) return 0;
          if (amount < 0) return -1;
          if (mem[amount] != Integer.MAX_VALUE) return mem[amount];
  
          int res = Integer.MAX_VALUE;
          for (int coin : coins) {
              int subproblem = coinChangeM(coins, amount - coin, mem);
              if (subproblem == -1) continue;
              res = Math.min(res, 1 + subproblem);
          }
  
          mem[amount] = (res == Integer.MAX_VALUE) ? -1 : res;
          return mem[amount];
      }
  
      public int coinChange2(int[] coins, int amount) {
          int[] mem = new int[amount + 1];
          for (int i = 0; i < mem.length; i++) {
              mem[i] = Integer.MAX_VALUE;
          }
          return coinChangeM(coins, amount, mem);
      }
  ```

  amount作为状态，如果某个状态在记忆集中已经存在，即已经计算过，那么直接返回它。

#### 回溯问题：求所有可能情况

基本解题套路见[labuladong](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/hui-su-suan-fa-xiang-jie-xiu-ding-ban)。

```
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

##### q46 全排列

```
给定一个 没有重复 数字的序列，返回其所有可能的全排列。

示例:
输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```

- 暴力解法：

  ```java
  class Solution {
      List<List<Integer>> res = new LinkedList<>();
      public List<List<Integer>> permute(int[] nums) {
          LinkedList<Integer> track = new LinkedList<>();
          backtrace(nums, track);
          return res;
      }
  
      public void backtrace(int[] nums, LinkedList<Integer> track) {
          if (track.size() == nums.length) {
              res.add(new LinkedList<Integer>(track));
              return;
          }
  
          for (int i = 0; i < nums.length; i++) {
              if (track.contains(nums[i]))
                  continue;
              track.add(nums[i]);
              backtrace(nums, track);
              track.removeLast();
          }
      }
  }
  ```

##### q51 N皇后

```
n 皇后问题研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。
给定一个整数 n，返回所有不同的 n 皇后问题的解决方案。
每一种解法包含一个明确的 n 皇后问题的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。

示例:
输入: 4
输出: [
 [".Q..",  // 解法 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // 解法 2
  "Q...",
  "...Q",
  ".Q.."]
]
解释: 4 皇后问题存在两个不同的解法。
```

- 暴力解法：

  ```java
  class Solution {
      List<List<String>> res = new LinkedList<>();
      public List<List<String>> solveNQueens(int n) {
          LinkedList<String> track = new LinkedList<>();
          char[] str = new char[n];
          for(int i=0; i<n; i++){
              str[i] = '.';
          }
          for(int i=0; i<n; i++){
              track.add(new String(str));
          }
          backtrace(track, 0);
          return res;            
      }
  
      public void backtrace(LinkedList<String> track, int row) {
          if (row == track.size()){
              res.add(new LinkedList<String>(track));
              return;
          }
  
          int n = track.size();
          for (int col = 0; col < n; col++) {
              if (!isValid(track, row, col)) 
                  continue;
              replace(track, row, col, 'Q');
              backtrace(track, row + 1);
              replace(track, row, col, '.');
          }
      }
  
      public boolean isValid(LinkedList<String> track, int row, int col) {
          int n = track.size();
          for (int i=0; i<row; i++) {
              if(track.get(i).charAt(col)=='Q')
                  return false;
          }
          for (int i=row-1, j=col-1; i>=0&&j>=0; i--,j--) {
              if(track.get(i).charAt(j)=='Q')
                  return false;
          }
          for (int i=row-1, j=col+1; i>=0&&j<n; i--,j++) {
              if(track.get(i).charAt(j)=='Q')
                  return false;
          }
          return true;
      }
  
      public void replace(LinkedList<String> track, int row, int col, char c) {
          String s = track.get(row);
          char[] chars = s.toCharArray();
          chars[col] = c;
          track.set(row, new String(chars));
      }
  }
  ```

#### BFS：在图中求起点到终点的最短距离

基本解题套路见[labuladong](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/bfs-kuang-jia)。

```
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路

    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```

##### q111 二叉树的最小深度

```
给定一个二叉树，找出其最小深度。
最小深度是从根节点到最近叶子节点的最短路径上的节点数量。
说明: 叶子节点是指没有子节点的节点。

示例:
给定二叉树 [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
返回它的最小深度  2.
```

- BFS解法：

  ```java
  /**
   * Definition for a binary tree node.
   * public class TreeNode {
   *     int val;
   *     TreeNode left;
   *     TreeNode right;
   *     TreeNode(int x) { val = x; }
   * }
   */
  class Solution {
      public int minDepth(TreeNode root) {
          if(root==null) return 0;
          
          Queue<TreeNode> queue = new LinkedList<>();
          queue.add(root);
          int step = 1;
          while(!queue.isEmpty()){
              TreeNode x = queue.poll();
              if (x == null) continue;
              if (x.left == null && x.right == null) {
                  return step;
              }
              queue.add(x.left);
              queue.add(x.right);
              step++;
          }
          return step;
      }
  }
  ```

  这段代码是错误的，错误之处在于，对于树结构，想要记录深度必须让一层节点在一次while循环中全部弹出。

  ```java
  		while(!queue.isEmpty()){
              int sz = queue.size();
              for (int i = 0; i < sz; i++) {
                  TreeNode x = queue.poll();
                  if (x == null) continue;
                  if (x.left == null && x.right == null) {
                      return step;
                  }
                  queue.add(x.left);
                  queue.add(x.right);
              }
              step++;
          }
  ```

- DFS解法

  ```java
  /**
   * Definition for a binary tree node.
   * public class TreeNode {
   *     int val;
   *     TreeNode left;
   *     TreeNode right;
   *     TreeNode(int x) { val = x; }
   * }
   */
  class Solution {
      public int minDepth(TreeNode root) {
          if(root==null) return 0;
          return minDepth(root, 1);
      }
  
      public int minDepth(TreeNode x, int depth) {
          if (x == null) {
              return Integer.MAX_VALUE;
          }
          if (x.left == null && x.right == null) {
              return depth;
          }
          int minLeft = minDepth(x.left, depth + 1);
          int minRight = minDepth(x.right, depth + 1);
          return Math.min(minLeft, minRight);
      }
  }
  ```

##### q752 打开转盘锁

```
你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' 。每个拨轮可以自由旋转：例如把 '9' 变为  '0'，'0' 变为 '9' 。每次旋转都只能旋转一个拨轮的一位数字。
锁的初始数字为 '0000' ，一个代表四个拨轮的数字的字符串。
列表 deadends 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。
字符串 target 代表可以解锁的数字，你需要给出最小的旋转次数，如果无论如何不能解锁，返回 -1。

示例 1:
输入：deadends = ["0201","0101","0102","1212","2002"], target = "0202"
输出：6
解释：可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，
因为当拨动到 "0102" 时这个锁就会被锁定。

示例 2:
输入: deadends = ["8888"], target = "0009"
输出：1
解释：把最后一位反向旋转一次即可 "0000" -> "0009"。

示例 3:
输入: deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"], target = "8888"
输出：-1
解释：无法旋转到目标数字且不被锁定。

示例 4:
输入: deadends = ["0000"], target = "8888"
输出：-1
```

- BFS解法：

  ```java
  class Solution {
      public int openLock(String[] deadends, String target) {
          Set<String> deads = new HashSet<>();
          for(String deadend : deadends) 
              deads.add(deadend);
          Set<String> visited = new HashSet<>();
          Queue<String> queue = new LinkedList<>();
  
          String cur = "0000";
          queue.add(cur);
          visited.add(cur);
          int steps = 0;
  
          while(!queue.isEmpty()) {
              int sz = queue.size();
              for (int i=0; i<sz; i++) {
                  cur = queue.poll();
                  if(deads.contains(cur))
                      continue;
                  if(cur.equals(target))
                      return steps;
                  for(int j=0; j<4; j++) {
                      String up = plusOne(cur, j);
                      if(!visited.contains(up)) {
                          queue.add(up);
                          visited.add(up);
                      }
                      String down = minusOne(cur, j);
                      if(!visited.contains(down)) {
                          queue.add(down);
                          visited.add(down);
                      }
                  }
              }
              steps++;
          }
          return -1;
      }
  
      public String plusOne(String cur, int j) {
          char[] chars = cur.toCharArray();
          if (chars[j] == '9')
              chars[j] = '0';
          else
              chars[j] += 1;
          return new String(chars);    
      }
  
      public String minusOne(String cur, int j) {
          char[] chars = cur.toCharArray();
          if (chars[j] == '0')
              chars[j] = '9';
          else
              chars[j] -= 1;
          return new String(chars);    
      }
  }
  ```

- 双向BFS

  ```java
  	public int openLock(String[] deadends, String target) {
          Set<String> deads = new HashSet<>();
          for (String s : deadends) deads.add(s);
          // 用集合不用队列，可以快速判断元素是否存在
          Set<String> q1 = new HashSet<>();
          Set<String> q2 = new HashSet<>();
          Set<String> visited = new HashSet<>();
  
          int step = 0;
          q1.add("0000");
          q2.add(target);
  
          while (!q1.isEmpty() && !q2.isEmpty()) {
              // 哈希集合在遍历的过程中不能修改，用 temp 存储扩散结果
              Set<String> temp = new HashSet<>();
  
              /* 将 q1 中的所有节点向周围扩散 */
              for (String cur : q1) {
                  /* 判断是否到达终点 */
                  if (deads.contains(cur))
                      continue;
                  if (q2.contains(cur))
                      return step;
                  visited.add(cur);
  
                  /* 将一个节点的未遍历相邻节点加入集合 */
                  for (int j = 0; j < 4; j++) {
                      String up = plusOne(cur, j);
                      if (!visited.contains(up))
                          temp.add(up);
                      String down = minusOne(cur, j);
                      if (!visited.contains(down))
                          temp.add(down);
                  }
              }
              /* 在这里增加步数 */
              step++;
              // temp 相当于 q1
              // 这里交换 q1 q2，下一轮 while 就是扩散 q2
              q1 = q2;
              q2 = temp;
          }
          return -1;
      }
  ```

#### 滑动窗口/双指针

滑动窗口多为给出两个字符串，一个是模版，需要在另一个中匹配的问题。

基本解题套路见[labuladong](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/hua-dong-chuang-kou-ji-qiao-jin-jie)。

```
int left = 0, right = 0;
while (right < s.size()) {`
    // 增⼤窗⼝
    window.add(s[right]);
    right++;
    while (window needs shrink) {
        // 缩⼩窗⼝
        window.remove(s[left]);
        left++;
    }
}
```

##### q19 删除链表的倒数第N个节点

##### q76 最小覆盖字串

```
给你一个字符串 S、一个字符串 T，请在字符串 S 里面找出：包含 T 所有字符的最小子串。

示例：
输入: S = "ADOBECODEBANC", T = "ABC"
输出: "BANC"

说明：
	如果 S 中不存这样的子串，则返回空字符串 ""。
	如果 S 中存在这样的子串，我们保证它是唯一的答案。
```

```java
class Solution {
    public String minWindow(String s, String t) {
        int[] needs = new int[128];
        int[] window = new int[128];

        for(char c : t.toCharArray()) {
            needs[c]++;
        }

        String res = "";
        int left = 0, right = 0;

        while(right < s.length()) {
            char c = s.charAt(right);
            window[c]++;
            right++;
            while (contains(needs, window)) {
                if ("".equals(res))
                    res = s.substring(left, right);
                else
                    res = res.length() < (right-left) ? res : s.substring(left, right);
                char d = s.charAt(left);
                window[d]--;
                left++;
            }
        }
        return res;
    }

    public boolean contains(int[] needs, int[] window) {
        for(int i=65; i<needs.length; i++)
            if(window[i]<needs[i])
                return false;
        return true;
    }
}
```

```java
class Solution {
    public String minWindow(String s, String t) {
        Map<Character, Integer> needs = new HashMap<>();
        Map<Character, Integer> window = new HashMap<>();

        for(char c : t.toCharArray()) {
            put(needs, c);
        }

        String res = "";
        int left = 0, right = 0;
        int valid = 0;

        while(right < s.length()) {
            char c = s.charAt(right);
            put(window, c);
            if(needs.containsKey(c))
                if(needs.get(c).equals(window.get(c)))
                    valid++;
            right++;
            while (valid == needs.size()) {
                if ("".equals(res))
                    res = s.substring(left, right);
                else
                    res = res.length() < (right-left) ? res : s.substring(left, right);
                c = s.charAt(left);
                if(needs.containsKey(c))
                    if(needs.get(c).equals(window.get(c)))
                        valid--;
                window.put(c, window.get(c) - 1);  
                left++;
            }
        }
        return res;
    }

    public void put(Map<Character, Integer> map, char c) {
        if(map.containsKey(c))
            map.put(c, map.get(c) + 1);
        else
            map.put(c, 1);
    }
}
```

- 用数组操作相对方便，Map的API较为繁琐。

##### q567 字符串的排列

```
给定两个字符串 s1 和 s2，写一个函数来判断 s2 是否包含 s1 的排列。
换句话说，第一个字符串的排列之一是第二个字符串的子串。

示例1:
输入: s1 = "ab" s2 = "eidbaooo"
输出: True
解释: s2 包含 s1 的排列之一 ("ba").

示例2:
输入: s1= "ab" s2 = "eidboaoo"
输出: False

注意：
	输入的字符串只包含小写字母
	两个字符串的长度都在 [1, 10,000] 之间
```

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int[] needs = new int[26];
        int[] window = new int[26];

        for (char c : s1.toCharArray())
            needs[c-'a']++;
        
        int left = 0, right = 0;
        
        while(right < s2.length()) {
            char c = s2.charAt(right);
            window[c-'a']++;
            right++;
            while(contains(needs, window)) {//这里的条件可以优化为right - left >= s1.length()
                if (onlycontains(needs, window))
                    return true;
                char d = s2.charAt(left);
                window[d-'a']--;
                left++;
            }
        }
        return false;
    }
    public boolean contains(int[] needs, int[] window) {
        for (int i=0; i<needs.length; i++)
            if (window[i]<needs[i])
                return false;
        return true;
    }
    public boolean onlycontains(int[] needs, int[] window) {
        for (int i=0; i<needs.length; i++)
            if (window[i] != needs[i])
                return false;
        return true;
    }
}
```

##### q438 找到字符串中所有字母异位词

```
给定一个字符串 s 和一个非空字符串 p，找到 s 中所有是 p 的字母异位词的子串，返回这些子串的起始索引。
字符串只包含小写英文字母，并且字符串 s 和 p 的长度都不超过 20100。

说明：
	字母异位词指字母相同，但排列不同的字符串。
	不考虑答案输出的顺序。

示例 1:
输入:
s: "cbaebabacd" p: "abc"
输出:
[0, 6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的字母异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的字母异位词。

示例 2:
输入:
s: "abab" p: "ab"
输出:
[0, 1, 2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的字母异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的字母异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的字母异位词。
```

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        int[] needs = new int[26];
        int[] window = new int[26];
        for(char c : p.toCharArray())
            needs[c-'a']++;
        
        int left = 0, right = 0;
        List<Integer> res = new ArrayList<>();

        while(right < s.length()) {
            char c = s.charAt(right);
            window[c-'a']++;
            right++;
            while (right - left >= p.length()) {
                if(onlycontains(needs, window))
                    res.add(left);
                char d = s.charAt(left);
                window[d-'a']--;
                left++;    
            }
        }
        return res;
    }
    public boolean onlycontains(int[] needs, int[] window) {
        for(int i=0; i<needs.length; i++)
            if (needs[i] != window[i])
                return false;
        return true;
    }
}
```

- 可以看到滑动窗口的模板代码非常明显，基本上只有缩小window时需要做的操作就是完成题目的操作。

##### q3 无重复字符的最长子串

```
给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

示例 1:
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。


示例 2:
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。


示例 3:
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int[] window = new int[128];

        int left = 0, right = 0;
        int res = 0;

        while(right < s.length()) {
            char c = s.charAt(right);
            window[c]++;
            right++;
            while(window[c] > 1) {//只要存在重复字符就缩小窗口
                char d = s.charAt(left);
                window[d]--;
                left++;
            }
            res = Math.max(res, right - left);//窗口缩小完毕后一定没有重复字符，更新结果
        }
        return res;
    }
}
```

- 基本模板没变，可以看到滑动窗口类型的问题关键在于缩小窗口的判定条件，以及如何获得结果，在哪个位置更新结果（缩小窗口时还是缩小窗口后）。

##### q11 盛最多水的容器

```
给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
说明：你不能倾斜容器，且 n 的值至少为 2。

图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

示例：
输入：[1,8,6,2,5,4,8,3,7]
输出：49
```

- 暴力解法

  对每个位置，向右遍历一遍剩余的位置。

- 双指针

  ```java
  class Solution {
      public int maxArea(int[] height) {
          int left = 0, right = height.length-1, max = 0;
          while(left < right) {
              int area = Math.min(height[left], height[right]) * (right-left);
              max = Math.max(max, area);
              if(height[left] < height[right])
                  left++;
              else
                  right--;
          }
          return max;
      }
  }
  ```

  不同于滑动窗口典型问题的两个指针都初始化为同一侧，这里从两侧开始往中间靠拢。关键是如何移动指针，两个位置上的高度谁小，谁就往中间靠拢。

##### q15 三数之和

```
给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

示例：
给定数组 nums = [-1, 0, 1, 2, -1, -4]，
满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

- 不能用回溯法，因为题目要求不能重复。

    ```java
    class Solution {
        List<List<Integer>> res = new ArrayList<>();
        boolean[] visited;
        public List<List<Integer>> threeSum(int[] nums) {
            visited = new boolean[nums.length];
            LinkedList<Integer> path = new LinkedList<>();
            backtrack(path, nums);
            return res;
        }

        private void backtrack(LinkedList<Integer> path, int[] nums) {
            if(path.size() == 3 && sumIs0(path)){
                res.add(new ArrayList(path));
                return;
            }
            if(path.size() == 3)
                return;
            int n = nums.length;
            for(int i=0; i<n; i++) {
                if(visited[i]==true)
                    continue;
                path.add(nums[i]);
                visited[i] = true;
                backtrack(path, nums);
                path.removeLast();
                visited[i] = false;
            }
        }

        private boolean sumIs0(LinkedList<Integer> path) {
            int sum = 0;
            for (int num : path) {
                sum += num;
            }
            return sum == 0;
        }
    }
    ```

- 排序+双指针

    ```java
    class Solution {
        public List<List<Integer>> threeSum(int[] nums) {
            List<List<Integer>> res = new ArrayList<>();
            Arrays.sort(nums);
            //-4 -1 -1 0 1 2
            int n = nums.length;
            for (int i = 0; i < n - 2; i++) {
                if (i > 0 && nums[i] == nums[i - 1]) {
                    continue;
                }
                int left = i + 1, right = n - 1;
                while (left < right) {
                    if (left > i + 1 && nums[left] == nums[left - 1]) {
                        left++;
                        continue;
                    }
                    //关键在于如何移动指针
                    if (nums[i] + nums[left] + nums[right] == 0) {
                        res.add(List.of(nums[i], nums[left], nums[right]));
                        left++;
                    } else if (nums[i] + nums[left] + nums[right] < 0)
                        left++;
                    else
                        right--;
                }
            }
            return res;
        }
    }
    ```

    思路：使用三重循环的框架枚举三元组，想要不重复的话首先排序，这样枚举$(a,b,c)$有序三元组，保证只有 $(a,b,c)$ 这个顺序会被枚举到，而 $(b,a,c)$、$(c,b,a)$ 等等这些不会，这样就减少了重复。

    同时，对于每一重循环而言，相邻两次枚举的元素不能相同，否则也会造成重复。

    ```
    [0, 1, 2, 2, 2, 3]
     ^  ^  ^
    ```

    使用三重循环枚举到的第一个三元组为 $(0,1,2)$，如果第三重循环继续枚举下一个元素，那么仍然是三元组$(0,1,2)$，产生了重复。因此我们需要将第三重循环「跳到」下一个不相同的元素，即数组中的最后一个元素 3，枚举三元组$(0,1,3)$。

    接着优化三重循环：可以发现如果我们固定了前两重循环枚举到的元素 a和 b，那么只有唯一的 c 满足 $a+b+c=0$。当第二重循环往后枚举一个元素 $b′$ 时，由于 $b′>b$，那么满足 $a+b′+c′=0$ 的 $c′$ 一定有 $c′<c$，即 $c′$ 在数组中一定出现在 c 的左侧。也就是说，我们可以从小到大枚举 b，同时从大到小枚举 c，即第二重循环和第三重循环实际上是并列的关系。

##### q16 最接近的三数之和

```
给定一个包括 n 个整数的数组 nums 和 一个目标值 target。找出 nums 中的三个整数，使得它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在唯一答案。

示例：
输入：nums = [-1,2,1,-4], target = 1
输出：2
解释：与 target 最接近的和是 2 (-1 + 2 + 1 = 2) 。

提示：
	3 <= nums.length <= 10^3
	-10^3 <= nums[i] <= 10^3
	-10^4 <= target <= 10^4
```

- 排序+双指针

  ```java
  class Solution {
      public int threeSumClosest(int[] nums, int target) {
          int res = nums[0] + nums[1] + nums[2];
          int n = nums.length;
          Arrays.sort(nums);
  
          //-4 -1 1 2
          for (int i = 0; i < n - 2; i++) {
              if(i > 0 && nums[i] == nums[i-1])//优化
                  continue;
              int left = i + 1, right = n - 1;
              while (left < right) {
                  if(left > i+1 && nums[left] == nums[left-1]){//优化
                      left++;
                      continue;
                  }
                  int t = nums[i] + nums[left] + nums[right];
                  if (t == target) return t;//优化
                  int diff_t = Math.abs(target - t);
                  int diff_res = Math.abs(target - res);
                  if (diff_t < diff_res) {
                      res = t;
                  }
                  if (target - t > 0) left++;
                  else right--;
              }
          }
  
          return res;
      }
  }
  ```

  思路与q15类似，原本相等的要求变成了最接近，可以计算每一个三元组之和与target的距离，更新这个和。在双指针移动部分，如果三元组之和小于target，那就left指针往右移动，否则right指针往左移动。

  优化之处：可以像q15一样遇到重复元素就跳过（在leetcode提供的测试用例中没有体现，7ms->7ms）；当三元组之和等于target时直接返回，因为相等一定是最接近的（在leetcode提供的测试用例中有体现，10ms->7ms）。

##### q26 删除排序数组中的重复项

```
给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

示例 1:
给定数组 nums = [1,1,2], 
函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。 
你不需要考虑数组中超出新长度后面的元素。

示例 2:
给定 nums = [0,0,1,1,1,2,2,3,3,4],
函数应该返回新的长度 5, 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4。
你不需要考虑数组中超出新长度后面的元素。

说明:
为什么返回数值是整数，但输出的答案是数组呢?
请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。
你可以想象内部操作如下:

// nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中该长度范围内的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

- 滑动窗口

  ```java
  class Solution {
      public int removeDuplicates(int[] nums) {
          int left = 0, right = 0;
          while (right < nums.length) {
              if (nums[right] == nums[left]) {
                  right++;
                  continue;
              }
              nums[left + 1] = nums[right];
              left++;
              right++;
          }
          return left + 1;
      }
  }
  ```

  这个问题的双指针运用更接近于典型字符串问题中的滑动窗口，即从同一侧开始移动，right指针只要和left指针的内容相同就右移，如果不同就停下来将left指针的内容更新，然后两个指针继续右移。

