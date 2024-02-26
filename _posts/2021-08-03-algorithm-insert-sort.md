---
layout: post
title: 插入排序算法实现
categories: Algorithm
description:
keywords: 算法
---

# 插入排序

## 插入排序（Insertion Sort）

插入排序是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序在实现上，通常采用in-place排序（即只需用到O(1)的额外空间的排序），因而在从后：向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。

## 分析插入排序的过程

假定数组arr = [23, 69, 51, 12, 76, 16],从第二个元素开始向前比较。

![](/images/posts/algorithm/insert_1.png)

插入排序过程中，只需要用到常量级的存储空间，因此空间复杂度为O(1)。元素每次都需要从后向前逐个扫描对比，平均时间复杂度为O(n2).

## 代码实现

```go
func insertionSort(array [6]int) [6]int {
    if len(array) == 1 {
        return array
    }

    for i := 1; i < len(array); i++ {
        value := array[i]
        j := i - 1
        for ; j >= 0; j-- {
            if array[j] > value {
                array[j+1] = array[j]
            } else {
                break
            }
        }

        array[j + 1] = value
    }

    return array
}
```