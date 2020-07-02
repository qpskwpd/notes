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

解法3：由于数组中的数字都在0~n-1范围内，如果没有重复数字，那排序后下标为i的数字就是i。因此可以利用这

个规律对数组进行重排：扫描到下标为$i$的数字$m$时，如果$i==m$，接着扫描下一个；如果$i!=m$，比较$m$与索

引为$m$的数字是否相等，如果$m=nums[m]$，则找到了一个重复数字，否则交换下标为$i$和$m$的数字，重复这个

过程。时间复杂度$O(n)$，空间复杂度$O(1)$。

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