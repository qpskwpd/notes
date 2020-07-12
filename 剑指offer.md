### 2 实现单例模式

```java
public class Q2Singleton {

    private static Q2Singleton singleton;
    private static Object object;
    
    private Q2Singleton() {
    }

    public static Q2Singleton getInstance() {
		if (singleton == null) {
            synchronized (object) {
                if (singleton == null) {
                    singleton = new Q2Singleton();
                }
            }
        }
        return singleton;
    }
}
```

初步创建单例模式：将构造函数设为私有的，添加一个静态的变量singleton，在静态方法中判断该变量是否为空，为空的时候创建一个实例。

用于多线程环境：创建实例的时候加上同步操作。

优化效率的版本：在加锁前再判断一次该变量是否为空，为空才加锁，避免加锁的开销。

```java
public class Q2Singleton {

    private static Q2Singleton singleton = new Q2Singleton();

    private Q2Singleton() {
    }

    public static Q2Singleton getSingleton() {
        return singleton;
    }
}
```

更好的写法：利用静态构造函数（类的初始化过程），确保只调用一次。

- Java虚拟机必须保证一个类的\<clinit>()方法在多线程环境中被正确地加锁同步，因此多个线程同时初始化一个类时，只会有一个线程去执行该类的\<clinit>()方法，其他线程都需要阻塞等待。

```java
public class Q2Singleton {

    private Q2Singleton() {
    }

    public static Q2Singleton getSingleton() {
        return Nested.singleton;
    }

    static class Nested {
        static Q2Singleton singleton = new Q2Singleton();
    }
}
```

按需创建的写法：定义一个内部类，将该变量定义到内部类中，第一次用到内部类时才创建实例。

### 3 数组中重复的数字

一个长度为n的数组，所有数字都在0~n-1范围内。找出某个重复的数字。

解法1：先排序再找，时间复杂度$O(nlogn)$。

```java
import java.util.Random;

public class Q3DupNum {
    public static void main(String[] args) {
        int N = 20;
        int[] nums = new int[N];
        Random random = new Random();
        for (int i = 0; i < N; i++) {
            nums[i] = random.nextInt(N-1);
        }
        System.out.println(dupNum(nums));
    }

    public static int dupNum(int[] nums) {
        int n = nums.length;
        Quick(nums);
        for (int i = 0; i < n - 1; i++) {
            if (nums[i] == nums[i + 1])
                return nums[i];
        }
        return -1;
    }

    private static void Quick(int[] nums) {
        Quick(nums, 0, nums.length - 1);
    }

    private static void Quick(int[] nums, int left, int right) {
        if (left > right) return;
        int j = partion(nums, left, right);
        Quick(nums, left, j - 1);
        Quick(nums, j + 1, right);
    }

    private static int partion(int[] nums, int left, int right) {
        int t = nums[left];
        int i = left + 1, j = right;
        while (true) {
            while (i <= right && nums[i] <= t) i++;
            while (j > left && nums[j] >= t) j--;
            if (i >= j)
                break;
            swap(nums, i, j);
        }
        swap(nums, left, j);
        return j;
    }

    private static void swap(int[] nums, int i, int j) {
        int t = nums[i];
        nums[i] = nums[j];
        nums[j] = t;
    }
}

[2, 11, 6, 12, 12, 4, 15, 7, 5, 15, 4, 17, 3, 13, 9, 1, 8, 12, 14, 10]
[1, 2, 3, 4, 4, 5, 6, 7, 8, 9, 10, 11, 12, 12, 12, 13, 14, 15, 15, 17]
4
```

解法2：利用散列表记录出现情况，时间复杂度$O(n)$，空间复杂度$O(n)$。这种解法不会修改数组。

```java
public class Q3DupNum {
    public static void main(String[] args) {
        int[] nums = new int[]{2, 3, 1, 0, 4, 5, 6};
        System.out.println(dupNum(nums));
    }

    public static int dupNum(int[] nums) {
        int n = nums.length;
        int[] count = new int[n];
        for (int i = 0; i < nums.length; i++) {
            count[nums[i]]++;
            if (count[nums[i]] > 1)
                return nums[i];
        }
        return -1;
    }
}
```

解法3：由于数组中的数字都在0~n-1范围内，如果没有重复数字，那排序后下标为i的数字就是i。因此可以利用这个规律对数组进行重排：扫描到下标为$i$的数字$m$时，如果$i==m$，接着扫描下一个；如果$i!=m$，比较$m$与索引为$m$的数字是否相等，如果$m=nums[m]$，则找到了一个重复数字，否则交换下标为$i$和$m$的数字，重复这个过程。时间复杂度$O(n)$，空间复杂度$O(1)$。

```java
import java.util.Arrays;
import java.util.Random;

public class Q3DupNum {
    public static void main(String[] args) {
        int N = 10;
        int[] nums = new int[N];
        Random random = new Random();
        for (int i = 0; i < N; i++) {
            nums[i] = random.nextInt(N - 1);
        }
        System.out.println(Arrays.toString(nums));
        System.out.println(dupNum(nums));
    }

    public static int dupNum(int[] nums) {
        if (nums == null && nums.length == 0) return -1;
        for (int num : nums) {
            if (num < 0 || num >= nums.length)
                return -1;
        }
        for (int i = 0; i < nums.length; i++) {
            if (i == nums[i]) continue;
            if (nums[i] == nums[nums[i]]) return nums[i];
            swap(nums, i, nums[i]);
        }
        return -1;
    }

    private static void swap(int[] nums, int i, int j) {
        int t = nums[i];
        nums[i] = nums[j];
        nums[j] = t;
    }
}


[4, 2, 5, 0, 2, 0, 4, 2, 4, 3]
0
```

一个长度为n+1的数组，所有数字都在1~n范围内。**不修改数组**找出某个重复的数字。

可以利用散列表，但空间使用为$O(n)$，下面为一种使用$O(1)$的方法。

```java
package cn.dut.offer;

import java.util.Arrays;
import java.util.Random;

public class Q3DupNum2 {
    public static void main(String[] args) {
        int N = 10;
        int[] nums = new int[N];
        Random random = new Random();
        for (int i = 0; i < N; i++) {
            nums[i] = random.nextInt(N - 1) + 1;
        }
        System.out.println(Arrays.toString(nums));
        System.out.println(dupNum(nums));
    }

    public static int dupNum(int[] nums) {
        if (nums == null && nums.length == 0) return -1;
        int start = 1, end = nums.length - 1;
        while (start <= end) {
            int mid = (end - start) / 2 + start;
            int count = countRange(nums, start, mid);
            if (start == end) {
                if (count > 1) return start;
                else break;
            }
            if (count > mid - start + 1)
                end = mid;
            else
                start = mid + 1;
        }
        return -1;
    }

    private static int countRange(int[] nums, int start, int end) {
        int count = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] >= start && nums[i] <= end)
                count++;
        }
        return count;
    }
}
```

