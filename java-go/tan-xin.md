---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 基础算法

### 二分搜索

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

```
// 区间[l, r]被划分成[l, mid - 1]和[mid, r]时使用：第一个临界点，l=mid,反推mid = l + r + 1 >> 1.r=mid -1.先减后加
int bsearch_1(int l, int r)
{
    while (l < r){
        // 根据check(mid) l = mid反推 mid = l + r + 1
        int mid = l + r + 1 >> 1;
        if (check(mid)) l = mid;
        else r = mid - 1;
    }
    return l;
}
```

```
// 区间[l, r]被划分成[l, mid]和[mid + 1, r]时使用：第二个临界点
int bsearch_2(int l, int r){
    while (l < r){
        int mid = l + r >> 1;
        if (check(mid)) r = mid;    // check()判断mid是否满足性质
        else l = mid + 1;
    }
    return l;
}

```

### 前缀和

```
// 下标从1开始 s[0] = 0 为了处理边界问题
S[i] = a[1] + a[2] + ... a[i]
a[l] + ... + a[r] = S[r] - S[l - 1]
for (int i = 1; i <= n; i++) {
    s[i] = s[i-1] + a[i];
}
// 二维前缀和
S[i, j] = 第i行j列格子左上部分所有元素的和
以(x1, y1)为左上角，(x2, y2)为右下角的子矩阵的和为：
S[x2, y2] - S[x1 - 1, y2] - S[x2, y1 - 1] + S[x1 - 1, y1 - 1]
```

## 差分

```
对差分数组求前缀和得到原数组b[i] += b[i-1]
原数组[a,b,c,d,e]
差分数组[a,b-a,c-b,d-c,e-c]

给区间[l, r]中的每个数加上c：B[l] += c, B[r + 1] -= c
给(l,r)每个数都加c,则第一个加c,因为对差分数组求前缀和，r+1只要在数组范围内则减c
注意：下标从1开始，与返回的数组从0 开始区分
原数组a===>差分数组b
int[] b = new int[100010];
for (int i = 1; i <= a.length; i++) {
// 构造差分数组
    insert(i,i,a[i]);
}
// 形成差分数组的关键
 public void insert(int l, int r, int c){
        b[l]+=c;
     if (r + 1 <= length()) {
           b[r+1] -=c;
     }
    }
 for (int i = 1; i <= length; i++) {
     b[i] += b[i-1];
 }
 
 // 二维差分
 给以(x1, y1)为左上角，(x2, y2)为右下角的子矩阵中的所有元素加上c：
S[x1, y1] += c, S[x2 + 1, y1] -= c, S[x1, y2 + 1] -= c, S[x2 + 1, y2 + 1] += c
```

## 双指针

```
// 滑动窗口
for(int i = 0, j = 0; i < m; i++) {
    // j表示与i的距离，最开始是0，向左移动，构成区间,但是J的下标并不是索引下标
    // j 从左开始，向右开始收缩，
    
    while(j < i && check(i,j)) {
        j++;
    }
    // 处理逻辑
}
```

## 单调栈

```
public void monotonicStack(int[] nums) {
        Stack<Integer> stack = new Stack();
        for ( int i = 0; i < nums.length; i++) {
            // 单调递增栈 >;单调递减栈 <; 
            while(!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
                stack.pop();
                // 相关逻辑
            }
            stack.push(i);
        }
    }
```

## BFS

```
public void bfs() {
        Queue queue = new LinkedList<>();
        queue.add();
        while(!queue.isEmpty()) {
            int len = queue.size();
            for ( int i = 0; i < len; i ++) {
                t = queue.poll();
            //    扩展 t
            }
        }
    }
```

## 并查集

1. 将两个集合合并
2. 询问两个元素是否在一个集合当中

```
1.p[x]表示x的父节点，根节点满足p[x]==x,其他节点都不满足
2. 树根的编号即当前集合的编号，求x的集合的编号： while (p[x] != x) { x = p[x];}
3.如何合并两个集合： px是x集合的编号，py是y集合的编号，p[x] = y 表示x集合是的父节点不再是x，而是y。
```

```
int p[N]; //存储每个点的祖宗节点
    // 返回x的祖宗节点
    int find(int x)
    {
        //精妙之处，利用递归，x = p[x] 反复找到根节点，且，还进行了路径压缩,直接p[x]指向了根节点
        if (p[x] != x) p[x] = find(p[x]);
        return p[x];
    }
    // 初始化，假定节点编号是1~n
    for (int i = 1; i <= n; i ++ ) p[i] = i;

    // 合并a和b所在的两个集合：a的祖宗节点的父亲是b的祖宗节点。直接把a拿来插到b集合里了。
    p[find(a)] = find(b);
```

## 链表

```
// 根据快指针来判断整个链路有奇数还是偶数个节点
// 题目：慢指针一边走，一边翻转，判断链表是否是回文链表
ListNode pre = null, fast = head, slow = head, cur = head;
while(fast != null && fast.next != null) {
    cur = slow;
    slow = slow.next;
    fast = fast.next.next;
    cur.next = pre;
    pre = cur;
}
// fast != null  true: 奇数个节点，且慢指针slow正好在中间位置节点上
// fast != null  false: 偶数个节点，此时慢指针走过了中间位置
if (fast != null) {
    slow = slow.next;
}
```

