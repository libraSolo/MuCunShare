# 力扣经典面试


# 力扣经典面试

# 数组和字符串

## 合并两个有序数组

```go
/*
给你两个按 非递减顺序 排列的整数数组 nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。

请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。

注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。

 

示例 1：

输入：nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
输出：[1,2,2,3,5,6]
解释：需要合并 [1,2,3] 和 [2,5,6] 。
合并结果是 [1,2,2,3,5,6] ，其中斜体加粗标注的为 nums1 中的元素。
示例 2：

输入：nums1 = [1], m = 1, nums2 = [], n = 0
输出：[1]
解释：需要合并 [1] 和 [] 。
合并结果是 [1] 。
示例 3：

输入：nums1 = [0], m = 0, nums2 = [1], n = 1
输出：[1]
解释：需要合并的数组是 [] 和 [1] 。
合并结果是 [1] 。
注意，因为 m = 0 ，所以 nums1 中没有元素。nums1 中仅存的 0 仅仅是为了确保合并结果可以顺利存放到 nums1 中。
*/

// 我的解题
func merge(nums1 []int, m int, nums2 []int, n int) {
	if 0 == len(nums1) {
		copy(nums1, nums2)
		return
	}

	if 0 == len(nums2) {
		return
	}

	nums := make([]int, 0, m + n)
	var p1, p2 int
	for {
		if p1 == m {
			nums = append(nums,nums2[p2:]...)
			break
		}
		if p2 == n {
			nums = append(nums,nums1[p1:]...)
			break
		}

		if nums1[p1] <= nums2[p2] {
			nums = append(nums, nums1[p1])
			p1++
		} else {
			nums = append(nums, nums2[p2])
			p2++
		}
	}
	copy(nums1, nums)
}
```

## 移除元素

```go
/*
给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

 

说明:

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

// nums 是以“引用”方式传递的。也就是说，不对实参作任何拷贝
int len = removeElement(nums, val);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
 

示例 1：

输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2]
解释：函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。你不需要考虑数组中超出新长度后面的元素。例如，函数返回的新长度为 2 ，而 nums = [2,2,3,3] 或 nums = [2,2,0,0]，也会被视作正确答案。
示例 2：

输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,3,0,4]
解释：函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。注意这五个元素可为任意顺序。你不需要考虑数组中超出新长度后面的元素。
*/

// 我的解题
// 思路： 倒序移除
func removeElement(nums []int, val int) int {
		for i := len(nums) - 1; i >= 0; i-- {
			if nums[i] == val {
				nums = append(nums[:i-1], nums[i-1:]...)
			}
		}

		return len(nums)
}

// 官网解题
/*
如果左指针 left 指向的元素等于 val，此时将右指针 right 指向的元素复制到左指针 lett 的位置,然后右指针 right 左移一位。如果赋值过来的元素恰好也等于 al，可以继续把右指针 right 指向的元素的值赋值过来(左指针 left 指向的等于 val 的元素的位置继续被覆盖)，直到左指针指向的元素的值不等于 val 为止。
当左指针 let 和右指针 rght 重合的时候，左右指针遍历完数组中所有的元素。
这样的方法两个指针在最坏的情况下合起来只遍历了数组一次。与方法一不同的是，方法二避免了需要保留的元素的重复赋值操作。
*/
func removeElement(nums []int, val int) int {
	left, right := 0, len(nums)
	for left < right {
		if nums[left] == val {
			nums[left] = nums[right - 1]
			right--
		} else {
			left++
		}
	}
	return left
}


```

## 删除有序数组中的重复项