思路：长度为n+1，所有数字在1~n之间，至少有一个重复数字。利用二分查找的想法，m为中间数字，首先检查1~m这个区间，如果数组中该区间的数字数目超过m，那么该区间一定包含重复数字；否则m+1~n的区间一定包含重复数字。

由于每次检查的区间是start~mid，因此该区间包含重复数字，那就让end设为mid。如果不包含，那就让start设为mid+1。

二分调用所需时次数为$O(logn)$，每次调用遍历一遍数组，为$O(n)$，因此总时间复杂度$O(nlogn)$。

### 4 二维数组中的查找

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排列。输入一个整数，判断数组中是否含有该整数。

```java
package cn.dut.offer;

public class Q4 {
    public static void main(String[] args) {
        int[][] array = new int[][]{
                {1, 2, 8, 9},
                {2, 4, 9, 12},
                {4, 7, 10, 13},
                {6, 8, 11, 15}
        };
        System.out.println(new Q4().find(array, 6));
    }

    public boolean find(int[][] array, int num) {
        if (array == null || array.length == 0) return false;
        int rows = array.length;
        int cols = array[0].length;
        int row = 0;
        int col = cols - 1;
        while (row < rows && col >= 0) {
            if (array[row][col] > num) {
                col--;
            } else if (array[row][col] < num) {
                row++;
            } else {
                return true;
            }
        }
        return false;
    }
}
```

思路：由于往右或往下均为增大的方向，因此可以从右上角或左下角的元素开始查找。右上角的元素大于被查找元素，则排除这一列，因为下面的元素都比它大；小于被查找元素，则排除这一行，因为左面的元素都比它小。

<img src="img/offer/q4.png" alt="image-20200706132611095" style="zoom:50%;" />

### 5 替换空格

把字符串中每个空格替换成“%20”。

```java
package cn.dut.offer;

import java.util.Arrays;

public class Q5 {
    public static void main(String[] args) {
        String str = "we are happy.";
        /*long t1 = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            str = str.replaceAll("\\s", "%20");
            str = replace(str);
            str = replace2(str);
        }
        System.out.println(str + ", cost time: " + (System.currentTimeMillis() - t1) / 1000.0);*/

        char[] chars = new char[18];
        for (int i = 0; i < str.length(); i++) {
            chars[i] = str.charAt(i);
        }
        System.out.println(Arrays.toString(chars));
        replace3(chars);
        System.out.println(Arrays.toString(chars));
    }

    public static String replace(String s) {
        StringBuilder sb = new StringBuilder();
        for (char c : s.toCharArray()) {
            if (c == ' ') sb.append("%20");
            else sb.append(c);
        }
        return sb.toString();
    }

    public static String replace2(String s) {
        String t = "";
        for (char c : s.toCharArray()) {
            if (c == ' ') t = t.concat("%20");
            else t = t.concat("" + c);
        }
        return t;
    }

    public static void replace3(char[] chars) {
        if (chars == null || chars.length == 0) return;
        int length = chars.length;

        int oldLength = 0, numOfSpace = 0;
        while (oldLength<length && chars[oldLength] != '\0') {
            if (chars[oldLength] == ' ') numOfSpace++;
            oldLength++;
        }

        int newLength = oldLength + 2 * numOfSpace;
        if (newLength > length) return;

        int right = newLength - 1, left = oldLength - 1;
        while (left >= 0 && left < right) {
            if (chars[left] != ' ') {
                chars[right--] = chars[left];
            }else {
                chars[right--] = '0';
                chars[right--] = '2';
                chars[right--] = '%';
            }
            left--;
        }
    }
}
```

这道题如果使用Java的一些api可以直接完成，如`replaceAll()`、直接拼接字符串/`concat()`或`StringBuilder`，而且Java的字符串无法在原地完成操作。如果使用C/C++且要求不能使用额外空间，那就应当考虑提前计算好替换后的长度，从后往前替换字符。

<img src="img/offer/q5.png" style="zoom:67%;" />

```
int right = newLength - 1, left = oldLength - 1;
```

Java中字符串末尾没有'\0'，索引应当初始化为`len-1`，而C/C++初始化为len，则是考虑到了'\0'。

### 6 从尾到头打印链表

输入一个链表的头节点，从尾到头反过来打印出每个节点的值。

```java
package cn.dut.offer;

public class Q6 {
    static class ListNode {
        int val;
        ListNode next;

        public ListNode(int val) {
            this.val = val;
        }
    }

    public static void main(String[] args) {
        int[] vals = new int[]{1, 3, 5, 6, 9, 8, 4, 3};
        ListNode dummy = new ListNode(-1);
        ListNode head = dummy;
        for (int i = 0; i < vals.length; i++) {
            head.next = new ListNode(vals[i]);
            head = head.next;
        }
        head = dummy.next;
        reversePrint(head);
    }

    public static void reversePrint(ListNode x) {
        if (x == null) return;
        reversePrint(x.next);
        System.out.print(x.val + " ");
    }
    
    public static void reversePrint2(ListNode x) {
        Stack<ListNode> stack = new Stack<>();

        while (x != null) {
            stack.push(x);
            x = x.next;
        }

        while (!stack.empty()) {
            ListNode pop = stack.pop();
            System.out.print(pop.val + " ");
        }
    }
}
```

先遍历的节点后打印，使用栈可以实现这种顺序。显式使用栈或直接后序遍历即可。

### 7 重建二叉树

输入二叉树的前序遍历和中序遍历结果，重建该二叉树。

```java
package cn.dut.offer;

import java.util.LinkedList;
import java.util.Queue;
import java.util.Stack;

public class Q7 {
    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        public TreeNode(int val) {
            this.val = val;
        }
    }

    public static void main(String[] args) {
        int[] pre = new int[]{1, 2, 4, 7, 3, 5, 6, 8};
        int[] mid = new int[]{4, 7, 2, 1, 5, 3, 8, 6};
        TreeNode head = reconstruct(pre, mid, 0, pre.length - 1, 0, mid.length - 1);
        preOrderPrint2(head);
        System.out.println();
        inOrderPrint(head);
        System.out.println();
        postOrderPrint(head);
        System.out.println();
        levelOrderPrint(head);

    }

    private static TreeNode reconstruct(int[] pre, int[] mid, int preLeft, int preRight, int midLeft, int midRight) {
        if (preLeft > preRight || midLeft > midRight) return null;
        int rootIndex = -1;
        for (int i = midLeft; i <= midRight; i++) {
            if (mid[i] == pre[preLeft]) {
                rootIndex = i;
                break;
            }
        }
        if (rootIndex == -1) throw new RuntimeException("invalid data.");
        TreeNode root = new TreeNode(pre[preLeft]);
        int leftLength = rootIndex - midLeft;
        int leftPreRight = preLeft + leftLength;
        root.left = reconstruct(pre, mid, preLeft + 1, leftPreRight, midLeft, rootIndex - 1);
        root.right = reconstruct(pre, mid, leftPreRight + 1, preRight, rootIndex + 1, midRight);
        return root;
    }

    private static void preOrderPrint(TreeNode root) {
        if (root == null) return;
        System.out.print(root.val + " ");
        preOrderPrint(root.left);
        preOrderPrint(root.right);
    }
}
```

