---
layout:     post
title:      "区间问题及相关问题 解题步骤"
subtitle:   "Interval problem"
date:       2021-09-26 17:22:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- leetcode
- 区间问题

---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=660 height=150 src="//music.163.com/outchain/player?type=2&id=1819840875&auto=1&height=100"></iframe>


我真的好爱这首歌



## [力扣435无重复区间](https://leetcode-cn.com/problems/non-overlapping-intervals/)



## [力扣253 会议室II](https://leetcode-cn.com/problems/meeting-rooms-ii/)

#### 题目描述：

> 给你一个会议时间安排的数组 intervals ，每个会议时间都会包括开始和结束的时间 intervals[i] = [starti, endi] ，为避免会议冲突，同时要考虑充分利用会议室资源，请你计算至少需要多少间会议室，才能满足这些会议安排。
>
> 
>

**示例1：**

```
输入：intervals = [[0,30],[5,10],[15,20]]
输出：2
```

**示例 2：**

```
输入：intervals = [[7,10],[2,4]]
输出：1
```

上下车解法：

这道题可以看成是一个简单的插旗问题：进入一个区间的时候将该点坐标对应的值+1，代表插上一面进入的🚩，离开时将该点坐标值-1，代表插上一面离开的🚩，在同一个点可以同时插进入的或离开的，因为这样并不形成区间重叠。上述思想只需要对数组进行一次遍历并利用map存储坐标和旗子数量，即可完成所有旗子的插入。由于C++里map是按照键的大小升序排列的，因此只需要遍历一次map，从头开始做累加sum，累加过程中最大的sum就是答案。

```cpp
class Solution {
public:
    int minMeetingRooms(vector<vector<int>>& time) {
        map<int, int> mp;
        for(auto& t : time){
            mp[t[0]]++;
            mp[t[1]]--;
        }
        int ans = 0, sum = 0;
        for(auto& t : mp){
            sum += t.second;
            ans = max(ans, sum);
        }
        return ans;
    }
};
```

优先队列解法：

我们可以将所有房间保存在最小堆中,堆中的键值是会议的结束时间，而不用手动迭代已分配的每个房间并检查房间是否可用。

这样，每当我们想要检查有没有 任何 房间是空的，只需要检查最小堆堆顶的元素，它是最先开完会腾出房间的。

如果堆顶的元素的房间并不空闲，那么其他所有房间都不空闲。这样，我们就可以直接开一个新房间。

1. 按照 开始时间 对会议进行排序。

2. 初始化一个新的 最小堆，将第一个会议的结束时间加入到堆中。我们只需要记录会议的结束时间，告诉我们什么时候房间会空。

3. 对每个会议，检查堆的最小元素（即堆顶部的房间）是否空闲。
   1. 若房间空闲，则从堆顶拿出该元素，将其改为我们处理的会议的结束时间，加回到堆中。
   2. 若房间不空闲。开新房间，并加入到堆中。
