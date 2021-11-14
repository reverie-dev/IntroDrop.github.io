---
layout:     post
title:      "编辑距离 入门动态规划 "
subtitle:   "Dynamic programming problem"
date:       2021-09-25 16:50:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- leetcode
- 动态规划
---

## 编辑距离等相关衍生题

编辑距离算法被数据科学家广泛应用，是用作机器翻译和语音识别评价标准的基本算法。

最直观的方法是暴力检查所有可能的编辑方法，取最短的一个。所有可能的编辑方法达到指数级，但我们不需要进行这么多计算，因为我们只需要找到距离最短的序列而不是所有可能的序列。

### 编辑距离

力扣[72题](https://leetcode-cn.com/problems/edit-distance/)

>给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数 。
>
>你可以对一个单词进行如下三种操作：
>
>- 插入一个字符
>- 删除一个字符
>- 替换一个字符

> 输入：word1 = "horse", word2 = "ros"
> 输出：3
> 解释：
> horse -> rorse (将 'h' 替换为 'r')
> rorse -> rose (删除 'r')
> rose -> ros (删除 'e')

我们可以将“对word1插入一个字符”操作，等效为 “对word2删除一个字符”操作。

此外，我们可以将“替换word1中的一个字符为word2中的一个字符”，等效为“对word1删除一个字符和对word2删除一个字符”。

这样，所有的操作就是 “对word1删除一个字符” 以及“对word2删除一个字符”

用一个dp\[i][j] 数组表示 表示 `A` 的前 `i` 个字母和 `B` 的前 `j` 个字母之间的编辑距离。

若 A 和 B 的最后一个字母相同：

>  dp\[i][j] =dp\[i-1][j-1]

若 A 和 B 的最后一个字母不同：

> dp\[i][j] = 1 + min(dp\[i - 1][j],  dp\[i][j - 1], dp\[i - 1][j - 1])

边界条件：当`i`或者`j`为`0`时，即长度为0，则需要删除`j`或者`i`个字符。有

> dp\[0][j] = i
>
> dp\[i][0] = j

code by Java:

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int n = word1.length(), m = word2.length();
        int[][] dp = new int[n + 1][m + 1];
        for(int i = 0; i <= n; i++){
            for(int j = 0; j <= m; j++){
                if(i == 0){
                    dp[i][j] = j; continue;
                }
                if(j == 0){
                    dp[i][j] = i; continue;
                }
                char c1 = word1.charAt(i - 1), c2 = word2.charAt(j - 1);
                if(c1 == c2){
                    dp[i][j] = dp[i - 1][j - 1];
                }else{
                    int min = Math.min(dp[i - 1][j], dp[i][j - 1]) + 1; // 对word1的删除操作，对word2的删除操作
                    dp[i][j] = Math.min(dp[i - 1][j - 1] + 1, min); //替换操作 dp[i-1][j-1]+1
                }
            }
        }
        return dp[n][m];
    }
}
```



### 两个字符串的删除操作

力扣[583](https://leetcode-cn.com/problems/delete-operation-for-two-strings/)题

> 给定两个单词 word1 和 word2，找到使得 word1 和 word2 相同所需的最小步数，每步可以删除任意一个字符串中的一个字符。

> 输入: "sea", "eat"

> 输出: 2

> 解释: 第一步将"sea"变为"ea"，第二步将"eat"变为"ea"

类似地有：

dp\[i][j] 数组代表使得word1[0:i]和word2[0:j]相同的最少操作次数

如果word1[i - 1] == word2[j - 1] 则 dp\[i][j] = dp\[i - 1][j - 1]

否则 dp\[i][j] = min(dp\[i - 1][j], dp\[i][j - 1]) + 1

边界条件：当i或者j为0时，dp\[i][0] = i  dp\[0][j] = j

code by Java

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
```



### 