前序遍历结果的第一个数一定是根节点，因此在中序遍历结果中找到它的位置，左边为根节点的左子树，右边为右子树。这样就得到了左子树和右子树的遍历序列，因此可以递归地进行重建。

<img src="img/offer/q7.png" style="zoom:50%;" />

#### 非递归遍历二叉树的方法

```java
    private static void preOrderPrint2(TreeNode root) {
        Stack<TreeNode> stack = new Stack<>();
        TreeNode t = root;
        while (t != null || !stack.empty()) {
            while (t != null) {
                System.out.print(t.val + " ");
                stack.push(t);
                t = t.left;
            }
            if (!stack.empty()) {
                t = stack.pop();
                t = t.right;
            }
        }
    }

    private static void inOrderPrint(TreeNode root) {
        Stack<TreeNode> stack = new Stack<>();
        TreeNode t = root;
        while (t != null || !stack.empty()) {
            while (t != null) {
                stack.push(t);
                t = t.left;
            }
            if (!stack.empty()) {
                t = stack.pop();
                System.out.print(t.val + " ");
                t = t.right;
            }
        }
    }

    private static void postOrderPrint(TreeNode root) {
        Stack<TreeNode> stack = new Stack<>();
        TreeNode cur;
        TreeNode pre = null;
        stack.push(root);
        while (!stack.empty()) {
            cur = stack.peek();
            if ((cur.left == null && cur.right == null) || (pre != null && (pre == cur.left || pre == cur.right))) {
                System.out.print(cur.val + " ");
                pre = stack.pop();
            } else {
                if (cur.right != null) stack.push(cur.right);
                if (cur.left != null) stack.push(cur.left);
            }
        }
    }

    private static void levelOrderPrint(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            System.out.print(node.val + " ");
            if (node.left != null) queue.add(node.left);
            if (node.right != null) queue.add(node.right);
        }
    }
```

前序：首先沿着左子树深入，每遇到一个节点就将其压入栈，同时打印（每个节点都是该子树的根节点，总先打印根节点），当左节点为空时，弹出栈，转向右节点。

中序：首先沿着左子树深入，每遇到一个节点就将其压入栈，当左节点为空时，弹出栈，此时打印（总先打印（相对于还没有被打印的根节点的）左节点），转向右节点。

后序：打印根节点有两种情况，一是它没有子节点，二是子节点都已经被访问过，这两种情况是根节点出栈的条件。因此可以使用一个指针记录上一个访问过的节点，如果是当前节点的子节点，就可以打印，并且出栈；否则依次将它的右节点、左节点入栈（左右根）。

层次：使用队列进行BFS即可。

### 8 二叉树的下一个节点

给定一课二叉树和其中一个节点，如何找出中序遍历的下一个节点？树中的节点除了有两个分别指向左、右子节点的指针，还有一个指向父节点的指针。

```java
private TreeNode inOrderNext(TreeNode query) {
    if (query == null) return null;
    TreeNode next = null;
    if (query.right != null) {
        TreeNode right = query.right;
        while (right.left != null) {
            right = right.left;
        }
        next = right;
    } else if (query.parent != null) {
        TreeNode current = query;
        TreeNode parent = query.parent;
        while (parent != null && current == parent.right) {
            current = parent;
            parent = parent.parent;
        }
        next = parent;
    }
    return next;
}
```

中序遍历的顺序为左、根、右。

对于一个节点，如果它有右节点，那么下一个遍历节点为右子树中最左边的节点，如b-e-h；

如果它没有右节点，那么先找到它的父节点，如果它是父节点的左节点，那么下一个遍历节点为它的父节点，如h-e；

否则往上回退，直到找到一个左节点，那么下一个遍历节点为这个节点的父节点，如i-e-b-a。

<img src="img/offer/q8.png" style="zoom: 50%;" />

### 9 用两个栈实现队列

用两个栈实现一个队列，具有在队列尾部插入节点和在队列头部删除节点的功能。

```java
package cn.dut.offer;

import java.util.Scanner;
import java.util.Stack;

public class Q9 {

    static class CQueue<T> {
        private Stack<T> stack1;
        private Stack<T> stack2;

        public CQueue() {
            stack1 = new Stack<>();
            stack2 = new Stack<>();
        }

        public void appendTail(T item) {
            stack1.push(item);
        }

        public T deleteHead() {
            if (!stack2.empty()) {
                return stack2.pop();
            } else {
                while (!stack1.empty()) {
                    stack2.push(stack1.pop());
                }
                if (!stack2.empty()) return stack2.pop();
                else throw new RuntimeException("queue is empty.");
            }
        }

        public boolean empty() {
            return stack1.empty() && stack2.empty();
        }
    }

    public static void main(String[] args) {
        CQueue<String> q = new CQueue<>();

        Scanner sc = new Scanner(System.in);
        while (sc.hasNext()) {
            String next = sc.next();
            if (!next.equals("-")) {
                q.appendTail(next);
            } else if (!q.empty()) {
                System.out.print(q.deleteHead() + " ");
            }
        }
    }
}
```

栈1用于加入元素时使用，此时先进后出；栈2用于删除元素时使用，将栈1的元素加入栈2，此时先进先出。

对于删除操作不用在栈1和栈2之间来回移动元素，当栈2为空时将栈1所有元素移动过来，栈2不为空时直接将栈顶元素出栈，并不影响新加入元素的出队顺序。

#### 用两个队列实现一个栈

```java
	static class CStack<T> {
        private Queue<T> queue1;
        private Queue<T> queue2;

        public void push(T item) {
            queue1.add(item);
        }

        public T pop() {
            while (queue1.size() > 1) {
                queue2.add(queue1.poll());
            }
            T t = queue1.poll();
            Queue q = queue1;
            queue1 = queue2;
            queue2 = q;
            return t;
        }

        public CStack() {
            queue1 = new LinkedList<>();
            queue2 = new LinkedList<>();
        }

        public boolean empty() {
            return queue1.size() == 0 && queue2.size() == 0;
        }
    }

    public static void main(String[] args) {
        CStack<String> stack = new CStack<>();

        Scanner sc = new Scanner(System.in);
        while (sc.hasNext()) {
            String next = sc.next();
            if (!next.equals("-")) {
                stack.push(next);
            } else if (!stack.empty()) {
                System.out.print(stack.pop() + " ");
            }
        }
    }
```