4. 处理完所有会议后，堆的大小即为开的房间数量。这就是容纳这些会议需要的最小房间数。

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {

    // Check for the base case. If there are no intervals, return 0
    if (intervals.length == 0) {
      return 0;
    }

    // Min heap
    PriorityQueue<Integer> allocator =
        new PriorityQueue<Integer>(
            intervals.length,
            new Comparator<Integer>() {
              public int compare(Integer a, Integer b) {
                return a - b;
              }
            });

    // Sort the intervals by start time
    Arrays.sort(
        intervals,
        new Comparator<int[]>() {
          public int compare(final int[] a, final int[] b) {
            return a[0] - b[0];
          }
        });

    // Add the first meeting
    allocator.add(intervals[0][1]);

    // Iterate over remaining intervals
    for (int i = 1; i < intervals.length; i++) {

      // If the room due to free up the earliest is free, assign that room to this meeting.
      if (intervals[i][0] >= allocator.peek()) {
        allocator.poll();
      }

      // If a new room is to be assigned, then also we add to the heap,
      // If an old room is allocated, then also we have to add to the heap with updated end time.
      allocator.add(intervals[i][1]);
    }

    // The size of the heap tells us the minimum rooms required for all the meetings.
    return allocator.size();
  }
}
```

方法二：有序化
思路

提供给我们的会议时间可以确定一天中所有事件的时间顺序。我们拿到了每个会议的开始和结束时间，这有助于我们定义此顺序。

根据会议的开始时间来安排会议有助于我们了解这些会议的自然顺序。然而，仅仅知道会议的开始时间，还不足以告诉我们会议的持续时间。我们还需要按照结束时间排序会议，因为一个“会议结束”事件告诉我们必然有对应的“会议开始”事件，更重要的是，“会议结束”事件可以告诉我们，一个之前被占用的会议室现在空闲了。

一个会议由其开始和结束时间定义。然而，在本算法中，我们需要 分别 处理开始时间和结束时间。这乍一听可能不太合理，毕竟开始和结束时间都是会议的一部分，如果我们将两个属性分离并分别处理，会议自身的身份就消失了。但是，这样做其实是可取的，因为：

当我们遇到“会议结束”事件时，意味着一些较早开始的会议已经结束。我们并不关心到底是哪个会议结束。我们所需要的只是 一些 会议结束,从而提供一个空房间。

考虑上一方法中使用的案例。 要考虑的会议为：(1, 10), (2, 7), (3, 19), (8, 12), (10, 20), (11, 30) 。像之前那样，第一张图说明前三个会议彼此冲突，需要分别分配房间。![pic-3](https://pic.leetcode-cn.com/c42177d609cf12ab80180219201caade8797770a3101923f3bca57007645cd00-image.png)

后两张图处理剩下的会议，可以看到，我们可以复用一些已有的会议室。最终的结果是相同的，我们一共需要4个会议室，这是最优的结果。

![pic-4](https://pic.leetcode-cn.com/5eb78af9f2763d15ee614c1916484dce017e4f84e35894103d4106892cd13ba3-image.png)

算法

1. 分别将开始时间和结束时间存进两个数组。
2. 分别对开始时间和结束时间进行排序。请注意，这将打乱开始时间和结束时间的原始对应关系。它们将被分别处理。
3. 考虑两个指针：s_ptr 和 e_ptr ，分别代表开始指针和结束指针。开始指针遍历每个会议，结束指针帮助我们跟踪会议是否结束。
4. 当考虑 s_ptr 指向的特定会议时，检查该开始时间是否大于 e_ptr 指向的会议。若如此，则说明 s_ptr 开始时，已经有会议结束。于是我们可以重用房间。否则，我们就需要开新房间。
5. 若有会议结束，换而言之，start[s_ptr] >= end[e_ptr] ，则自增 e_ptr 。
6. 重复这一过程，直到 s_ptr 处理完所有会议。

code by Java

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {

    // Check for the base case. If there are no intervals, return 0
    if (intervals.length == 0) {
      return 0;
    }

    Integer[] start = new Integer[intervals.length];
    Integer[] end = new Integer[intervals.length];

    for (int i = 0; i < intervals.length; i++) {
      start[i] = intervals[i][0];
      end[i] = intervals[i][1];
    }

    // Sort the intervals by end time
    Arrays.sort(
        end,
        new Comparator<Integer>() {
          public int compare(Integer a, Integer b) {
            return a - b;
          }
        });

    // Sort the intervals by start time
    Arrays.sort(
        start,
        new Comparator<Integer>() {
          public int compare(Integer a, Integer b) {
            return a - b;
          }
        });

    // The two pointers in the algorithm: e_ptr and s_ptr.
    int startPointer = 0, endPointer = 0;

    // Variables to keep track of maximum number of rooms used.
    int usedRooms = 0;

    // Iterate over intervals.
    while (startPointer < intervals.length) {

      // If there is a meeting that has ended by the time the meeting at `start_pointer` starts
      if (start[startPointer] >= end[endPointer]) {
        usedRooms -= 1;
        endPointer += 1;
      }

      // We do this irrespective of whether a room frees up or not.
      // If a room got free, then this used_rooms += 1 wouldn't have any effect. used_rooms would
      // remain the same in that case. If no room was free, then this would increase used_rooms
      usedRooms += 1;
      startPointer += 1;

    }

    return usedRooms;
  }
}
```

