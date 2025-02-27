# 力扣热榜100算法


# 力扣热榜100算法

# TOP 100

**链接** : https://leetcode.cn/studyplan/top-100-liked/

## 哈希

### 两数之和

```go
// 001
// 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
//
// 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
//
// 你可以按任意顺序返回答案。
// 输入：nums = [2,7,11,15], target = 9
// 输出：[0,1]
// 解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
func twoSum(nums []int, target int) []int {
	for i := 0; i < len(nums)-1; i++ {
		for j := i + 1; j < len(nums); j++ {
			if nums[i]+nums[j] == target {
				return []int{i, j}
			}
		}
	}
	return []int{}
}

func twoSum2(nums []int, target int) []int {
	targetMap := make(map[int]int)
	for index, num := range nums {
		expect := target - num
		if i, ok := targetMap[expect]; ok {
			return []int{index, i}
		}
		targetMap[num] = index
	}
	return nil
}
```

### 字母异位词

```go
// 049
// 给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。
//
// 字母异位词 是由重新排列源单词的所有字母得到的一个新单词
// 输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
// 输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
func groupAnagrams(strs []string) [][]string {
	strMap := make(map[string][]string)
	for _, str := range strs {

		s := []rune(str)
		sort.Slice(s, func(i, j int) bool {
			return s[i] < s[j]
		})
		sortStr := string(s)
		strMap[sortStr] = append(strMap[sortStr], str)
	}

	ans := make([][]string, 0, len(strMap))
	for _, v := range strMap {
		ans = append(ans, v)
	}
	return ans
}
```

### 最长连续序列

```go
// 128
// 给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。
//
// 请你设计并实现时间复杂度为 O(n) 的算法解决此问题。
// 输入：nums = [100,4,200,1,3,2]
// 输出：4
// 解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
func longestConsecutive(nums []int) int {
	numMap := make(map[int]bool)
	maxLen := 0
	for _, num := range nums {
		numMap[num] = true
	}
	for num, _ := range numMap {
		if _, ok := numMap[num-1]; !ok {
			currentNum := num
			currentStreak := 1
			for numMap[currentNum+1] {
				currentNum++
				currentStreak++
			}
			if maxLen < currentStreak {
				maxLen = currentStreak
			}
		}
	}

	return maxLen
}
```

## 双指针

### 移动零

```go
// 283
// 给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
//
// 请注意 ，必须在不复制数组的情况下原地对数组进行操作。
// 输入: nums = [2,1,0,3,12]
// 输出: [1,3,12,0,0]
func moveZeroes(nums []int) {
	left, right, n := 0, 0, len(nums)
	for right < n {
		if nums[right] != 0 {
			nums[left], nums[right] = nums[right], nums[left]
			left++
		}
		right++
	}
}
```

### 盛最多水的容器

```go
// 011
// 给定一个长度为 n 的整数数组 height 。有 n 条垂线，第 i 条线的两个端点是 (i, 0) 和 (i, height[i]) 。
//
// 找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
//
// 返回容器可以储存的最大水量。
func maxArea(height []int) int {
	ans := 0
	left, right := 0, len(height)-1

	for left < right {
		area := (right - left) * min(height[left], height[right])
		ans = max(area, ans)
		if height[left] < height[right] {
			left++
		} else {
			right--
		}
	}
	return ans
}
```

### 三数之和

```go
// 015
// 给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请你返回所有和为 0 且不重复的三元组。
//
// 注意：答案中不可以包含重复的三元组。
// 输入：nums = [-1,0,1,2,-1,-4]
// 输出：[[-1,-1,2],[-1,0,1]]
func threeSum(nums []int) [][]int {
	ans := make([][]int, 0)
	sort.Ints(nums)

	for i := 0; i < len(nums)-2; i++ {
		if i > 0 && nums[i] == nums[i-1] {
			continue
		}
		j := i + 1
		k := len(nums) - 1

		for j < k {
			s := nums[i] + nums[j] + nums[k]
			if 0 == s {
				ans = append(ans, []int{nums[i], nums[j], nums[k]})
				j++
				for j < k && nums[j] == nums[j-1] {
					j++
				}
				k--
				for j < k && nums[k] == nums[k+1] {
					k--
				}
			} else if s < 0 {
				j++
			} else {
				k--
			}
		}
	}
	return ans
}
```

### 接雨水

```go
// 42
// 给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。
// 输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
// 输出：6
// 解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。
func trap(height []int) int {
	leftList := make([]int, len(height))
	rightList := make([]int, len(height))

	leftList[0] = height[0]
	for i := 1; i < len(height); i++ {
		leftList[i] = max(leftList[i-1], height[i])
	}

	rightList[len(height)-1] = height[len(height)-1]
	for i := len(height) - 2; i >= 0; i-- {
		rightList[i] = max(rightList[i+1], height[i])
	}

	ans := 0
	for i, h := range height {
		ans += min(leftList[i], rightList[i]) - h
	}
	return ans
}
```

## 滑动窗口

### 无重复字符的最长子串

```go
// 003
// 给定一个字符串 s ，请你找出其中不含有重复字符的 最长子串的长度。
// 输入: s = "abcabcbb"
// 输出: 3
// 解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
func lengthOfLongestSubstring(s string) int {
	mp := map[uint8]int{}
	ans := 0
	for left, right := 0, 0; right < len(s); right++ {
		mp[s[right]] += 1
		for mp[s[right]] > 1 {
			mp[s[left]] -= 1
			left++
		}
		ans = max(ans, right-left+1)
	}
	return ans
}
```

### 找到字符串中所有字母异位词

```go

// 438
// 找到字符串中所有字母异位词
// 给定两个字符串 s 和 p，找到 s 中所有 p 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。
// 异位词 指由相同字母重排列形成的字符串（包括相同的字符串）。
// 输入: s = "abab", p = "ab"
// 输出: [0,1,2]
func findAnagrams(s, p string) (ans []int) {
	sLen, pLen := len(s), len(p)
	if sLen < pLen {
		return
	}

	var sCount, pCount [26]int
	for i, ch := range p {
		sCount[s[i]-'a']++
		pCount[ch-'a']++
	}
	if sCount == pCount {
		ans = append(ans, 0)
	}

	for i, ch := range s[:sLen-pLen] {
		sCount[ch-'a']--
		sCount[s[i+pLen]-'a']++
		if sCount == pCount {
			ans = append(ans, i+1)
		}
	}
	return
}
