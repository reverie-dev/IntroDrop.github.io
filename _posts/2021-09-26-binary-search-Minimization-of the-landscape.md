---
layout:     post
title:      "二分查找之「最大值极小化」相关问题及解题步骤"
subtitle:   "Minimization of the landscape problem"
date:       2021-09-26 11:12:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- leetcode
- 二分查找

---

## 二分查找之「最大值极小化」相关问题及解题步骤

今天为大家总结一下「力扣」第 202 场周赛的第 3 题「两球之间的磁力」（题号：1552）。这道问题在周赛、春季团体赛中都出现过若干次，不过还是有不少朋友们觉得很难理解。我第一次接触到这个问题的时候也觉得很绕，因此今天花一点时间和大家做一个总结。

这一类问题的原形是「木棍分割」问题，大家可以在互联网上搜索一下。我们基于「力扣」第 410 题「分割数组的最大值」开始说起，其实是一样的。



## 例题：「力扣」第 410 题：「分割数组的最大值」

> 给定一个非负整数数组和一个整数 `m`，你需要将这个数组分成 `m` 个非空的连续子数组。设计一个算法使得这 `m` 个子数组各自和的最大值最小。

### 题意分析

各自和的最大值最小，这句话读起来有一点点绕。我们拆分一下：

- 由于数组是确定的，其中一组分得多，相应地另一组分到的值就少。所以对于任意一种拆分（切成 `m` 段），这 `m` 段可以取最大值 `val`；
- 我们需要找到一种拆分，使得这个最大值 `val` 的值是所有分成 `m` 段拆分里值最小的那个；**具体怎么拆，题目不用我们求，只需要我们计算出 `val` 的值**。

### 关键字分析

这个题目非常关键的字眼是：**非负整数数组**、**非空** 和 **连续**。尤其是「**非负整数数组**」和「**连续**」这两个信息，请读者看完下面分析以后再来体会「连续」为什么那么重要。

### 解题思路的直觉来源

基于「力扣」第 69 题、第 287 题，知道二分查找的应用：可以用于查找一个有范围的 **整数**，就能想到是不是可以使用二分查找去解决这道问题。

事实上，二分查找最典型的应用我们都见过，《幸运 52》猜价格游戏，主持人说「低了」，我们就应该往高了猜。

这种二分查找的应用大家普遍的叫法是「二分答案」，即「对答案二分」。它是相对于二分查找的最原始形式「在一个有序数组里查找一个数」而言的。

### 挖掘单调性

使用二分查找的一个前提是「数组具有单调性」，我们就去想想第 410 题有没有类似的单调性。「最大值最小」就提示了单调性：

- 如果设置「数组各自和的最大值」很大，那么必然导致分割数很小；
- 如果设置「数组各自和的最大值」很小，那么必然导致分割数很大。

仔细想想，这里「**数组各自和的最大值**」就决定了一种分割的方法。再联系一下我们刚刚向大家强调的题目的要求 **连续** 和题目中给出的输入数组的特点: **非负整数数组**。

那么，我们就可以通过调整「数组各自和的最大值」来达到：使得分割数恰好为 `m` 的效果。这里要注意一个问题：

### 注意事项

如果某个 **数组各自和的最大值** 恰恰好使得分割数为 `m` ，此时不能放弃搜索，因为我们要使得这个最大值 **最小化**，此时还应该继续尝试缩小这个 **数组各自和的最大值** ，使得分割数超过 `m` ，超过 `m` 的最后一个使得分割数为 `m` 的 **数组各自和的最大值**  就是我们要找的 **最小值**。

这里想不太明白的话，可以举一个具体的例子：

例如：（题目中给出的示例）输入数组为 `[7, 2, 5, 10, 8]` ，`m = 2` 。如果设置 **数组各自和的最大值** 为 `21`，那么分割是 `[7, 2, 5, | 10, 8]`，此时 `m = 2`，此时，这个值太大，尝试一点一点缩小：

