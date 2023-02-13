# 插入排序

## 核心思想

将待排数组分为两个区间，**已排区间**和**未排区间**。初始已排区间只有一个元素，即数组第一个元素。取未排区间中的元素，在已排区间中找到**合适位置**将其插入，并保证已排区间数据一直有序。**重复**该过程，直到未排区间中的元素为空。

## 算法特点

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)
- 稳定性：稳定

## 代码实现

```go
func InsertionSort(arr []int) []int {
	if len(arr) == 1 {
		return arr
	}
	for i := 1; i < len(arr); i++ {
		v := arr[i]
		j := i - 1
		for ; j >= 0; j-- {
			if v < arr[j] {
				arr[j+1] = arr[j]
			} else {
				break
			}
		}
		arr[j+1] = v
	}
	return arr
}
```