# 队列

## 队列的定义

队列是一种操作受限的线性表，队列支持两个基本操作：入队（`enqueue`）和出队（`dequeue`）。**入队** 指将数据放入队列尾部（`tail`），**出队** 指从队列头部取数据（`head`）。符合先进先出（**FIFO**）的特性。

队列的应用非常广泛，特别是一些具有高级特性的特殊队列，如 **循环队列**、**阻塞队列**、**并发队列**，它们在很多偏低层的系统、框架、中间件的开发中，起到关键性作用。

## 顺序队列和链式队列

```go
// Queue 提供的操作接口
type Q interface {
Enqueue(any)
    Dequeue() any
    IsEmpty() bool
    Reset()
    Dump() []any
    Peek() any
}
```

### 顺序队列

- 内部数据容器依赖 go 的动态数组类型 slice。
- 为了保证线程安全，sync.RWMutex 以嵌入字段的方式组合进 Queue，因为这样调用的时候更舒服。建议：锁谁就放谁上面 ；）
- Reset() 将数据容器 items 置空。由于 slice 是“零值安全”的，所以重置后的 Queue 依然可以正常使用。

```go
// Queue built by slice
type Queue struct {
    sync.RWMutex
    items      []any
}

// Enqueue an item to the rear of queue
func (q *Queue) Enqueue(item any) {
    q.Lock()
    defer q.Unlock()

    q.items = append(q.items, item)
}

// Dequeue an item from the front of queue
func (q *Queue) Dequeue() any {
    q.Lock()
    defer q.Unlock()

    if len(q.items) == 0 {
        return nil
    }

    ret := q.items[0]
    q.items = q.items[1:]

    return ret
}

// IsEmpty return the queue is empty or not
func (q *Queue) IsEmpty() bool {
    q.RLock()
    defer q.RUnlock()

    return len(q.items) == 0
}

// Reset the container of queue's items
func (q *Queue) Reset() {
    q.Lock()
    defer q.Unlock()

    q.items = nil
}

// Dump returns all the items in the queue
func (q *Queue) Dump() []any {
    q.RLock()
    defer q.RUnlock()

    var copiedQueue = make([]any, len(q.items))
    copy(copiedQueue, q.items)

    return copiedQueue
}

// Peek the head item in the queue
func (q *Queue) Peek() any {
    q.RLock()
    defer q.RUnlock()

    if len(q.items) == 0 {
        return nil
    }

    return q.items[0]
}
```

### 链式队列

- container/list 是 go 提供的一个链表标准库

```go
// Queue built by go std library list
type Queue struct {
    sync.RWMutex
    items *list.List
}

func NewQueue() *Queue {
    return &Queue{items: list.New()}
}

// Enqueue an item to the rear of queue
func (q *Queue) Enqueue(item any) {
    q.Lock()
    defer q.Unlock()
    _ = q.items.PushBack(item)
}

// Dequeue an item from the front of queue
func (q *Queue) Dequeue() any {
    q.Lock()
    defer q.Unlock()
    if e := q.items.Front(); e != nil {
        return q.items.Remove(e)
    }
    return nil
}

// IsEmpty return the queue is empty or not
func (q *Queue) IsEmpty() bool {
    q.RLock()
    defer q.RUnlock()
    return q.items.Len() == 0
}

// Reset the container of queue's items
func (q *Queue) Reset() {
    q.Lock()
    defer q.Unlock()
    q.items = list.New()
}

// Dump returns all the items in the stack
func (q *Queue) Dump() []any {
    q.RLock()
    defer q.RUnlock()

    copiedQueue := make([]any, q.items.Len())
    for i, e := 0, q.items.Front(); e != nil; e = e.Next() {
        copiedQueue[i] = e.Value
        i++
    }
    return copiedQueue
}

// Peek the head item in the queue
func (q *Queue) Peek() any {
    q.RLock()
    defer q.RUnlock()
    if e := q.items.Front(); e != nil {
        return e.Value
    }
    return nil
}
```