---
title: leetcode-go-03
date: 2025-03-21 22:21:33
tags:
---

# leetcode热题100-GO解-2025.3.21

12/100

~~今日主要在调整butterfly主题配色等，简单堆堆数量了今天~~

堆数量失败，两道困难题



### 560.和为K的子数组

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

子数组是数组中元素的连续非空序列。



**示例 1：**

```
输入：nums = [1,1,1], k = 2
输出：2
```

**示例 2：**

```
输入：nums = [1,2,3], k = 3
输出：2
```



**提示：**

- `1 <= nums.length <= 2 * 104`
- `-1000 <= nums[i] <= 1000`
- `-107 <= k <= 107`



采用前缀和加哈希表标准解法

```go
func subarraySum(nums []int, k int) int {
	m := make(map[int]int)
	//map m: key为前缀和，value为前缀和出现的次数
	//初始化，当和为0时，出现了1次
	m[0] = 1
	count := 0
	//count：和为k的子数组的个数
	pre := 0
	//pre：当前前缀和
	for i := 0; i < len(nums); i++ {
		pre += nums[i]
		//从头遍历数组，逐一计算前缀和
		if _, ok := m[pre-k]; ok {
			//若前缀和为pre-k的前缀和存在
			//即存在若干个(m(pre-k)个)前缀和为pre-k的子项
			//即sum(nums[:i+1])-sum(nums[:j])=k
			//即sum(nums[j+1:i+1])=k
			//即存在和为k的子数组
			//故count+=m[pre-k]，即可计入的子数组个数
			count += m[pre-k]
		}
		m[pre] += 1
		//将当前前缀和加入map
	}
	return count
}
```



### 239.滑动窗口最大值

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。



**示例 1：**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值			和
---------------               -----
[1  3  -1] -3  5  3  6  7       3			3
 1 [3  -1  -3] 5  3  6  7       3			-1
 1  3 [-1  -3  5] 3  6  7       5			1
 1  3  -1 [-3  5  3] 6  7       5			5
 1  3  -1  -3 [5  3  6] 7       6			14
 1  3  -1  -3  5 [3  6  7]      7			16
 
 [7  2] 4						7			9
  7 [2  4]						6			6
```

**示例 2：**

```
输入：nums = [1], k = 1
输出：[1]
```

 

**提示：**

- `1 <= nums.length <= 105`
- `-104 <= nums[i] <= 104`
- `1 <= k <= nums.length`



预计采用邪道解法

```go
// 能解出来，但是超时
// 时间复杂度：O(nk)，空间复杂度：O(n-k+1)
func maxSlidingWindow(nums []int, k int) []int {
	if k == 1 {
		return nums
	} else if k == len(nums) {
		return []int{Max(nums)}
	}
	res := make([]int, 0)
	l := len(nums)
	currentMax := Max(nums[:k])
	res = append(res, currentMax)
	for i, v := range nums[:l-k] {
		if v == currentMax {
			currentMax = Max(nums[i+1 : i+k+1])
		} else if nums[i+k] > currentMax {
			currentMax = nums[i+k]
		}
		res = append(res, currentMax)
	}
	return res
}
```



采用正常解法

> 采用单调栈，这种解法之前确实没专门看过，不知道具体如何维护，上述解法实际上约等于记录max和2ndMax来进行迭代，过于粗糙了
>
> 依此代码逻辑，队列为降序存储

```go
func maxSlidingWindow(nums []int, k int) []int {
	if k == 1 {
		return nums
	} else if k == len(nums) {
		return []int{Max(nums)}
	}

	n := len(nums)
	result := make([]int, n-k+1)
	// 单调队列存储的是数组的索引
    // 降序存储
	var queue []int

	for i := 0; i < n; i++ {
		// 移除队列中不在当前窗口内的元素
		for len(queue) > 0 && queue[0] <= i-k {
            // 使用for遍历所有元素，清除所有过期项
			queue = queue[1:]
            // 取[1,len)切片
		}
		// 移除队列中比当前元素小的元素
		for len(queue) > 0 && nums[queue[len(queue)-1]] < nums[i] {
            // 同理，持续迭代，这里的len(queue) > 0条件是保险存在
            // 保证在多次迭代后，队列尾不会比当前元素小
            // 反复判定队尾的值是否大于当前值，如果都不大于就等于清空栈
			queue = queue[:len(queue)-1]
		}
        // 若是当前元素最大，队列已被清空，则当前元素直接加入队列
        // 若当前元素小于当前的队尾元素，则直接加入队尾
        // 若当前元素比队尾大，则从队尾开始逐个清算
		// 将当前元素的索引加入队列
		queue = append(queue, i)
		// 当窗口形成后，记录窗口的最大值
		if i >= k-1 {
			result[i-k+1] = nums[queue[0]]
		}
	}
	return result
}
```



### 76.最小覆盖子串

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。



**注意：**

- 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
- 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。



**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

**示例 2：**

```
输入：s = "a", t = "a"
输出："a"
解释：整个字符串 s 是最小覆盖子串。
```

**示例 3:**

```
输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```



还是滑动窗口

> 使用`target map[byte]int`来统计结果中需要的字母及数量
>
> 使用`valid int`与`len(target)`作比较判断右指针是否已经移动到了能包含子串的位置
>
> 使用`window map(byte)int`来统计窗口内字母及数量
>
> 在`valid == len(target)`条件达成后
>
> 1. 先判断当前左右差是否能更新最短长度，若可以，更新指向子串起始的start指针及长度
> 2. 暂存要移出的最左字符（因为其是两个统计map的key），再移动左指针
> 3. 先判断移出的是否是所需字母，不是则直接进入下一次循环
> 4. 移出的是所需字母，再判断`window[d]`与`target[d]`是否相等，不相等则破坏条件，`window[d]--`，同时`valid--`导致不满足`valid == len(target)`，跳出循环
> 5. 回到移动右指针过程

```go
func minWindow(s string, t string) string {
	// 先判断特殊情况
	if len(s) < len(t) {
		return ""
	}
	if len(s) == len(t) && len(t) == 1 {
		return func() string {
			if s == t {
				return s
			}
			return ""
		}()
	}

	// 统计字符串 t 中每个字符的出现次数
	target := make(map[byte]int)
	for i := 0; i < len(t); i++ {
		target[t[i]]++
	}

	// 窗口中字符的出现次数
	window := make(map[byte]int)
	// 窗口中已经满足条件的字符种类数
	valid := 0
	// 最小覆盖子串的起始位置和长度
	start, minLen := 0, len(s)+1
	// 左右指针
	left, right := 0, 0

	for right < len(s) {
		// 移入窗口的字符
		c := s[right]
		right++
		// 更新窗口内的数据
		if _, ok := target[c]; ok {
			window[c]++
			if window[c] == target[c] {
				valid++
			}
		}

		// 判断左侧窗口是否要收缩
		for valid == len(target) {
			// 更新最小覆盖子串
			if right-left < minLen {
				start = left
				minLen = right - left
			}
			// 移出窗口的字符
			d := s[left]
			left++
			// 更新窗口内的数据
			if _, ok := target[d]; ok {
				if window[d] == target[d] {
					valid--
				}
				window[d]--
			}
		}
	}

	if minLen == len(s)+1 {
		return ""
	}
	return s[start : start+minLen]
}
```



------

虽然只写了两题，但两题困难难度，也当自己任务完成吧。
