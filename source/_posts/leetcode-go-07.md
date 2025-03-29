---
title: leetcode-go-07
date: 2025-03-28 23:08:33
tags:
---

# leetcode热题100-GO-解-2025.3.29

35/100

剩余主题：

- 二叉树
- 图论
- 回溯
- 二分查找
- 栈
- 堆
- 贪心算法
- 动态规划
- 多维动态规划
- 技巧

占位

- ~~3.26应有 刷题 面试总结~~

- 3.27应有 刷题 java复习

- 3.28应有 JavaSSM复习 芋道微服务框架学习 毕设过程笔记



### 24.两两交换链表中的节点

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/swap_ex1.jpg)

```
输入：head = [1,2,3,4]
输出：[2,1,4,3]
```

**示例 2：**

```
输入：head = []
输出：[]
```

**示例 3：**

```
输入：head = [1]
输出：[1]
```



水题

解一 递归

```go
func swapPairs(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
    // 递归结束条件
	next := head.Next
	head.Next = swapPairs(next.Next)
	next.Next = head
    // 两两互换，调用递归
	return next
}
```



解二 迭代

```go
func swapPairs2(head *ListNode) *ListNode {
	dummy := &ListNode{0, head}
	pre := dummy
	for head != nil && head.Next != nil {
		first := head
		second := head.Next
        // pre -> first -> second -> ...
        
		pre.Next = second
		first.Next = second.Next
		second.Next = first
        // pre -> second -> first -> ...
        
		pre = first
		head = first.Next
        // 更新迭代
	}
	return dummy.Next
}
```



### 25.K个一组翻转链表

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex1.jpg)

```
输入：head = [1,2,3,4,5], k = 2
输出：[2,1,4,3,5]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex2.jpg)

```
输入：head = [1,2,3,4,5], k = 3
输出：[3,2,1,4,5]
```



思路：

直接对每n个为一段，执行翻转

最后一次计数达不到n时直接返回

共用辅助函数 翻转链表：

```go
// 辅助函数：翻转链表
// 反转从 head 到 tail 之间的节点
// 不包含 tail 节点
// @return 反转后的头节点
func reverseListReturnHead(head, tail *ListNode) *ListNode {
	var prev *ListNode
	cur := head
	for cur != tail {
		next := cur.Next
		cur.Next = prev
		prev = cur
		cur = next
	}
	return prev
}
```



解一 递归

```go
func reverseKGroup(head *ListNode, k int) *ListNode {
	// 检查是否有足够的节点
	cur := head
	for i := 0; i < k; i++ {
		if cur == nil {
			return head
		}
		cur = cur.Next
	}
	// 翻转前 k 个节点
	newHead := reverseListReturnHead(head, cur)
	// 递归翻转剩余的节点
	head.Next = reverseKGroup(cur, k)
	return newHead
}
```



解二 迭代

```go
func reverseKGroup2(head *ListNode, k int) *ListNode {
	dummy := &ListNode{0, head}
	pre := dummy
	for head != nil {
		tail := pre
		for i := 0; i < k; i++ {
			tail = tail.Next
			if tail == nil {
				return dummy.Next
			}
		}
		next := tail.Next
		newHead := reverseListReturnHead(head, next) // 反转从 head 到 tail.Next 之间的节点
		pre.Next = newHead                           // pre 指向反转后链表的头节点
		head.Next = next                             // 反转后原 head 变为尾节点，连接到 next
		pre = head                                   // pre 移动到下一组的前一个节点
		head = next                                  // head 移动到下一组的头节点
	}
	return dummy.Next
}
```



### 138.随机链表的复制

给你一个长度为 `n` 的链表，每个节点包含一个额外增加的随机指针 `random` ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 **[深拷贝](https://baike.baidu.com/item/深拷贝/22785317?fr=aladdin)**。 深拷贝应该正好由 `n` 个 **全新** 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 `next` 指针和 `random` 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。**复制链表中的指针都不应指向原链表中的节点** 。

例如，如果原链表中有 `X` 和 `Y` 两个节点，其中 `X.random --> Y` 。那么在复制链表中对应的两个节点 `x` 和 `y` ，同样有 `x.random --> y` 。

返回复制链表的头节点。

用一个由 `n` 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 `[val, random_index]` 表示：

- `val`：一个表示 `Node.val` 的整数。
- `random_index`：随机指针指向的节点索引（范围从 `0` 到 `n-1`）；如果不指向任何节点，则为 `null` 。

你的代码 **只** 接受原链表的头节点 `head` 作为传入参数。



**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e1.png)

```
输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e2.png)

