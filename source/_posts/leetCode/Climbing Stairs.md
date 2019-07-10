---
title: Climbing Stairs
categories:
- 好好学习
tags:
  - Algorithm
  - LeetCode
date: 2019-07-09 11:19:46
---

# Climbing Stairs

<!-- more -->



## 问题描述

You are climbing a stair case. It takes n steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

Note: Given n will be a positive integer.

Example 1:

Input: 2
Output: 2
Explanation: There are two ways to climb to the top.
1. 1 step + 1 step
2. 2 steps
Example 2:

Input: 3
Output: 3
Explanation: There are three ways to climb to the top.
1. 1 step + 1 step + 1 step
2. 1 step + 2 steps
3. 2 steps + 1 step

## 思路分析

这是一个Fibonacci问题。

从动态规划的角度来看，可以看作是一个简单的动态规划问题。

针对一个动态规划问题，我们一般分为四步来进行解决：

1. 寻找问题的最优子结构
2. 从上往下递归
3. 边界值
4. 从下往上求解

## 代码实现

递归实现：

```java
public int climbStairs(int n) {
    if(0 == n) {
        return 0;
    } else if(1 == n) {
        return 1;
    } else if(2 == n) {
        return 2;
    }

    return climbStairs(n-1) + climbStairs(n-2);
}
```

递归实现的时间复杂度为O(2^n). 含有大量重复的操作。

自底向上**非递归实现**:

```java
public int climbStairs(int n) {
    if(0 >= n) {
        return 0;
    } else if(1 == n) {
        return 1;
    } else if(2 == n) {
        return 2;
    }

    int a = 1;
    int b = 2;
    for(int i = 3; i <= n; i++) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}
```

非递归实现的时间复杂度为O(n), 空间复杂度为O(1).



具体代码实现请看[<https://github.com/shawn520/algorithms/blob/master/src/leetcode/dynamicProgramming/ClimbingStairs.java> ](<https://github.com/shawn520/algorithms/blob/master/src/leetcode/dynamicProgramming/ClimbingStairs.java> )

## 参考资料

<https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653190796&idx=1&sn=2bf42e5783f3efd03bfb0ecd3cbbc380&chksm=8c990856bbee8140055c3429f59c8f46dc05be20b859f00fe8168efe1e6a954fdc5cfc7246b0&mpshare=1&scene=1&srcid=0709OvMHdeNK7eaMPIHxhGzi&key=95b17fddd06e963c3d29f369a67d55bdd472084d90b6e9ba9a2227393d7558aea7fdb904b5be96c98679ced6ce5256cf1cf5a2d0d390acd4803d7e78b7eaee9131ccaea9f37c5efcc8ae94f2f0558ced&ascene=1&uin=NDUxNjcyMjU3&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=O7C6j9RSY4oPV7QAs9MDZcZ%2F6vxPmw3fkRCQdM1JQFwZANsbL%2FNlcpAdszFVSCEt> 

