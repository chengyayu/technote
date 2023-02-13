# 冒泡排序

## 核心思想

数组中前一个元素和后一个元素进行比较如果大于（或者小于）前者就进行交换，最终返回最大（或者最小）都“冒”到数组的最后。

## 算法特点

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)
- 稳定性：稳定

## 代码实现

```go
func BubbleSort(arr []int) []int {
	for i := 0; i < len(arr)-1; i++ {
		for j := i + 1; j < len(arr); j++ {
			if arr[i] > arr[j] {
				arr[i], arr[j] = arr[j], arr[i]
			}
		}
	}
	return arr
}
```