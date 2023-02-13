# 快速排序

## 核心思想

如果要排序数据序列中下标从 start 到 end 之间的一组数据，我们选择 start 到 end 之间的任意一个数据作为 pivot（分区点），假设对应下标是 p。

遍历 start 到 end 之间的数据，将小于 pivot 的放到 p 左边，将大于 pivot 的放到 p 右边，将 pivot 放到中间（即 p 对应的位置）。我们将这个步骤叫做“**分区操作**”。

经过分区操作之后，数据序列 `[start,end]` 之间的数据就被分成了三个部分，`[start,p-1]` 之间都是小于 pivot 的，中间是 `pivot`，`[p+1,end]` 之间是大于 pivot 的。依此类推，**递归处理** `[start,p-1]` 与 `[p+1,end]` 两部分，**终止条件**是 `start >= end`。

## 分区操作图示

![快排-分区操作](./static/quick_sort.png)

## 算法特点

- 时间复杂度：`O(nlgn)`
- 空间复杂度：`O(nlgn)`
- 稳定性：`不稳定`

## 代码实现

```go
func QuickSort(nums []int, start int, end int) {
    // 递归终止条件
    if start >= end {
        return
    }
    pivot := partition(nums, start, end)
    QuickSort(nums, start, pivot - 1)
    QuickSort(nums, pivot + 1, end)
}

// 分区操作，确定分区点位置
func partition(nums []int, start int, end int) int {
    pivot := nums[end]
    
    // i 左侧都是小于 pivot 的值
    i := start
    for j := start; j < end; j++ {
        if nums[j] < pivot {
            nums[i], nums[j] = nums[j], nums[i]
            i++
        }
    }

    // i 与 pivot 交换值
    nums[i], nums[end] = pivot, nums[i]
    
    // 返回 i 作为 pivot 分区位置
    return i
}
```