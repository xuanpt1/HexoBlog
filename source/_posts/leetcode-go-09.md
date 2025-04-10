---
title: leetcode-go-09
date: 2025-04-10 13:24:36
tags:
---

# leetcode热题100-Go解-2025.4.10

90/100

终于要入门了

笔试记录等维护明天再做吧

累了

### 4.寻找两个正序数组的中位数

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。

算法的时间复杂度应该为 `O(log (m+n))` 。



**示例 1：**

```
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并数组 = [1,2,3] ，中位数 2
```

**示例 2：**

```
输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
```



解一 O(log(m+n)) 思路

官方题解 等同于找到第`k`小的元素，`k = (m + n + 1) / 2`

逐步排除小数直到排除了k个后自然得到



解二 更优解 O(log(min(m, n))) 

```go
func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
	// 思路：
	// 等同于对两个数组求一个划分，使得划分后的左半部分的元素都小于等于右半部分的元素
	// 且左半部分的元素个数等于右半部分的元素个数/左右半部分的元素个数差为1
	// 因此可以用二分查找的方法找到这个划分点
	// 时间复杂度：O(log(min(m,n)))
	// 这算是最佳思路，O(log(m+n))的解法懒得看了
	// 空间复杂度：O(1)
	m, n := len(nums1), len(nums2)
	// 保证nums1的长度小于nums2的长度
	if m > n {
		nums1, nums2 = nums2, nums1
		m, n = n, m
	}
	// 二分查找的左右边界
	left, right := 0, m
	// 二分查找
	for left <= right {
		// 划分点
		// 对第一个数组进行划分，划分点左边的元素个数为i，右边的元素个数为m-i
		// 对第二个数组进行划分，划分点左边的元素个数为j，右边的元素个数为n-j
		// 划分点左边的元素个数等于划分点右边的元素个数/左右半部分的元素个数差为1
		// 因此j=(m+n+1)/2-i 这里利用了int类型的向下取整
		i := (left + right) / 2
		j := (m+n+1)/2 - i

		// 划分点左边的最大值
		nums1LeftMax := math.MinInt32
		if i > 0 {
			nums1LeftMax = nums1[i-1]
		}
		nums2LeftMax := math.MinInt32
		if j > 0 {
			nums2LeftMax = nums2[j-1]
		}
		// 划分点右边的最小值
		nums1RightMin := math.MaxInt32
		if i < m {
			nums1RightMin = nums1[i]
		}
		nums2RightMin := math.MaxInt32
		if j < n {
			nums2RightMin = nums2[j]
		}
		// 满足条件，找到了划分点
		if nums1LeftMax <= nums2RightMin && nums2LeftMax <= nums1RightMin {
			if (m+n)%2 == 0 {
				return float64(max(nums1LeftMax, nums2LeftMax)+min(nums1RightMin, nums2RightMin)) / 2
			} else {
				return float64(max(nums1LeftMax, nums2LeftMax))
			}
		} else if nums1LeftMax > nums2RightMin {
			// 划分点在nums1的左边，需要将划分点右移
			right = i - 1
		} else {
			// 划分点在nums1的右边，需要将划分点左移
			left = i + 1
		}
	}
	return 0.0
}
```



### 20. 有效的括号

水题



```go
// 20.有效的括号
func isValid(s string) bool {
	stack := []rune{}
	for _, c := range s {
		if c == '(' || c == '[' || c == '{' {
			stack = append(stack, c)
		} else if c == ')' {
			if len(stack) == 0 || stack[len(stack)-1] != '(' {
				return false
			} else {
				stack = stack[:len(stack)-1]
			}
		} else if c == ']' {
			if len(stack) == 0 || stack[len(stack)-1] != '[' {
				return false
			} else {
				stack = stack[:len(stack)-1]
			}
		} else if c == '}' {
			if len(stack) == 0 || stack[len(stack)-1] != '{' {
				return false
			} else {
				stack = stack[:len(stack)-1]
			}
		} else {
			return false
		}
	}

	if len(stack) == 0 {
		return true
	} else {
		return false
	}
}
```



### 155.最小栈

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

实现 `MinStack` 类:

- `MinStack()` 初始化堆栈对象。
- `void push(int val)` 将元素val推入堆栈。
- `void pop()` 删除堆栈顶部的元素。
- `int top()` 获取堆栈顶部的元素。
- `int getMin()` 获取堆栈中的最小元素。



