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

# 反转链表 7.27

给你单链表的头节点 head ，请你反转链表，并返回反转后的链表。

输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */

// 循环
var reverseList = function(head) {
    let pre = null
    let curr = head
    while (curr) {
        let tmp = curr.next
        curr.next = pre
        pre = curr
        curr = tmp
    }
    return pre
};

// 递归
var reverseList = function(head) {
    const reverse = (pre, curr) => { // 定义前、后指针
        if (!curr) return pre
        let next = curr.next
        curr.next = pre
        return reverse(curr, next) // curr 前指针，当前指针为往前移的 next
    }

    return reverse(null, head)
}
```

# 数组中的第K个最大元素 7.30

给定整数数组 nums 和整数 k，请返回数组中第 k 个最大的元素。
请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

示例 1:

输入: [3,2,1,5,6,4] 和 k = 2
输出: 5

示例 2:

输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4

```js
// 使用js的方法
var findKthLargest = function(nums, k) {
    nums.sort((a, b) => b - a) // 按数字从小到大排序
    return nums[nums.length - 1 -k]
};

// 小顶堆


// 快速排序 + 二分
// 将数组中比第 k 个元素大的元素放在 n-k 的右边, 
var findKthLargest = function(nums, k) {
    let n = nums.length
    let left = 0, right = n - 1
    let target = n - k

    while(true) {
        let index = partition(nums, left, right) // 获取切分好的index , 左边比该值都小, 右边都大

        if (index === target) {
            return nums[index]
        } else if (index < target) {
            // index 的值偏左，说明比较小，而 index 左边的值比index还小，那么就将左边界移到 index 右
            left = index + 1
        } else {
            right = index - 1
        }
    }

    function partition(nums, left, right) {
        // 锚点的选取可以用随机数，可以一定程度避免最坏情况-------
        // 选取一个随机数作为锚点
        let anchor = Math.floor(Math.random() * (right - left + 1)) + left
        // 与left交换，比锚点小的值从最左开始累计
        swap(nums, anchor, left)
        // ---------------------------

        let pivot = nums[left] // 设定锚点, 以 p 为界限分成两波
        let j = left // 记录小于 p 的个数

        // 循环：以 p 为锚点，从下一位比起，比 p 小就累加 j，再和 j 位置的交换，
        // 那么 p 后就多了小于 p 的值，出现几个比 p 小的值，其后就有几个，直到 j 位置止步
        for (let i = left + 1; i <= right; i++) { 
            if (nums[i] < pivot) {
                j++
                swap(nums, i, j)
            }
        }

        // 循环完成后，p 点后直到第 j 位均比 p 小，那么将 p 与 j 位交换，就完成以 p 为中点的划分
        swap(nums, j, left)
        return j
    }

    function swap(nums, a, b) {
        let temp = nums[a]
        nums[a] = nums[b]
        nums[b] = temp
    }
}

```

# 无重复字符的最长子串 8.1

给定一个字符串 s ，请你找出其中不含有重复字符的 最长子串 的长度。

示例 1:

输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。

示例 2:

输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。

```js
// 使用 set 去重
var lengthOfLongestSubstring = function(s) {
    let n = s.length
    let i = j = 0
    let res = 0
    let set = new Set()

    while (i < n && j < n) {
        if (!set.has(s[j])) { // 检查当前指针是否与前边重复
            set.add(s[j++]) // 若无将下一个添加进筛选表
            res = Math.max(res, j - i)
        } else {
            check.delete(s[i++]) // 若存在重复，i前进一步继续判断
        }
    }

    return res
}