```
输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]
```

**示例 3：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e3.png)**

```
输入：head = [[3,null],[3,0],[3,null]]
输出：[[3,null],[3,0],[3,null]]
```



关键点在于对于复制出的链表节点如何复制他们的对应关系

解一 哈希

保存原节点 -> 复制节点的映射

```go
func copyRandomList(head *Node) *Node {
	if head == nil {
		return nil
	}
	// 用于存储原节点和新节点的映射关系
	nodeMap := make(map[*Node]*Node)

	// 第一次遍历，创建新节点并存储映射关系
	cur := head
	for cur != nil {
		nodeMap[cur] = &Node{Val: cur.Val}
		cur = cur.Next
	}

	// 第二次遍历，为新节点的 Next 和 Random 指针赋值
	cur = head
	for cur != nil {
		if cur.Next != nil {
			nodeMap[cur].Next = nodeMap[cur.Next]
		}
		if cur.Random != nil {
			nodeMap[cur].Random = nodeMap[cur.Random]
		}
		cur = cur.Next
	}

	return nodeMap[head]
}
```



解二 迭代

通过将复制出的新节点直接插入到原节点的后面

再复制对应的random关系，最后拆分链表

从而实现了空间复杂度为 O(1) 的解法

```go
func copyRandomList2(head *Node) *Node {
	if head == nil {
		return nil
	}
	// 第一次遍历，复制节点并插入到原节点的后面
	cur := head
	for cur != nil {
		newNode := &Node{Val: cur.Val, Next: cur.Next}
		cur.Next = newNode
		cur = newNode.Next
	}

	// 第二次遍历，设置新节点的 Random 指针
	cur = head
	for cur != nil {
		if cur.Random != nil {
			cur.Next.Random = cur.Random.Next
		}
		cur = cur.Next.Next
	}

	// 第三次遍历，拆分链表
	cur = head
	newHead := head.Next
	newCur := newHead
	for cur != nil {
		cur.Next = newCur.Next
		cur = cur.Next
		if cur != nil {
			newCur.Next = cur.Next
			newCur = newCur.Next
		}
	}
	return newHead
}
```



### 148.排序链表

给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/09/14/sort_list_1.jpg)

```
输入：head = [4,2,1,3]
输出：[1,2,3,4]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/09/14/sort_list_2.jpg)

```
输入：head = [-1,5,3,4,0]
输出：[-1,0,3,4,5]
```

**示例 3：**

```
输入：head = []
输出：[]
```



常用的冒泡 插入时间上都无法满足

考虑归并处理

解一 递归 自顶向下

> 感觉这个方式好理解些

`mergeTwoList(left,right)`为之前写过的合并有序链表操作

```go
if head == nil || head.Next == nil {
		return head
	}
	// 快慢指针找到链表的中间节点
	slow, fast := head, head.Next
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
	}
	// 断开链表
	mid := slow.Next
	slow.Next = nil
	// 递归排序左右子链表
	left := sortList(head)
	right := sortList(mid)

	// 合并排序后的左右子链表
	return mergeTwoLists(left, right)
}
```



解二 迭代 自底向上

```go
func sortList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	// 计算链表长度
	length := 0
	cur := head
	for cur != nil {
		length++
		cur = cur.Next
	}
	// 创建一个虚拟头节点
	dummy := &ListNode{0, head}
    
	// 自底向上合并子链表
    // subLength 子链表最长长度
	for subLength := 1; subLength < length; subLength <<= 1 {
		prev, cur := dummy, dummy.Next
        
        //处理完一轮后，subLength长度限制翻倍，prev和cur归位
        //每轮循环后，相当于以subLenth长度为单元进行排序并合并
        
        // 关键点在于内层循环
		for cur != nil {
			// 找到第一个子链表的头节点
			head1 := cur
            
			for i := 1; i < subLength && cur.Next != nil; i++ {
				cur = cur.Next
			}
			// 找到第二个子链表的头节点
            // 即第subLen个
			head2 := cur.Next
            
			cur.Next = nil // 断开第一个子链表和第二个子链表的连接
            
			cur = head2
			for i := 1; i < subLength && cur != nil && cur.Next != nil; i++ {
				cur = cur.Next
			}
			// 断开第二个子链表和后面的连接
			var next *ListNode
			if cur != nil {
				next = cur.Next
				cur.Next = nil
			}
			// 合并两个子链表
			prev.Next = mergeTwoLists(head1, head2)
			// 移动到合并后链表的末尾
			for prev.Next != nil {
				prev = prev.Next
			}
			cur = next // 移动到下**一对**子链表的头节点
		}

	}
	return dummy.Next
}
```

### 23.合并K个升序链表

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

示例 1：

```
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
1->4->5,
1->3->4,
2->6
]
将它们合并到一个有序链表中得到。
1->1->2->3->4->4->5->6
```

示例 2：

```
输入：lists = []
输出：[]
```

示例 3：

```
输入：lists = [[]]
输出：[]
```

解法： **最小堆**
```go
// 定义最小堆
type MinHeap []*ListNode

