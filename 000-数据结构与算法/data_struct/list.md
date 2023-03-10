# 链表

## 链表的定义

在链表中，为了将所有节点串联起来，每个节点除了存储 **数据本身** 之外，还需要额外存储 **上一个/下一个节点的地址**。记录上一个节点地址的指针称为 **pre 指针** 或 **前继指针**，记录下一个节点地址的指针称为 **next 指针** 或 **后继指针**。

其中，有两个节点比较特殊，第一个节点被称为**头节点**，最后一个节点被称为**尾节点**。头节点用来记录链表的基地址，也就是起始地址。我们从头节点开始，可以遍历整个链表。尾节点时最后一个节点，它的 next 指针指向一个空地址。

## 内部存储结构

链表不需要一块连续的内存空间，它通过指针将一组零散的内存块（节点）串联起来使用。这样就能避免在创建数组时一次性申请过大的内存空间而导致有可能创建失败的问题。

## 几种形态结构

- 单向链表：节点存储 **数据本身** 外，额外存储一个 **next 指针**。
- 双向链表：节点存储 **数据本身** 外，额外存储一个 **pre 指针** 和 **next 指针**。
- 循环链表：**尾节点** 的 **next 指针** 指向 **头节点**。
- 双向循环链表：**尾节点** 的 **next 指针** 指向 **头节点**，**头节点** 的 **pre 指针** 指向 **尾节点**。

## 基本操作

### 定义

```go
// SingleLinkedList
type SingleLinkedList struct {
	head   *Node
	length uint
}

type Node struct {
	next *Node
	val  int
}
```

```go
// DoubleLinkedList
type DoubleLinkedList struct {
	head   *Node
	length int
}

type Node struct {
	Val  int
	Pre  *Node
	Next *Node
}
```

### 删除

在实际的软件开发中，从链表中删除一个数据无外乎下面两种情况。

- 删除“值等于给定值”的节点
- 删除给定指针指向的节点

对于第一种情况，无论单链表还是双链表，为了查找值等于给定值的节点，我们都需要从链表的头节点开始依次遍历并对比，直到找到值等于给定值的节点，然后将其删除。尽管删除操作时间复杂度是 `O(1)`，但是遍历查找是主要的耗时点，对应的时间复杂度为 `O(n)`。因此，无论单链表还是双链表，「删除“值等于给定值”的节点」对应的时间复杂度为 `O(n)`。

```go
// Remove from SingleLinkedList by val
func (this *SingleLinkedList) Remove(val int) {
	q := this.head
	var p *Node

	for q != nil && q.val != val {
		p = q
		q = q.next
	}

	if q != nil { // got it
		if p == nil {
			this.head = q.next
		} else {
			p.next = q.next
		}
		this.length -= 1
	}
}
```

```go
// Remove from DoubleLinkedList by val
func (this *DoubleLinkedList) Remove(val int) {
	q := this.head
	
    for q != nil && q.Val != val {
		q = q.Next
	}
	
    if q != nil { // got it
		if q.Pre == nil {
			this.head = q.Next
		} else {
			q.Pre.Next = q.Next
		}
        this.length -= 1
	}
}
```

对于第二种情况，已知要被删除的节点，但是删除节点 q 需要借助前驱节点，而单链表无法直接获取前驱节点，因此，为了找到节点 q 的前驱节点，我们需要从头遍历链表，直到 `p.next = q` 为止。说明 p 是 q 的前驱节点。但是对于双向链表来说，节点中已经保存了前驱节点的指针，无需遍历。因此，针对「删除给定指针指向的节点」，单链表的时间复杂度是 `O(n)`，双向链表的时间复杂度是 `O(1)`。

```go
// Remove from SingleLinkedList by node pointer
func (this *SingleLinkedList) Remove(q *Node) {
	if q == nil {
		return
	}
	if this.head == q {
		this.head = q.next
		return
	}
	p := this.head
	for p.next != nil && p.next != q {
		p = p.next
	}
	if p.next != nil {
		p.next = q.next
		this.length -= 1
	}
}
```

```go
// Remove from DoubleLinkedList by node pointer
func (this *DoubleLinkedList) Remove(q *Node) {
	if q == nil {
		return
	}
	if this.head == q {
		this.head = q.Next
		return
	}
	q.Next.Pre = q.Pre
	q.Pre.Next = q.Next
	this.length -= 1
}
```

同理，对于在某个指定节点前插入一个节点这样的操作，双向链表比单链表也更有优势。双向链表的时间复杂度是 `O(1)`，而单链表需要 `O(n)` 时间复杂度。

实际的软件开发中，双向链表虽然会占用更多的内存，但比单链表的应用更加广泛。原因就在于双向链表可以快速找到某个节点的前驱节点，支持双向遍历。

## 链表与数组的性能对比

链表和数组是两种截然不同的内存组织方式。实际软件开发中，数据结构的选择需要多方面的考虑。

数组使用连续的内存空间来存储数据。可以有效的利用 CPU 的缓存机制，预读数组中的数据，提高访问效率。而链表在内存中不是连续存储的，因此，无法预读，对 CPU 缓存不友好。

数组大小固定，一经声明就要占用整块的连续内存空间，无法更改。声明的数组过大，可能会导致“内存不足”（out of memory）异常。如果声明的过小，就会出现不够用的问题，需要扩容，申请一个更大的空间，将原数组中的数据都复制过去，非常耗时。而链表恰恰相反，天然支持动态扩容。

## 各项操作时间复杂度

| 操作 | 时间复杂度 | 备注 |
| --- | --- |--- |
| prepend | O(1) ||
| append | O(1) ||
| lookup | O(n) |在有序的前提下，可以通过升维成跳表，解决链表的查找性能问题|
| insert | O(1) ||
| delete | O(1) ||

## LeetCode Problems

- [反转链表](https://leetcode.cn/problems/reverse-linked-list/)
- [两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)
- [环形链表](https://leetcode.cn/problems/linked-list-cycle/)
- [环形链表 II 环的入口点](https://leetcode.cn/problems/linked-list-cycle-ii/)
- [K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)
- [合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)
- [删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
- [删除链表的中间节点](https://leetcode.cn/problems/delete-the-middle-node-of-a-linked-list/)
- [寻找链表的中间节点](https://leetcode.cn/problems/middle-of-the-linked-list/)