---
title: leetcode-go-06
date: 2025-03-25 19:43:09
tags:
---

# leetcode热题100-GO-解-2025.3.25

29/100

> 当日懒得写了，题量是3.28补的（

### 234.回文链表

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/03/03/pal1linked-list.jpg)

```
输入：head = [1,2,2,1]
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/03/03/pal2linked-list.jpg)

```
输入：head = [1,2]
输出：false
```

 

解一

一眼用栈，先入后出匹配一下就行

时间复杂度O(n),空间复杂度O(n)

```go
// 234.回文链表
// 一解 空间复杂度O(n)
// 有些繁琐，索性直接懒得写，AIGC了
// 大致思路是将链表值存入栈
// 再从栈顶开始比较，直到栈为空
func isPalindrome(head *ListNode) bool {
	// 先判断特殊情况
	if head == nil || head.Next == nil {
		return true
	}
	if head.Next.Next == nil {
		return head.Val == head.Next.Val
	}
	// 初始化变量
	stack := make([]int, 0)

	// 遍历链表，将值存入栈
	curr := head
	for curr.Next != nil && curr.Next.Next != nil {
		stack = append(stack, curr.Val)
		curr = curr.Next
	}
	// 处理奇数个节点的情况
	if curr.Next == nil {
		stack = append(stack, curr.Val)
	}
	// 处理偶数个节点的情况
	if curr.Next.Next == nil {
		stack = append(stack, curr.Val)
		stack = append(stack, curr.Next.Val)
	}
	//fmt.Println(stack)
	// 从栈顶开始比较，直到栈为空
	for i := len(stack) - 1; i >= 0; i-- {
		//fmt.Printf("stack[i]: %v, curr.Val: %v, equal: %v\n", stack[i], head.Val, stack[i] == head.Val)
		if stack[i] != head.Val {
			return false
		}
		head = head.Next
	}
	return true
}
```



解二

快慢指针找中点 空间复杂度O(1)

```go
// 二解 空间复杂度O(1)
// 快慢指针找到链表中点
// 反转后半部分链表
// 比较前半部分和后半部分链表
func isPalindrome2(head *ListNode) bool {
	// 先判断特殊情况
	if head == nil || head.Next == nil {
		return true
	}
	if head.Next.Next == nil {
		return head.Val == head.Next.Val
	}

	// 找到链表中点
	slow, fast := head, head
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
	}

	// 反转后半部分链表
	var prev *ListNode
	for slow != nil {
		next := slow.Next
		slow.Next = prev
		prev = slow
		slow = next
	}

	// 比较前半部分和后半部分链表
	for prev != nil {
		if head.Val != prev.Val {
			return false
		}
		head = head.Next
		prev = prev.Next
	}
	return true
}
```



### 141.环形链表

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

*如果链表中存在环* ，则返回 `true` 。 否则，返回 `false` 。



**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist.png)

```
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png)

```
输入：head = [1,2], pos = 0
输出：true
解释：链表中有一个环，其尾部连接到第一个节点。
```

**示例 3：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png)

```
输入：head = [1], pos = -1
输出：false
解释：链表中没有环。
```



快慢指针，两者若能指向同一地址则有环，否则无环

```go
// 直接用快慢指针解，空间复杂度O(1)，时间复杂度O(n)
func hasCycle(head *ListNode) bool {
	// 先判断特殊情况
	if head == nil || head.Next == nil {
		return false
	}
	if head.Next.Next == nil {
		return false
	}
	// 初始化变量
	slow, fast := head, head
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
		if slow == fast {
			return true
		}
	}
	return false
}
```



### 142.环形链表 II

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)

