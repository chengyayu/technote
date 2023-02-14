# 桶排序（Bucket Sort）

## 核心思想

- 将要排序的数据 **分到几个有序的桶里**，每个桶里的数据再 **单独进行排序**。
- 每个桶内的排序可能使用 **别的排序算法** 或 **以递归方式继续使用桶排序算法** 进行排序。

## 算法描述

1. 创建 n（`n = (max-min) / bucketSize + 1`）个空桶。 
2. 将待排数组中的每个元素放到合适的桶（`buckets[idx]`）内。（`idx := (v - min) / bucketSize`）
3. 对每个桶内的元素进行排序。 
4. 连接每个桶内元素。

## 代码实现

```go
func BucketSort(arr []int, bucketSize int) []int {
    // 找到待排数据边界
    var max, min int
    for _, v := range arr {
        if v < min {
            min = v
        }
        if v > max {
            max = v
        }
    }

    // 设置空桶
    buketCount := (max-min)/bucketSize + 1
    buckets := make([][]int, buketCount)
    for i := 0; i < buketCount; i++ {
        buckets[i] = make([]int, 0)
    }

    // 分流数据到空桶
    for _, v := range arr {
        idx := (v - min) / bucketSize
        buckets[idx] = append(buckets[idx], v)
    }

    // 分别排序，合并结果
    sorted := make([]int, 0)
    for _, bucket := range buckets {
        if len(bucket) > 0 {
            // 桶内排序算法，此处使用快速排序
            QuickSort(bucket, 0, len(bucket)-1) 
            sorted = append(sorted, bucket...)
        }
    }

    return sorted
}
```

## 性能评估

如果要排序的数据有 `n` 个，我们把它们均匀地划分到 `m` 个桶内，每个桶里就有 `k=n/m` 个元素。每个桶内部使用快速排序，时间复杂度为 `O(k * logk)`。m 个桶排序的时间复杂度就是 `O(m * k * logk)`，因为 `k=n/m`，所以整个桶排序的时间复杂度就是 `O(n*log(n/m))`。当桶的个数 `m` 接近数据个数 `n` 时，`log(n/m)` 就是一个非常小的常量，这个时候桶排序的时间复杂度接近 `O(n)`。

- 最坏时间复杂度：`O(n^2)`
- 最好时间复杂度：`O(n)`
- 平均时间复杂度：`O(n)`
- 空间复杂度：`O(n+k)`
- 稳定性：`稳定`
- 适用场景：待排序数据`值域较大`但`分布比较均匀`。

## 参考资料

- [wikipedia bucket sort](https://zh.wikipedia.org/zh-hans/%E6%A1%B6%E6%8E%92%E5%BA%8F)
- [https://time.geekbang.org/column/article/42038](https://time.geekbang.org/column/article/42038)