- 设置 **数组各自和的最大值** 为 `20`，此时分割依然是 `[7, 2, 5, | 10, 8]`，`m = 2`；
- 设置 **数组各自和的最大值** 为 `19`，此时分割依然是 `[7, 2, 5, | 10, 8]`，`m = 2`；
- 设置 **数组各自和的最大值** 为 `18`，此时分割依然是 `[7, 2, 5, | 10, 8]`，`m = 2`；
- 设置 **数组各自和的最大值** 为 `17`，此时分割就变成了 `[7, 2, 5, | 10, | 8]`，这时 `m = 3`。

`m` 变成 `3` 之前的值 **数组各自和的最大值** `18` 是这个问题的最小值，所以输出 `18`。

### 代码实现

说明：

- 以下代码实现中采用 `while (left < right)` 的写法表示退出循环以后 `left == right` 成立，这种通过「左右边界」向中间逼近，最后得到一个数的二分查找思考路径，我在「力扣」的题解区已经多次向大家介绍，并且强调了使用细节和注意事项，这里就不赘述了；
- `if (splits > m)` 的反面是 `splits <= m` 此时，下一轮搜索区间是 `[left, mid]` ，这个时候我们没有排除掉 `mid` 这个值，符合我们上面的逻辑：当分割数恰好等于 `m` 的时候，尝试缩小 **数组各自和的最大值**。

```java
public class Solution {

    public int splitArray(int[] nums, int m) {
        int max = 0;
        int sum = 0;

        // 计算「子数组各自的和的最大值」的上下界
        for (int num : nums) {
            max = Math.max(max, num);
            sum += num;
        }

        // 使用「二分查找」确定一个恰当的「子数组各自的和的最大值」，
        // 使得它对应的「子数组的分割数」恰好等于 m
        int left = max;
        int right = sum;
        while (left < right) {
            int mid = left + (right - left) / 2;

            int splits = split(nums, mid);
            if (splits > m) {
                // 如果分割数太多，说明「子数组各自的和的最大值」太小，此时需要将「子数组各自的和的最大值」调大
                // 下一轮搜索的区间是 [mid + 1, right]
                left = mid + 1;
            } else {
                // 下一轮搜索的区间是上一轮的反面区间 [left, mid]
                right = mid;
            }
        }
        return left;
    }

    /***
     *
     * @param nums 原始数组
     * @param maxIntervalSum 子数组各自的和的最大值
     * @return 满足不超过「子数组各自的和的最大值」的分割数
     */
    private int split(int[] nums, int maxIntervalSum) {
        // 至少是一个分割
        int splits = 1;
        // 当前区间的和
        int curIntervalSum = 0;
        for (int num : nums) {
            // 尝试加上当前遍历的这个数，如果加上去超过了「子数组各自的和的最大值」，就不加这个数，另起炉灶
            if (curIntervalSum + num > maxIntervalSum) {
                curIntervalSum = 0;
                splits++;
            }
            curIntervalSum += num;
        }
        return splits;
    }

    public static void main(String[] args) {
        int[] nums = new int[]{7, 2, 5, 10, 8};
        int m = 2;
        Solution solution = new Solution();
        int res = solution.splitArray(nums, m);
        System.out.println(res);
    }
}
```

**复杂度分析**：

- 时间复杂度：O(Nlog⁡∑nums)，这里 N 表示输入数组的长度，∑nums表示输入数组的和，代码在 \[max⁡(nums),∑nums] 区间里使用二分查找找到目标元素，而每一次判断分支需要遍历一遍数组，时间复杂度为 O(N)；
- 空间复杂度：O(1) ，只使用到常数个临时变量。

## 总结

- 这道题让我们「查找一个有范围的整数」，以后遇到类似问题，要想到可以尝试使用「二分」；
- 遇到类似使「最大值」最小化，这样的题目描述，可以好好跟自己做过的这些问题进行比较，看看能不能找到关联；
- 在代码层面上，这些问题的特点都是：**在二分查找的判别函数里，需要遍历数组一次**。



## 练习

这些问题都如出一辙，请大家特别留意题目中出现的关键字「非负整数」、分割「连续」，**思考清楚设计算法的关键步骤和原因**，以后遇到类似的问题就能轻松应对。

- 「力扣」第 875 题：爱吃香蕉的珂珂（中等）
-  「力扣」LCP 12. 小张刷题计划（中等）
- 「力扣」第 1482 题：制作 m 束花所需的最少天数（中等）
- 「力扣」第 1011 题：在 D 天内送达包裹的能力（中等）
- 「力扣」第 1552 题：两球之间的磁力（中等）

