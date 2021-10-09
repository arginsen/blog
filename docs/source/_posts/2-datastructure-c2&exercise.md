---
title: 数据结构 | C 与 C++ 描述 | 二、线性表练习题
date: 2020-9-30
tags: 
  - dataStructure
  - c
  - 考研
categories: notes
photos:
  - /blog/img/dataStructure.jpg
---

<br>
<!--more-->


# 顺序表

## 定义一个顺序表

```c++
// 万能头文件
# include <bits/stdc++.h>
using namespace std;

// 一般
// 静态分配
# define MaxSize 50

typedef struct {
    ElemType data[MaxSize];
    int length;
}SqList;


// 动态分配
# define InitSize 100

typedef struct {
    ElemType *data;
    int MaxSize, length;
}SeqList;

// 之后再分配大小
SeqList L;
L.data = (ElemType*)malloc(sizeof(ElemType)*InitSize);

// c++ 写法
L.data = new ElemType[InitSize];
```

## 顺序表习题

1. 从顺序表中删除具有最小值的元素（假设唯一）并由函数返回被删元素的值。空出的位置由最后一个元素填补，若顺序表为空则显示出错信息并退出运行

```c++
// 由于天勤家定义顺序表是直接在参数添加数组，再传入另一个参数length来控制顺序表的长度，
// 王道这里是直接用现成的顺序表结构体来传入

// 将最小值元素赋值给引用型参数 value，这样就可以传出最小值
bool del_min(SqList &L, ElemType &value) {
    if(L.length == 0)
        return false;
    value = L.data[0];
    int pos = 0;
    for(int i=1; i < L.length; i++) {
        if(L.data[i] < value) {
            value = L.data[i]; // value循环完后返回最小的值
            pos = i; // 记录最小值的位置
        }
    }
    L.data[pos] = L.data[L.length - 1]; // 给最小值赋值顺序表最后一个值
    L.length--; // 缩短一位
    return true;
}
```

2. 设计一个高效的算法，将顺序表L的所有元素逆置，要求算法的空间复杂度为Ｏ(1)

```c++
void reverse(SqList &L) {
    ElemType temp;
    for(int i = 0;i < L.length;i++) {
        temp = L.data[i];
        L.data[i] = L.data[L.length - 1 - i];
        L.data[L.length - 1 - i] = temp;
    }
}
```

3. 对长度为n的顺序表L，编写一个时间复杂度为Ｏ(n)，空间复杂度为Ｏ(1)的算法，该算法删除线性表中所有值为x的数据元素

```C++
void del_x(SqList &L, ElemType x) {
    int k = 0;
    for(int i = 0; i < L.lengh; i++) {
        if(L.data[i] != x) {
            L.data[k] = L.data[i]; // 当前i是x继续递增，而k不递增，等到不等于x了，另此时k处的值等于递增后i的值，相当于后边整体前移一位
            k++;
        }
    L.length -= k;
}
```

4. 从有序顺序表中删除其值在给定值s与t之间（要求`s<t`）的所有元素，如果s或t不合理或顺序表为空，显示出错信息并退出运行。

```c++
// 定义 l 和 r，分别记录到 s (不包含s)时的下标和到 t (包含t)时的下标
bool del_s_t(SqList &L, ElemType s, ElemType t) {
    if(s>=t || L.length == 0)
        return false;
    int l = 0, r = 0;
    for(int i = 0; i < L.length; i++) {
        if(L.data[i] <= s)
            l++;
        if(L.data[i] < t)
            r++;
    }
    for(int j = 0;j < L.length - l;j++) {
        L.data[l + j] = L.data[r + j];
    }
    L.length -= r - l;
    return true;
}
```

```js
// 浏览器js测试
let r = 0, l = 0;
const s = 2, t = 9;
const arr = [1,2,3,5,7,8,9,11,12];
for(let i = 0;i < arr.length; i++) {
    if(arr[i] <= s) l++;
    if(arr[i] < t) r++;
}
for(let j = 0;j < arr.length - l; j++) {
    arr[l + j] = arr[r + j];
}
arr.length -= r - l;
```

5. 从有序顺序表中删除其值在给定值s与t之间（包含s和t，要求 `s<t`）的所有元素，如果s或t不合理或顺序表为空，显示出错信息并退出运行。