```go
/*
给你一个 非严格递增排列 的数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致 。然后返回 nums 中唯一元素的个数。

考虑 nums 的唯一元素的数量为 k ，你需要做以下事情确保你的题解可以被通过：

更改数组 nums ，使 nums 的前 k 个元素包含唯一元素，并按照它们最初在 nums 中出现的顺序排列。nums 的其余元素与 nums 的大小不重要。
返回 k 。

示例 1：

输入：nums = [1,1,2]
输出：2, nums = [1,2,_]
解释：函数应该返回新的长度 2 ，并且原数组 nums 的前两个元素被修改为 1, 2 。不需要考虑数组中超出新长度后面的元素。
示例 2：

输入：nums = [0,0,1,1,1,2,2,3,3,4]
输出：5, nums = [0,1,2,3,4]
解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。

*/

// 我的解题
// 两个指针，紧邻的数字相同时，right++，直到right大于left时，right 和 left+1 交换位置。
// 当 right 大于数组长度时，left+1 为前N项都为唯一元素。
func removeDuplicates(nums []int) int {
	left, right := 0, 1
	for right <= len(nums) - 1 {
		if nums[left] < nums[right] {
			if left+1 != right {
				nums[left+1] = nums[right]
			}
			left++
		}
		right++
	}
	return left + 1
}

// 官方解题

func removeDuplicates(nums []int) int {
    n := len(nums)
    if n == 0 {
        return 0
    }
    slow := 1
    for fast := 1; fast < n; fast++ {
        if nums[fast] != nums[fast-1] {
            nums[slow] = nums[fast]
            slow++
        }
    }
    return slow
}
```

## 删除有序数组中的重复项2

```go
/*
给你一个有序数组 nums ，请你 原地 删除重复出现的元素，使得出现次数超过两次的元素只出现两次 ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

 

说明：

为什么返回数值是整数，但输出的答案是数组呢？

请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

// nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
 

示例 1：

输入：nums = [1,1,1,2,2,3]
输出：5, nums = [1,1,2,2,3]
解释：函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3。 不需要考虑数组中超出新长度后面的元素。
示例 2：

输入：nums = [0,0,1,1,1,1,2,3,3]
输出：7, nums = [0,0,1,1,2,3,3]
解释：函数应返回新长度 length = 7, 并且原数组的前七个元素被修改为 0, 0, 1, 1, 2, 3, 3。不需要考虑数组中超出新长度后面的元素。
*/

// 我的解题
// 类似上面解题，双指针但是我写的bug了

// 官方解题
func removeDuplicates(nums []int) int {
    n := len(nums)
    if n <= 2 {
        return n
    }
    slow, fast := 2, 2
    for fast < n {
        if nums[slow-2] != nums[fast] {
            nums[slow] = nums[fast]
            slow++
        }
        fast++
    }
    return slow
}
```

## 多数元素

```go
/*
给定一个大小为 n 的数组 nums ，返回其中的多数元素。多数元素是指在数组中出现次数 大于 ⌊ n/2 ⌋ 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

 

示例 1：

输入：nums = [3,2,3]
输出：3
示例 2：

输入：nums = [2,2,1,1,1,2,2]
输出：2
*/

// 我的解题
// 遍历获取一个 dir, 统计出现次数，获取出现频率最高的
func majorityElement(nums []int) int {
	counts := make(map[int]int)

	// 统计每个元素的出现次数
	for _, num := range nums {
		counts[num]++
	}

	// 找到出现次数最多的元素
	maxCount := 0
	mostFrequent := 0
	for num, count := range counts {
		if count > maxCount {
			maxCount = count
			mostFrequent = num
		}
	}

	return mostFrequent
}

// 官方解题
// 摩尔投票法（Boyer–Moore majority vote algorithm），也被称作「多数投票法」
// 题目为大于一半的票，即"高个子"打一群，一换一的最后还是留下高个子。一群当中不管是不是同样的人，都可以是一换一的元素。因为他们的票至少比高个子少一个。
func majorityElement(nums []int) int {
    major := 0
    count := 0

    for _, num := range nums {
        if count == 0 {
            major = num
        }
        if major == num {
            count += 1
        } else {
            count -= 1
        }
    }
  
    return major
}

```