再次强调一下：

- 搜索一个有范围的整数，可以用二分；
- 分析题目中的单调性，通常这种单调性表现为两个变量此增彼减，即 **负相关**；
- 题目中 **连续** 和 **正整数** 这两个前提缺一不可，非常非常重要，希望大家能够抓住题目中这两个关键信息，进而理解这种做法的有效性。

再次强调：**写对二分查找不能靠模板，需要理解加练习**。

### 例 1：「力扣」第 875 题：[爱吃香蕉的珂珂](https://leetcode-cn.com/problems/koko-eating-bananas/)

**分析题意**：哪一堆香蕉先吃是无关紧要的，每一堆香蕉的根数是正数，符合「连续」、「正整数」条件。

**一句话题解**：如果目前尝试的速度恰好使得珂珂在规定的时间内吃完香蕉的时候，还应该去尝试更小的速度是不是还可以保证在规定的时间内吃完香蕉。

**参考代码**：

```java
public class Solution {

    // 速度越小，耗时越大，搜索的是速度
    // 会选择一堆香蕉，只能选择一堆，因此需要向上取整

    public int minEatingSpeed(int[] piles, int H) {
        int maxVal = 1;
        for (int pile : piles) {
            maxVal = Math.max(maxVal, pile);
        }

        // 速度最小的时候，耗时最长
        int left = 1;
        // 速度最大的时候，耗时最短
        int right = maxVal;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (calculateSum(piles, mid) > H) {
                // 耗时太多，说明速度太慢了，下一轮搜索区间在 [mid + 1, right]
                left = mid + 1;
            } else {
                // 下一轮搜索区间在 [left, mid]
                right = mid;
            }
        }
        return left;
    }

    /**
     * 如果返回的小时数严格大于 H，就不符合题意
     *
     * @param piles
     * @param speed
     * @return 需要的小时数
     */
    private int calculateSum(int[] piles, int speed) {
        int sum = 0;
        for (int pile : piles) {
            // 上取整可以这样写
            sum += (pile + speed - 1) / speed;
        }
        return sum;
    }
}
```

**复杂度分析**（后面的问题的复杂度分析一模一样，故全部略去）：

- 时间复杂度：
  $$
  O(Nlog⁡max⁡(piles))
  $$
  ，这里 N 表示数组 `piles` 的长度。我们在
  $$
  [1,max⁡piles]
  $$
  里使用二分查找定位最小速度，而每一次执行判别函数的时间复杂度是 O(N)；

- 空间复杂度：O(1)，算法只使用了常数个临时变量。

### 例 2 ：「力扣」第 1011 题：[在 D 天内送达包裹的能力](https://leetcode-cn.com/problems/capacity-to-ship-packages-within-d-days/)

**分析题意**：我们装载的重量不会超过船的最大运载重量（找一个有范围的整数），返回能在 D 天内将传送带上的所有包裹送达的船的最低运载能力（最大值最小化）。

题目中 **按给出重量的顺序** 这个条件非常重要，保证连续性。运输重量是正数。

- 重点 1：货物必须按照给定的顺序装运；
- 重点 2：最低运载能力：表示超过了就要另外起一艘航船。

运载能力最低是数组 `weights` 中的最大值，至少得拉走一个。最高是数组 `weights` 中的和。

**一句话题解**：单调性是：运载能力越低，需要的天数越多；运载能力越高，需要的天数越少。如果目前尝试的 **最低运载能力** 恰好使得传送带上的包裹在 D 天（注意：恰好是 D 天）从一个港口运送到另一个港口，还应该继续尝试减少 **最低运载能力** 。

**参考代码**：