利用队列的当前容量作为辅助手段，队列1用于入栈操作，每当需要出栈操作时，将队列1的前N-1个元素出队，加入到队列2中，然后队列1出队最后一个元素，即为返回元素，满足后进先出。然后将两个队列的地址互换即可。

### 10 斐波那契数列

求斐波那契数列的第n项。

```java
package cn.dut.offer;

public class Q10 {

    public static void main(String[] args) {
        Q10 q10 = new Q10();
        final int n = 45;

        long t1 = System.currentTimeMillis();
        System.out.println(q10.fib(n) + " cost: " + (System.currentTimeMillis() - t1) / 1000.0);

        long t2 = System.currentTimeMillis();
        int[] mem = new int[n + 1];
        System.out.println(q10.fib(n, mem) + " cost: " + (System.currentTimeMillis() - t2) / 1000.0);

        long t3 = System.currentTimeMillis();
        System.out.println(q10.fib2(n) + " cost: " + (System.currentTimeMillis() - t3) / 1000.0);
    }

    public int fib2(int n) {
        if (n <= 0) return 0;
        if (n == 1 || n == 2) return 1;
        int pre = 1, cur = 1;
        int res;
        for (int i = 3; i <= n; i++) {
            res = pre + cur;
            pre = cur;
            cur = res;
        }
        return cur;
    }

    public int fib(int n) {
        if (n <= 0) return 0;
        if (n == 1 || n == 2) return 1;
        return fib(n - 1) + fib(n - 2);
    }

    public int fib(int n, int[] mem) {
        if (n <= 0) return 0;
        if (n == 1 || n == 2) return 1;
        if (mem[n] != 0) return mem[n];
        mem[n] = fib(n - 1, mem) + fib(n - 2, mem);
        return mem[n];
    }
}
```

为了健壮性，最好将`n<=0`和`n==1`、`n==2`的情况直接返回。

#### 青蛙跳台阶问题

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个n级台阶总共有多少种跳法。

解法其实与斐波那契数列相同，但它包含着动态规划的思想。

跳上n级台阶，那么一定是从n-1或n-2级台阶跳上来的，因此问题转化为跳上n-1或n-2级台阶共有多少种跳法。以此类推，一定会到达基本情况，即跳上1级或2级台阶。

或者从基本情况出发，跳上1级台阶只有1种跳法，2级台阶有两种跳法。跳上n级台阶，可以先选择跳1级，那么剩下n-1级，或者选择跳2级，那么剩下n-2级。

<img src="img/offer/q10.png" style="zoom: 67%;" />

### 11 旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。如数组`{3,4,5,1,2}`为`{1,2,3,4,5}`的一个旋转，该数组的最小值为1。

- 不合意的解法

  ```java
  class Solution {
      public int minArray(int[] numbers) {
          int n = numbers.length;
          int res = Integer.MAX_VALUE;
          for(int i = n-1; i>=0; i--)
              if(numbers[i] < res) res = numbers[i];
          return res; 
      }
  }
  ```

  没有用到旋转的特性，需要遍历一遍数组，复杂度为$O(n)$。

- 二分查找

  ```java
  class Solution {
      public int minArray(int[] numbers) {
          int n = numbers.length;
          int left = 0, right = n - 1;
          int mid = left;
          while(numbers[left] >= numbers[right]) {
              mid = (right - left) / 2 + left;
              if(numbers[mid] == numbers[left] && numbers[left] == numbers[right])
                  return mainOrder(numbers, left, right);
              if(numbers[mid] >= numbers[left]) left = mid;
              else if(numbers[mid] <= numbers[right]) right = mid;
              if(right - left == 1){
                  mid = right;
                  break;
              }
          }
          return numbers[mid];
      }
  
      private int mainOrder(int[] numbers, int left, int right) {
          int res = numbers[left];
          for(int i = left + 1; i <= right; i++)
              if(numbers[i] <= res) res = numbers[i];
          return res;
      }
  }
  ```
  
  由于通常情况下旋转数组`{3,4,5,1,2}`的前半部分总比后半部分大，因此循环条件设置为left元素大于等于right元素，每次获得mid位置元素后，如果它比left元素大，说明它还在前半部分，于是将left设置为mid；如果它比right元素小，说明它在后半部分，于是将right设置为mid。如果right元素与left元素相邻，此时仍然满足条件left元素大于等于right元素，则right元素即为最小元素。
  
  特殊情况处理：
  
  <img src="img/offer/q11.png" style="zoom:67%;" />
  
  当left、mid、right元素都相同时，此时无法判断mid元素到底属于前半部分还是后半部分。在这种情况下只能进行顺序查找。

### 12 矩阵中的路径

```
请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

示例 1：
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true


示例 2：
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false


提示：
	1 <= board.length <= 200
	1 <= board[i].length <= 200
```

- 垃圾写法

  ```java
  package cn.dut.offer;
  
  import java.util.ArrayList;
  import java.util.List;
  
  public class Q12 {
  
      public static void main(String[] args) {
          char[][] board = new char[][]{  {'a', 'b', 'c', 'e'},
                                          {'s', 'f', 'c', 's'},
                                          {'a', 'd', 'e', 'e'}};
          Q12 q12 = new Q12();
          System.out.println(q12.exist(board, "abce"));
      }
  
      List<String> results = new ArrayList<>();
      public boolean exist(char[][] board, String word) {
          if (board == null || board.length == 0 || board[0].length == 0) return false;
          int rows = board.length;
          int cols = board[0].length;
          boolean[][] visited = new boolean[rows][cols];
          StringBuilder sb = new StringBuilder();
          for (int i = 0; i < rows; i++) {
              for (int j = 0; j < cols; j++) {
                  backtrack(board, word, i, j, sb, visited);
                  if (!results.isEmpty() && word.equals(results.get(0))) {
                      return true;
                  }
              }
          }
          return false;
      }
  
      public void backtrack(char[][] board, String word, int i, int j, StringBuilder sb, boolean[][] visited) {
          if (sb.length() == word.length() && word.equals(sb.toString())) {
              results.add(sb.toString());
              return;
          }
          if (i >= 0 && i <= board.length - 1 && j >= 0 && j <= board[0].length - 1 && !visited[i][j]) {
              sb.append(board[i][j]);
              visited[i][j] = true;
              backtrack(board, word, i - 1, j, sb, visited);
              backtrack(board, word, i, j - 1, sb, visited);
              backtrack(board, word, i + 1, j, sb, visited);
              backtrack(board, word, i, j + 1, sb, visited);
              sb.deleteCharAt(sb.length() - 1);
              visited[i][j] = false;
          }
      }
  }
  ```