**示例 1:**

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.
```



思路：

实际上是维护两个栈，一个数据栈，一个最小值栈

数据出栈时判断是否为最小值，是则最小值栈同步更新

```go
// 155.最小栈
// 思路：
// 1. 定义两个栈，一个栈用来存储数据，另一个栈用来存储最小值
// 2. 当栈为空时，将数据和最小值都存入栈中
// 3. 当栈不为空时，将数据存入数据栈中，将最小值存入最小值栈中
type MinStack struct {
	stack    []int
	minStack []int
}

func Constructor() MinStack {
	return MinStack{
		stack:    []int{},
		minStack: []int{},
	}
}

func (this *MinStack) Push(val int) {
	this.stack = append(this.stack, val)
	// TIP: leetcode中的测试集有对重复的数字压栈出栈，所以需要判断是否小于等于最小值栈的栈顶元素
	if len(this.minStack) == 0 || val <= this.minStack[len(this.minStack)-1] {
		this.minStack = append(this.minStack, val)
	}
}

func (this *MinStack) Pop() {
	if len(this.stack) == 0 {
		return
	}
	if this.stack[len(this.stack)-1] == this.minStack[len(this.minStack)-1] {
		this.minStack = this.minStack[:len(this.minStack)-1]
	}
	this.stack = this.stack[:len(this.stack)-1]
}

func (this *MinStack) Top() int {
	if len(this.stack) == 0 {
		return 0
	}
	return this.stack[len(this.stack)-1]
}

func (this *MinStack) GetMin() int {
	if len(this.minStack) == 0 {
		return 0
	}
	return this.minStack[len(this.minStack)-1]
}
```



### 394.字符串解码

水题 类似的还有逆波兰表达式

```go
// 394.字符串解码
func decodeString(s string) string {
	stack := []byte{}
	for i := 0; i < len(s); i++ {
		if s[i] != ']' {
			stack = append(stack, s[i])
		} else {
			// 1. 取出[]中的字符串
			str := []byte{}
			for len(stack) > 0 && stack[len(stack)-1] != '[' {
				str = append(str, stack[len(stack)-1])
				stack = stack[:len(stack)-1]
			}
			// 2. 取出[]前面的数字
			stack = stack[:len(stack)-1]
			num := 0
			base := 1
			for len(stack) > 0 && stack[len(stack)-1] >= '0' && stack[len(stack)-1] <= '9' {
				num += int(stack[len(stack)-1]-'0') * base
				stack = stack[:len(stack)-1]
				base *= 10
			}
			// 3. 将字符串重复num次
			for j := 0; j < num; j++ {
				for k := len(str) - 1; k >= 0; k-- {
					stack = append(stack, str[k])
				}
			}

		}
	}
	return string(stack)
}
```



### 84.柱状图中最大的矩形

给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。

求在该柱状图中，能够勾勒出来的矩形的最大面积。



**示例 1:**

![img](https://assets.leetcode.com/uploads/2021/01/04/histogram.jpg)

```
输入：heights = [2,1,5,6,2,3]
输出：10
解释：最大的矩形为图中红色区域，面积为 10
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/01/04/histogram-1.jpg)

```
输入： heights = [2,4]
输出： 4
```



解法 单调栈 一轮遍历解决

```go
// 84.柱状图中最大的矩形
func largestRectangleArea(heights []int) int {
	// 为了方便处理边界情况，在原数组前后添加高度为 0 的柱子
	heights = append([]int{0}, heights...)
	heights = append(heights, 0)
	stack := []int{}
	maxArea := 0
	for i := 0; i < len(heights); i++ {

		// 当栈不为空且当前柱子高度小于栈顶柱子高度时
		for len(stack) > 0 && heights[i] < heights[stack[len(stack)-1]] {
			// 弹出栈顶元素
			top := stack[len(stack)-1]
			stack = stack[:len(stack)-1]
			// 计算以弹出柱子为高的矩形面积
			area := heights[top] * (i - stack[len(stack)-1] - 1)
			if area > maxArea {
				maxArea = area
			}
		}

		// 将当前柱子的索引压入栈
		stack = append(stack, i)
	}
	return maxArea
}
```



### 215.数组中的第K个最大元素

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。



解一 堆排 O(nlogn)

