---
layout: post
title: 冒泡排序算法实现
categories: Algorithm
description: 
keywords: 算法
---

# 冒泡排序

## 冒泡排序（Bubble Sort）

冒泡排序（英语：Bubble Sort）又称为泡式排序，是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

## 演示冒泡排序的整个过程

假设定义一个数组 arr = [23, 16, 51, 12, 76, 69]

第一次冒泡排序：

![](/images/posts/algorithm/Untitled.png)

可以看到，经过第一轮排序后，76这个数字已经存储在正确的位置上，相应的想要对整个数组由小到大依次排序，最多只需进行6次相同的操作即可。

冒泡排序的最终结果：

![](/images/posts/algorithm/Untitled%201.png)

实际上，当第N次冒泡排序结果未发生变化时，表示数组排序已经完成，可以提前退出排序。

## 代码实现

```go
func bubbleSort(array [6]int) [6]int {

	if len(array) == 1 {
		return array
	}

	for i := 0; i < len(array) - 1; i++ {
		var flag bool = false
		for j := 0; j < len(array) - 1 - i; j++ {
			if array[j + 1] < array[j] {
				temp := array[j]
				array[j] = array[j + 1]
				array[j + 1] = temp
				flag = true
			}
		}

		if !flag {
			break
		}
	}

	return array
	// result = [12 16 23 51 69 76]
}

```

## 复杂度分析

冒泡排序的平均时间复杂度为O(n2)，在算法执行过程中，只用到了一个临时变量temp，因此空间复杂度为O(1).
