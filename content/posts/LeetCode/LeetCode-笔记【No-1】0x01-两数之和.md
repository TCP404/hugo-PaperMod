---
title: "LeetCode-笔记【No-1】0x01-两数之和"
tags:
  - 算法
  - LeetCode
categories: ["LeetCode"]
date: 2019-04-25 10:44:47
draft: false
toc: false
images:
math: true
---

LeetCode 第1题


<!--more-->



### 题目

给定一个整数数组  `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 **两个** 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

### 示例

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[**0**] + nums[**1**] = 2 + 7 = 9

所以返回 [**0, 1**]

### 思路

思路非常简单，就是把要找的数字target，减去第一个的差，是否等于第二个数

如果是，返回这两个数的下标，如果不是，循环继续。

### 代码

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length - 1; i++) {
            for (int j = i+1; j < nums.length; j ++) {
                if (nums[j] == target - nums[i]) {
                    return new int[] { i , j };
                }
            }
        }
        return null;
    }
}
```



第一层循环取第一个数i，然后进入第二层循环，

第二层循环从第二个数j 开始，判断要找的数-第一个数i 的差 是否等于 第二个数 j

如果不是第二层取第三个数，再进行比较。

如果第二层遍历完还没结果说明第一层的第一个数i不对，所以第一层取第二个数

再次进入第二层依次遍历。

当差等于第二层的某个数的时候，说明找到了，返回i和j的下标。

时间复杂度：O(n)

空间复杂度：O(n)

