---
title: 字节编程题
date: 2020-9-16
tags: 
  - interview
  - Frontend
categories: practice
hide: true
photos:
    - /blog/img/interview.jpg
---

<br>
<!--more-->


# 用户喜好

> 输入描述:
> 输入： 第1行为n代表用户的个数 第2行为n个整数，第i个代表用户标号为i的用户对某类文章的喜好度 第3行为一个正整数q代表查询的组数  第4行到第（3+q）行，每行包含3个整数l,r,k代表一组查询，即标号为l<=i<=r的用户中对这类文章喜好值为k的用户的个数。 数据范围n <= 300000,q<=300000 k是整型
>
> 输出描述:
> 输出：一共q行，每行一个整数代表喜好值为k的用户的个数
>
> 输入例子1:
> 5
> 1 2 3 3 5
> 3
> 1 2 1
> 2 4 5
> 3 5 3
>
> 输出例子1:
> 1
> 0
> 2
>
> 例子说明1:
> 样例解释:
> 有5个用户，喜好值为分别为1、2、3、3、5，
> 第一组询问对于标号[1,2]的用户喜好值为1的用户的个数是1
> 第二组询问对于标号[2,4]的用户喜好值为5的用户的个数是0
> 第三组询问对于标号[3,5]的用户喜好值为3的用户的个数是2

```js
const n = readline();
const k = readline().split(' ');
let q = readline();
while(q--) {
    const arr = readline().split(' ');
    console.log(check(arr));
}
function check(arr) {
    let num = 0;
    for(let i = arr[0] - 1; i <= arr[1] - arr[0] + 1; i++) {
        if(arr[2] === k[i]) { num += 1; }
    }
    return num;
}
```

# 手串

> 作为一个手串艺人，有金主向你订购了一条包含n个杂色串珠的手串——每个串珠要么无色，要么涂了若干种颜色。为了使手串的色彩看起来不那么单调，金主要求，手串上的任意一种颜色（不包含无色），在任意连续的m个串珠里至多出现一次（注意这里手串是一个环形）。手串上的颜色一共有c种。现在按顺时针序告诉你n个串珠的手串上，每个串珠用所包含的颜色分别有哪些。请你判断该手串上有多少种颜色不符合要求。即询问有多少种颜色在任意连续m个串珠中出现了至少两次。
>
> 输入描述:
> 第一行输入n，m，c三个数，用空格隔开。(1 <= n <= 10000, 1 <= m <= 1000, 1 <= c <= 50) 接下来n行每行的第一个数num_i(0 <= num_i <= c)表示第i颗珠子有多少种颜色。接下来依次读入num_i个数字，每个数字x表示第i颗柱子上包含第x种颜色(1 <= x <= c)
>
> 输出描述:
> 一个非负整数，表示该手链上有多少种颜色不符需求。
>
> 输入例子1:
> 5 2 3
> 3 1 2 3
> 0
> 2 2 3
> 1 2
> 1 3
>
> 输出例子1:
> 2
>
> 例子说明1:
> 第一种颜色出现在第1颗串珠，与规则无冲突。
> 第二种颜色分别出现在第 1，3，4颗串珠，第3颗与第4颗串珠相邻，所以不合要求。
> 第三种颜色分别出现在第1，3，5颗串珠，第5颗串珠的下一个是第1颗，所以不合要求。
> 总计有2种颜色的分布是有问题的。 
> 这里第2颗串珠是透明的。


```js
const f = readline().split(' ');
const n = f[0];
const m = f[1];
// const c = f[2];
const arr = [];
let err = 0;
for(let i = 0; i < n; i++) {
    const tmp = readline().substring(1).split(' ').join('');
    arr[i] = tmp;
}
// console.log(arr);
for(let i = 0; i < n; i += 1) {
    let [curr, back, cc] = [arr[i], '', m - 1];
    const cir = (x, i) => {
        let next = i + 1;
        while(x--){
            if(arr[next] === undefined) next = 0;
            back += arr[next];
            next += 1;
        }
    };
    cir(cc, i);
    // console.log(back);
    for(let j = 0; j < arr[i].length; j += 1) {
        if (back.indexOf(curr[j]) !== -1) {
            err += 1;
        }
    }
}
console.log(err)
```