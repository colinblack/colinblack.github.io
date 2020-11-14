---
title: leetcode题解
date: 2020-08-27 07:33:46
tags: 面试
mathjax: true
---

### 动态规划 
#### 5.最长回文子串 

状态转移方程:
子串长度大于２:
$$
P(i,j) = P(i+1, j-1) ∧ (Si == Sj) 
$$
否则:
$$
\begin{cases} P(i,i) = true \\ P(i, i+1) = (Si == Si+1) \end{cases}
$$


遍历方式:

1. 斜着遍历
2. 从上至下遍历



#### 91.解码方法

状态转移方程:
$$
f(i)=\begin{cases} 1, \quad i = 0\\ f(i-1)+f(i-2)\{i-1=1 or i-1=2  and i < 7\},\quad i>0 \end{cases}
$$


