---
layout:     post
title:      "编辑距离 两个字符串的删除操作 等衍生动态规划题"
subtitle:   "Dynamic programming problem"
date:       2021-09-25 16:50:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- leetcode
---

首先看下力扣583题：两个字符串的删除操作

> 给定两个单词 word1 和 word2，找到使得 word1 和 word2 相同所需的最小步数，每步可以删除任意一个字符串中的一个字符。

> 输入: "sea", "eat"

> 输出: 2

解释: 第一步将"sea"变为"ea"，第二步将"eat"变为"ea"

动态规划解法

 dp[i][j] 数组代表使得word1[0:i]和word2[0:j]相同的最少操作次数

如果word1[i - 1] == word2[j - 1] 则 dp[i][j] = dp[i - 1][j - 1]

否则 dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + 1

边界条件：当i或者j为0时，dp[i][0] = i dp[0][j] = j

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int n = word1.length(), m = word2.length();
        int[][] dp = new int[n + 1][m + 1];
        for(int i = 0; i <= n; i++){
            dp[i][0] = i;
        }
        for(int j = 0; j <= m; j++){
            dp[0][j] = j;
        }
        for(int i = 1; i <= n; i++){
            for(int j = 1; j <= m; j++){
                char c1 = word1.charAt(i - 1), c2 = word2.charAt(j - 1);
                if(c1 == c2){
                    dp[i][j] = dp[i - 1][j - 1];
                }else{
                    dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + 1;
                }
            }
        }
        return dp[n][m];
    }
}
