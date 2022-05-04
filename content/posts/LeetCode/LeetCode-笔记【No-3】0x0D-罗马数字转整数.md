---
title: "LeetCode-笔记【No-3】0x0D-罗马数字转整数"
tags:
  - 算法
  - LeetCode
categories: [LeetCode]
date: 2019-04-25 11:29:48
draft: false
toc: false
images:
math: true
---

LeetCode 第13题

<!--more-->



### 题目

罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

| 字符 | 数值 |
| ---- | ---- |
| I    | 1    |
| V    | 5    |
| X    | 10   |
| L    | 50   |
| C    | 100  |
| D    | 500  |
| M    | 1000 |

例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

> 通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

- I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
- X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
- C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

> 示例 1:

输入: "III"

输出: 3

> 示例 2:

输入: "IV"

输出: 4

> 示例 3:

输入: "IX"

输出: 9

> 示例 4:

输入: "LVIII"

输出: 58

解释: L = 50, V= 5, III = 3.

> 示例 5:

输入: "MCMXCIV"

输出: 1994

解释: M = 1000, CM = 900, XC = 90, IV = 4.

### 思路

1. 首先建立一个HashMap来映射符号和值
2. 然后对字符串从左到右来，如果当前字符代表的值不小于其右边，就加上该值；否则就减去该值。
3. 以此类推到最右边的数，最终得到的结果即是答案

这是LeetCode评论上的一条思路。非常简单粗暴。

### 代码

```java
class Solution {
    public int romanToInt(String s) {
        Map<Character,Integer> map = new HashMap<>();
        map.put('I',1);
        map.put('V',5);
        map.put('X',10);
        map.put('L',50);
        map.put('C',100);
        map.put('D',500);
        map.put('M',1000);
        int res = 0;
        int size = s.length();
        //最后一位不需要判断所以循环次数是size-1
        for (int i = 0; i < size - 1; i ++) {
            //如果左边的数字小于右边的数字
            if (map.get(s.charAt(i)) >= map.get(s.charAt(i+1))){
                 res += map.get(s.charAt(i));
            } else {
                //如果左边数字小于右边的数字（也就是通常4和9的情况）
                res -= map.get(s.charAt(i)); 
            }
        }
        //最后一位没被判断到，所以没被加上，记得加上去
        res += map.get(s.charAt(size - 1));
        return res;
    }
}

```

这里的if判断先写多数的情况，也就是左边的数大于右边的数。

这样才不会把时间浪费在 先判断少数情况（左边 小于右边），再执行多数情况的运算

