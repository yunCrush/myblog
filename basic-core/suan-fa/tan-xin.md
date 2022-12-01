# 贪心

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