## 旋转数组

```go
/*
给定一个整数数组 nums，将数组中的元素向右轮转 k 个位置，其中 k 是非负数。

 

示例 1:

输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右轮转 1 步: [7,1,2,3,4,5,6]
向右轮转 2 步: [6,7,1,2,3,4,5]
向右轮转 3 步: [5,6,7,1,2,3,4]
示例 2:

输入：nums = [-1,-100,3,99], k = 2
输出：[3,99,-1,-100]
解释: 
向右轮转 1 步: [99,-1,-100,3]
向右轮转 2 步: [3,99,-1,-100]
*/

// 我的解题
// 看k值来判断nums的循环次数
func rotate(nums []int, k int) {
    newNums := make([]int, len(nums))
    for i, v := range nums {
        newNums[(i+k)%len(nums)] = v
    }
    copy(nums, newNums)
}

// 官方解题
// 翻转数组
/*
nums = "----->-->"; k =3
result = "-->----->";

reverse "----->-->" we can get "<--<-----"
reverse "<--" we can get "--><-----"
reverse "<-----" we can get "-->----->"
*/
func reverse(a []int) {
    for i, n := 0, len(a); i < n/2; i++ {
        a[i], a[n-1-i] = a[n-1-i], a[i]
    }
}

func rotate(nums []int, k int) {
    k %= len(nums)
    reverse(nums)
    reverse(nums[:k])
    reverse(nums[k:])
}

```

## 买卖股票的最佳时机

```go
/*
给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

 

示例 1：

输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
示例 2：

输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。
*/

// 我的解题
// 双重循环获取最大利润，但是会超时
// 向后看方法
func maxProfit(prices []int) int {
	maxPrice := 0
	for i, price := range prices {
		for j := i + 1; j < len(prices); j++ {
			if prices[j] > price && prices[j] - price > maxPrice {
			 	maxPrice = prices[j] - price
			}
		}
	}
	return maxPrice
}

// 官方解题
// 向前看，默认自己就是最大的卖出时价格，找到前N项的最低购买价格
func maxProfit(prices []int) int {
	maxProfit := 0
	minPrice := prices[0]
	for _,price := range prices {
		if price < minPrice {
			minPrice = price
		}
		if price - minPrice > maxProfit {
			maxProfit = price - minPrice
		}
	}

	return maxProfit
}
```

## 买卖股票的最佳时机2

```go
/*
给你一个整数数组 prices ，其中 prices[i] 表示某支股票第 i 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 最多 只能持有 一股 股票。你也可以先购买，然后在 同一天 出售。

返回 你能获得的 最大 利润 。

 

示例 1：

输入：prices = [7,1,5,3,6,4]
输出：7
解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3 。
     总利润为 4 + 3 = 7 。
示例 2：

输入：prices = [1,2,3,4,5]
输出：4
解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     总利润为 4 。
示例 3：

输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 交易无法获得正利润，所以不参与交易可以获得最大利润，最大利润为 0 。
*/

// 我的解题
// 能卖则卖， 当前价格比前一天大，就获得利润
func maxProfit(prices []int) int {
	maxProfit := 0
	for i,price := range prices[1:] {
		if price > prices[i] {
			maxProfit += price - prices[i]
		}
	}
	return maxProfit
}

// 动态规划

```

## 跳跃游戏

```go
/*
给你一个非负整数数组 nums ，你最初位于数组的 第一个下标 。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 true ；否则，返回 false 。

 

示例 1：

输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
示例 2：

输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。
*/

// 官方解题
// 贪心 把跳跃想成覆盖，只要覆盖到最后一个位置即可到达最后
func canJump(nums []int) bool {
	if 1 == len(nums) {
		return true
	}
	cover := 0
	for i := 0; i <= cover; i++ {
		cover = max(i + nums[i], cover)
		if cover >= len(nums) - 1 {
			return true
		}
	}
	return false
}
```

##
