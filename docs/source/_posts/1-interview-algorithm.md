---
title: 面试 | 数据结构与算法
date: 2020-8-29
tags: 
  - interview
  - Frontend
categories: notes
photos:
    - /blog/img/interview.jpg
---

<br>
<!--more-->

# 合并两条有序链表

```js
var mergeTwoLists  = function (l1, l2) {
    if (!l1) return l2;
    if (!l2) return l1;
    if (l1.val > l2.val) {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    } else {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    }
};
```

# 写出数组的冒泡排序与快速排序

```js
// 冒泡
var bubbleSort = function(arr) {
  for (let j = 0; j < arr.length; j++) {
    for (let i = 0; i < arr.length - j; i++) {
      if (arr[i] > arr[i + 1]) {
        [arr[i], arr[i + 1]] = [arr[i + 1], arr[i]];
      }
    }
  }
  return arr;
}

// 快排
var quickSort = function(arr) {
  if (arr.length < 2) {
    return arr
  }

  const target = arr[0]
  const left = []
  const right = []

  for (let i = 1; i < arr.length; i++) {
    if (arr[i] < target) {
      left.push(arr[i])
    } else {
      right.push(arr[i])
    }
  }

  return [...quickSort(left), target, ...quickSort(right)]
}
```

# 求两个数组的中位数

例：
nums1 = [1, 3, 4]
nums2 = [1, 2, 5, 6]

nums1 的中位数 3.0
nums2 的中位数 3.5
nums1 和 nums2 的中位数为 3.3

```js
var findMedianSortedArrays = function(nums1, nums2) {
    const arr = [...nums1, ...nums2].sort((a, b) => a - b);
    const { length } = arr;
    return length % 2 ? arr[Math.floor(length / 2)] : (arr[length / 2] + arr[length / 2 - 1]) / 2;
};

// 双指针排序法，时间复杂度为O(m + n)。
var findMedianSortedArrays = function(nums1, nums2) {
    let reIndex = nums2.length - 1;
    for (let i = nums1.length - 1; i >= 0; i--) {
        while (nums1[i] <= nums2[reIndex] && reIndex > -1) {
            nums1.splice(i + 1, 0, ...(nums2.splice(reIndex, 1)));
            reIndex--;
        }
    }
    const arr = nums2.concat(nums1);
    const { length } = arr;
    return length % 2 ? arr[Math.floor(length / 2)] : (arr[length / 2] + arr[length / 2 - 1]) / 2;
};


// 二分法 时间复杂度O(log(min(m, n)))
var findMedianSortedArrays = function(nums1, nums2) {
    // 保证num1是比较短的数组
    if (nums1.length > nums2.length) {
        [nums1, nums2] = [nums2, nums1];
    }
    const length1 = nums1.length;
    const length2 = nums2.length;
    let min = 0;
    let max = length1;
    let half = Math.floor((length1 + length2 + 1) / 2);
    while (max >= min) {
        const i = Math.floor((max + min) / 2);
        const j = half - i;
        if (i > min && nums1[i - 1] > nums2[j]) {
            max = i - 1;
        } else if (i < max && nums1[i] < nums2[j - 1]) {
            min = i + 1;
        } else {
            let left,right;
            if (i === 0) left = nums2[j - 1];
            else if (j === 0) left = nums1[i - 1];
            else left = Math.max(nums1[i - 1], nums2[j - 1]);
            
            if (i === length1) right = nums2[j];
            else if (j === length2) right = nums1[i];
            else right = Math.min(nums1[i], nums2[j]);
            
            return (length1 + length2) % 2 ? left : (left + right) / 2;
        }
    }
    return 0;
};

```

# 如何实现深拷贝

```js
// 通过 json 的方法
function deepClone(obj) {
  const obj_json = JSON.stringify(obj);
  const obj_clone = JSON.parse(obj_json);
  return obj_clone;
}

// 通过判断类型再递归生成
function deepClone(obj) {
  let objClone = Array.isArray(obj) ? [] : {};
  for(let item in obj){
    if (typeof obj[item] === "object") {
      objClone[item] = deepClone(obj[item]);
    } else {
      objClone[item] = obj[item];
    }
  }
  return objClone;
}
```

# 使用js实现斐波那契数列

```js
async fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    await curr;
    [prev, curr] = [curr, prev + curr];
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
```

# 输出斐波那契数列的第n项

```js
function Fibonacci(n)
{
    // write code here
    let [prev, curr] = [0, 1];
    for (let i = 1; i <= n; i++) {
        [prev, curr] = [curr, prev + curr];
    }
    return prev;
}
```

# 输出1-10000所有对称数字

```js
function getNumber(num) {
  const arr = [];
  for (let i = 10; i < num; i++) {
    let a = i.toString();
    let b = a.split("").reverse().join("");
    if (a === b) {
      arr.push(b)
    }
  }return arr;
}

console.log(getNumber(10000)) ;
```