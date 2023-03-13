# 二分查找算法

二分查找算法作用于**有序数组**。查找思想有点类似**分治思想**。每次都通过跟区间的中间元素对比，将待查找的区间缩小为之前的一半，直到找到要查找的元素，或者区间被缩小为空。查找某个元素的时间复杂度为 `O(logn)`。

## 应用条件

1. 有序性：保证每次去半的特性。
2. 数组：按照下标寻址的时间复杂度 O(1)。（链表不行，寻址时间复杂度为 O(n)）

## 代码模版

```go
func search(nums []int, target int) int {
    left, right := 0, len(nums)-1
    // 1. 终止条件（left == right 是闭区间，还有一个元素，需要和 target 比较）
    for left <= right {
        // 2. 中间值选择（防止溢出）
        mid := left + (right-left)/2
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            // 3. 区间边界的更新（防止陷入死循环）
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return -1
}
```