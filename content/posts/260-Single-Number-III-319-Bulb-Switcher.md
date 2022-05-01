---
title: "260 Single Number III 319 Bulb Switcher"
date: 2016-01-24T15:44:25+08:00
tags:
    - Leetcode
draft: true
---

最近也开始刷Leetcode,而且感觉也是时候更新一下Blog了,所以拿Leetcode的笔记来凑数(大雾

<!--more-->

## 260. Single Number III

> Given an array of numbers nums, in which exactly two elements appear only once and all the other elements appear exactly twice. Find the two elements that appear only once.
>
> For example:
> 
> Given nums = [1, 2, 1, 3, 2, 5], return [3, 5].

#### 分析

找出数组中两个不成对的数字，顺序不限，和Single Number I不同，不能用直接异或的方法进行计算。这里我们要用到位运算

> x & -x = 二进制中第一个1出现的位置
>
> 如 0b1010 & -0b1010 = 2 ^ 1 = 2

之后只要对mask & nums[i]不等于0的元素进行异或，遍历一轮后就能得出两个中的其中一个值，最后再和另一个数相异或即可

最终代码

```javascript
var singleNumber = function(nums) {
    var xor = nums.reduce((p, n) => p ^ n)
    var mask = xor & -xor
    var ans = 0
    nums.forEach(item => {
        if (mask & item) ans ^= item
    })
    return [ans, ans ^ xor]
};
```

## 319. Bulb Switcher

> There are n bulbs that are initially off. You first turn on all the bulbs. Then, you turn off every second bulb. On the third round, you toggle every third bulb (turning on if it's off or turning off if it's on). For the nth round, you only toggle the last bulb. Find how many bulbs are on after n rounds.
>
> Example:
>
> > Given n = 3.
> >
> > At first, the three bulbs are [off, off, off].
> >
> > After first round, the three bulbs are [on, on, on].
> >
> > After second round, the three bulbs are [on, off, on].
> >
> > After third round, the three bulbs are [on, off, off].
>
> So you should return 1, because there is only one bulb is on.

#### 分析

这里我们可以举几个例子进行分析，这里就取n = 20，不难看出，第m个按钮Toggle的次数就是它因子的个数

> m = 10 在 2 5 10 处翻转
>
> m = 15 在 3 5 15 处翻转
>
> m = 16 在 2 4 8 16 处翻转

从上述例子可以看出，当m开方后是整数（因子重复）时，翻转奇数次（包括第一次），翻转奇数次的最终结果就是灯泡为On，所以最终灯泡On的个数就是n个数中有因子重复的数，这个数字和sqrt(n)后向下取整在数值上是相等的。

最终代码（竟然就一行，2333333）

```javascript
var bulbSwitch = function(n) {
    return Math.floor(Math.sqrt(n))
};
```