```java
public class Solution {

    public int shipWithinDays(int[] weights, int D) {
        int maxVal = 0;
        int sum = 0;

        for (int weight : weights) {
            maxVal = Math.max(maxVal, weight);
            sum += weight;
        }

        int left = maxVal;
        int right = sum;
        while (left < right) {
            // 运载能力
            int mid = left + (right - left ) / 2;
            // 运载能力越大，天数越少
            // 运载能力越小，天数越多
            // 负相关
            if (calculateDays(weights, mid) > D) {
                // 太多，下一轮搜索区间 [mid + 1, right]
                left = mid + 1;
            } else {
                // 下一轮搜索区间 [left, mid]
                right = mid;
            }
        }
        return left;
    }

    /**
     * 返回多少天
     *
     * @param weights
     * @param capacity
     * @return
     */
    private int calculateDays(int[] weights, int capacity) {
        int days = 1;
        int currentWeightSum = 0;
        for (int weight : weights) {
            if (currentWeightSum + weight > capacity) {
                currentWeightSum = 0;
                days++;
            }
            currentWeightSum += weight;
        }
        return days;
    }
}
```

### 例 3：「力扣」春季团体赛第 3 题：[LCP 12. 小张刷题计划](https://leetcode-cn.com/problems/xiao-zhang-shua-ti-ji-hua/)

**题意分析**：题目中强调了 **必须按照顺序做完题目**（连续），并且做题数量这件事情肯定是正数。

**一句话题解**：每天耗时最多的问题留给小杨去做。

**参考代码**：

```java
public class Solution {

    public int minTime(int[] time, int m) {
        int sum = 0;
        for (int num : time) {
            sum += num;
        }

        int left = 0;
        int right = sum;

        while (left < right) {
            int mid = (left + right) >>> 1;
            // mid 是 T 值的意思
            int splits = split(time, mid);
            if (splits > m) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return left;
    }

    private int split(int[] nums, long maxSum) {
        int splits = 1;

        int len = nums.length;
        int curMax = nums[0];
        int tempSum = nums[0];

        for (int i = 1; i < len; i++) {
            curMax = Math.max(curMax, nums[i]);
            if (tempSum + nums[i] - curMax > maxSum) {
                tempSum = 0;
                curMax = nums[i];
                splits++;
            }
            tempSum += nums[i];
        }
        return splits;
    }
}
```

### 例 4：「力扣」第 1482 题：[制作 m 束花所需的最少天数](https://leetcode-cn.com/problems/minimum-number-of-days-to-make-m-bouquets/)

**题意分析**：依然是题目用着重号强调了「需要使用花园中 **相邻的 `k` 朵花**」（连续），并且花朵数量这件事情肯定是正数。

**参考代码**：

```java
public class Solution {

    public int minTime(int[] time, int m) {
        int sum = 0;
        for (int num : time) {
            sum += num;
        }

        int left = 0;
        int right = sum;

        while (left < right) {
            int mid = (left + right) >>> 1;
            // mid 是 T 值的意思
            int splits = split(time, mid);
            if (splits > m) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return left;
    }

    private int split(int[] nums, long maxSum) {
        int splits = 1;

        int len = nums.length;
        int curMax = nums[0];
        int tempSum = nums[0];

        for (int i = 1; i < len; i++) {
            curMax = Math.max(curMax, nums[i]);
            if (tempSum + nums[i] - curMax > maxSum) {
                tempSum = 0;
                curMax = nums[i];
                splits++;
            }
            tempSum += nums[i];
        }
        return splits;
    }
}
```

### 例 5：「力扣」第 1552 题：[1552. 两球之间的磁力](https://leetcode-cn.com/problems/magnetic-force-between-two-balls/)

**题意分析**：距离这件事情天然具有连续性，并且距离肯定是正数。并且题目都告诉我们要我们求「最大化的最小磁力」，很显然往「最大值极小化」这一类问题上靠。

**参考代码**：

```java
import java.util.Arrays;

public class Solution {

    public int maxDistance(int[] position, int m) {
        Arrays.sort(position);
        int len = position.length;
        int left = 1;
        int right = position[len - 1] - position[0];

        while (left < right) {
            int mid = left + (right - left + 1) / 2;
            if (countSplits(position, mid) >= m) {
                left = mid;
            } else {
                right = mid - 1;
            }
        }
        return left;
    }

    private int countSplits(int[] position, int distance) {
        int len = position.length;
        int pre = position[0];

        int M = 1;
        for (int i = 1; i < len; i++) {
            if (position[i] - pre >= distance) {
                M++;
                pre = position[i];
            }
        }
        return M;
    }
}
```

