---
title: go 排序算法
date: 2020-08-17 13:45:07
tags:
- sort
- 排序
- golang
categories:
- interview
---

## 冒泡



```go
package main

import "fmt"

// 稳定
// 时间复杂度 O(n^2)
// 空间复杂度 O(1)

func BubbleSort(arr []int) {
	// n个元素只要走n-1趟
	n := len(arr)
	for i:=0; i<n-1; i++ {
		for j:=0; j<n-i-1; j++ {
			if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
			}
		}
	}
}

func main() {
	var arr = []int{5, 12, 1, 4, 2}
	BubbleSort(arr)
	fmt.Println(arr)
}

```

## 插入

```go
package main

import "fmt"

// 基本思想 不断地将尚未排好序的数插入到已经排好序的部分
// 稳定
// 时间复杂度 O(n^2)
// 空间复杂度 O(1)

func Insertionsort(arr []int) {
	n := len(arr)
	// 第一个认为是有序的，从第二个元素开始，一共n-1次插入
	for i:=0; i<n-1; i++ {
		for j:=i+1; j>0; j-- {
			if arr[j] < arr[j-1] {
				arr[j], arr[j-1] = arr[j-1], arr[j]
			} else {
				break
			}
		}

	}
}

func main() {
	var arr = []int{5, 12, 1, 4, 2}
	Insertionsort(arr)
	fmt.Println(arr)
}

```

## 选择

```go
package main

import "fmt"

// 基本思想 在每次遍历后，把未排序的最小的元素放到前面
// 不稳定  eg  3 2 3 1 第一个3和1交换  破坏了稳定性
// 时间复杂度O(n^2)
// 空间复杂度 O(1)

func SelectionSort(arr []int) {
	n := len(arr)
	var minIndex int = 0
	for i:=0; i<n-1; i++ {
		for j:=i+1; j<n; j++ {
			if arr[j] < arr[minIndex] {
				minIndex = j
			}
		}

		if minIndex != i {
			arr[i], arr[minIndex] = arr[minIndex], arr[i]
		}
	}
}

func main() {
	var arr = []int{5, 12, 1, 4, 2}
	SelectionSort(arr)
	fmt.Println(arr)
}

```

## 希尔

```go
package main

import "fmt"

// 基本思想 希尔排序是按照不同步长对元素进行插入排序
// 不稳定  eg  5 2 2 步长为2时，5和第二个2交换，破坏稳定性
// 时间复杂度O(n^2)
// 空间复杂度 O(1)

func ShellSort(arr []int) {
	n := len(arr)
	// 遍历间隔
	for gap:=n/2; gap>0; gap/=2 {
		// 从第gap个开始
		for i:=gap; i<n; i++ {
			// 之前的分组都要过一遍，比如gap=2时，42比较，21比较
			for j:=i; j-gap>=0 && arr[j-gap] > arr[j]; j -= gap{
				arr[j-gap], arr[j] = arr[j], arr[j-gap]
			}
		}
	}
}

func main() {
	var arr = []int{5, 4, 3, 2, 1}
	ShellSort(arr)
	fmt.Println(arr)
}

```

## 归并

```go
package main

import (
	"fmt"
)

// 基本思想 一开始先把数组从中间划分成两个子数组，一直递归地把子数组划分成更小的子数组，直到子数组里面只有一个元素，这个时候才开始排序，排序的方法就是按照大小顺序合并两个元素，接着依次按照递归的返回顺序，不断地合并排好序的子数组，直到最后把整个数组的顺序排好。
// 稳定
// 时间复杂度 O(nlogn)  进行logn层切分, 每层合并复杂度都是O(n)
// 空间复杂度 O(n)	结果需要用一个O(n)的数组

func Merge(left []int, right []int) (res []int){
	res = make([]int, 0)
	m, n := len(left), len(right)
	i, j:=0, 0

	for ; i<m && j<n;  {
		if left[i] < right[j] {
			res = append(res, left[i])
			i += 1
		} else {
			res = append(res, right[j])
			j += 1
		}
	}

	if i == m {
		res = append(res, right[j:]...)
	}
	if j == n {
		res = append(res, left[i:]...)
	}

	return
}

func MergeSort(arr []int) (res []int){
	if len(arr) < 2 {
		return arr
	}

	i := len(arr) / 2

	left := MergeSort(arr[0:i])
	right := MergeSort(arr[i:])

	result := Merge(left, right)
	return result
}

func main() {
	var arr = []int{1, 4, 5, 2, 3, 6}
	res := MergeSort(arr)
	fmt.Println(res)
}

```

## 快速排序

```go
package main

import (
	"fmt"
)

// 基本思想 一开始先把数组从中间划分成两个子数组，一直递归地把子数组划分成更小的子数组，直到子数组里面只有一个元素，这个时候才开始排序，排序的方法就是按照大小顺序合并两个元素，接着依次按照递归的返回顺序，不断地合并排好序的子数组，直到最后把整个数组的顺序排好。
// 不稳定  eg 1 2 2* 3 把2*作为key 那么2将会放在2*后
// 时间复杂度O(nlogn) logn次分解，每次都要进行n次比较  最佳情况：每次选出来的数都是中间值  最坏：数组为逆序
// 空间复杂度 O(logn)	递归次数为logn, 每次都需要额外的O(1)空间

func QuickSort(arr []int, start int, end int) {
	if start >= end {
		return
	}

	i, j := start, end
	key := arr[(start+end)/2]

	for i <= j {
		for arr[i] < key  {
			i++
		}

		for arr[j] > key {
			j--
		}

		if i <= j {
			arr[i], arr[j] = arr[j], arr[i]
			i++
			j--
		}
	}

	if start < j {
		QuickSort(arr, start, j)
	}

	if i < end {
		QuickSort(arr, i, end)
	}
}

func main() {
	var arr = []int{1, 4, 5, 2, 3, 6}
	QuickSort(arr, 0, len(arr)-1)
	fmt.Println(arr)
}

```

## 堆排序

```go
package main

import "fmt"

func HeapAdjust(arr []int, k, length int) {
	for {
		// k为父节点，i为左子节点
		i := 2*k+1
		if i >= length {
			break
		}
		// 选择左右节点中最大的, 如果没有有右节点就是左节点最大
		if i+1 < length && arr[i] < arr[i+1] {
			i++
		}
		// 父节点和最大的子节点交换
		if arr[k] < arr[i] {
			arr[k], arr[i] = arr[i], arr[k]
		}
		// 继续调整子节点
		k = i
	}
}

func HeapSort(arr []int) {
	n := len(arr)
	// 自底向上调整
	for i:=n/2; i>=0; i-- {
		HeapAdjust(arr, i, n)
	}

	for n > 0 {
		arr[n-1], arr[0] = arr[0], arr[n-1]
		n--
		HeapAdjust(arr, 0, n)
	}
}

func main() {
	var arr = []int{1, 4, 2, 3, 5, 2, 1, 24, 14, 124}
	HeapSort(arr)
	fmt.Println(arr)
}

```