```c++
// 1.与 4 相同，定位的 l r 变为闭区间
bool del_s_t(SqList &L, ElemType s, ElemType t) {
    if(s>=t || L.length == 0)
        return false;
    int l = 0, r = 0;
    for(int i = 0; i < L.length; i++) {
        if(L.data[i] < s)
            l++;
        if(L.data[i] <= t)
            r++;
    }
    for(int j = 0;j < L.length - l;j++) {
        L.data[l + j] = L.data[r + j];
    }
    L.length -= r - l;
    return true;
}

// 2.若元素属于 [s, t] 闭区间，累加 k，因为是连续的，等过了这个区间，再将当前元素赋值给第前 k 个元素
bool del_s_t(SqList &L, ElemType s, ElemType t) {
    int k = 0;
    if(s >= t || L.length == 0)
        return false;
    for(int i = 0; i < L.length; i++) {
        if(L.data[i] >= s && L.data[i] <= t)
            k++;
        else
            L.data[i-k] = L.data[i];
    }
    L.length -= k;
    return true;
```

6. 从有序顺序表中删除所有其值重复的元素，使表中所有元素的值均不同。

```c++
// 1 2 2 2 2 3 3 3 4 4 5
bool del_same(SeqList &L) {
    if(L.length == 0)
        return false;
    int i, j; // 相当于两个指针，i负责不重复的部分，j负责往后遍历与i对比，不相同了将j赋值给i+1，从i+1继续
    for(i = 0, j = 1;j < L.length; j++) {
        if(L.data[i] != L.data[j])
            L.data[++i] = L.data[j]; // 如果相等则不会执行，i停留
    }
    L.length = i + 1;
    return true;
}
```

7. 将两个有序顺序表合并为一个新的有序顺序表，并由函数返回结果顺序表

```c++
bool merge(SeqList A, SeqList B, SeqList &C) {
    if(A.length + B.length > C.maxSize)
        return false;
    int i = 0, j = 0, k = 0;
    // 先遍历对比两个顺序表的值，小的添在C里，直到A或B遍历完了
    while(i < A.length && j < B.length) {
        if(A.data[i] < B.data[j])
            C.data[k] = A.data[i];
            k++;i++;
        else
            C.data[k] = B.data[j];
            k++;j++;
    }
    // 如果A还没遍历完，说明A剩下的值都大，全都存入C中即可
    while(i < A.length) {
        C.data[k] = A.data[i];
        k++;i++;
    }
    while(j < B.length) {
        C.data[k] = B.data[j];
        k++;j++;
    }
    C.length = k; // 循环最后k已递增
    return true;
}
```

8. ![](/blog/img/2020/data-structure/linearlist/seqlist/8.png)

```c++
void resverse(int A[], int l, int r, int arraySize) {
    int tmp;
    // 判断传入的左值要比右值小，右值不能超出数组长度
    if(l >= r || r >= arraySize)
        return;
    // 数组为偶数则一半一半，为奇数则中间的数刚好不用操作
    for(int i = l; i < (r-l)/2; i++) {
        tmp = A[i];
        A[i] = A[r-i];
        A[r-i] = tmp;
    }
}
void exchange(int A[], int m, int n, int arraySize) {
    resverse(A, 0, m-1, arraySize); // 先反转前 m 个
    resverse(A, m, m+n-1, arraySize); // 再反转后 n 个
    resverse(A, 0, m+n-1, arraySize); // 再全部反转
}
```

9. ![](/blog/img/2020/data-structure/linearlist/seqlist/9.png)

```c++

```

10. ![](/blog/img/2020/data-structure/linearlist/seqlist/10.png)

```c++

```

11. ![](/blog/img/2020/data-structure/linearlist/seqlist/11.png)

```c++

```

12. ![](/blog/img/2020/data-structure/linearlist/seqlist/12.png)

```c++

```

13. ![](/blog/img/2020/data-structure/linearlist/seqlist/13.png)

```c++

```




# 链表

## 定义一个链表

