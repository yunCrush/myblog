# Untitled

1.链表翻转：1-&gt;2-&gt;3-&gt;4-&gt;5  ===&gt;  5-&gt;4-&gt;3-&gt;2-&gt;1

```text
# 需要一个pre节点，记录当前节点的前一个节点，记录下一个节点的next节点，记录当前节点的cur,翻转
# 链表的关键在于时cur.next指向pre，随后移动pre与cur
ListNode cur=head;
ListNode next = null;
ListNode pre = null;
while(cur!=null){
    next = cur.next;
    cur.next = pre;
    pre = cur;
    cur = next;
}
return pre;
```

2.判断链表有环

```text
# 快慢指针
while(fast && fast.next && fast.next.next){
    fast = fast.next.next;
    slow = slow.next;
}
```

3.约瑟夫环问题

```text
# 循环链表来解决，将取出的数放入队列尾部，如果出队列次数i，刚好等于M,则不再进入队列
while (q.size() > 0) {
        element = q.poll();
        if (i < m) {
            q.add(element);
            i++;
        } else {
            i = 1;
            System.out.println(element);
        }
    }
```

4.快排

```text
 private static void help(int[] arr, int low, int high) {

        if(low >= high){
            return ;
        }
        int i = low;
        int j = high;
        int temp = arr[low];
        while(i<j){
//            找到右边第一个小于temp的
            while(i<j && temp<=arr[j] ){
                j--;
            }
//            找到左边大于temp的
            while(i<j && temp >=arr[i]){
                i++;
            }
            int t = arr[i];
            arr[i]=arr[j];
            arr[j]=t;
        }
        arr[low] = arr[i];
        arr[i]=temp;
        help(arr, low, j-1);
        help(arr,j+1,high);

    }
```

```text
# 冒泡、选择、插入、快排、归并、堆排序、桶排序
# 数学技巧
n & (n-1) == 0  判断是否是2的幂
# 判断n中1的个数
int count=0;
while(n>0){
 count++;
 n &= n-1;

}
```



