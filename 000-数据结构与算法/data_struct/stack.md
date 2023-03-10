# 栈

## 栈的定义

栈是一种操作受限的线性表，只允许在一端（也就是 **栈顶**）插入和删除数据，并且满足后进先出（**LIFO**）、先进后出（**FILO**）的特性。

## 顺序栈和链式栈

从栈的定义得出，栈主要包含两个操作：入栈（push）和出栈（pop）。入栈就是在栈顶插入一个数据，出栈就是在栈顶删除一个数据。使用 **链表** 和 **数组** 都可以实现栈。

不论是顺序栈还是链式栈，入栈和出栈仅需要操作栈顶元素，所以时间复杂度为 `O(1)`。入栈和出栈过程中仅需申请一两个临时变量，所以空间复杂度为 `O(1)`。这是针对 **容量固定** 的栈来说，如果栈支持动态扩缩容，便会对顺序栈提出更高的要求（链式栈不受影响），顺序栈依赖的数组应该换成 **动态数组**。因为扩容时需要搬移全部数据，所以最坏时间复杂度为 `O(n)`，不扩容时，时间复杂度不变，即最好时间复杂度为 `O(1)`，均摊时间复杂度为 `O(1)`。

### 顺序栈实现

- 依赖 go 语言内置的动态数组类型 slice。
- 使用 sync.RWMutex 来保证线程安全。
- any 类型（interface{} 的别名）保证可以存取任意类型的元素。

```go
// Stack built by slice
type Stack struct {
	sync.RWMutex
	items []any
}

// Push an item to the top of stack
func (q *Stack) Push(item any) {
	q.Lock()
	defer q.Unlock()

	q.items = append(q.items, item)
}

// Pop an item from the top of stack
func (q *Stack) Pop() any {
	q.Lock()
	defer q.Unlock()

	if len(q.items) == 0 {
		return nil
	}

	ret := q.items[:len(q.items)-1]
	q.items = q.items[:len(q.items)-1]

	return ret
}

// IsEmpty return the stack is empty or not
func (q *Stack) IsEmpty() bool {
	q.RLock()
	defer q.RUnlock()

	return len(q.items) == 0
}

// Reset the items of stack
func (q *Stack) Reset() {
	q.Lock()
	defer q.Unlock()

	q.items = nil
}

// Dump returns all the items in the stack
func (q *Stack) Dump() []any {
	q.RLock()
	defer q.RUnlock()

	var copiedStack = make([]any, len(q.items))
	copy(copiedStack, q.items)

	return copiedStack
}

// Peek the top item in the stack
func (q *Stack) Peek() any {
	q.RLock()
	defer q.RUnlock()

	if len(q.items) == 0 {
		return nil
	}

	return q.items[len(q.items)-1]
}
```

### 链式栈实现

- container/list 是 go 提供的一个链表标准库

```go
// Stack built by "container/list"
type Stack struct {
	sync.RWMutex
	items *list.List
}

// NewStack creates a new Stack
func NewStack() *Stack {
	return &Stack{items: list.New()}
}

// Push adds an Item to the top of the stack
func (s *Stack) Push(value any) {
	s.Lock()
	defer s.Unlock()
	s.items.PushBack(value)
}

// Pop removes an Item from the top of the stack
func (s *Stack) Pop() any {
	s.Lock()
	defer s.Unlock()
	if e := s.items.Back(); e != nil {
		s.items.Remove(e)
		return e.Value
	}
	return nil
}

func (s *Stack) Peak() any {
	s.RLock()
	defer s.RUnlock()
	if e := s.items.Back(); e != nil {
		return e.Value
	}
	return nil
}

func (s *Stack) Len() int {
	s.RLock()
	defer s.RUnlock()
	return s.items.Len()
}

func (s *Stack) Empty() bool {
	s.RLock()
	defer s.RUnlock()
	return s.items.Len() == 0
}
```

## LeetCode Problems

- [有效的括号](https://leetcode.cn/problems/valid-parentheses/)
- [最小栈](https://leetcode.cn/problems/min-stack/)