func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i].Val < h[j].Val }
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *MinHeap) Push(x interface{}) {
    *h = append(*h, x.(*ListNode))
}

func (h *MinHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}

func mergeKLists(lists []*ListNode) *ListNode {
    // 先排除特况
	if len(lists) == 0 {
		return nil
	}
	// 初始化小顶堆
	h := &MinHeap{}
	heap.Init(h)
	for _, head := range lists {
		if head != nil {
			heap.Push(h, head)
		}
	}

	// 创建一个虚拟头节点
	dummy := &ListNode{}
	curr := dummy
	// 从堆中取出最小的节点，将其加入到结果链表中
	for h.Len() > 0 {
		node := heap.Pop(h).(*ListNode)
		curr.Next = node
		curr = curr.Next
		if node.Next != nil {
			heap.Push(h, node.Next)
		}
	}
	return dummy.Next
}
```

### 146.LRU 缓存
请你设计并实现一个满足**LRU(最近最少使用)缓存**约束的数据结构。

实现`LRUCache`类：

`LRUCache(int capacity)`以 正整数 作为容量 capacity 初始化 LRU 缓存

`int get(int key)` 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。

`void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该*逐出*最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

示例：
```
输入:
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]

输出:
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

解法 hashmap+双向链表
map用于建立索引，双向链表用于建立FIFO队列
```go
/ LRUCache 是一个LRU缓存的实现
type LRUCache struct {
    size       int
    evictList  *DoublyLinkedList
    items      map[int]*DoublyLinkedListNode
}

// DoublyLinkedList 是一个双向链表
type DoublyLinkedList struct {
    head *DoublyLinkedListNode
    tail *DoublyLinkedListNode
}

// DoublyLinkedListNode 是双向链表的节点
type DoublyLinkedListNode struct {
    key  int
    val  int
    prev *DoublyLinkedListNode
    next *DoublyLinkedListNode
}

// Constructor 创建一个新的LRU缓存
// Constructor 创建一个新的LRU缓存
func Constructor(size int) LRUCache {
    return LRUCache{
        size:       size,
        evictList:  &DoublyLinkedList{},
        items:      make(map[int]*DoublyLinkedListNode),
    }
}

// Put 向缓存中添加一个元素
func (c *LRUCache) Put(key int, value int) {
    // 如果元素已经存在，更新值并移动到链表头部
    if node, ok := c.items[key]; ok {
        node.val = value
        c.evictList.moveToFront(node)
        return
    }

    // 如果缓存已满，移除最久未使用的元素
    if len(c.items) == c.size {
        removedKey := c.evictList.tail.key
        c.evictList.removeNode(c.evictList.tail)
        delete(c.items, removedKey)
    }

    // 创建新节点并添加到链表头部和哈希表中
    node := &DoublyLinkedListNode{key: key, val: value}
    c.evictList.addNode(node)
    c.items[key] = node
}

// Get 从缓存中获取一个元素
func (c *LRUCache) Get(key int) int {
    if node, ok := c.items[key]; ok {
        // 访问后将节点移动到链表头部
        c.evictList.moveToFront(node)
        return node.val
    }
    return -1 // 如果元素不存在，返回-1
}

// removeNode 从双向链表中移除一个节点
func (l *DoublyLinkedList) removeNode(node *DoublyLinkedListNode) {
    if node == l.head {
        l.head = node.next
    }
    if node == l.tail {
        l.tail = node.prev
    }
    if node.prev != nil {
        node.prev.next = node.next
    }
    if node.next != nil {
        node.next.prev = node.prev
    }
    node.prev = nil
    node.next = nil
}

// addNode 在双向链表的头部添加一个节点
func (l *DoublyLinkedList) addNode(node *DoublyLinkedListNode) {
    node.next = l.head
    node.prev = nil
    if l.head != nil {
        l.head.prev = node
    }
    l.head = node
    if l.tail == nil {
        l.tail = node
    }
}

// moveToFront 将一个节点移动到双向链表的头部
func (l *DoublyLinkedList) moveToFront(node *DoublyLinkedListNode) {
    l.removeNode(node)
    l.addNode(node)
}
```
