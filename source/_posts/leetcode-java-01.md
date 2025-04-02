---
title: leetcode-java-01
date: 2025-04-02 06:19:26
tags:
---

# leetcode热题100-Java解-2025.4.2

主要是复习下Java语法，顺带写下题



### 78.子集

给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。



**示例 1：**

```
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

**示例 2：**

```
输入：nums = [0]
输出：[[],[0]]
```



回溯算法

```java
	public static List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        if (nums == null || nums.length == 0){
            return result;
        }
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    private static void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> result) {
        // 每一次递归调用都将当前路径加入结果集
        result.add(new ArrayList<>(path));
        for (int i = start; i < nums.length; i++) {
            // 做出选择
            path.add(nums[i]);
            // 递归调用，继续从下一个元素开始
            backtrack(nums, i + 1, path, result);
            // 回溯，撤销选择
            path.remove(path.size() - 1);
        }
    }
```



### 35.搜索插入位置

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。



**示例 1:**

```
输入: nums = [1,3,5,6], target = 5
输出: 2
```

**示例 2:**

```
输入: nums = [1,3,5,6], target = 2
输出: 1
```

**示例 3:**

```
输入: nums = [1,3,5,6], target = 7
输出: 4
```



```java
	public static int searchInsert(int[] nums, int target) {
        int n = nums.length;
        int left = 0, right = n - 1, ans = n;
        while (left <= right) {
            int mid = ((right - left) >> 1) + left;
            if (target <= nums[mid]) {
                ans = mid;
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return ans;
    }
```



### 136.只出现过一次的数字

给你一个 **非空** 整数数组 `nums` ，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

你必须设计并实现线性时间复杂度的算法来解决此问题，且该算法只使用常量额外空间。



**示例 1 ：**

>  **输入：**nums = [2,2,1]
>
> **输出：**1

**示例 2 ：**

>  **输入：**nums = [4,1,2,1,2]
>
> **输出：**4

**示例 3 ：**

>  **输入：**nums = [1]
>
> **输出：**1



1. **交换律：a ^ b ^ c <=> a ^ c ^ b**
2. **任何数于0异或为任何数 0 ^ n => n**
3. **相同的数异或为0: n ^ n => 0**

```
var a = [2,3,2,4,4]
2 ^ 3 ^ 2 ^ 4 ^ 4等价于 2 ^ 2 ^ 4 ^ 4 ^ 3 => 0 ^ 0 ^3 => 3
```



```java
	public int singleNumber(int[] nums) {
        int single = 0;
        for (int num : nums) {
            single ^= num;
        }
        return single;
    }
```

