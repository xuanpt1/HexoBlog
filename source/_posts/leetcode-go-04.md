---
title: leetcode-go-04
date: 2025-03-23 23:57:57
tags:
---

# leetcode热题100-GO解-2025.3.21

15/100

今日主要在忙双选会事宜，毕竟号称本校春招最大规模双选会

累一天但是每日日常不能断



### 53.最大子数组和

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组**是数组中的一个连续部分。



**示例 1：**

```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

**示例 2：**

```
输入：nums = [1]
输出：1
```

**示例 3：**

```
输入：nums = [5,4,-1,7,8]
输出：23
```



很经典的基础题，这里用动态规划方式来解

> Kadane算法

```go
func maxSubArray(nums []int) int {
	if len(nums) == 1 {
		return nums[0]
	}

	currentMax := nums[0]
    //记录当前最大值
	globalMax := nums[0]
    //记录全局最大值
    
	for i := 1; i < len(nums); i++ {
        //初始化之后，对于nums[i]项考虑是否计入当前最大值，
		currentMax = max(nums[i], currentMax+nums[i])
        //再比较更新全局最大值
		globalMax = max(globalMax, currentMax)
	}
	return globalMax
}
```

Kadane算法的核心在于它采用了动态规划的思想，通过一次遍历数组，同时维护两个变量：

1. **当前最大子数组和（`currentMax`）**：表示**以当前元素结尾**的最大子数组和。
2. **全局最大子数组和（`globalMax`）**：表示整个数组中找到的最大子数组和。

在遍历数组的过程中，对于每个元素，算法会根据以下规则更新这两个变量：

- 更新 `currentMax`：
- 对于当前元素`nums[i]`，`currentMax`有两种选择：
  - 要么只取当前元素 `nums[i]`。
  - 要么将当前元素 `nums[i]` 加到之前的 `currentMax` 上。 选择这两种情况中的较大值作为新的 `currentMax`，即 `currentMax = max(nums[i], currentMax + nums[i])`。
- **更新 `globalMax`**：将 `currentMax` 和 `globalMax` 进行比较，如果 `currentMax` 大于 `globalMax`，则更新 `globalMax` 为 `currentMax`，即 `globalMax = max(globalMax, currentMax)`。



二解用分治法 类似递归 遍历二叉树的思路

leetcode题解有详细解释，在此代码中附上我的浅显理解

```go
func maxSubArray(nums []int) int {
	return get(nums, 0, len(nums)-1).mSum
}
type Status struct {
	lSum, rSum, mSum, iSum int
	// 这个Status的内容定义是关键
	// lSum: 以左端点为起点的最大子段和
	// rSum: 以右端点为起点的最大子段和
	// mSum: 区间[l,r]内的最大子段和
	// iSum: 区间[l,r]的区间和
}
func pushUp(l, r Status) Status {
	// 合并两个子区间的信息
	// 主要从次级叶子节点的状况来描述,即区间[x,x+1]的状况
	// 例如：
	// 	左子区间：[l,l]
	// 	右子区间：[r,r]
	// 	合并后的区间：[l,r]
	// 此时，lSum: 以左端点为起点的最大子段和为max(l.lSum, l.iSum+r.lSum)
	// 即，要么是左区间的最大子段和，要么是左区间的区间和+右区间的最大左起始子段和
	// rSum同理
	// 以上述为例子，则为:
	// 	lSum = max(l, l+r)
	// 	rSum = max(r, r+l)
	// 	mSum = max(l, r, l + r)
	// 后续递归逻辑以此类推
	iSum := l.iSum + r.iSum
	lSum := max(l.lSum, l.iSum+r.lSum)
	rSum := max(r.rSum, r.iSum+l.rSum)
	mSum := max(max(l.mSum, r.mSum), l.rSum+r.lSum)
	return Status{lSum, rSum, mSum, iSum}
}
func get(nums []int, l, r int) Status {
	if l == r {
		// 递归终止条件 遍历到叶子节点
		return Status{nums[l], nums[l], nums[l], nums[l]}
	}
	m := (l + r) >> 1
	// m >> 1 相当于 m / 2 int类型的除法是向下取整的
	lSub := get(nums, l, m)
	// 递归左子树
	rSub := get(nums, m+1, r)
	// 递归右子树
	return pushUp(lSub, rSub)
}
```



### 56.合并区间

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。



**示例 1：**

```
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```

**示例 2：**

```
输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。
```



**提示：**

- `1 <= intervals.length <= 104`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 104`



简单的排序解

算是用上了GO的语法糖`sort.Slice`

```go
func merge(intervals [][]int) [][]int {
	// 如果区间数量小于等于 1，直接返回原区间数组
	if len(intervals) <= 1 {
		return intervals
	}

	// 按区间的左端点进行排序
	sort.Slice(intervals, func(i, j int) bool {
		return intervals[i][0] < intervals[j][0]
	})

	// 存储合并后的区间
	merged := [][]int{intervals[0]}
	for _, interval := range intervals[1:] {
		// 获取最后一个合并后的区间
		last := merged[len(merged)-1]
		// 如果当前区间的左端点小于等于最后一个合并区间的右端点，说明有重叠
		if interval[0] <= last[1] {
			// 更新最后一个合并区间的右端点为两者右端点的最大值
			last[1] = max(last[1], interval[1])
		} else {
			// 没有重叠，将当前区间添加到合并后的区间数组中
			merged = append(merged, interval)
		}
	}
	return merged
}
```

关键在于直接使用**`sort.Slice`**定义了对区间的排序方式，依**左边界从小到大排序**

再取**最后一个合并后的区间**，即当前包含了的区间的最右端，与**排序后的当前区间**进行比较

