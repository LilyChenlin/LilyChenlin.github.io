---
title: leetcode-55.跳跃游戏
date: 2022-02-12 23:42:41
update: 2022-02-12 23:42:41
tags: 动态规划
categories: Algorithm
---
[来源] 🔗[55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

# 题目

给定一个非负整数数组 nums ，你最初位于数组的 第一个下标 。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标。

**示例 1:**

```JavaScript
输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
```

**示例 2：**

```JavaScript
输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。

```

# 思路

1. 首先这个思路和数组元素的索引值关系非常大。
2. 对于数组 [2, 3, 1, 1, 4] 我们尽可能的取最大的跳跃长度，那么最大跳跃长度能跳到的索引位置为 [2, 4, 3, 4](不包含数组最后一个元素)

    我们可以得出 k = Math.max(k, nums[i] + i)， k为能够跳到的最远位置。 

    如果此时k == nums.length - 1 那么就能够到达最后一个标
1. 此时考虑到以下情况，[1, 0, 2, 1, 4]，针对每一个元素，最大跳跃长度能跳到的索引位置为[1, 1, 4, 4]。但是此时i = 1时，不能进行跳跃，也就是无法跳到下一个节点，那么尽管后面的元素跳跃长度能够到达最后一个下标，此时结果也为false。

    所以我们需要通过i 和 k值进行对比，当且仅当i ≤ k时，该节点能够进行下一个跳跃

# 代码

```JavaScript
function canJump(nums: number[]): boolean {
    let maxLen = 0, len = nums.length;
    for (let i = 0; i < len; i++) {
        if (i > maxLen) return false;
        maxLen = Math.max(maxLen, nums[i] + i);
    }
    return true;
};
```