- 更巧妙的写法

  ```java
  class Solution {
      
      public boolean exist(char[][] board, String word) {
          if (board == null || board.length == 0 || board[0].length == 0) return false;
          int rows = board.length;
          int cols = board[0].length;
          boolean[][] visited = new boolean[rows][cols];
          for (int i = 0; i < rows; i++) {
              for (int j = 0; j < cols; j++) {
                  if(backtrack(board, word, i, j, 0, visited)) return true;
              }
          }
          return false;
      }
  
      public boolean backtrack(char[][] board, String word, int i, int j, int curLen, boolean[][] visited) {
          if (curLen == word.length()) {
              return true;
          }
          boolean flag = false;
          if (i >= 0 && i <= board.length - 1 && j >= 0 && j <= board[0].length - 1 && board[i][j] == word.charAt(curLen) && !visited[i][j]) {
              curLen++;
              visited[i][j] = true;
              flag = backtrack(board, word, i - 1, j, curLen, visited) ||
                      backtrack(board, word, i, j - 1, curLen, visited) ||
                      backtrack(board, word, i + 1, j, curLen, visited) ||
                      backtrack(board, word, i, j + 1, curLen, visited);
              curLen--;
              visited[i][j] = false;
          }
          return flag;
      }
  }
  ```

  使用回溯法，由于不知道起点在矩阵中的哪个位置，因此只能对每个位置都进行一遍回溯。

  但这道题并不需要将路径保存下来，因为target字符串已经是路径了，在回溯过程中使用一个变量保存当前匹配的长度，判断当前字符是否是路径中的字符即可，如果不是就完全不需要继续回溯。

  另外回溯函数也可以使用返回值，只要四个方向上有一个能达成target路径即可，使用或关系完成。

### 13 机器人的运动范围

```
地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

示例 1：
输入：m = 2, n = 3, k = 1
输出：3

示例 2：
输入：m = 3, n = 1, k = 0
输出：1

提示：
	1 <= n,m <= 100
	0 <= k <= 20
```

- DFS

  ```java
  class Solution {
      int m;
      int n;
      int k;
      boolean[][] visited;
  
      public int movingCount(int m, int n, int k) {
          //dfs
          this.m = m;
          this.n = n;
          this.k = k;
          this.visited = new boolean[m][n];
          return dfs(0, 0, 0, 0);
      }
  
      public int dfs(int i, int j, int si, int sj) {
          if (i > m - 1 || j > n - 1 || k < si + sj || visited[i][j] == true) return 0;
          visited[i][j] = true;
          return 1 + dfs(i + 1, j, (i + 1) % 10 == 0 ? si - 8 : si + 1, sj) + dfs(i, j + 1, si, (j + 1) % 10 == 0 ? sj - 8 : sj + 1);
      }
  }
  ```

  dfs的参数列表为当前的坐标$(i,j)$以及当前坐标的数位和。

  当一个坐标从$9\rightarrow10$时，数位和会$-8$，否则数为和$+1$，注意这里的数位和是从0累加上来的，不需要单独计算。

- BFS

  ```java
  class Solution {
      public int movingCount(int m, int n, int k) {
  
          Queue<int[]> queue = new LinkedList<>();
          boolean[][] visited = new boolean[m][n];
          queue.add(new int[]{0, 0});
          visited[0][0] = true;
          int res = 0;
  
          while (!queue.isEmpty()) {
              int[] pos = queue.poll();
              int i = pos[0], j = pos[1];
              if (i >= 0 && i < m && j + 1 >= 0 && j + 1 < n && !visited[i][j + 1] && can(i, j + 1, k)) {
                  queue.add(new int[]{i, j + 1});
                  visited[i][j + 1] = true;
              }
              if (i + 1 >= 0 && i + 1 < m && j >= 0 && j < n && !visited[i + 1][j] && can(i + 1, j, k)) {
                  queue.add(new int[]{i + 1, j});
                  visited[i + 1][j] = true;
              }
              res++;
          }
          return res;
      }
  
      public boolean can(int i, int j, int k) {
          int sum = 0;
          while (i > 0) {
              sum += i % 10;
              i /= 10;
          }
          while (j > 0) {
              sum += j % 10;
              j /= 10;
          }
          return sum <= k;
      }
  }
  ```

### 14 剪绳子

```
给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m-1] 。请问 k[0]*k[1]*...*k[m-1] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

示例 1：
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1

示例 2:
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36

提示：
	2 <= n <= 58
```

- 动态规划

  ```java
  class Solution {
      public int cuttingRope(int n) {
          if(n < 2) return 0;
          if(n == 2) return 1;
          if(n == 3) return 2;
  
          int[] res = new int[n+1];
          res[0] = 0;
          res[1] = 1;
          res[2] = 2;
          res[3] = 3;
          return cuttingRope(n, res);
      }
  
      public int cuttingRope(int n, int[] res) {
          if(res[n] != 0) return res[n];
          for(int i = 1; i < n; i++) {
              res[n] = Math.max(res[n], cuttingRope(i, res) * cuttingRope(n-i, res));
          }
          return res[n];
      }
  }
  ```

  状态方程：$f(n)=max(f(i)\times f(n-i))，i\in(0,n)$，即一段长度为n的绳子，首先剪一刀，得到长度为i和长度为n-i的两端绳子，乘积结果应当等于这两段绳子的乘积结果的积。由于并不知道一分为二的时候要剪多少，因此需要做出选择，即1/n-1，2/n-2等等，这些情况下的最大值即为最终结果。

  注意之处：绳子长度本身为1、2、3时，最终结果应当为0、1、2。但是当绳子长度大于3时，计算过程中遇到的子绳子长度为1、2、3，它们应当为1、2、3。如长度为4，剪为1/3，乘积为3；剪为2/2，乘积为4。

- 贪婪

  ```java
  class Solution {
      public int cuttingRope(int n) {
          if(n < 2) return 0;
          if(n == 2) return 1;
          if(n == 3) return 2;
  
          int timesOf3 = n / 3;
          if(n - 3 * timesOf3 == 1) 
              timesOf3 -= 1;
          int timesOf2 = (n - 3 * timesOf3)/2;
          return (int) Math.pow(3, timesOf3) * (int) Math.pow(2, timesOf2);
      }
  }
  ```

  尽可能地划分出3，剩下4时，划分2。

  这是由于$n\geq5$时，$2(n-2)\gt n$，$3(n-3)\gt n$且$3(n-3)\gt 2(n-2)$，因此尽可能多剪为长度为3的绳子。而当$n=4$时，$2\times2\gt1\times3$，因此剩余长度为4时剪为长度为2的绳子。

### 15 二进制中1的个数

```
请实现一个函数，输入一个整数，输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。
```