若**目前区间的左端点**小于**合并后的最右端**，证明此区间可以并入最后一个合并后的区间，右端点取**二者最大值**

否则将当前区间添加到合并后区间数组集中

遍历得出最终结果



### 189.轮转数组

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。



**示例 1:**

```
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右轮转 1 步: [7,1,2,3,4,5,6]
向右轮转 2 步: [6,7,1,2,3,4,5]
向右轮转 3 步: [5,6,7,1,2,3,4]
```

**示例 2:**

```
输入：nums = [-1,-100,3,99], k = 2
输出：[3,99,-1,-100]
解释: 
向右轮转 1 步: [99,-1,-100,3]
向右轮转 2 步: [3,99,-1,-100]
```



**提示：**

- `1 <= nums.length <= 105`
- `-231 <= nums[i] <= 231 - 1`
- `0 <= k <= 105`



你知道的，我们GO语言的数组是`Slice`，是有语法糖的()

一步到位解：

> 这里第一次写时出现了问题
>
> 函数内部对`nums`切片的重新赋值不会影响到函数外部传入的原始切片。
>
> 在 Go 语言里，虽然切片是引用类型，但当对`nums`进行重新赋值时，实际上只是改变了函数内部的 `nums` 变量指向，而原始切片本身并没有被修改。
>
> 使用`temp`暂存，再使用`copy`函数将`temp`中元素复制回原切片`nums`

```go
func rotate(nums []int, k int)  {
    // 先判断特殊情况
	if k == 0 || len(nums) == 1 {
		return
	}

    k %= len(nums)
	// 采用临时切片存储轮转后的元素
    temp := append(nums[len(nums)-k:], nums[:len(nums)-k]...)
    // 将临时切片的元素复制回原切片
    copy(nums, temp)
}
```



二解 标准答案 反转反转再反转

先整体反转，再反转其

```go
func rotate(nums []int, k int) {
	// 先判断特殊情况
	if k == 0 || len(nums) == 1 {
		return
	}
	// 取模操作，确保 k 在数组长度范围内
	k %= len(nums)
	// 反转整个数组
	reverse(nums)
	// 反转前 k 个元素
	reverse(nums[:k])
	// 反转剩余的 n - k 个元素
	reverse(nums[k:])
}
// reverse 函数用于反转数组中的元素
func reverse(nums []int) {
	for i, j := 0, len(nums)-1; i < j; i, j = i+1, j-1 {
		// 交换元素位置
		nums[i], nums[j] = nums[j], nums[i]
	}
}
```



三解，使用一个与原切片相同长度的新切片，来放置移动位置后的元素

再将新切片复制到原切片

```go
func rotate(nums []int, k int) {
	// 先判断特殊情况
	if k == 0 || len(nums) == 1 {
		return
	}
	k %= len(nums)
	// 新建一个切片，长度和原切片相同
	newNums := make([]int, len(nums))
	for i, v := range nums {
		newNums[(i+k)%len(nums)] = v
	}
	// 将新切片的值复制到原切片
	copy(nums, newNums)
}
```



四解，三解的优化，直接环状替换

> **Q：为什么必须要使用双层循环？**
>
> A：环状替换的核心思路：
>
> ​	`n % k`结果为同一值，即Hash结果为同一值的所有元素，构成了一个需要轮换的环
>
> ​	外层循环相当于在遍历所有的这些“环”
>
> ​	内层循环则是在某个“环”内部，对这些元素进行轮换

```go
func rotate(nums []int, k int) {
	// 先判断特殊情况
	if k == 0 || len(nums) == 1 {
		return
	}
	n := len(nums)
	k %= n
	count := 0
	// 循环次数不会超过数组长度
	for start := 0; count < n; start++ {
		current := start
		prev := nums[start]
		for {
			next := (current + k) % n
			temp := nums[next]
			nums[next] = prev
			prev = temp
			current = next
			count++
			// 如果回到起始位置，说明当前环替换完成
			if start == current {
				break
			}
		}
	}
}
```

详细些的解释：

**外层循环**

```go
for start := 0; count < n; start++ {
    // ...
}
```

- 处理多个不相交的环：在某些情况下，数组可能会被分成多个不相交的环。外层循环的作用是从不同的起始位置开始，尝试启动新的环替换操作，确保所有元素都能被正确处理。`start` 变量表示当前开始替换的元素索引，`count` 用于记录已经替换的元素数量，当 `count` 达到数组长度 `n` 时，说明所有元素都已经被替换过，外层循环结束。

**内层循环**

```go
current := start
prev := nums[start]
for {
    next := (current + k) % n
    temp := nums[next]
    nums[next] = prev
    prev = temp
    current = next
    count++
    if start == current {
        break
    }
}
```

- **完成一个环的替换**：内层循环从`start`位置开始，不断将元素移动到其轮转后的位置，直到回到起始位置`start`。具体步骤如下：
  - 计算当前元素要移动到的位置 `next`。
  - 保存 `next` 位置原来的元素到 `temp`。
  - 将当前元素 `prev` 放到 `next` 位置。
  - 更新 `prev` 为 `temp` 的值。
  - 更新 `current` 为 `next`。
  - `count` 加 1 表示已经替换了一个元素。
  - 如果 `current` 等于 `start`，说明已经回到起始位置，当前环的替换完成，跳出内层循环。

**多个不相交环示例：**

```go
nums := {1, 2, 3, 4}
k := 2
```

此时存在两个不相交环

```
1 -> 3 -> 1
2 -> 4 -> 2
```

若是如同题目示例一`nums = [1, 2, 3, 4, 5, 6, 7]  k = 3`，则只有一个环存在，即：

```
1 -> 4 -> 7 -> 3 -> 6 -> 2 -> 5 -> 1
```