```c++
#include <bits/stdc++.h>
using namespace std;
#define maxSize 50

typedef struct LNode{
    int data;
    struct LNode *next;
}LNode, *LinkList;

// 建表
LinkList InitList(LinkList &L) {
	L = (LinkList)malloc(sizeof(LNode)); // L 指向链表的第一个结点地址
	L->next = NULL;
	LNode *p, *r = L;
	int n;
	cin>>n;
	for(int i = 0; i < n; i++) {
		p = (LNode*)malloc(sizeof(LNode));
		p->next = NULL; // 初始化 next 
		cin>>p->data; // 初始化data 
		p->next = r->next;
		r->next = p;
		r = p;
	}
	return L;
}
```

## 链表习题

1. 设计一个递归算法，删除不带头结点的单链表`L`中所有值为`x`的结点。

```c++
void del_x(LinkList &L, ElemType x) {
    LNode *p;
    if(L == NULL)
        return;
    if(L->data = x) {
        p = L;
        L = L->next;
        free(p);
        del_x(L, x);
    } else {
        del_x(L->next, x);
    }
}
```

2. 在带头结点的单链表`L`中，删除所有值为`x`的结点，并释放其空间，假设值为`x`的结点不唯一，试编写算法以实现上述操作。

```c++
void del_x(LinkList &L, ElemType x) {
    LNode *p = L->next, *q = L, *r;
    while(p != NULL){
        if(p->data == x) {
            r = p;
            p = p->next;
            q->next = p;
            free(r);
        } else {
            q = p;
            p = p->next;
        }
    }
}
```

3. 设L为带头结点的单链表，编写算法实现从尾到头反向输出每个结点的值。

```c++
// 使用循环，逆置链表后再依次输出

// 递归
void reverse(LinkList L) {
    if(L->next) {
        reverse(L->next);
    }
    if(L)
        cout<<L->data<<" ";
}
```

4. 试编写在带头结点的单链表`L`中删除一个最小值结点的高效算法(假设最小值结点是唯一的)。

```c++
// 定义p工作指针，q为min的前一位；实质上是用p->next与min进行比较
LinkList del_min(LinkList L) {
    LNode *p = L->next, *q = L, *min = p;
    while(p->next) {
        if(p->next->data < min->data) {
            q = p;
            p = p->next;
            min = p;
        } else {
            p = p->next;
        }
    }
    q->next = min->next;
    free(min);
    return L;
}
```

5. 试编写算法将带头结点的单链表就地逆置，所谓“就地”是指辅助空间复杂度为0(1).

```c++
// 头插法
LinkList reverse_1(LinkList L) {
    LNode *p, *r;
    p = L->next; // p指向第一个元素
    L->next = NULL; // L变为新链
    while(p) {
        r = p->next; // 存储p的下一个地址
        p-next = L->next; // p的next指向L头节点的next，开始头插
        L->next = p; // L这边头节点next指向p
        p = r; // 更新p指向为原链下一个地址
    }
    return L;
}

// 遍历，使用两个指针逆置链表，最后接回头节点
LinkList reverse_2(LinkList L) {
    LNode *p = L->next, *r = p->next, *tmp; // 定义p和r，r是p后一位，r用于存储p后边的地址
    p->next = NULL; // 第一个元素变尾元素
    while(r) {
        tmp = p; // 暂存p指针
        p = r; // p往后移
        r = r->next; // r往后移
        p->next = tmp; // 更新p指向原链上一个元素
    }
    // r为空退出循环，此时元素逆置完成
    L->next = p;
    return L;
}

// 因为循环时是p和r两个指针共同移动，所以tmp可以存储p，也可以存储r
LinkList reverse_3(LinkList L) {
    LNode *p = L->next, *r = p->next, *tmp;
    p->next = NULL;
    while(r) {
        tmp = r; 
        r = r->next;
        tmp->next = p;
        p = tmp;
    }
    L->next = p;
    return L;
}
```

6. 有一个带头结点的单链表`L`，设计一个算法使其元素递增有序。

```c++
void increase_sort(LinkList &L) {
    LNode *p =L->next, *q, *r = p->next;
    p->next = NULL; // 保留头节点作为新链 
    p = r; // p回到旧链继续做对比 
    while(p) {
    	while(q->next && q->next->data < p->data) {
   	r = p->next; // r往后推移 
    	q = L; // q在新链每次从头开始 
     		q = q->next; 
        }
        p->next = q->next; // 旧链的值插入新链合适的位置 
        q->next = p;
        p = r; 
    }
}
```

