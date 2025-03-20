---
title: leetcode-go-02
date: 2025-03-20 14:32:17
tags:
---

# leetcode热题100-GO解-2025.3.20



### 42.接雨水

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。



**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rainwatertrap.png)

```
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 
```

**示例 2：**

```
输入：height = [4,2,0,3,2,5]
输出：9
```



leetcode官方题解很详细，不再赘述

```go
//采用双指针解法
func trap(height []int) int {
	left, right := 0, len(height)-1
    //左右双指针
	leftHeight, rightHeight := 0, 0
    //左右最高高度
	result := 0
    
    //依旧先排除特殊情况
	if len(height) <= 2 {
		return 0
	}

	for left < right {
        //在左右双指针相遇前
		if height[left] < height[right] {
            //若右边更高
			if height[left] > leftHeight {
                //当前左指针所在高度比记录的最高高度高，就更新左最高
				leftHeight = height[left]
			} else {
                //比左最高低，则只需与左最高对比累加结果，右最高是以判断过比左最高高的
                //木桶效应，只需考虑短板即可
				result += leftHeight - height[left]
			}
            //处理完移动左指针
			left++
		} else {
            //反之亦然
			if height[right] > rightHeight {
				rightHeight = height[right]
			} else {
				result += rightHeight - height[right]
			}
			right--
		}
	}

	return result
}
```



### 3.无重复字符的最长子串

给定一个字符串 `s` ，请你找出其中不含有重复字符的**最长子串**的长度。



**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```



> tips:实际提示中有：`s` 由英文字母、数字、符号和空格组成，即必为AsicII字符，无需使用rune，使用byte已经可以全覆盖



滑动窗口经典题

```go
func lengthOfLongestSubstring(s string) int {
	//还是先判断特殊情况
	switch len(s) {
	case 0:
		return 0
	case 1:
		return 1
	}

	//使用map建立字符出现位置索引
	m := make(map[rune]int)
	//fix:这里使用rune类型，因为字符串中可能会有中文等非ASCII字符
	//使用双指针，left，right分别指向子串的首尾
	left, right := 0, 0
	//maxLength记录最长子串的长度
	maxLength := 0

	// 遍历字符串
	for _, char := range s {
		// 使用range遍历，适用UTF-8编码的字符串，避免有非ASCII字符的情况

		// 4.尝试，如果字符已经出现过，并且其最后一次出现的位置在当前左指针的右侧
		if lastIndex, ok := m[char]; ok && lastIndex >= left {
			// 将左指针移动到重复字符的下一个位置
			left = lastIndex + 1
		}

		// 实际初次执行顺序
		// 1.更新字符最后一次出现的索引
		m[char] = right
		// 2.计算当前子串的长度
		length := right - left + 1
		// 3.如果当前子串的长度大于 maxLength，更新 maxLength
		if length > maxLength {
			maxLength = length
		}
		// 4.移动右指针
		right++
	}
	return maxLength
}
```



### 438. 找到字符串中所有的字母异位词

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。



**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

 **示例 2:**

```
输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
```



> 提示中仍有仅包含小写字母，但我仍用rune保证稳健性



**一解 暴力解 超时**

大概是因使用`sort.Slice()`

直接转为uint8数组排序比较

> 重复验证过了，方法雀食没错，就是性能开销大了超时

```go
	func findAnagrams(s string, p string) []int {
		subLen := len(p)
		sInt := make([]uint8, 0, len(s))
		sInt = append(sInt, []byte(s)...)
		pInt := make([]uint8, 0, subLen)
		pInt = append(pInt, []byte(p)...)
		sort.Slice(pInt, func(i, j int) bool {
			return pInt[i] < pInt[j]
		})
		res := make([]int, 0)
		for i, _ := range sInt {
			if i+subLen > len(sInt) {
				break
			}

			tmp := make([]uint8, 0, subLen)
			tmp = append(tmp, sInt[i:i+subLen]...)
			sort.Slice(tmp, func(i, j int) bool {
				return tmp[i] < tmp[j]
			})
			if string(tmp) == string(pInt) {
				res = append(res, i)
			}
		}
		return res
	}
```



**二解 滑动窗口，但是map**

能过，但性能差了，大概是因为用的map统计

> 但用的`map[rune]int`包有适用性的（性能去他妈

```go
	func findAnagrams(s string, p string) []int {
		// 先判断特殊情况
		if len(s) == 1 && len(p) == 1 {
			if s == p {
				return []int{0}
			} else {
				return []int{}
			}
		}

		pLen := len(p)
		pMap := make(map[rune]int)

		res := make([]int, 0)
		for _, v := range p {
			pMap[v]++
		}

		for i := 0; i+pLen <= len(s); i++ {
			tmpMap := make(map[rune]int)
			tmpStr := s[i : i+pLen]
			for _, v := range tmpStr {
				tmpMap[v]++
			}
			flag := false
			for k, v := range pMap {
				if tmpMap[k] != v {
					flag = false
					break
				} else {
					flag = true
					continue
				}
			}
			if flag {
				res = append(res, i)
			}
		}
		return res
	}
```



**三解 参考原题解的标准解**

极度依赖原题提示：***`s` 和 `p` 仅包含小写字母***

```go
func findAnagrams(s string, p string) []int {
	sLen, pLen := len(s), len(p)
	if sLen < pLen {
		return []int{}
	}
	if sLen == pLen && pLen == 1 {
		if s == p {
			return []int{0}
		} else {
			return []int{}
		}
	}
	//先处理特殊情况

	sCount, pCount := [26]int{}, [26]int{}
	//sCount: s的滑动窗口中每个字符出现的次数
	//pCount: p中每个字符出现的次数
	for i, v := range p {
		sCount[s[i]-'a']++
		pCount[v-'a']++
        //分别统计两个数组对应字母数量
        //因为用的"-'a'"，自然是按字母表顺序的
	}

	res := make([]int, 0)
    // 初始化答案数组
    // 尝试首次匹配
	if sCount == pCount {
		res = append(res, 0)
	}

	for i, v := range s[:sLen-pLen] {
        // 开始遍历 因要有pLen长度在，故只需滑起点在[0,sLen-pLen)空间
		sCount[v-'a']--
        // 减去离开窗口的当前字母
		sCount[s[i+pLen]-'a']++
        // 加上新进入窗口的下一字母
		if sCount == pCount {
			res = append(res, i+1)
		}
	}

	return res
}
```

