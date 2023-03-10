# 堆排序（Heap Sort）

## 算法原理

堆排序算法，分为**建堆**和**排序**两个步骤。

### 建堆

依赖二叉堆这个数据结构，对待排数组**原地建堆**。

### 排序

以大顶堆为例，建堆结束后，数组中的数据已经是按照大顶堆的特性来组织的。数组中的第一个元素就是**堆顶**，也就是最大的元素。我们把它**跟最后一个元素交换**，那最大元素就放到了下标为 n-1 的位置。

这个过程有点类似上面讲的“删除堆顶元素”的操作，当堆顶元素移除之后，我们把下标为 n-1 的元素放到堆顶，然后再通过**堆化**的方法，**将剩下的 n−2 个元素重新构建成堆**。堆化完成之后，我们再取堆顶的元素，放到下标是 n−2 的位置，**一直重复这个过程，直到最后堆中只剩下标为 0 的一个元素**，排序工作就完成了。

## 代码实现

```go
func heapSort(arr []int) {
    l := arr
    h := &hp{sort.IntSlice(l)}
    
    // 堆化
    heap.Init(h)

    // 排序
    for {
        // 循环终止条件
        if h.Len() == 1 {
            return
        }
        // 首尾元素交换，对剩余堆向下堆化
        // 此时 Pop 前栈顶元素已经被放置在正确位置
        _ = heap.Pop(h)
    }
}

// 构建大顶堆
type hp struct {
    sort.IntSlice
}

// go heap 默认构建小顶堆
// 如果需要构建大顶堆，需要对默认 Less 结果取反
func (p *hp) Less(i, j int) bool {
    return p.IntSlice.Less(j, i)
}
func (p *hp) Push(v interface{}) {
    p.IntSlice = append(p.IntSlice, v.(int))
}

func (p *hp) Pop() interface{} {
    item := p.IntSlice[p.Len()-1]
    p.IntSlice = p.IntSlice[:p.Len()-1]
    return item
}
```

## 特点

整个堆排序的过程，都只需要极个别临时存储空间，所以堆排序是**原地排序算法**。

堆排序包括建堆和排序两个操作，建堆过程的时间复杂度是 `O(n)`，排序过程的时间复杂度是 `O(nlogn)`，所以，堆排序整体的时间复杂度是 `O(nlogn)`。

堆排序**不是稳定的排序算法**，因为在排序的过程，存在将堆的最后一个节点跟堆顶节点互换的操作，所以就有可能改变值相同数据的原始相对顺序。