```go
// 解一 堆排序
// 构建一个大顶堆，删除K-1个元素后，堆顶元素即为第K个最大元素
// 参数 heap 是一个整数切片，表示堆。
// 参数 i 是当前需要维护堆性质的节点的索引。
func heapify(heap []int, i int) {
	// 计算左子节点的索引
	left := 2*i + 1
	// 计算右子节点的索引
	right := 2*i + 2
	// 初始化最大值索引为当前节点的索引
	largest := i
	// 如果左子节点存在且左子节点的值大于当前最大值节点的值
	if left < len(heap) && heap[left] > heap[largest] {
		// 更新最大值索引为左子节点的索引
		largest = left
	}
	// 如果右子节点存在且右子节点的值大于当前最大值节点的值
	if right < len(heap) && heap[right] > heap[largest] {
		// 更新最大值索引为右子节点的索引
		largest = right
	}
	// 最大值索引不为当前节点索引时，不论与左子节点还是右子节点交换，都是等效的，发生在一轮交换中
	// 如果最大值索引不等于当前节点的索引，说明需要交换节点
	if largest != i {
		// 交换当前节点和最大值节点的值
		heap[i], heap[largest] = heap[largest], heap[i]
		// 递归调用 heapify 函数，继续维护以最大值节点为根的子树的堆性质
		heapify(heap, largest)
	}
}

// 时间复杂度：O(nlogn)
func findKthLargest(nums []int, k int) int {
	// 构建一个大顶堆
	heap := make([]int, len(nums))
	// 将 nums 数组中的元素复制到 heap 数组中
	for i := 0; i < len(nums); i++ {
		heap[i] = nums[i]
	}
	// 从最后一个非叶子节点开始，自下而上调整堆，使其满足大顶堆的性质
	for i := len(heap)/2 - 1; i >= 0; i-- {
		heapify(heap, i)
	}

	// 删除 K-1 个元素
	for i := 0; i < k-1; i++ {
		// 将堆顶元素（最大值）替换为堆的最后一个元素
		heap[0] = heap[len(heap)-1]
		// 移除堆的最后一个元素
		heap = heap[:len(heap)-1]
		// 从堆顶开始调整堆，使其重新满足大顶堆的性质
		heapify(heap, 0)
	}

	// 经过 K-1 次删除操作后，堆顶元素即为第 K 大的元素
	return heap[0]
}
```



解二 快排变式，快速选择

```go
func findKthLargest(nums []int, k int) int {
    // 获取数组的长度
    n := len(nums)
    // 调用 quickSelect 函数，查找第 n-k 小的元素，即第 k 大的元素
    return quickSelect(nums, 0, n-1, n-k)
}
func quickSelect(nums []int, l, r, k int) int {
	// 当左边界等于右边界时，说明区间内只有一个元素，直接返回该元素
	if l == r {
		return nums[k]
	}
	// 选择中间位置的元素作为基准值
	pivot := nums[(l+r)/2]
	// 初始化左右指针
	i, j := l, r
	// 进行分区操作，将小于基准值的元素移到左边，大于基准值的元素移到右边
	for i <= j {
		// 从左向右找到第一个大于等于基准值的元素
		for nums[i] < pivot {
			// l -> r
			i++
		}
		// 从右向左找到第一个小于等于基准值的元素
		for nums[j] > pivot {
			// r -> l
			j--
		}
		// 如果左指针小于等于右指针，交换两个元素的位置
		if i <= j {
			nums[i], nums[j] = nums[j], nums[i]
			// 移动左指针
			i++
			// 移动右指针
			j--
		}
	}
	// 若是快排，则需要对左右两部分分别进行递归调用
	// 若是快速选择，只需要对包含第 k 大元素的那一部分进行递归调用
	// 如果 k 小于等于 j，说明第 k 大的元素在左半部分，递归调用 quickSelect 函数在左半部分查找
	if k <= j {
		return quickSelect(nums, l, j, k)
	}
	// 如果 k 大于等于 i，说明第 k 大的元素在右半部分，递归调用 quickSelect 函数在右半部分查找
	if k >= i {
		return quickSelect(nums, i, r, k)
	}
	// 否则，说明第 k 小的元素就是当前位置的元素
	return nums[k]
}
```



### 347.前 K 个高频元素

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。



**示例 1:**