- 巧妙方法

  ```java
  public class Solution {
      // you need to treat n as an unsigned value
      public int hammingWeight(int n) {
          int count = 0;
          while(n != 0) {
              n = n & (n - 1);
              count++;
          }
          return count;
      }
  }
  ```

  直接将整数右移，由于负数时右移需要保持负号，因此填补的是1。这样写无法处理负数。

  或者将一个数1每次左移1位，与整数位与，直到它变为负，这样需要循环32次。

  巧妙方法为每次将整数减1再和原整数位与，就是让它最右边的1归零，而左边各位均不变。因此可以让n与n-1位与，这样有几个1就循环几次。如$1100-1=1011$，$1100 \& 1011=1000$。

  <img src="img/offer/q15.png" style="zoom: 67%;" />

### 16 数值的整数次方

```
实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

示例 1:
输入: 2.00000, 10
输出: 1024.00000

示例 2:
输入: 2.10000, 3
输出: 9.26100

示例 3:
输入: 2.00000, -2
输出: 0.25000
解释: 2-2 = 1/22 = 1/4 = 0.25

说明:
	-100.0 < x < 100.0
	n 是 32 位有符号整数，其数值范围是 [−231, 231 − 1] 
```

- 比较傻的方法

  ```java
  class Solution {
      public double myPow(double x, int n) {
          if(x == 0.0) return 0.0;
          if(n == 0.0) return 1.0;
          if(x == 1.0) return x;
          if(x == -1.0) return n % 2 == 0 ? -x : x;
          
          final double eps = 1e-10;
          long N = n;
          N = N > 0 ? N : -N;
          x = n > 0 ? x : 1.0 / x;
          double res = 1.0;
          for(long i = 1; i <= N; i++) {
              res = res * x;
              if(abs(res) < eps) return x < 0 ? -0.0 : 0.0;
          }
          return res;
      }
  
      public double abs(double x) {
          return x < 0 ? -x : x;
      }
  }
  ```

  对各种特殊情况直接处理；n可能取到整数的最小值，此时取反会超过范围，因此放入long类型再取绝对值；

  对于类似$0.1^{100000000}$的情况，使用一个epsilon进行限制，直接返回0。

