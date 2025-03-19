---
title: leetcode-go-01
date: 2025-03-19 18:56:45
tags:
---

# leetcode热题100-GO解

2025.03.19 

## 6/100



### 1.两数之和

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。



**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

**示例 2：**

```
输入：nums = [3,2,4], target = 6
输出：[1,2]
```

**示例 3：**

```
输入：nums = [3,3], target = 6
输出：[0,1]
```



暴力枚举O(n^2)略

使用map，将原本array中的（index，value）反向制作成（value，index）

再通过遍历array，检测map中是否存在target-value项，存在则去除返回二者下标

```go
func twoSum(nums []int, target int) []int {
    numMap := make(map[int]int)
	for index, value := range nums {
		numMap[value] = index
	}
	for i, num := range nums {
		complement := target - num
		if j, ok := numMap[complement]; ok && j != i {
			return []int{i, j}
		}
	}
	return []int{}
}
```



### 49.字母异位词分组

给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

**字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。



**示例 1:**

```
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

**示例 2:**

```
输入: strs = [""]
输出: [[""]]
```

**示例 3:**

```
输入: strs = ["a"]
输出: [["a"]]
```



**提示：**

- `1 <= strs.length <= 104`
- `0 <= strs[i].length <= 100`
- **`strs[i]` 仅包含小写字母**



提示三相对重要，可以决定我选择的这个方法的可行性（或者说关键值

首先做个特殊情况处理

```go
if len(strs) <= 1 {
		return [][]string{strs}
	}
```

再初始化一个特殊的map，它的key为我们对原字符串array中每个str计数后的产物，value为该字符串

（是的，仍然是用map更方便

```go
m := make(map[[26]int][]string)
```

再对strs遍历，数数每个字母有几个

```go
for _, str := range strs {
		arr := [26]int{}
		for _, c := range str {
			arr[c-'a']++
		}
		m[arr] = append(m[arr], str)
	}
```

这里有个小点，取`m[arr]`时这里的`arr`为值而非地址/指针，故相同时会append到map `m`中的同一位置

再将m中值取出得到结果

完整流程如下：

```go
func groupAnagrams(strs []string) [][]string {
	if len(strs) <= 1 {
		return [][]string{strs}
	}
	m := make(map[[26]int][]string)
	for _, str := range strs {
		arr := [26]int{}
		for _, c := range str {
			arr[c-'a']++
		}
		m[arr] = append(m[arr], str)
	}
	res := make([][]string, 0, len(m))
	for _, v := range m {
		res = append(res, v)
	}
	return res
}
```

参考leetcode官方题解内容

> 合理设计的哈希键能够有效地整合原始信息，找出对于解题有用的结果信息
> 常见哈希键设计：

> - 在字符串和数组当中，当**每个元素的顺序不重要**时，可以使用**排序后的字符串或数组**作为键
> - 如果只关心**每个值的偏移量**，例如第一个值的偏移量，则可以使用**偏移量**作为键
> - 在树中，我们通常会用**子树的序列化表述**作为键



### 128.最长连续数列

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

> 我永远喜欢map（不是



**示例 1：**

```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

**示例 2：**

```
输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
```

**示例 3：**

```
输入：nums = [1,0,1,2]
输出：3
```



依然是hashmap的使用，原本的数作为`key`,设定bool类型的`value`来确认该数是否存在，同时去重

```go
func longestConsecutive(nums []int) int {
	if len(nums) == 1 {
		return 1
	} else if len(nums) == 0 {
		return 0
	}
    //先处理特殊情况

	m := make(map[int]bool)
	for _, v := range nums {
		m[v] = true
	}
    //记录存在哪些数并去重

    //longestStreak: 最长连续长度
	longestStreak := 0
	for num := range m {
        //遍历map，仅当m[num-1]，即该数的前一个数不在map中，该数为起始时开始记录
		if !m[num-1] {
			currentNum := num
			currentStreak := 1
            //记录当前数及初始化长度
            //若下一个数存在则累计奖池
			for m[currentNum+1] {
				currentNum++
				currentStreak++
                //（奖池还在累加！）
			}
            //结束后比较并记录最长连续长度
			if longestStreak < currentStreak {
				longestStreak = currentStreak
			}
		}
	}
	return longestStreak
    //输出喵~
}
```



### 283.移动零

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

> 很基础的题目，但是思想蛮重要的



**示例 1:**

```
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

**示例 2:**

```
输入: nums = [0]
输出: [0]
```



解一：

从数组两头往中间缩

```go
func moveZeroes(nums []int) {
	//思路为cpp的双指针，但是go中没有指针，所以用下标代替
	//left：左指针，right：右指针
	left, right := 0, len(nums)-1
	for left < right {
		if nums[left] == 0 {
			nums = append(nums[:left], nums[left+1:]...)
			nums = append(nums, 0)
			right--
			continue
		}
		left++
	}
}
```

**tips: GO语法糖**

- `nums[:left]`为取`num[0]`到`num[left-1]`的切片，即[0,left)
- `nums[left+1:]`为取`nums[left+1]`到`nums[]`末尾的切片
- `...`在GO中为展开符，代表将`nums[left+1:]`顺序展开为单个元素
- 需要展开的原因是GO的append函数，`input_1`类型为切片，`input_2...`为追加到切片末尾的具体元素，不能为切片



解二：

leetcode官方题解

```go
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