```
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

**示例 2:**

```
输入: nums = [1], k = 1
输出: [1]
```



解法 小顶堆

先遍历建立 值-频率 HashMap

再变量Map建立k容量小顶堆，达到上限时优先移除堆顶元素

最后剩下k容量即为前k高频的元素



小顶堆的实现：

```go
type Item struct {
	Val   int
	Count int
}

type PriorityQueue []*Item

// 实现 heap.Interface 的方法
// 等于实现了小顶堆
func (pq PriorityQueue) Len() int           { return len(pq) }
func (pq PriorityQueue) Less(i, j int) bool { return pq[i].Count < pq[j].Count }
func (pq PriorityQueue) Swap(i, j int)      { pq[i], pq[j] = pq[j], pq[i] }
func (pq *PriorityQueue) Push(x interface{}) {
	item := x.(*Item)
	*pq = append(*pq, item)
}
func (pq *PriorityQueue) Pop() interface{} {
	old := *pq
	n := len(old)
	item := old[n-1]
	*pq = old[0 : n-1]
	return item
}
```



具体方法：

```go
func topKFrequent(nums []int, k int) []int {
	// 统计每个元素的出现频率
	frequency := make(map[int]int)
	for _, num := range nums {
		frequency[num]++
	}

	// 初始化最小堆
	pq := make(PriorityQueue, 0)
	heap.Init(&pq)

	// 遍历频率字典，将元素加入堆中
	for value, count := range frequency {
		heap.Push(&pq, &Item{value, count})
		// 如果堆的大小超过了 k，弹出堆顶元素
		// 因为堆顶元素是最小的，所以弹出堆顶元素后，堆中剩下的元素就是前 k 个高频元素
		if pq.Len() > k {
			heap.Pop(&pq)
		}
	}

	// 从堆中取出前 k 个高频元素
	result := make([]int, k)
	for i := k - 1; i >= 0; i-- {
		result[i] = heap.Pop(&pq).(*Item).Val
	}

	return result
}
```



### 295.数据流的中位数

**中位数**是有序整数列表中的中间值。如果列表的大小是偶数，则没有中间值，中位数是两个中间值的平均值。

- 例如 `arr = [2,3,4]` 的中位数是 `3` 。
- 例如 `arr = [2,3]` 的中位数是 `(2 + 3) / 2 = 2.5` 。

实现 MedianFinder 类:

- `MedianFinder() `初始化 `MedianFinder` 对象。
- `void addNum(int num)` 将数据流中的整数 `num` 添加到数据结构中。
- `double findMedian()` 返回到目前为止所有元素的中位数。与实际答案相差 `10-5` 以内的答案将被接受。



**示例 1：**

```
输入
["MedianFinder", "addNum", "addNum", "findMedian", "addNum", "findMedian"]
[[], [1], [2], [], [3], []]
输出
[null, null, null, 1.5, null, 2.0]

解释
MedianFinder medianFinder = new MedianFinder();
medianFinder.addNum(1);    // arr = [1]
medianFinder.addNum(2);    // arr = [1, 2]
medianFinder.findMedian(); // 返回 1.5 ((1 + 2) / 2)
medianFinder.addNum(3);    // arr[1, 2, 3]
medianFinder.findMedian(); // return 2.0
```



解法：

采用两个小顶堆，

对于大于当前中位数的值，我们直接存储，便于直接调出最小值

对于小于当前中位数的值，我们做负值存储，便于直接调出最大值

小顶堆：

```go
type PriorityQueue2 struct {
	sort.IntSlice
}

func (pq *PriorityQueue2) Push(v interface{}) {
	pq.IntSlice = append(pq.IntSlice, v.(int))
}
func (pq *PriorityQueue2) Pop() interface{} {
	a := pq.IntSlice
	v := a[len(a)-1]
	pq.IntSlice = a[:len(a)-1]
	return v
}
```



题解：

> 注意负值的处理

```go
type MedianFinder struct {
	minHeap *PriorityQueue2
	maxHeap *PriorityQueue2
}

func Constructor2() MedianFinder {
	return MedianFinder{
		minHeap: &PriorityQueue2{},
		maxHeap: &PriorityQueue2{},
	}
}

func (this *MedianFinder) AddNum(num int) {
	minh, maxh := this.minHeap, this.maxHeap
	if minh.Len() == 0 || num <= -minh.IntSlice[0] {
		heap.Push(minh, -num)
		if minh.Len() > maxh.Len()+1 {
			heap.Push(maxh, -heap.Pop(minh).(int))
		}
	} else {
		heap.Push(maxh, num)
		if maxh.Len() > minh.Len() {
			heap.Push(minh, -heap.Pop(maxh).(int))
		}
	}
}

