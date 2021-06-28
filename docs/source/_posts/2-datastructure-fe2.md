---
title: 前端数据结构 | 二、链表
date: 2020-6-13
tags: 
- dataStructure
- JavaScript
categories: notes
hide: false
photos:
    - /blog/img/dataStructure.jpg
---

<br>
<!--more-->

# 单链表

> - 链表是数据结构之一，其中的数据呈线性排列。
> - 用一组任意存储的单元来存储线性表的数据元素。一个对象存储着本身的值和下一个元素的地址。

- 优点：
添加和删除比较方便

- 缺点：
查询时速度比较慢

- 特点：
链表中的每个数据都有一个**指针**，用于指向下一个数据的内存地址；在链表中，数据一般都是**分散**存储于内存中的，无须存储在连续空间内。

- 查找数据
由于数据是分散存储，查找数据时，只能从第一个数据开始，顺着指针的指向一一往下访问(顺序访问)。

- 添加数据
添加数据时，只需要改变添加位置前后的指针指向就可以。
```
例如,a > c > d > e
现在想要在a和c之间添加b元素，将a的指针指向b，将b的指针指向c即可。
```

- 删除数据
数据的删除也一样，只需改变指针的指向就可以。
```
例如:a > b > c > d
现在想要删除b元素，只需要将a元素的指针指向c即可。 
```

# 循环链表

> 链表尾部使用指针，并将指针指向链表头部的数据，称之为循环链表

# 双向链表

> 链表里每个数据都有两个指针，并且他们分别指向前后数据，称之为双向链表。

- 优点
不仅可以从前往后，还可以从后往前遍历数据。

- 缺点
指针数的增加会导致存储空间需求增加
添加和删除数据时需要改变更多指针的指向


# 遍历链表

> **val** 属性存储当前的值，**next** 属性存储下一个节点的引用。使用头节点 head 来代表整个列表

> 要求：
> 输入：1->2->3->4->5->NULL
> 输出：[5, 4, 3, 2, 1]


```js
function TailToHead(head) {
  const array = []
  // 头节点不为 null，整个链表还有值，进行遍历存值，没值了把已存的输出
  while (head) {
    // 将链表当前值存入头部 —— 队列存储
    array.unshift(head.val)
    // 头节点往后移一个，链表缩短一个
    head = head.next
  }
  return array
}
```


# 反转链表

## 简单反转

> 要求：
> 输入：1->2->3->4->5->NULL
> 输出：5->4->3->2->1->NULL
>
> leetcode：[地址](https://leetcode-cn.com/problems/reverse-linked-list/)

### 循环

```js
// head 为 null 时，输出 null
// 初始化一条新链 newList（空链），
// 保留 head.next，用于更新 head
// 使 head 的 next 指向 newList，更新 newList 为此时的 head
// 更新 head
// 每轮下来相当于 head - 1，newList + 1，都是头部进行
// 等到 head 减的剩下 null 了，输出 newList
function reverseList(head) {
  if (!head) {
    return null
  }
  let newList = null
  while (head) {
    let next = head.next
    head.next = newList
    newList = head
    head = next
  }
  return newList
}
```

### 递归

```js
// 自递归法 -- 反转后尾接: 自递归无法存储推进状态所以无法尾递归，
// 不断将 next 放入递归方法反转链表，结果.next = 当前节点. 
// （Tip: 记得推进结果直到 next.next 为空）
function reverseList(head) {
  if (!head || !head.next) return head;
  let next = head.next; // next节点，反转后是最后一个节点
  let reverseHead = reverseList(next);
  head.next = null; // 裁减 head
  next.next = head; // 尾接
  return reverseHead;
}


// 尾递归法: 用 head 和 next 存储推进状态，直到 head 为空则输出结果
function reverseList(head) {
  const reverse = (newList, head) => {
    if (!head) return newList;
    let next = head.next;
    head.next = newList;
    return reverse(head, next); // 相当于循环里边 newList = head，head = next
  };
  return reverse(null, head);
};
```


## 区间反转

## 两个值一组反转链表

## K 个值一组反转链表


# 合并链表

## 合并两个有序链表

> 要求：
> 输入：1->2->4, 1->3->4
> 输出：1->1->2->3->4->4
> 
> leetcode：[地址](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

### 循环

```js
var mergeTwoLists = function(l1, l2) {
  if(!l1) return l2;
  if(!l2) return l1;
  let p = dummyHead = new ListNode();
  let p1 = l1, p2 = l2;
  while(p1 && p2) {
    if(p1.val > p2.val) {
      p.next = p2;
      p = p.next;
      p2 = p2.next;
    }else {
      p.next = p1;
      p = p.next;
      p1 = p1.next;
    }
  }
  // 循环完成后务必检查剩下的部分
  if(p1) p.next = p1;
  else p.next = p2;
  return dummyHead.next;
};
```


### 递归

```js
// 输入 l1 l2 两链表，比较两个链表的头部值，
// 将小节点的一条作为主链，向下递归，
// 两条链作为参数，小的一方就接着用 .next 剩下的链接着比头部值大小
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



## 合并 K 个有序链表


### 循环


# 参考链接

[leetbook - 链表](https://leetcode-cn.com/leetbook/read/linked-list/jsumh/)
[awesome-coding-js - 链表](http://www.conardli.top/docs/dataStructure/#%E9%93%BE%E8%A1%A8)
[神三元 - 前端算法系统练习指南 - 链表](http://47.98.159.95/leetcode-js/linkedlist/001.html)
[图雀社区 - 前端学习数据结构与算法](http://cn.hk.uy/pkJ)