## 递归

```
// 终止条件

// 处理当前逻辑确定递归返回值与入参

// 进入下一层

//清理环境 可有可无

//返回值
```

### 贪心

1. 跳跃游戏

　　Description: 给定一个非负整数数组 nums ，你最初位于数组的 第一个下标 。数组中的每个元素代表你在该位置可以跳跃的最大长度。判断你是否能够到达最后一个下标

```
// 从后向前遍历，判断是否能到达endIndex
public boolean canJump(int[] nums) {
    if (nums == null) {
        return false;
    }
    int endIndex = nums.length-1;
    for (int i = endIndex; i >= 0; i--) {
        if (nums[i] + i >= endIndex） {
            endIndex = i;
        }
    }
    // 判断是否endIndex可否到达0,即可
    return endIndex == 0;
}
```

2\. 跳跃游戏2

给你一个非负整数数组 nums ，你最初位于数组的第一个位置。数组中的每个元素代表你在该位置可以跳跃的最大长度。你的目标是使用最少的跳跃次数到达数组的最后一个位置。假设你总是可以到达数组的最后一个位置。

```
    public int jump(int[] nums) {
        if (nums.length == 1) {
            return 0;
        }
        int end = 0;
        int maxIndex = nums[0];
        int step = 0;
        // 每次到达的最远位置，开始计算,注意点，中途可能直接跳出边界
        for (int i = 0; i < nums.length; i++) {
             maxIndex = Math.max(maxIndex, nums[i] + i);
             if (maxIndex >= nums.length - 1) {
                 return step + 1;
             }
            if (i == end){
                step++;
                end = maxIndex;
            }
        }
        return step;
    }
```

```
数组中:
    left与right中间有多少个值: right-left-1
    left到right需要走多少步: right-left
    left到right共有多少个数: right-left+1
    left到right之间长度： right - left
```

## 排序算法

2. 冒泡排序

```
    // 外层控制循环次数，内层筛选每次的max
    public void bubbleSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            for (int j = 0; j < arr.length - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    swap(arr, j, j + 1);
                }
            }
        }
    }
```

3. 插入排序

```

    // 最后一个数temp，往左开始慢慢一个个的比对，如果自己小，则一直往左移动，直到0,
    public void insertSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            int j;
            int temp = arr[i];
            for (j = i-1; j >= 0; j--) {
                if (temp < arr[j]) {
                    arr[j + 1] = arr[j];
                } else {
                    break;
                } 
            }
            arr[j + 1] = temp;
        }
    }
```

4. 选择排序

```

    public void selectSort(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            int minIndex = i;
            for (int j = i; j < arr.length; j++) {
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            if (minIndex != i) {
                swap(arr, i, minIndex);
            }
        }
    }
```

5.  快速排序

    ```

    // quickSort(nums,0,nums.length-1)     
    public void quickSort(int[] nums, int left, int right) {
        // left == right 无意义
            if (left >= right) {
                return ;
            }
            int x = nums[left];
            int i = left - 1, j = right + 1;
            while (i < j ) {
                while(nums[++i] < x);
                
                 while(nums[--j] > x);
                if (i < j) {
                    int t = nums[i];
                    nums[i] = nums[j];
                    nums[j] = t;
                }
            }
            quickSort(nums,left,j);
            quickSort(nums,j+1,right);
        }
    ```
6.  归并排序

    ```

    // mergeSort(nums,0,nums.length-1) 
    public void mergeSort(int[] nums, int left, int right) {
            if (left >= right) {
                return ;
            }
            int mid = left + right >> 1;
            mergeSort(nums,left, mid);
            mergeSort(nums,mid+1,right);
            int k = 0;
            int[] temp = new int [right - left + 1];
            int i = left, j = mid+1;
            while(i <= mid && j <= right) {
                if(nums[i] <= nums[j] ){
                    temp[k++] = nums[i++];
                } else {
                    temp[k++] = nums[j++];
                }
            }
            while(i <= mid) {
                temp[k++] = nums[i++];
            }
            while(j <= right) {
                temp[k++] = nums[j++];
            }
            for( i = left, j = 0; i <= right; i++,j++) {
                nums[i] = temp[j];
            }
        }

    ```

输入输出:

```
import java.util.*;
import java.io.*;
public class Main {
    public static void main(String[] args)  throws IOException {
         BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String str;
        while ((str = br.readLine()) != null) {
            // 处理
            String[] arr = str.split("\\s+");
            int row = Integer.parseInt(arr[0]);
            int col = Integer.parseInt(arr[1]);
            int[][] nums = new int[row][col];
            for ( int i = 0; i < row; i++) {
                String[] temp = br.readLine().split("\\s+");
                for (int j = 0; j < col; j++) {
                    nums[row][j] = Integer.parseInt(temp[j]);
                }
                
            }
        }
    }
}
```