大致相当于一条`x x 0 0 0 x x x x x x`000的毛毛虫蠕动向前并吸收路上经过了的0

仅在`right`指向非0，此时`left`必定指向0时互换，故不会更改顺序



解三：

直接计数并标记非0元素位置，后续补0

很好理解，但相对官方解等于多做了一次遍历

```go
func moveZeroes(nums []int) {
	//另起数组记录非0数，后续补0
	var newNums []int
	for _, num := range nums {
		if num != 0 {
			newNums = append(newNums, num)
		}
	}
	for i := 0; i < len(newNums); i++ {
		nums[i] = newNums[i]
	}
	for i := len(newNums); i < len(nums); i++ {
		nums[i] = 0
	}
}
```



### 11.盛最多水的容器

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。



**示例 1：**

![img](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

**示例 2：**

```
输入：height = [1,1]
输出：1
```



使用类似贪心算法，先取左右两端，再往中间缩，逐步取最大水量

很简单，但错了一个小点

**使用双指针算法时请确认每次只移动一个指针！！！！！**

```go
func maxArea(height []int) int {
	//双指针，i，j分别指向头尾，每次移动较小的指针
	i, j := 0, len(height)-1
	maxArea := 0
	for i < j {
		area := (j - i) * min(height[i], height[j])
		if area > maxArea {
			maxArea = area
		}
		
		//if height[i] < height[j] {
		//	i++
		//}
		//if height[i] >= height[j] {
		//	j--
		//}
		
		if height[i] < height[j] {
			i++
		} else {
			j--
		}
	}
	return maxArea
}
```

如对`height[]=[1,8,6,2,5,4,8,3,7]`

**初始状态**
	初始时，`i = 0，j = 8，height[i] = 1，height[j] = 7，maxArea = 0`。
**原代码执行过程**

- 第一次循环：
  - 因为 `height[i] = 1` 小于 `height[j] = 7`，所以` i `右移一位，此时` i = 1，height[i] = 8`。
  - 接着检查` height[i] >= height[j]`，由于 `height[i] = 8 `大于` height[j] = 7`，所以` j `左移一位，此时` j = 7`。
  - 在这一次循环中，两个指针` i `和` j `都发生了移动，这不符合 **“每次只移动一个指针”** 的双指针算法逻辑，会导致一些可能的组合被跳过，从而影响最终结果。



### 15.三数之和

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。



**示例 1：**

```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
解释：
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。
```

**示例 2：**

```
输入：nums = [0,1,1]
输出：[]
解释：唯一可能的三元组和不为 0 。
```

**示例 3：**

```
输入：nums = [0,0,0]
输出：[[0,0,0]]
解释：唯一可能的三元组和为 0 。
```



思路为先将数组排序，然后遍历数组，固定一个数，再使用双指针，一个指向固定数的下一个数，一个指向数组的末尾。

```go
func threeSum(nums []int) [][]int {
    // 首先对数组进行排序
    sort.Ints(nums)
    var result [][]int
    n := len(nums)

    // 遍历数组，固定第一个数
    // 排序之后，如果第一个数大于0，那么三数之和一定大于0，所以直接返回
    if nums[0] > 0 {
       return result
    }
    for i := 0; i < n-2; i++ {
       // 排除第一个数为0的情况后，若nums[i] == nums[i-1],则取[nums[i], nums[i-1], ...]的情况在
       // x=i-1时已经计算过了，包含[nums[i], nums[nums[i:]..., ...]的状况在x=i时也已经计算过了，
       // 所以直接跳过
       if i > 0 && nums[i] == nums[i-1] {
          continue
       }
       // 双指针，分别指向剩余元素的首尾
       left, right := i+1, n-1
       target := -nums[i]

       for left < right {
          sum := nums[left] + nums[right]
          if sum == target {
             // 计算三个数的和，如果和为0，则将这三个数加入结果集中，然后移动指针，
             result = append(result, []int{nums[i], nums[left], nums[right]})
             // 此时需要注意去重，如果左指针和右指针指向的数和它们前一个数相同，则需要移动指针，直到指向的数和前一个数不同。
             // 跳过重复的第二个数
             for left < right && nums[left] == nums[left+1] {
                left++
             }
             // 跳过重复的第三个数
             for left < right && nums[right] == nums[right-1] {
                right--
             }
             // 移动指针
             left++
             right--
          } else if sum < target {
             // 如果和小于0，则将左指针右移，因为数组已经排序，所以右移后三个数的和会更大，
             left++
          } else {
             // 如果和大于0，则将右指针左移，因为数组已经排序，所以左移后三个数的和会更小。
             right--
          }
       }
    }
    return result
}
```