```
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png)

```
输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
```

**示例 3：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png)

```
输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。
```



```go
// 直接用快慢指针解，空间复杂度O(1)，时间复杂度O(n)
func detectCycle(head *ListNode) *ListNode {
	// 初始化快慢指针
	slow, fast := head, head
	// 标记是否有环
	hasCycle := false

	// 快慢指针遍历链表
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
		// 如果快慢指针相遇，说明有环
		if slow == fast {
			hasCycle = true
			break
		}
	}

	// 如果没有环，返回 nil
	if !hasCycle {
		return nil
	}

	// 让慢指针回到头节点
	slow = head
	// 快慢指针以相同速度继续遍历，直到相遇
	for slow != fast {
		slow = slow.Next
		fast = fast.Next
	}

	// 相遇的节点即为环的入口节点
	return slow
}
```

> 设链表头节点到环的起始节点的距离为 `a`，环的起始节点到快慢指针相遇节点的距离为 `b`，相遇节点再到环的起始节点的距离为 `c`。
>
> - **快慢指针移动距离关系**：
>   - 当快慢指针相遇时，慢指针 `slow` 移动的距离为 `a + b`。
>   - 快指针 `fast` 移动的距离为 `a + b + k * (b + c)`，其中 `k` 为快指针在环内绕的圈数（`k >= 1`）。因为快指针速度是慢指针的两倍，所以快指针移动的距离也是慢指针的两倍，即 `2 * (a + b) = a + b + k * (b + c)`。
> - **等式化简**：
>   - 对 `2 * (a + b) = a + b + k * (b + c)` 进行化简，得到 `a + b = k * (b + c)`，进一步变形为 `a = (k - 1) * (b + c) + c`。
>   - 这个等式表明，从链表头节点到环的起始节点的距离 `a`，等于快指针在环内绕 `k - 1` 圈后再加上从相遇节点到环的起始节点的距离 `c`。
> - **寻找环的起始节点**：
>   - 当快慢指针相遇后，将慢指针 `slow` 重新移动到链表头节点，然后让慢指针 `slow` 和快指针 `fast` 以相同的速度（每次移动一步）继续移动。
>   - 由于 `a = (k - 1) * (b + c) + c`，当慢指针 `slow` 移动距离 `a` 到达环的起始节点时，快指针 `fast` 也刚好在环内绕了 `k - 1` 圈后再移动距离 `c`，同样到达环的起始节点，此时快慢指针再次相遇，相遇节点就是环的起始节点。



### 21.合并两个有序链表

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

> 水题，略

```go
func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
	// 先判断特殊情况
	if list1 == nil {
		return list2
	}
	if list2 == nil {
		return list1
	}
	// 初始化变量
	dummy := &ListNode{}
	curr := dummy
	// 遍历两个链表，将较小的节点加入新链表
	for list1 != nil && list2 != nil {
		if list1.Val < list2.Val {
			curr.Next = list1
			list1 = list1.Next
		} else {
			curr.Next = list2
			list2 = list2.Next
		}
		curr = curr.Next
	}
	// 将剩余的节点加入新链表
	if list1 != nil {
		curr.Next = list1
	}
	if list2 != nil {
		curr.Next = list2
	}
	return dummy.Next
}
```



### 2.两数相加

给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。



**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2021/01/02/addtwonumber1.jpg)

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```

**示例 2：**

```
输入：l1 = [0], l2 = [0]
输出：[0]
```

**示例 3：**

```
输入：l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出：[8,9,9,9,0,0,0,1]
```



**典型错误解：**

```go
// int位数有限，链表过长时会错误
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
	sum1 := 0
	sum2 := 0
	for i := 0; l1 != nil; i++ {
		sum1 += l1.Val * int(math.Pow10(i))
		l1 = l1.Next
	}
	for i := 0; l2 != nil; i++ {
		sum2 += l2.Val * int(math.Pow10(i))
		l2 = l2.Next
	}
	sum := sum1 + sum2
	// 先判断特殊情况
	if sum == 0 {
		return &ListNode{Val: 0}
	}
	// 初始化变量
	dummy := &ListNode{}
	curr := dummy
	// 从低位开始遍历，将每个数字加入新链表
	for sum != 0 {
		curr.Next = &ListNode{Val: sum % 10}
		sum /= 10
		curr = curr.Next
	}
	return dummy.Next
}
```

正解：

逐位相加，记录进位

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    dummy := &ListNode{}
    curr := dummy
    carry := 0 // 进位

    for l1 != nil || l2 != nil || carry != 0 {
        sum := carry
        if l1 != nil {
            sum += l1.Val
            l1 = l1.Next
        }
        if l2 != nil {
            sum += l2.Val
            l2 = l2.Next
        }
        carry = sum / 10
        curr.Next = &ListNode{Val: sum % 10}
        curr = curr.Next
    }

    return dummy.Next
}
```



### 19.删除链表的倒数第N个节点

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。



**示例 1：**

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

**示例 2：**

```
输入：head = [1], n = 1
输出：[]
```

**示例 3：**

```
输入：head = [1,2], n = 1
输出：[1]
```



快慢指针

指针先走N步，慢指针再开始走

快指针到达链表尾时慢指针为倒数第N个

删除操作不必多说

```go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
	// 创建一个虚拟头节点，方便处理删除头节点的情况
	dummy := &ListNode{Next: head}
	// 定义快慢指针，初始都指向虚拟头节点
	slow, fast := dummy, dummy
	// 快指针先移动 n + 1 步
	for i := 0; i <= n; i++ {
		fast = fast.Next
	}
	// 快慢指针同时移动，直到快指针到达链表末尾
	for fast != nil {
		slow = slow.Next
		fast = fast.Next
	}
	// 此时慢指针的下一个节点就是要删除的节点
	slow.Next = slow.Next.Next
	// 返回虚拟头节点的下一个节点，即新的链表头节点
	return dummy.Next
}
```