func (this *MedianFinder) FindMedian() float64 {
	if this.minHeap.Len() > this.maxHeap.Len() {
		return float64(-this.minHeap.IntSlice[0])
	}
	return float64(this.maxHeap.IntSlice[0]-this.minHeap.IntSlice[0]) / 2
}
```



### 121.买卖股票的最佳时机

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。



**示例 1：**

```
输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

**示例 2：**

```
输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。
```



贪心算法入门

```go
func maxProfit(prices []int) int {
	minPrice := prices[0]
    //将最低买入值初始化
	maxProfit := 0
    //初始化最大收益
	for i := 1; i < len(prices); i++ {
        //遍历
		if prices[i] < minPrice {
            // 若当前价低于历史最低，考虑在当前买入
			minPrice = prices[i]
		}else if prices[i] - minPrice > maxProfit {
            // 这里是else if在前一个条件后判断，保证这里最高的prices[i]一定是在minPrice后产生
            // 目前获利更大则更新
			maxProfit = prices[i] - minPrice
		}else {
			continue
		}
	}
	return maxProfit
}
```



### 55. 跳跃游戏

水题



```go
func canJump(nums []int) bool {
	maxPosition := 0
	for i := 0; i < len(nums); i++ {
		if i <= maxPosition {
			maxPosition = max(maxPosition, i+nums[i])
			if maxPosition >= len(nums)-1 {
				return true
			}
		}
	}
	return false
}
```



### 45. 跳跃游戏 II

比上题稍微难点



解一 正向 O(n)

```go
// 每次选择能跳的最远的位置，直到跳到最后一个位置
func jump(nums []int) int {
	end := 0
	maxPosition := 0
	steps := 0
	for i := 0; i < len(nums)-1; i++ {
		maxPosition = max(maxPosition, i+nums[i])
		if i == end {
			end = maxPosition
			steps++
		}

	}
	return steps
}
```



解二 反向 O(n^2)

```go
// 反向查找出发位置
func jump2(nums []int) int {
	position := len(nums) - 1
	steps := 0
	for position > 0 {
		for i := 0; i < position; i++ {
			if i+nums[i] >= position {
				position = i
				steps++
				break
			}
		}
	}
	return steps
}
```



### 70.爬楼梯

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？



**示例 1：**

```
输入：n = 2
输出：2
解释：有两种方法可以爬到楼顶。
1. 1 阶 + 1 阶
2. 2 阶
```

**示例 2：**

```
输入：n = 3
输出：3
解释：有三种方法可以爬到楼顶。
1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶
```



```go
// dp[i] = dp[i-1] + dp[i-2]
func climbStairs(n int) int {
	if n == 1 {
		return 1
	}
	dp := make([]int, n+1)
	dp[1] = 1
	dp[2] = 2
	for i := 3; i <= n; i++ {
		dp[i] = dp[i-1] + dp[i-2]
	}
	return dp[n]
}
```



### 118.杨辉三角

水题



```go
// dp[i][j] = dp[i-1][j-1] + dp[i-1][j]
func generate(numRows int) [][]int {
	res := make([][]int, numRows)
	for i := 0; i < numRows; i++ {
		res[i] = make([]int, i+1)
	}
	// 初始化
	for i := 0; i < numRows; i++ {
		for j := 0; j <= i; j++ {
			if j == 0 || j == i {
				res[i][j] = 1
			}
		}
	}
	
	// 计算
	for i := 2; i < numRows; i++ {
		for j := 1; j < i; j++ {
			res[i][j] = res[i-1][j-1] + res[i-1][j]
		}
	}
	return res
}
```



### 198.打家劫舍

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。



**示例 1：**