- 递归写法

  ```java
  class Solution {
      public double myPow(double x, int n) {
          if(n == 0) return 1.0;
          if(n == 1) return x;
          if(n == -1) return 1.0 / x;
          double half = myPow(x, n / 2);
          double mod = myPow(x, n % 2);
          return half * half * mod;
      }
  }
  ```

  由于$a^n=\left\{\begin{aligned} &a^\frac{n}{2}\times a^\frac{n}{2}\times a^0& n为偶数 \\ &a^\frac{n-1}{2}\times a^\frac{n-1}{2}\times a^1&n为奇数\end{aligned}\right.$，因此可以使用递归在`O(log(n))`时间内完成。

### 17 打印从1到最大的n位数

```
输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

示例 1:
输入: n = 1
输出: [1,2,3,4,5,6,7,8,9]
```

- 字符串相加

  ```java
  package cn.dut.offer;
  
  public class Q17 {
      public static void main(String[] args) {
          final int n = 3;
          StringBuilder res = new StringBuilder("1");
          while (res.length() <= n) {
              System.out.print(res.toString() + " ");
              res = strAdd(res);
          }
      }
  
      private static StringBuilder strAdd(StringBuilder res) {
          int j = res.length() - 1;
          int num = res.charAt(j) - '0' + 1;
          int carry = num / 10;
          res.deleteCharAt(j);
          res.insert(j, num % 10 + "");
          while (carry == 1 && j > 0) {
              j--;
              num = res.charAt(j) - '0' + carry;
              carry = num / 10;
              res.deleteCharAt(j);
              res.insert(j, num % 10 + "");
          }
          if (carry == 1) res.insert(0, '1');
          return res;
      }
  }
  ```

  由于n没有指定，因此无法保证数字类型能够容纳，因此需要考虑使用字符串或者数组表示大数。Java中字符串可能更灵活一点。C++中的字符串与数组类似，需要提前指定好字符串的长度。

  字符串初始化为“1”，然后采用字符串模拟数字相加的方法，每次为字符串表示的数字+1，并打印。

  这里需要考虑进位的情况，例如`99+1=100`，当进位为1且没有到达数字最高位时，循环相加，最高位处理完之后如果还有进位就在字符串开头新加一个“1”。

- 递归写法

  ```java
      public void printMaxNDigits(int n) {
          if (n <= 0) return;
          char[] number = new char[n];
  
          for (int i = 0; i < 10; i++) {
              number[0] = (char) (i + '0');
              printMaxNDigitsRecursively(number, n, 0);
          }
      }
  
      public void printMaxNDigitsRecursively(char[] number, int length, int index) {
          if (index == length - 1) {
              print(number);
              return;
          }
          for (int i = 0; i < 10; i++) {
              number[index + 1] = (char) (i + '0');
              printMaxNDigitsRecursively(number, length, index + 1);
          }
          System.out.println();
      }
  
      public void print(char[] number) {
          boolean isBeginning0 = true;
          for (int i = 0; i < number.length; i++) {
              if (isBeginning0 && number[i] != '0')
                  isBeginning0 = false;
              if (!isBeginning0)
                  System.out.print(number[i]);
          }
          System.out.print("\t");
      }
  ```

  打印1-999…就是所有n位数字的全排列，因此可以使用类似回溯的方法递归解决。

### 18 O(1)时间删除链表的节点

给定单向链表的头指针和一个节点指针，定义一个函数在`O(1)`时间内删除该节点。

<img src="img/offer/q18.png" style="zoom: 67%;" />

如果只给定了值，那么需要遍历链表，找到被删除节点的前一个节点，才能完成删除操作。

现在给出了被删除节点的指针，那么是知道它后面节点的信息的，因此可以将它的值替换为下一个节点的值，然后将它指向下下个节点，这样与删除的效果类似。

需要处理的特殊情况是，如果链表中只有一个节点，那么直接将头指针设为null即可。如果被删除节点是最后一个节点，那么只能通过遍历找到它的前一个节点，然后进行删除。

平均时间复杂度为$[(n-1)O(1)+O(n)]/n$仍然为`O(1)`。

另外`O(1)`的时间复杂度基于节点指针确实指向了链表中的节点，否则为了确定此事也需要`O(n)`的时间。

#### 删除链表中重复的节点

在一个链表中删除重复节点。

```
0->0->1->1->1->3->4->5->5->6->6->7->7->8->8->8->8->8->9->9
3->4
```

```java
package cn.dut.offer;

import java.util.Random;

public class Q18 {

    static class ListNode {
        int val;
        ListNode next;

        public ListNode(int val) {
            this.val = val;
        }
    }

    public static void main(String[] args) {
        ListNode head = null;
        Random random = new Random();
        for (int i = 0; i < 20; i++) {
            int val = random.nextInt(10);
            if (head == null) {
                head = new ListNode(val);
            } else {
                if (head.val >= val) {
                    ListNode oldHead = head;
                    head = new ListNode(val);
                    head.next = oldHead;
                } else {
                    ListNode tmpHead = head;
                    while (tmpHead.next != null && tmpHead.next.val < val) tmpHead = tmpHead.next;
                    ListNode tmp = tmpHead.next;
                    tmpHead.next = new ListNode(val);
                    tmpHead.next.next = tmp;
                }
            }
        }

        ListNode tmpHead = head;
        while (tmpHead != null) {
            System.out.print(tmpHead.val + "->");
            tmpHead = tmpHead.next;
        }
        System.out.println();
        tmpHead = new Q18().deleteDup(head);
        while (tmpHead != null) {
            System.out.print(tmpHead.val + "->");
            tmpHead = tmpHead.next;
        }
    }

    public ListNode deleteDup(ListNode head) {
        if (head == null) return null;
        ListNode pre = null;
        ListNode cur = head;
        ListNode next = head.next;
        while (next != null) {
            if (cur.val != next.val) {
                pre = cur;
                cur = next;
                next = next.next;
            } else {
                while (next != null && cur.val == next.val) next = next.next;
                if (pre == null) head = next;//说明开始的节点就重复
                else pre.next = next;
                cur = next;
                if (next != null) next = next.next;
            }
        }
        return head;
    }
}
```

由于这个题需要把重复的节点全部删掉，一个不留，因此使用三个指针操作。

### 19 正则表达式

```
请实现一个函数用来匹配包含'. '和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（含0次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但与"aa.a"和"ab*a"均不匹配。

示例 1:
输入:
s = "aa"
p = "a"
输出: false
解释: "a" 无法匹配 "aa" 整个字符串。


示例 2:
输入:
s = "aa"
p = "a*"
输出: true
解释: 因为 '*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。


示例 3:
输入:
s = "ab"
p = ".*"
输出: true
解释: ".*" 表示可匹配零个或多个（'*'）任意字符（'.'）。


示例 4:
输入:
s = "aab"
p = "c*a*b"
输出: true
解释: 因为 '*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。


示例 5:
输入:
s = "mississippi"
p = "mis*is*p*."
输出: false

	s 可能为空，且只包含从 a-z 的小写字母。
	p 可能为空，且只包含从 a-z 的小写字母以及字符 . 和 *，无连续的 '*'。
```

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if (s == null || p == null)
            return false;
        return match(s, p, 0, 0);
    }

    private boolean match(String s, String p, int sPointer, int pPointer) {
        if (sPointer == s.length() && pPointer == p.length())
            return true;
        if (sPointer != s.length() && pPointer == p.length())
            return false;
        if (pPointer + 1 < p.length() && p.charAt(pPointer + 1) == '*') {
            if (sPointer < s.length() && (p.charAt(pPointer) == s.charAt(sPointer) || p.charAt(pPointer) == '.'))
                return match(s, p, sPointer + 1, pPointer + 2)
                        || match(s, p, sPointer + 1, pPointer)
                        || match(s, p, sPointer, pPointer + 2);
            else return match(s, p, sPointer, pPointer + 2);
        }
        if (sPointer < s.length() && (s.charAt(sPointer) == p.charAt(pPointer) || p.charAt(pPointer) == '.'))
            return match(s, p, sPointer + 1, pPointer + 1);
        return false;
    }
}
```

正则表达式匹配就是画出模式字符串对应的非确定有限状态机（NFA），然后在给定字符串上模拟它的运行。

NFA是一个有向图，可以用有向图的模型来实现。这本书给出的解法使用递归挺巧妙的，非确定的情况是使用或运算模拟的。

由于Java没有指针并且字符串末尾也没有'\0'，因此需要用一个int值模拟指针，并且使用该值判断匹配字符串是否越界（因为判断匹配成功与否的条件只有两个：一是两个指针均到达了末尾，说明匹配成功；二是模式字符串的指针已经到达了末尾，但匹配字符串的指针还没有，说明匹配失败。但是注意状态机运行的情况中存在模式字符串指针不动，匹配字符串指针移动的情况，因此可能会发生匹配字符串越界）。

<img src="img/offer/q19.png" style="zoom:67%;" />

状态机遇到”*“且与当前字符相同时，接下来的三种可能运行情况：

`match(s, p, sPointer + 1, pPointer + 2)`：进入到下一个状态，并且检查下一个字符。
`match(s, p, sPointer + 1, pPointer)`：保持原状态不变，并且检查下一个字符。
`match(s, p, sPointer, pPointer + 2)`：忽视掉当前状态，仍然检查当前字符。因为"*"代表可能出现0~任意次，因此可以直接跳过它，如字符串”bab“只有在这种情况下会匹配成功。

### 20 表示数值的字符串

```
请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"、"-1E-16"及"12e+5.4"都不是。
```

- 有限状态自动机（DFA）

  <img src="img/offer/q20.png" style="zoom:50%;" />

  作者：jyd
  链接：https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/solution/mian-shi-ti-20-biao-shi-shu-zhi-de-zi-fu-chuan-y-2/
  来源：力扣（LeetCode）
  著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

  虽然知道应该用DFA，但这个题的情况太复杂了。一方面状态转移图考虑不全面，另一方面使用if-else编程也不太清晰明了。

  ```java
  class Solution {
      public boolean isNumber(String s) {
          if (s == null || s.length() == 0) return false;
          Map[] states = {
                  new HashMap(){{ put(' ',0); put('s',1); put('d',2); put('.',4); }},
                  new HashMap(){{ put('d',2); put('.',4); }},
                  new HashMap(){{ put('d',2); put('e',5); put('.',3); put(' ',8); }},
                  new HashMap(){{ put('d',3); put('e',5); put(' ',8); }},
                  new HashMap(){{ put('d',3); }},
                  new HashMap(){{ put('d',7); put('s',6); }},
                  new HashMap(){{ put('d',7); }},
                  new HashMap(){{ put('d',7); put(' ',8); }},
                  new HashMap(){{ put(' ',8); }}
          };
          char t;
          int p = 0;
          for (char c : s.toCharArray()) {
              if (c >= '0' && c <= '9') t = 'd';
              else if (c == '+' || c == '-') t = 's';
              else if (c == ' ' || c == 'e' || c == '.') t = c;
              else t = '?';
              if (!states[p].containsKey(t)) return false;
              p = (int) states[p].get(t);
          }
          return p == 2 || p == 3 || p == 7 || p == 8;
      }
  }
  ```

  使用map数组存储状态转移表，这里有个初始化map的奇技淫巧：匿名内部类+实例化代码块。但仅仅为了代码简洁，最好不要用这种方式初始化，它会生成很多不必要的内部类，加重类加载器负担，还可能造成内存泄漏。

- 基于规则

  ```java
  class Solution {
      public boolean isNumber(String s) {
          if(s==null||s.length()==0) return false;
  
          boolean numSeen = false;
          boolean dotSeen = false;
          boolean eSeen = false;
  
          char[] chars = s.strip().toCharArray();
          for(int i=0; i<chars.length; i++) {
              char c = chars[i];
              if(c >= '0' && c <= '9') {
                  numSeen = true;
              } else if(c == '.') {
                  if(dotSeen || eSeen) return false;
                  dotSeen = true;
              } else if(c == 'e') {
                  if(!numSeen || eSeen) return false;
                  eSeen = true;
                  numSeen = false;
              } else if(c == '+' || c == '-') {
                  if(i != 0 && (chars[i-1] != 'e' || chars[i-1] != 'E')) return false;
              } else {
                  return false;
              }
          }
          return numSeen;
      }
  }
  ```

  首先去掉字符串两边的空格字符；定义三个状态，numSeen、dotSeen、eSeen。

  规则：

  1. 遇到数字，numSeen为true。
  2. 遇到"."，如果已经遇到过"."或者"e"，返回false，如2.2.2，1e2.2；否则dotSeen为true。
  3. 遇到"e"，如果没有遇到过数字或者已经遇到过"e"，返回false，如e2，1e2e3；否则eSeen为true，并且将numSeen重置为false。
  4. 遇到"+"、"-"，如果它不在开头并且前一个字符不是"e"，返回false。
  5. 其他情况均为false。
  6. 结束遍历后，有效状态为numSeen，而不是直接返回true。如1.e，最后numSeen被重置，仍然返回false。

### 21 调整数组顺序使奇数位于偶数前面

```
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

