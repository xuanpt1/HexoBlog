---
title: leetcode-java-04
date: 2025-04-07 13:17:53
tags:
---

# leetcode热题100-Java解-2025.4.7

69/100

### 74. 搜索二维矩阵

给你一个满足下述两条属性的 `m x n` 整数矩阵：

- 每行中的整数从左到右按非严格递增顺序排列。
- 每行的第一个整数大于前一行的最后一个整数。

给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。



示例略



写完前面的这题纯水题

从右上或左下入手就是个二叉搜索树

解一 dfs

```java
	public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length;
        int n = matrix[0].length;
        return dfs(matrix, target, m - 1, 0);
    }

    private boolean dfs(int[][] matrix, int target, int i, int j) {
        if (i < 0 || i >= matrix.length || j < 0 || j >= matrix[0].length) {
            return false;
        }
        if (matrix[i][j] == target) {
            return true;
        }
        if (matrix[i][j] > target) {
            return dfs(matrix, target, i - 1, j);
        }else {
            return dfs(matrix, target, i, j + 1);
        }
    }
```



解二 二分查找

针对这题可以使用

题目保证了本行的每个元素都大于上一行的任一元素

先排查行，再二分

```java
	public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length;
        int n = matrix[0].length;
        int i = m - 1;
        int j = 0;
        while( target < matrix[i][j] ){
            i--;
            if( i < 0 ){
                return false;
            }
        }

        int k = n - 1;
        //二分查找
        while( j <= k ){
            int mid = (j + k) / 2;
            if( matrix[i][mid] == target ){
                return true;
            }else if( matrix[i][mid] < target ){
                j = mid + 1;
            }else{
                k = mid - 1;
            }
        }
        return false;
    }
```



### 34. 在排序数组中查找元素的第一个和最后一个位置

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。



**示例 1：**

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

**示例 2：**

```
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]
```

**示例 3：**

```
输入：nums = [], target = 0
输出：[-1,-1]
```



解一 递归

我是递归高手

```java
	// 简单的说就是找到0的index，k = length - index
    // 然后将数组分为两个部分，分别进行二分查找
    // ...下标从0开始计数，但是数组就不一定了........
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        int index = 0;
        if (nums.length == 1) {
            return nums[0] == target ? 0 : -1;
        }

        index = findIndex(nums, left, right);

        int res1 = binarySearch(nums, target, 0, index - 1);
        int res2 = binarySearch(nums, target, index, nums.length - 1);

        return res1 == res2 || res1 != -1 ? res1 : res2;
    }

    private int findIndex(int[] nums, int left, int right) {
        if (left > right) {
            return -1;
        }
        int mid = (left + right) / 2;
        if (nums[mid] > nums[right]) {
            if(right - left == 1) {
                return right;
            }
            // 左边大于右边，继续递归
            return findIndex(nums, mid, right);
        }else if( nums[mid] < nums[right]) {
            //左边小于右边，开始寻找升序起点
            int begin = nums[mid];
            for (int i = mid; i >= 1; i--) {
                if (nums[i] < nums[i - 1]) {
                    return i;
                }
            }
            return -1;
        }else {
            return -1;
        }
    }

    private int binarySearch(int[] nums, int target, int left, int right) {
        if (left > right) {
            return -1;
        }
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            return mid;
        }else if (nums[mid] > target) {
            if (mid >= 1) {
                return binarySearch(nums, target, left, mid - 1);
            }
        }else {
            if (mid <= nums.length - 1) {
                return binarySearch(nums, target, mid + 1, right);
            }
        }
        return -1;
    }
```



解二 官方解

相对来看，我的解法等于把数组遍历了两遍，第一遍区分有序部分，第二遍二分查找

O(2n) ~ O(n)

大致是这个效果

