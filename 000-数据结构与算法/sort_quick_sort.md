# 快速排序

## 核心思想

如果要排序数据序列中下标从 p 到 r 之间的一组数据，我们选择 p 到 r 之间的任意一个数据作为 pivot（分区点），假设对应下标是 q。

遍历 p 到 r 之间的数据，将小于 pivot 的放到 q 左边，将大于 pivot 的放到 q 右边，将 pivot 放到中间（即 q 对应的位置）。我们将这个步骤叫做“**分区操作**”。

经过分区操作之后，数据序列 [p,r] 之间的数据就被分成了三个部分，[p,q-1] 之间都是小于 pivot 的，中间是 pivot，[q+1,r] 之间是大于 pivot 的。依此类推，**递归处理** [p,q-1] 与 [q+1,r] 两部分，**终止条件**是 p >= r。

## 分区操作图示

![快排-分区操作](./static/quick_sort.png)

## 算法特点

- 时间复杂度：O(nlgn)
- 空间复杂度：O(nlgn)
- 稳定性：不稳定

## 代码实现

```go
func QuickSort(nums []int, p int, r int) {
    // 递归终止条件
    if p >= r {
        return
    }
    q := partition(nums, p, r)
    QuickSort(nums, p, q - 1)
    QuickSort(nums, q + 1, r)
}

// 分区操作，确定分区点位置
func partition(nums []int, p int, r int) int {
    pivot := nums[r]
    // i 预标记 pivot
    i := p
    for j := p; j < r; j++ {
        if nums[j] < pivot {
            nums[i], nums[j] = nums[j], nums[i]
            i++
        }
    }

    // i 位置的值与q位置的值交换
    nums[i], nums[r] = pivot, nums[i]
    
    // 返回 i 作为 pivot 分区位置
    return i
}
```