// 使用map进一步优化
var lengthOfLongestSubstring = function(s) {
    let n = s.length
    let res = 0
    let map = new Map()

    for (let j = 0, i = 0; j < n; j++) {
        if (map.has(s[j])) { // 如果map结构中有下个字符，那么就更新i的位置
            i = Math.max(map.get(s[j]), i) // 存的是字符的下一位的下标，获取的就直接是重复字符的下一位，i接着开始
        }
        res = Math.max(res, j - i + 1) // 当前的不重复子串长度与之前的对比, 下标加1才是子串长度
        map.set(s[j], j + 1) // 设置下一个j进map继续判断
        console.log(i, j, res, map)
    }

    return res
}
```

# K 个一组翻转链表 8.02

给你一个链表，每 k 个节点一组进行翻转，请你返回翻转后的链表。

k 是一个正整数，它的值小于或等于链表的长度。

如果节点总数不是 k 的整数倍，那么请将最后剩余的节点保持原有顺序

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */

// 循环 不断前进 head
var reverseKGroup = function(head, k) {
    let s = new ListNode(0, head) // 在链前加一个结点，用于固定整链
    let p = s // 设置一个 head 前的指针 p
    while(head) { // head 在整个进程中不断前进，直到指向为空
        let r = p // 定义一个用于遍历子链的指针 r
        for (let i = 0; i < k; i++) {
            r = r.next
            if (!r) return s.next // 如果 r 指针为空，直接返回整链，也就是上一个子链后的结点不够组成一个子链
        }

        let front = head // 保存当前的 head，下边 head 会不断前进

        while(head !== r) { // 当前子链前有 p 指针，子链头一直为 head，链尾结点为 r 指针，满足逆置条件
            let temp = head.next // 保留 head 后的子链，用于更新 head
            head.next = r.next // 将子链头结点提出插到 r 后
            r.next = head
            head = temp
        }
        p.next = head // 将子链拼接回来，head 不断前进，最后移动端新子链的头，也就和 r 指向一样了，r 整个过程中其实没动，在不断往下一个结点插值

        p = front // 之前的子链逆置完头结点自然也就成了尾结点，将 p 移动到下一个子链之前
        head = front.next // 更新下一个子链头结点
    }

    return s.next
};

// 递归 重复的环节是 定位子链，子链逆置，完后再接着将 head 作为一个新链再处理，直到 r 不再前几
var reverseKGroup = function(head, k) {
    let s = new ListNode(0, head)
    let p = s

    const gohead = (head, p) => {
        let r = p
        for (let i = 0; i < k; i++) {
            r = r.next
            if (!r) return s.next
        }

        let front = head // 子链第一个结点
        while(head !== r) { // 整个过程 head 是不断前进的，子链遍历完后，head 会在子链头部
            let temp = head.next
            head.next = r.next
            r.next = head
            head = temp
        }
        p.next = head

        p = front // 逆置后 front 为子链最后一个结点，也是下一个子链头结点的前一位
        head = front.next // 下一位即是子链的头结点
        gohead(head, p)
    }
    gohead(head, p)

    return s.next
};
```

# 三数之和

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。
注意：答案中不可以包含重复的三元组。

示例 1：

输入：`nums = [-1,0,1,2,-1,-4]`
输出：`[[-1,-1,2],[-1,0,1]]`

示例 2：

输入：`nums = []`
输出：`[]`

示例 3：

输入：`nums = [0]`
输出：`[]`

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var threeSum = function(nums) {

};

```

# 买卖股票的最佳时机

给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。
你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。
返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

示例 1：

输入：`[7,1,5,3,6,4]`
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
示例 2：

输入：`prices = [7,6,4,3,1]`
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。

```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function(prices) {
    let buy = prices[0], max = 0
    for (let i = 1; i < prices.length; i++) {
        if (prices[i] < buy) {
            buy = prices[i]
        } else {
            let temp = prices[i] - buy
            max = temp > max ? temp : max
        }
    }
    return max
};
```

# 环形链表 II

给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。

说明：不允许修改给定的链表。

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */

/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var detectCycle = function(head) {
    let fast = head, slow = head // 定义快慢指针
    while (true) {
        if (fast === null || fast.next === null) { // 环形链表是一直循环的
            return null
        }
        fast = fast.next.next
        slow = slow.next

        // 快慢指针会在环内追赶相遇, 要想慢针进环后与快针相遇
        // 那么快针至少在环里转了一圈，除过两者经过的相同部分：
        // 也就是慢针的路程 -- 从起点到入环点到相遇点
        // 剩下的路程则是快针走的部分，而这部分与慢针的路程相等，速度 2:1 
        // 那么快针多走的实际也就是从相遇点绕环一圈又回到相遇点
        // 这部分与慢针的公共部分也就是入环点到相遇点重复
        // 那么则可以得到相遇点到入环点的距离，与起点到入环点的距离相等
        if (fast === slow) break
    }

    fast = head // 将快指针再移到链头，慢指针还在相遇的结点
    while (slow !== fast) {
        slow = slow.next
        fast = fast.next // 此次快针就和慢针一样的步频
    }

    return fast
};
```