```java
	public int search(int[] nums, int target) {
        // 获取数组的长度
        int n = nums.length;
        // 如果数组为空，直接返回 -1
        if (n == 0) {
            return -1;
        }
        // 如果数组只有一个元素，检查该元素是否等于目标值
        if (n == 1) {
            return nums[0] == target ? 0 : -1;
        }
        
        // 初始化左指针为数组的起始位置
        int l = 0, r = n - 1;
        // 使用二分查找算法
        while (l <= r) {
            // 计算中间位置
            int mid = (l + r) / 2;
            // 如果中间元素等于目标值，返回中间位置的索引
            if (nums[mid] == target) {
                return mid;
            }
            
            // 判断左半部分是否有序
            if (nums[0] <= nums[mid]) {
                // 左半部分有序
                if (nums[0] <= target && target < nums[mid]) {
                    // 如果目标值在左半部分的有序区间内，缩小右边界
                    r = mid - 1;
                } else {
                    // 否则，目标值在右半部分，缩小左边界
                    l = mid + 1;
                }
            } else {
                // 左半部分无序，说明右半部分有序
                if (nums[mid] < target && target <= nums[n - 1]) {
                    // 如果目标值在右半部分的有序区间内，缩小左边界
                    l = mid + 1;
                } else {
                    // 否则，目标值在左半部分，缩小右边界
                    r = mid - 1;
                }
            }
        }
        // 未找到目标值，返回 -1
        return -1;
    }
```



### 153. 寻找旋转排序数组中的最小值

已知一个长度为 `n` 的数组，预先按照升序排列，经由 `1` 到 `n` 次 **旋转** 后，得到输入数组。例如，原数组 `nums = [0,1,2,4,5,6,7]` 在变化后可能得到：

- 若旋转 `4` 次，则可以得到 `[4,5,6,7,0,1,2]`
- 若旋转 `7` 次，则可以得到 `[0,1,2,4,5,6,7]`

注意，数组 `[a[0], a[1], a[2], ..., a[n-1]]` **旋转一次** 的结果为数组 `[a[n-1], a[0], a[1], a[2], ..., a[n-2]]` 。

给你一个元素值 **互不相同** 的数组 `nums` ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 **最小元素** 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。



**示例 1：**

```
输入：nums = [3,4,5,1,2]
输出：1
解释：原数组为 [1,2,3,4,5] ，旋转 3 次得到输入数组。
```

**示例 2：**

```
输入：nums = [4,5,6,7,0,1,2]
输出：0
解释：原数组为 [0,1,2,4,5,6,7] ，旋转 4 次得到输入数组。
```

**示例 3：**

```
输入：nums = [11,13,15,17]
输出：11
解释：原数组为 [11,13,15,17] ，旋转 4 次得到输入数组。
```



解一  递归，复用上一题写的`findIndex`

```java
	// 刚写完上一题，吓老子一跳
    // 上一题是根据坐标k旋转，对应的这题就是差不多旋转k次
    // 没啥差别，直接复用上一题的findIndex方法
    // index位置就是最小值
    public int findMin(int[] nums) {

        int index = findIndex(nums, 0, nums.length - 1);
        if (index == -1) {
            return nums[0];
        }
        return nums[index];
    }
    private int findIndex(int[] nums, int left, int right) {
        if (left > right) {
            return -1;
        }
        int mid = (left + right) / 2;
        if (nums[mid] > nums[right]) {
            if(right - left == 1) {
                return right;
            }
            // 左边大于右边，继续递归
            return findIndex(nums, mid, right);
        }else if( nums[mid] < nums[right]) {
            //左边小于右边，开始寻找升序起点
            int begin = nums[mid];
            for (int i = mid; i >= 1; i--) {
                if (nums[i] < nums[i - 1]) {
                    return i;
                }
            }
            return -1;
        }else {
            return -1;
        }
    }
```



解二  官方解，迭代

```java
	public int findMin(int[] nums) {
        // 初始化左指针，指向数组的起始位置
        int low = 0;
        // 初始化右指针，指向数组的末尾位置
        int high = nums.length - 1;
        // 当左指针小于右指针时，继续循环
        while (low < high) {
            // 计算中间位置，避免整数溢出
            int pivot = low + (high - low) / 2;
            // 如果中间元素小于右指针指向的元素，说明最小值在左半部分
            if (nums[pivot] < nums[high]) {
                // 缩小右边界到中间位置
                high = pivot;
            } else {
                // 否则，最小值在右半部分
                // 缩小左边界到中间位置的下一个位置
                low = pivot + 1;
            }
        }
        // 当 low 和 high 相遇时，该位置即为最小值所在位置
        return nums[low];
    }
```