```
输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

**示例 2：**

```
输入：[2,7,9,3,1]
输出：12
解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。
```



```go
// dp[i] = max(dp[i-1], dp[i-2]+nums[i])
func rob(nums []int) int {
	if len(nums) == 0 {
		return 0
	}
	if len(nums) == 1 {
		return nums[0]
	}
	dp := make([]int, len(nums))
	dp[0] = nums[0]
	dp[1] = max(nums[0], nums[1])
	for i := 2; i < len(nums); i++ {
		dp[i] = max(dp[i-1], dp[i-2]+nums[i])
	}
	return dp[len(nums)-1]
}
```



### 763.划分字母区间

给你一个字符串 `s` 。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。例如，字符串 `"ababcc"` 能够被分为 `["abab", "cc"]`，但类似 `["aba", "bcc"]` 或 `["ab", "ab", "cc"]` 的划分是非法的。

注意，划分结果需要满足：将所有划分结果按顺序连接，得到的字符串仍然是 `s` 。

返回一个表示每个字符串片段的长度的列表。



**示例 1：**

```
输入：s = "ababcbacadefegdehijhklij"
输出：[9,7,8]
解释：
划分结果为 "ababcbaca"、"defegde"、"hijhklij" 。
每个字母最多出现在一个片段中。
像 "ababcbacadefegde", "hijhklij" 这样的划分是错误的，因为划分的片段数较少。 
```

**示例 2：**

```
输入：s = "eccbbbbdec"
输出：[10]
```



```go
func partitionLabels(s string) []int {
	// lastPos 数组用于记录每个小写字母在字符串 s 中最后一次出现的位置
	lastPos := [26]int{}
	// 遍历字符串 s，记录每个字母的最后出现位置
	for i, c := range s {
		// 通过 c-'a' 计算字母在 lastPos 数组中的索引
		lastPos[c-'a'] = i
	}
	// start 表示当前划分片段的起始位置
	start, end := 0, 0
	// partition 用于存储每个划分片段的长度
	var partition []int
	// 再次遍历字符串 s
	for i, c := range s {
		// 如果当前字母的最后出现位置大于当前划分片段的结束位置
		if lastPos[c-'a'] > end {
			// 更新当前划分片段的结束位置
			end = lastPos[c-'a']
		}
		// 当遍历到当前划分片段的结束位置时
		if i == end {
			// 计算当前划分片段的长度并添加到 partition 中
			partition = append(partition, end-start+1)
			// 更新下一个划分片段的起始位置
			start = end + 1
		}
	}
	// 返回划分片段的长度列表
	return partition
}
```



### 279. 完全平方数

给你一个整数 `n` ，返回 *和为 `n` 的完全平方数的最少数量* 。

**完全平方数** 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，`1`、`4`、`9` 和 `16` 都是完全平方数，而 `3` 和 `11` 不是。



**示例 1：**

```
输入：n = 12
输出：3 
解释：12 = 4 + 4 + 4
```

**示例 2：**

```
输入：n = 13
输出：2
解释：13 = 4 + 9
```



```go
func numSquares(n int) int {
    // 创建一个长度为 n+1 的数组 dp，用于存储每个数的最少完全平方数组合
    dp := make([]int, n+1)
    // 初始化 dp 数组，每个元素初始值为最大值
    for i := range dp {
       dp[i] = math.MaxInt32
    }
    // 0 只需要 0 个完全平方数
    dp[0] = 0

    // 遍历从 1 到 n 的每个数
    for i := 1; i <= n; i++ {
       // 遍历所有可能的完全平方数
       for j := 1; j*j <= i; j++ {
          // 更新 dp[i] 的值，取当前值和 dp[i-j*j]+1 中的较小值
          dp[i] = min(dp[i], dp[i-j*j]+1)
       }
    }

    // 返回 n 的最少完全平方数组合
    return dp[n]
}
```



### 322.零钱兑换

给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。

计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

你可以认为每种硬币的数量是无限的。



**示例 1：**

```
输入：coins = [1, 2, 5], amount = 11
输出：3 
解释：11 = 5 + 5 + 1
```

**示例 2：**

```
输入：coins = [2], amount = 3
输出：-1
```

**示例 3：**

```
输入：coins = [1], amount = 0
输出：0
```



```go
// dp[i] = min(dp[i], for each coin dp[i-coin]+1)
func coinChange(coins []int, amount int) int {
	// 创建一个长度为 amount+1 的数组 dp，用于存储每个金额所需的最少硬币数
	dp := make([]int, amount+1)
	// 初始化 dp 数组，每个元素初始值为最大值
	for i := range dp {
		dp[i] = math.MaxInt32
	}
	// 0 元需要 0 个硬币
	dp[0] = 0
	// 遍历从 1 到 amount 的每个金额
	for i := 1; i <= amount; i++ {
		// 遍历所有可能的硬币面额
		for _, coin := range coins {
			// 如果当前硬币面额小于等于当前金额
			if coin <= i {
				// 更新 dp[i] 的值，取当前值和 dp[i-coin]+1 中的较小值
				dp[i] = min(dp[i], dp[i-coin]+1)
			}

		}

	}
	// 如果 dp[amount] 仍然是最大值，则无法凑出指定金额，返回 -1
	if dp[amount] == math.MaxInt32 {
		return -1
	}
	// 返回凑出指定金额所需的最少硬币数
	return dp[amount]
}
```



### 139. 单词拆分

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意：**不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。



**示例 1：**

```
输入: s = "leetcode", wordDict = ["leet", "code"]
输出: true
解释: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。
```

**示例 2：**

```
输入: s = "applepenapple", wordDict = ["apple", "pen"]
输出: true
解释: 返回 true 因为 "applepenapple" 可以由 "apple" "pen" "apple" 拼接成。
     注意，你可以重复使用字典中的单词。
