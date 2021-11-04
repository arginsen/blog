---
title: 前端算法系列 | 二、排序
date: 2020-6-17 8:16:41
tags: 
- Algorithm
categories: notes
photos:
    - /blog/img/algorithm.jpg
---

<br>
<!--more-->

> 默认从小到大，测试均在 leetcode 对应的数组排序题： [912. Sort an Array](https://leetcode-cn.com/problems/sort-an-array/)

# 概览

- 冒泡排序
循环数组，比较当前元素和下一个元素，如果当前元素比下一个元素大，向上冒泡。下一次循环继续上面的操作，不循环已经排序好的数。

- 选择排序
每次排序取一个最大或最小的数字放到前面的有序序列中。

- 插入排序
将左侧序列看成一个有序序列，每次将一个数字插入该有序序列。插入时，从有序序列最右侧开始比较，若比较的数较大，后移一位。

- 快速排序
选择一个目标值，比目标值小的放左边，比目标值大的放右边，目标值的位置已排好，将左右两侧再进行快排。

- 归并排序
将大序列二分成小序列，将小序列排序后再将排序后的小序列归并成大序列。

- 堆排序
创建一个大顶堆，大顶堆的堆顶一定是最大的元素。交换第一个元素和最后一个元素，让剩余的元素继续调整为大顶堆。从后往前以此和第一个元素交换并重新构建，排序完成。



# 冒泡排序

时间复杂度：O(n^2)

> - 循环数组，比较当前元素和下一个元素，如果当前元素比下一个元素大，向上冒泡。这样一次循环之后最后一个数就是本数组最大的数。下一次循环继续上面的操作，不循环已经排序好的数。

```js
// 相邻的两个元素比大小，大的放后面，下次循环排除已经排好的
function bubbleSort(array) {
  for (let j = 0; j < array.length; j++) {
    for (let i = 0; i < array.length - j; i++) { // 只排后边的
      // 比较相邻数
      if (array[i] > array[i + 1]) {
        [array[i], array[i + 1]] = [array[i + 1], array[i]];
      }
    }
  }
  return array;
}


// 优化：当一次循环没有发生冒泡，说明已经排序完成，停止循环。
function bubbleSort(array) {
  for (let j = 0; j < array.length; j++) {
    let finished = true
    for (let i = 0; i < array.length - j; i++) {
      // 比较相邻数
      if (array[i] > array[i + 1]) {
        [array[i], array[i + 1]] = [array[i + 1], array[i]];
        finished = false;
      }
    }
    if (finished) {
      return array
    }
  }
}
```

- 运行时间: 5792 ms
- 内存消耗: 42.3 MB

记录状态优化后：
- 运行时间: 6244 ms	
- 内存消耗: 42.3 MB


# 选择排序

> 每次循环选取一个最小的数字放到前面的有序序列中。

时间复杂度：O(n^2)

```js
function selectionSort(array) {
  for (let i = 0; i < array.length - 1; i++) { // 每轮的初始最小值直到倒数第二个，最后一位当然不用再比了
    let minIndex = i

    for (let j = i + 1; j < array.length; j++) {
      // 通过暂存的最小索引，往后查找不断替换为比它更小的，找到最小
      if (array[j] < array[minIndex]) {
        minIndex = j
      }
    }
    // 一轮遍历后得到最小数，与当前位互换，再进行下一位
    [array[i], array[minIndex]] = [array[minIndex], array[i]]
  }
  return array
}


// 第二次循环会不断将更小的数往前提，因此不断替换元素而不是找到索引再替换，相对上边更耗时间
function bubbleSort(array) {
  for (let i = 0; i < array.length; i++) {
    for (let j = i + 1; j < array.length; j++) {
      if (array[i] > array[j]) {
        [array[i], array[j]] = [array[j], array[i]];
      }
    }
  }
  return array;
}
```

- 运行时间: 1252 ms
- 内存消耗: 42.5 MB

- 运行时间: 5836 ms
- 内存消耗: 40.8 MB


# 插入排序

> 将左侧序列看成一个有序序列，每次将一个数字插入该有序序列。插入时，从有序序列最右侧开始比较，若比较的数较大，后移一位。

时间复杂度：O(n^2)

```js
// 从第二个开始，小于前一个值就往前移，
// 比前一个值大就原地待着，终止循环进入下一个 i
function InsertSort(array) {
  for (let i = 1; i < array.length; i++) {
    // 将用于比较的那个值的索引储存
    let compare = i
    for (let j = i - 1; j >= 0; j--) {
      if (array[compare] < array[j]) {
        [array[compare], array[j]] = [array[j], array[compare]]
        compare = j
      } else {
        break  // 终止循环
      }
    } 
  }

  return array
}
```

- 运行时间: 1940 ms
- 内存消耗: 42.5 MB


# 快速排序

> 通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据比另一部分的所有数据要小，再按这种方法对这两部分数据分别进行快速排序，整个排序过程可以**递归**进行，使整个数据变成有序序列。

时间复杂度：O(nlogn)

实现步骤：
1. 选择一个基准元素 target（一般选择第一个数）
2. 将比 target 小的元素移动到数组左边，比 target 大的元素移动到数组右边
3. 分别对 target 左侧和右侧的元素进行快速排序

```js
function quickSort(array) {
  if (array.length < 2) {
    return array
  }

  const target = array[0]
  const left = []
  const right = []

  for (let i = 1; i < array.length; i++) {
    if (array[i] < target) {
      left.push(array[i])
    } else {
      right.push(array[i])
    }
  }

  return [...quickSort(left), target, ...quickSort(right)]
}
```

- 运行时间: 124 ms
- 内存消耗: 51.5 MB








# 参考链接

[awesome-coding-js - 排序](http://www.conardli.top/docs/algorithm/#%E6%8E%92%E5%BA%8F)
[图雀社区 - 前端学习数据结构与算法](http://cn.hk.uy/pkJ)
