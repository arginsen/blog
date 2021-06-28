---
title: 算法 | leetcode
date: 2020-5-1 8:12:11
tags: 
- Algorithm
- leetcode
categories: practice
photos:
    - /blog/img/algorithm.jpg
---

<br>
<!--more-->

# 两数之和

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].


>使用JS编写

```js
// 基本两个 for 循环
var twoSum = function(nums, target) {
    for (i = 0; i < nums.length; i++) {
        for (j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] === target) {
                return [i, j];
            }
        }
    }
}

// 内层循环查找差值很浪费时间 利用数组就可以减少查询时间
// 创建一个空数组 temp 用于存储
// 使用一层循环，每遍历到 nums 数组的一个元素就计算该元素与 target 之间的差值 dif
// 如果说 temp 中不存在第 dif 个元素, 
//   那么就以当前遍历的 nums 数组中第 i 个元素为索引, 当前递增的 i 为元素, 存储进 temp 数组中
//   那么也必定存在一个 dif, 与之前的 num[i] 相同
//   而此时以 dif 作为索引在 temp 数组中当然可以查到
// 因此输出之前的 i, 与现在的 i, 即 [temp[dif], i]
var twoSum = function(nums, target) {
  let temp = []
  for (let i = 0; i < nums.length; i++) {
    let dif = target-nums[i]
    if (temp[dif] !== undefined) {
      return [temp[dif], i]
    }
    temp[nums[i]] = i;    // num[0] = 2, dif = 7, temp[2] = 0; 
  }                       // num[1] = 7, dif = 2, temp[2] !== undefined
};
```

# 删除链表的倒数第n个

```js
// 
var removeNthFromEnd = function(head, n) {
    let k = head, m = head;
    // 需要考虑到长度只有1，此时k已经被赋值为null，if就不能判断k.next了
    while(--n) {
        k = k.next;
    }
    if(!k.next) return head.next;
    k = k.next;
    while(k.next) {
        k = k.next;
        m = m.next;
    }
    m.next = m.next.next;
    return head;
};


var removeNthFromEnd = function(head, n) {
    let m = head, k = head;
    while (k.next) {
        if (!n) {
            m = m.next;
        }
        n = n ? n - 1 : n;
        k = k.next;
    }
    if (n) {
        head = head.next;
    } else {
        if (m.next)
            m.next = m.next.next;
        else
            m.next = null;
    }
    return head;
};
```

# 有效的括号

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。

```js
// 采用栈的方式
var isValid = function(s) {
  // 左括号限制
  let map = {
    '(': 1,
    ')': -1,
    '[': 2,
    ']': -2,
    '{': 3,
    '}': -3
  }
  let stack = []
  for (let i = 0; i < s.length; i++) {
    if (map[s[i]] > 0) {
      stack.push(s[i])
    } else {
      let last = stack.pop()
      if (map[s[i]] + map[last] != 0) return false
    }
  }
  return stack.length === 0
};
```

# 合并两个有序链表

将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]

```js
// 采用递归的方式合并
var mergeTwoLists = function(l1, l2) {
    const merge = (l1, l2) => {
        if(!l1) return l2
        if(!l2) return l1
        if(l1.val <= l2.val) {
            l1.next = merge(l1.next, l2)
            return l1
        } else {
            l2.next = merge(l1, l2.next)
            return l2
        }
    }
    return merge(l1, l2)
};
```