```

**示例 3：**

```
输入: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
输出: false
```



```go
// dp[i] = dp[j] && s[j:i] in wordDict
func wordBreak(s string, wordDict []string) bool {
	// 创建一个长度为 len(s)+1 的布尔数组 dp，用于存储每个位置是否可以拆分
	dp := make([]bool, len(s)+1)
	// 初始化 dp[0] 为 true，表示空字符串可以拆分
	dp[0] = true
	// 遍历从 1 到 len(s) 的每个位置
	for i := 1; i <= len(s); i++ {
		// 遍历从 0 到 i-1 的每个位置
		for j := 0; j < i; j++ {
			// 如果 dp[j] 为 true 且 s[j:i] 在 wordDict 中
			if dp[j] && contains(wordDict, s[j:i]) {
				// 将 dp[i] 设置为 true，表示从位置 j 到位置 i 的子串可以拆分
				dp[i] = true
				// 跳出内层循环，继续处理下一个位置
				break
			}
		}
	}
	// 返回 dp[len(s)]，表示整个字符串是否可以拆分
	return dp[len(s)]
}

// 判断字符串 s 是否在字符串数组 wordDict 中
func contains(wordDict []string, s string) bool {
	for _, word := range wordDict {
		if word == s {
			return true
		}
	}
	return false
}
```



### 300.最长递增子序列

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。



**示例 1：**

```
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
```

**示例 2：**

```
输入：nums = [0,1,0,3,2,3]
输出：4
```

**示例 3：**

```
输入：nums = [7,7,7,7,7,7,7]
输出：1
```



```go
// dp[i] = max(dp[i], dp[j]+1) for each j < i and nums[j] < nums[i]
func lengthOfLIS(nums []int) int {
	n := len(nums)
	if n == 0 {
		return 0
	}
	// dp[i] 表示以 nums[i] 结尾的最长递增子序列的长度
	dp := make([]int, n)
	for i := range dp {
		dp[i] = 1
	}
	maxLen := 1
	for i := 1; i < n; i++ {
		for j := 0; j < i; j++ {
			if nums[i] > nums[j] {
				dp[i] = max(dp[i], dp[j]+1)
			}
		}
		maxLen = max(maxLen, dp[i])
	}
	return maxLen
}
```



### 152. 乘积最大子数组

给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个 **32-位** 整数。



**示例 1:**

```
输入: nums = [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```

**示例 2:**

```
输入: nums = [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```



数组里存在负数，这会让最大乘积和最小乘积相互转变，所以要同时维护当前最大乘积 `maxDp` 和当前最小乘积 `minDp`。在遍历数组期间，持续更新这两个值以及全局最大乘积 `ans`，最终返回 `ans`。

```go
func maxProduct(nums []int) int {
	n := len(nums)
	if n == 0 {
		return 0
	}
	// 由于存在负数，会导致最大的变最小的，最小的变最大的，所以需要同时维护当前最大和最小乘积
	maxDp, minDp, ans := nums[0], nums[0], nums[0]
	for i := 1; i < n; i++ {
		mx, mn := maxDp, minDp
		// 更新当前最大乘积
		maxDp = max(mx*nums[i], max(nums[i], mn*nums[i]))
		// 更新当前最小乘积
		minDp = min(mn*nums[i], min(nums[i], mx*nums[i]))
		// 更新全局最大乘积
		ans = max(ans, maxDp)
	}
	return ans
}
```

