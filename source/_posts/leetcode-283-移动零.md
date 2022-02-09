---
title: leetcode-283.移动零
date: 2022-02-09 20:22:29
update: 2022-02-09 20:22:29
tags: 双指针
categories: Algorithm
---
[来源] 🔗[283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)

# 题目

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

**示例 1:**

```JavaScript
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

# 思路

1. 这里由于只需要将0移动到数组的末尾，所以我们考虑在遍历数组的时候直接跳过0，等后续再去补齐。
2. 题目要求原地对数组进行操作，意味着我们需要定义一个变量来记录从哪里开始补齐0。
3. 使用双指针，快指针不断向右移动，遍历数组元素，慢指针用于记录已经处理好的序列的尾部。

# 代码

```JavaScript
/**
 Do not return anything, modify nums in-place instead.
 */
function moveZeroes(nums: number[]): void {
    let slow = 0, fast = 0;
    while (fast < nums.length) {
        if (nums[fast] !== 0) {
            nums[slow] = nums[fast];
            slow++;
        }
        fast++;
    }
    while(slow < nums.length) {
        nums[slow++] = 0;
    }
};
```