7. 设在一个带表头结点的单链表中所有元素结点的数据值无序，试编写一个函数，删除表中所有介于给定的两个值(作为函数参数给出)之间的元素的元素(若存在).

```c++
void delete_s_t(LinkList &L, int s, int t) {
    LNode *p = L->next; *q = L;
    while(p) {
        if(p->data > s && p->data < t) {
            q->next = p->next;
            free(p);
            p = q->next;
        } else {
            q = p;
            p = p->next;
        }
    }
}
```

8. 给定两个单链表，编写算法找出两个链表的公共结点。

```c++

```

9. 给定一个带表头结点的单链表，设`head`为头指针，结点结构为`(data, next)`，`data`为整型元素，`next` 为指针，试写出算法:按递增次序输出单链表中各结点的数据元素，并释放结点所占的存储空间(要求:不允许使用数组作为辅助空间)。

```c++

```

10. 将一个带头结点的单链表`A`分解为两个带头结点的单链表`A`和`B`,使得A表中含有原表中序号为奇数的元素,而B表中含有原表中序号为偶数的元素，且保持其相对顺序不变。

```c++

```

11. ![](/blog/img/2020/data-structure/linearlist/linkedlist/11.png)

```c++

```

12. 在一个递增有序的线性表中，有数值相同的元素存在。若存储方式为单链表，设计算法去掉数值相同的元素,使表中不再有重复的元素，例如`(7, 10, 10, 21, 30, 42, 42, 42, 51, 70)`将变为`(7, 10, 21, 30, 42, 51, 70)`。

```c++

```

13. 假设有两个按元素值递增次序排列的线性表，均以单链表形式存储。请编写算法将这两个单链表归并为一个按元素值递减次序排列的单链表，并要求利用原来两个单链表的结点存放归并后的单链表。

```c++

```

14. 设A和B是两个单链表(带头结点),其中元素递增有序。设计一个算法从A和B中的公共元素产生单链表C,要求不破坏A、B的结点。

```c++

```

15. 已知两个链表A和B分别表示两个集合，其元素递增排列。编制函数，求A与B的交集，并存放于A链表中。

```c++

```

16. ![](/blog/img/2020/data-structure/linearlist/linkedlist/16.png)

```c++

```

17. 设计一个算法用于判断带头结点的循环双链表是否对称。

```c++

```

18. 有两个循环单链表，链表头指针分别为`h1`和`h2`,编写一个函数将链表`h2`链接到链表`h1`之后，要求链接后的链表仍保持循环链表形式。

```c++

```

19. 设有一个带头结点的循环单链表，其结点值均为正整数。设计一个算法，反复找出单链表中结点值最小的结点并输出，然后将该结点从中删除，直到单链表空为止，再删除表头结点。

```c++

```

20. 设头指针为L的带有表头结点的非循环双向链表，其每个结点中除有`pred` (前驱指针)、`data` (数据)和`next` (后继指针)域外，还有一个访问频度城`freq`.在链表被启用前，其值均初始化为零。每当在链表中进行一次`Locate (L,x)`运算时,令元素值为x的结点中`freq`域的值增1,并使此链表中结点保持按访问频度非增(递减)的顺序排列，同时最近访问的结点排在频度相同的结点前面，以便使频繁访问的结点总是靠近表头。试编写符合上述要求的`Locate(L,x)`运算的算法，该运算为函数过程，返回找到结点的地址，类型为指针型.

```c++

```

21. ![](/blog/img/2020/data-structure/linearlist/linkedlist/21.png)

```c++

```

22. ![](/blog/img/2020/data-structure/linearlist/linkedlist/22.png)

```c++

```

23. ![](/blog/img/2020/data-structure/linearlist/linkedlist/23.png)

```c++

```

24. 设计一个算法完成以下功能:判断一个链表是否有环,如果有,找出环的入口点并返回,否则返回NULL.

```c++

```

25. ![](/blog/img/2020/data-structure/linearlist/linkedlist/25.png)

```c++

```
