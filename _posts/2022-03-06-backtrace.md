---
layout:     post
title:      "经典回溯题总结"
subtitle:   "backtrace template"
date:       2022-03-06 19:00:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- leetcode
- backtrace
---

## Tips
> 凡是含有duplicate的都需要之前sorted，才能保证没有结果中没有重复

### 46 Permutation
```golang
func permute(nums []int) [][]int {
	ans := make([][]int, 0)
	vis := make(map[int]bool)
	var backtrace func(path []int)
	backtrace = func(path []int) {
		if len(path) == len(nums) {
			ans = append(ans, append([]int{}, path...))
			return
		}
		for _, i := range nums {
			if !vis[i] {
				vis[i] = true
				path = append(path, i)
				backtrace(path)
				path = path[:len(path)-1]
				vis[i] = false
			}
		}
	}
	backtrace(make([]int, 0))
	return ans
}
```

### 47 Permutations II
输入可能包含重复数字，所以就要保证去重。先排序然后创建Array记录访问过的数字，然后前面的一个数是否和自己相等，相等的时候则前面的数必须使用了，自己才能使用，这样就不会产生重复的排列了
```golang
func permuteUnique(nums []int) [][]int {
	ans := make([][]int, 0)
	vis := make([]bool, len(nums))
	// 排序方便去重
	sort.Ints(nums)
	var backtrace func(path []int)
	backtrace = func(path []int) {
		if len(path) == len(nums) {
			ans = append(ans, append([]int{}, path...))
		}
		for i := 0; i < len(nums); i++ {
			// 剪枝，如果这个值被vis了，过掉。或者:这个值和上一个值相等，并且上个值未被vis，那么必然会重复。
            // [1,1,2] 当path为空，从第二个1开始的时候，必然重复
			if vis[i] || (i > 0 && nums[i-1] == nums[i] && !vis[i-1]) {
				continue
			}
			vis[i] = true
			path = append(path, nums[i])
			backtrace(path)
			path = path[:len(path)-1]
			vis[i] = false
		}
	}
	backtrace(make([]int, 0))
	return ans
}
```
### 78. Subsets
元素互不相同，则用start坐标来保证唯一性
```golang
func subsets(nums []int) [][]int {
	ans := make([][]int, 0)
	var backtrace func(path []int, start int)
	backtrace = func(path []int, start int) {
		ans = append(ans, append([]int{}, path...))
		for i := start; i < len(nums); i++{
			path = append(path, nums[i])
			backtrace(path, i + 1)
			path = path[:len(path) - 1]
		}
	}
	backtrace(make([]int, 0), 0)
	return ans
}
```
### 90. Subsets II
需要排序，加去重
```golang
func subsetsWithDup(nums []int) [][]int {
	ans := make([][]int, 0)
	sort.Ints(nums)
	var backtrace func(path []int, start int)
	backtrace = func(path []int, start int) {
		ans = append(ans, append([]int{}, path...))
		for i := start; i < len(nums); i++ {
			// 在同一个循环里，过滤掉重复的
			if i > start && nums[i] == nums[i-1]{
				continue
			}
			path = append(path, nums[i])
			backtrace(path, i+1)
			path = path[:len(path)-1]
		}
	}
    backtrace(make([]int, 0), 0)
	return ans
}
```

### 39 Combination Sum
可以利用重复数字，index等于自己
```golang
func combinationSum(candidates []int, target int) [][]int {
	ans := make([][]int, 0)
	var backtrace func(path []int, start, tar int)
	backtrace = func(path []int, start, tar int) {
		if tar < 0 {
			return
		} else if tar == 0 {
			ans = append(ans, append([]int{}, path...))
		} else {
			for i := start; i < len(candidates); i++ {
				path = append(path, candidates[i])
                // 不断的选自己
				backtrace(path, i, tar-candidates[i])
                // 到下一轮循环，就会选下个元素
				path = path[:len(path)-1]
			}
		}
	}
	backtrace(make([]int, 0), 0, target)
	return ans
}
```

### 40 Combination Sum II
不可以重复利用数字，index+1
```golang
func combinationSum2(candidates []int, target int) [][]int {
	ans := make([][]int, 0)
	// 需要去重，排序帮助去重
	sort.Ints(candidates)
	var backtrace func(path []int, start, tar int)
	backtrace = func(path []int, start, tar int) {
		if tar < 0 {
			return
		} else if tar == 0 {
			ans = append(ans, append([]int{}, path...))
		} else {
			for i := start; i < len(candidates); i++ {
                // 去重
				if i > start && candidates[i] == candidates[i-1] {
					continue
				}
				path = append(path, candidates[i])
                // 不能利用重复数字
				backtrace(path, i + 1, tar-candidates[i])
				path = path[:len(path)-1]
			}
		}
	}
	backtrace(make([]int, 0), 0, target)
	return ans
}
```

### 216 Combination Sum III
区别在于输入为1-9
```golang
func combinationSum3(k int, n int) [][]int {
	ans := make([][]int, 0)
	var backtrace func(path []int, start, tar, step int)
	backtrace = func(path []int, start, tar, step int) {
		if tar < 0 {
			return
		} else if tar == 0 && step == k {
			ans = append(ans, append([]int{}, path...))
		} else {
			for i := start; i <= 9; i++ {
                if i > tar{
                    break
                }
				path = append(path, i)
				backtrace(path, i+1, tar-i, step+1)
				path = path[:len(path)-1]
			}
		}
	}
	backtrace(make([]int, 0), 1, n, 0)
	return ans
}
```

### 22 Generate Parentheses
注意 left < right
```golang
func generateParenthesis(n int) []string {
	ans := make([]string, 0)
	var backtrace func(path []rune, left, right int)
	backtrace = func(path []rune, left, right int) {
		if left == right && left == n {
			ans = append(ans, string(path))
		}
		if left < n {
			path = append(path, '(')
			backtrace(path, left+1, right)
			path = path[:len(path)-1]
		}
		if right < left {
			path = append(path, ')')
			backtrace(path, left, right+1)
			path = path[:len(path)-1]
		}
	}
	backtrace(make([]rune, 0), 0, 0)
	return ans
}
```