示例：
输入：nums = [1,2,3,4]
输出：[1,3,2,4] 
注：[3,1,2,4] 也是正确的答案之一。

提示：
	1 <= nums.length <= 50000
	1 <= nums[i] <= 10000
```

- 双指针

```java
class Solution {
    public int[] exchange(int[] nums) {
        if(nums==null || nums.length==0) return new int[0];
        int n = nums.length;
        int left = 0, right = n-1;

        while(left<right) {
            while(left<right && nums[left]%2!=0) left++;
            while(left<right && nums[right]%2==0) right--;
            if(left<right) {
                int t = nums[left];
                nums[left] = nums[right];
                nums[right] = t;
            }
        }
        return nums;
    }
}
```

### 22 链表中倒数第k个节点

```
输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。例如，一个链表有6个节点，从头节点开始，它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个节点是值为4的节点。

示例：
给定一个链表: 1->2->3->4->5, 和 k = 2.
返回链表 4->5.
```

- 先遍历一遍获取长度

- 双指针

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
      public ListNode getKthFromEnd(ListNode head, int k) {
          if(head==null || k<=0) return null;
          ListNode left = head;
          ListNode right = head;
          int i=1;
          for(;i<=k;i++) {
              if(right==null) break;
              right = right.next;
          }
          if(i<=k) return null;
          while(right!=null){
              left = left.next;
              right = right.next;
          }
          return left;
      }
  }
  ```

  right先移动k步，然后同时移动left和right，直到right为null，此时left指向倒数第n个节点。

  关键是考虑鲁棒性，处理k大于链表长度的情况。

  当k小于等于链表长度时，移动k步后正常退出循环，计数i比k大1。

  当k大于链表长度时，移动到链表末尾靠break退出循环，计数i小于等于k。

  如果采用先移动k-1步的方式，那么可以用right是否为null判断k合不合理。

  ```java
  class Solution {
      public ListNode getKthFromEnd(ListNode head, int k) {
          if(head == null || k <= 0) return null;
          ListNode left = head;
          ListNode right = head;
          int i = 0;
          for(; i < k - 1; i++) {
              if(right == null) break;
              right = right.next;
          }
          if(right == null) return null;
          while(right.next != null){
              left = left.next;
              right = right.next;
          }
          return left;
      }
  }
  ```

### 23 链表中环的入口节点

如果一个链表中包含环，如何找出环的入口节点？

- 使用set记忆

  ```
  public ListNode entry(ListNode head) {
  	Set<ListNode> mem = new HashSet<>();
  	while(head != null) {
  		if(mem.contains(head)) return head;
  		mem.add(head);
  		head = head.next;
  	}
  	return head;
  }
  ```

- 双指针

  ```java
  public ListNode entry(ListNode head) {
  	if(head == null) return null;
  	ListNode slow = head;
  	ListNode fast = head.next;
  	while(slow != fast) {
  		if(slow != null) slow = slow.next;
  		if(fast != null) fast = fast.next;
  		if(fast != null) fast = fast.next;
  	}
  	if(slow == null) return null;
      
  	int count = 0;
  	fast = slow.next;
  	while(slow != fast) {
  		slow = slow.next;
  		fast = fast.next.next;
  		count++;
  	}
  	count++;
      
  	slow = head;
  	fast = head;
  	for(int i=0; i<count; i++, fast = fast.next);
  	while(slow != fast) {
  		slow = slow.next;
  		fast = fast.next;
  	}
  	return slow;
  }
  ```

  首先用快慢指针判断是否有环，当slow==fast时，它们为null则没有环。

  然后继续用快慢指针对环的节点计数，当slow==fast时，计数器还需要加1。例如slow：3-4-5-6，fast：4-6-4-6，循环进行了三次，但有4个节点。

  最后用双指针，类似于22的思路，环有n个节点，则fast先移动n步，然后slow和fast同时移动，当它们相等时则为入口节点。

  <img src="img/offer/q23.png" style="zoom:67%;" />

### 24 反转链表

```
定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

示例:
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

限制：
0 <= 节点个数 <= 5000
```

- 循环

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
          if(head == null) return null;
          ListNode pre = null;
          ListNode cur = head;
          ListNode next;
  
          while(cur != null) {
              next = cur.next;
              cur.next = pre;
              pre = cur;
              cur = next;
          }
          return pre;
      }
  }
  ```

- 递归

  ```java
  class Solution {
      public ListNode reverseList(ListNode head) {
          if(head == null) return null;
          if(head.next == null) return head;
          ListNode rt = reverseList(head.next);
          head.next.next = head;
          head.next = null;
          return rt;
      }
  }
  ```

  首先沿着链表深入，到达尾节点时停止，返回尾节点。沿着递归树回退时，修改每个节点和下一个节点的指向。如返回到4时，`head.next.next = head;`：$1\rightarrow2\rightarrow3\rightarrow4\leftrightarrow5$，`head.next = null;`：$1\rightarrow2\rightarrow3\rightarrow4\leftarrow5$。