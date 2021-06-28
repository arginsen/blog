---
title: 数据结构 | C 与 C++ 描述 | 二、线性表
date: 2020-9-25
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

# 线性表的定义

## 逻辑结构 - 当然是线性了

> 线性表(Linear List)是具有**相同数据类型**的 n (n>=0) 个数据元素的**有限序列**。其中 n 为表长，当 n=0 时线性表是一个空表。

`L = (a1, a2, ..., ai, ai+1, ..., an)`

1. ai 是线性表中的 “第i个”元素线性表中的位序（位序是从`1`开始的，数组下标是从`0`开始的）
2. a1 是表头元素，an是表尾元素
3. 除第一个元素外， 每个元素有且仅有一个直接前驱；除最后一个元素，每个元素有且仅有一个直接后驱

## 基本运算

增删减改

## 存储结构

### 顺序表 - 顺序存储

> 顺序表即是使用顺序存储的方式实现线性表。把逻辑上相邻的元素存储在物理位置上也相邻的存储单元中。

- 因为线性表具有相同数据类型，所以意味着每个数据元素所占空间一样大。
- 通过 `sizeof(ElemType)`，即可以得到一个数据元素的大小；像之后的链表使用结构体定义的，一个 `LNode` 包含 `data` + `*next`，那么我们新加入一个链表元素就需要给它分配存储空间：`(Lnode*)malloc(sizeof(LNode))`

- 特点：
1. 随机访问 - 能在`O(0)`时间内找到第 i 个元素
2. 存储密度高
3. 扩展容量不方便
4. 插入、删除数据元素不方便

#### 初始化

1. 静态分配

> 用静态的“数组”来存放数据元素，按 `MaxSize` 直接初始化一片连续的存储空间，大小为`MaxSize*sizeof(ElemType)`，即初始定义最大长度为10，存储10个数据元素，每个数据元素大小通过计算得知，两个相乘就是该顺序表总共占据的空间大小。

```c++
#define MaxSize 10

typedef struct{
    ElemType data[MaxSize]; // 按初始定义分配存储空间
    int length; // 设定顺序表当前实际的长度
}SqList;

// 再进行初始化
void InitList(SqList &L) {
    int n;
    cout<<"顺序表初始化长度：\n";
    cin>>n;
    if(n>=MaxSize)
        L.length = 0;
    L.length = n;
    for(int i = 0; i < L.length; i++) {
        L.data[i] = 0;
    }
}
```

- 顺序表定义后需要进行初始化，`L.length = 0`，便于之后的存值操作，否则会输出内存中遗留的脏数据
- 静态分配的顺序表长度不可更改

2. 动态分配

```c++
#include <stdlib.h>
#include <iostream>
#define InitSize 10

typedef struct {
    // 此处定义了data数据类型，分配空间会使用该类型计算单个所需空间大小，及将malloc返回的指针类型转换为定义的该类型
    ElemType *data;
    int MaxSize;
    int length;
}SeqList;

// 初始化
void InitList(SeqList &L) {
    // 使用malloc申请一片连续的存储空间
    // 计算data的数据类型空间大小，再乘以表的初始长度，就为总共占据的空间大小
    L.data = (ElemType*)malloc(sizeof(ElemType)*InitSize); 
    L.length = 0;
    L.MaxSize = InitSize;
}

// 动态增加数据长度
void IcreaseSize(SeqList &L, int len) {
    int *p = L.data; // 使用指针p保留data那一片连续空间的地址
    L.data = (ElemType*)malloc(sizeof(ElemType)*(L.MaxSize + len)); // 把新加入的长度给补上，重新分配存储空间大小
    for(int i = 0; i < L.length; i++) {
        L.data[i] = p[i]; // 从p那里复制数据到新空间
    }
    L.MaxSize += len; // 长度补加
    free(p); // 释放原来的内存空间
}
```

> 注：`malloc`函数返回一个指针，所以前边需要添加目的数据元素类型**以使返回的指针类型转为目的数据元素指针类型**

#### 插入

```c++
// 将值 e 插入为表 L 的第 i 个元素 
bool ListInsert(SqList &L, int i, int e) {
    if(i < 1 || i > L.length + 1)
        return false;
    if(L.length >= MaxSize)
        return false;
    for(int j = L.length; j >= i; j--) {
        L.data[j] = L.data[j - 1]; // 后移
    }
    L.data[i - 1] = e;
    L.length++;
    return true;
}
```

算法分析：
最好情况：插入到表尾，不需要移动元素，O(1)
最坏情况：插入到表头，需要移动原有的 n 个元素，O(n)
平均情况：插入每个位置概率相同，则平均移动 n/2 个元素，依旧 O(n)

#### 删除

> 需要注意是删除位序为 i 的元素，还是数组下标为 i 的元素

```c++
// 删除表 L 的第 i 个元素，并返回该元素的值 e（e 为引用型，外部就可以读取到被删除元素）
bool ListDelete(SqList &L, int i, int &e) {
    if(i < 1 || i > L.length)
        return false;
    e = L.data[i - 1];
    for(int j = i; j < L.length; j++) {
        L.data[j - 1] = L.data[j]; 
    }
    L.length--;
    return true;
}
```

算法分析：
最好情况：删除表尾元素，不需要移动元素，O(1)
最坏情况：删除表头元素，需要移动原有的 n 个元素，O(n)
平均情况：插入每个位置概率相同，则平均移动 n/2 个元素，依旧 O(n)

#### 逆置

```c++
// 按位逆置
void Reverse(SqList &L, int left, int right) {
    ElemType temp;
    for(int i = left, j = right; i < j; ++i,--j) {
        temp = L.data[i];
        L.data[i] = L.data[j];
        L.data[j] = temp;
    }
}
```

#### 取最值

```c++
// 取最小值
void Get_min(SqList L, int &min, int &minIdx) {
    min = L.data[0];
    minIdx = 0;
    for(int i = 1; i < L.length; i++) {
        if(L.data[i] < min)
            min = L.data[i];
            minIdx = i;
    }
}
```

#### 归并

```c++
// 合并两条递增的表
void mergeS(SqList A, SqList B, SqList &C) {
    int i = 0, j = 0, k = 0;
    // 当一个表遍历完退出
    while(i < A.length && j < B.length) {
        if(A.data[i] < B.data[j])
            C[k++] = A[i++];
        else
            C[k++] = B[j++];
    }
    while(i < A.length) {
        C[k++] = A[i++];
    }
    while(j < B.length) {
        C[k++] = A[j++];
    }
}
```

### 链表 - 链式存储

> 线性表的链式存储又称单链表，是指通过一组**任意的存储单元**来存储线性表中的数据元素。

- `LinkList` 写法更加强调这是一个链表
- `LNode*` 写法更强调这是一个结点

#### 单链表

1. 不要求大片连续空间，改变容量方便
2. 不可随机存取，要耗费一定空间存放指针
3. 插入、删除操作方便

##### 初始化

> 头结点是单链表第一个结点，通常不存储信息，其指针域指向线性表的第一个元素结点。

- 由于第一个数据结点的位置被存放在头结点的指针域中，所以在**链表的第一个位置上的操作和表的其他位置上操作一致**
- 无论链表是否为空，其头指针都指向头节点的非空指针，因此空表和非空表的处理也就得到了统一

1. 不带头结点

```c++
#include <stdlib.h>

// 定义
typedef struct LNode{
    ElemType data;
    struct LNode *next;
}LNode, *LinkList; // LNode 表示该数据类型，LinkList 表示指向该数据类型的指针

// ----------------------
// 给链表新节点分配存储空间
LNode *p = (LNode*)malloc(sizeof(LNode)); 
// LinkList 就等价于 LNode*
// 因此链表初始化，创建头节点也可以写作：
LinkList L = (LinkList)malloc(sizeof(LNode));
// ----------------------

// 初始化
bool InitList (LinkList &L) {
    L = NULL;
    return true;
}

// 判断链表是否为空
bool Empty(LinkList L) {
    return (L == NULL);
}

int main() {
    LinkList L; // 声明一个指向单链表头结点的指针
    InitList(L); // 初始化一个空表
    Empty(L);
}
```

2. 带头结点

```c++
#include <stdlib.h>

// 定义
typedef struct LNode{
    ElemType data;
    struct LNode *next;
}LNode, *LinkList;

// 初始化
bool InitList(LinkList &L) {
    L = (LinkList)malloc(sizeof(LNode)); // 分配头结点
    if(L == NULL)
        return false;
    L->next = NULL;
    return true;
}

int main() {
    LinkList L;
    InitList(L);
    // ...
    return 0;
}
```

##### 建表（带头）

1. 头插法

```c++
LinkList InitList(LinkList &L) {
	L = (LinkList)malloc(sizeof(LNode));
	L->next = NULL;
	LNode *p;
	int n;
	cin>>n; // 输入一个数表示表的元素个数
	for(int i = 0; i < n; i++) {
		p = (LNode*)malloc(sizeof(LNode));
		p->next = NULL; // 初始化 next 
		cin>>p->data; // 初始化data 
		p->next = L->next;
        L->next = p;
	}
	return L;
}
```

2. 尾插法

```c++
LinkList InitList(LinkList &L) {
	L = (LinkList)malloc(sizeof(LNode));
	L->next = NULL;
	LNode *p, *r = L;
	int n;
	cin>>n; // 输入一个数表示表的元素个数
	for(int i = 0; i < n; i++) {
		p = (LNode*)malloc(sizeof(LNode));
		p->next = NULL; // 初始化 next 
		cin>>p->data; // 初始化data 
		p->next = r->next; // 依照输入顺序建表
		r->next = p;
		r = p;
	}
	return L;
}
```

##### 插入

1. 指定位序插入

- 带头结点

```c++
// 在表 L 位序为 i 的位置上插入 元素 e
bool InsertList(LinkList &L, int i, ElemType e) {
    if(i < 1)
        return false;
    LNode *p = L;
    int j = 0;
    // p 指针停留在待插入位之前
    // 若插入第一位，则跳过循环直接插在p指针后
    while(p && j < i -1) {
        p = p->next;
        j++;
    }
    // 循环到 i 位置前，建插入新元素
    // 如果插入的点不存在，p 就到 NULL 了
    if(p == NULL)
        return false;
    LNode *q = (LNode*)malloc(sizeof(LNode));
    q->data = e;
    q->next = p->next;
    p->next = q;
    return true;
}


// 对指针的理解
// p 指针初始指向 L，即链表 L 的头结点，p 实际保存的是该头结点的内存地址
// 所以数据的地址和数据本身没有任何关系，头结点可能为空，可能有脏数据，但其已经被分配了存储空间，那么地址必然不为空
// 因此对比链表结构，除非指针明确的指向 NULL，否则均不等于 NULL。即 p->next = NULL
cout<<"p指针存放L的地址："<<p<<endl;
cout<<"p指针的大小："<<sizeof(p)<<endl;

p指针存放L的地址：0xc71530
p指针的大小：8
```

平均时间复杂度：O(n)

- 不带头结点

```c++
// 在表 L 位序为 i 的位置上插入 元素 e
bool InsertList(LinkList &L, int i, ElemType e) {
    if(i < 1)
        return false;
    if(i == 1) {
        LNode *h = (LNode*)malloc(sizeof(LNode)); // 分配结点直接指向 L 
        h.data = e;
        h.next = L;
        L = h;
        return true;
    }
    LNode *p = L;
    int j = 1; // 第 1 位序的情况已经处理，所以从 1 开始
    while(p && j < i-1) {
        p = p->next;
        j++;
    }
    if(p == NULL)
        return false;
    LNode *q = (LNode*)malloc(sizeof(LNode));
    q->data = e;
    q->next = p->next;
    p->next = q;
    return true;
}
```

2. 指定结点插入

- 在指定结点之后

```c++
bool InsertNextNode(LNode *p, ElemType e) {
    if(p == NUll)
        return false;
    LNode *q = (LNode*)malloc(sizeof(LNode));
    if(q == NULL) // 可能内存分配失败
        return false;
    q->data = e;
    q->next = p->next;
    p->next = q;
    return true;
}
```

- 在指定节点之前

```c++
bool InsertPriorNode(LNode *p, ElemType e) {
    if(p == NUll)
        return false;
    LNode *q = (LNode*)malloc(sizeof(LNode));
    if(q == NULL) // 可能内存分配失败
        return false;
    q->data = p->data;
    p->data = e;
    q->next = p->next;
    p->next = q;
    return true;
}
```

##### 删除

1. 指定位序

> 通过带头结点与不带头结点插入、删除操作的对比，可以看出带与不带的区别：**链表的第一个位置上的操作和表的其他位置上操作是否一致**，不带头结点就需要单独处理

- 带头结点

```c++
// 删除表 L 位序为 i 的元素，并返回为 e
bool ListDelete(LinkList &L, int i, ElemType &e) {
    if(i < 1)
        return false; // 给定的 i 过小
    LNode *p = L;
    int j = 0;
    while(p->next && j < i - 1) {
        p = p->next;
        j++;
    }
    if(p == NULL || p->next == NULL) // 给定的 i 超标或者 L 是空链表
        return false; 
    e = p->next->data;
    LNode *q = p->next;
    p->next = q->next;
    free(q);
    return true;
}
```

- 不带头结点

```c++
bool ListDelete(LinkList &L, int i, ElemType &e) {
    if(i < 1)
        return false; // 给定的 i 过小
    if(i == 1 && p != NULL)
        L = L->next;
        return true;
    LNode *p = L;
    int j = 1; // 少了头结点，位序 1 已特殊处理，就从 1 开始累计
    while(p && j < i - 1) {
        p = p->next;
        j++;
    }
    if(p == NULL || p->next == NULL)
        return false; // 给定的 i 超标
    e = p->next->data;
    LNode *q = p->next;
    p->next = q->next;
    free(q);
    return true;
}
```

2. 指定结点删除

```c++
bool DeleteNextNode(LinkList &L, int i, ElemType e) {
    if(p == NULL)
        return false;
    LNode *q = p->next;
    p->data = p->next->data;
    p->next = q->next;
    free(q);
    return true;
}
```

##### 逆置

```c++
// 将链表的 left 至 right 位元素逆置
// p 指针作为运行指针，到 left 前一位了用 tmp 保存下位置，接着往下走到 right，用 q 标记，p 再返回 left 前一位
// 利用循环将这一段链表逆置
void Reverse(LinkList &L, int left, int right) {
    if(left >= right)
        return false;
    LNode *p = L, *q, *tmp;
    int i = 1;
    while(p != NULL && i < left) {
        p = p->next;
        i++;
    }
    if(p == NULL)
        return false;
    tmp = p; j = i - 1;
    while(p != NULL && j < right) {
        p = p->next;
        j++;
    }
    if(p == NULL)
        return false;
    q = p;
    p = tmp;
    // 循环逆置 相当于尾插，逐次将 p 后的元素，插到 q 后，直到 p->next = q
    while(p->next != q) {
        tmp = p->next;
        p->next = tmp->next;
        tmp->next = q->next;
        q->next = tmp;
    }
}

// 也可利用头插法
LNode *r;
r = p->next;
p->next = q->next;
while(p->next != q) {
    tmp = r;
    tmp->next = p->next;
    p->next = tmp;
    r = r->next;
}
```

##### 取最值

```c++
// 取最小值
void Get_min(LinkList L, int &min) {
    LNode *p = L->next; min = p->data;
    while(p != NULL) {
        p = p->next;
        if(p->data < min)
            min = p->data;
    }
}
```

##### 归并

```c++
// 两条递增链表合并为一条逆序链表，取出较小的一个，在新链头插
void mergeR(LinkList A, LinkList B, LinkList &C) {
    LNode *p = A->next, *q = B->next, *s;
    C = A;
    C->next = NULL;
    free(B);
    while(p != NULL && q != NULL) {
        if(p->data <= q->data) {
            s = p;
            p = p->next;
            s->next = C->next;
            C->next = s;
        } else {
            s = q;
            q = q->next;
            s->next = C->next;
            C->next;
        }
    }
    while(p != NULL) {
        s = p;
        p = p->next;
        s->next = C->next;
        C->next = s;
    }
    while(q != NULL) {
        s = q;
        q = q->next;
        s->next = C->next;
        C->next;
    }
}
```

#### 双链表

> 很方便找到结点的前后结点，进行插入删除操作

##### 初始化

```c++
#include <stdlib.h>

typedef struct DNode{
    ElemType data;
    struct DNode *prior, *next;
}DNode, *DLinkList;

// typedef DNode* DLinkList;

bool InitDlinkList(DLinkList &L) {
    L = (DLinkList)malloc(sizeof(DNode));
    if(L == NULL)
        return false;
    L->prior = NULL;
    L->next = NULL;
    return true;
}

// 判断双链表是否为空
// 双链表头结点的 prior 指针永远指向 NULL
bool Empty(DLinkList L) {
    return (L->next == NULL);
}
```

##### 插入

```c++
// 在 p 结点之后插入 s 结点
bool InsertDNode(DNode *p, DNode *s) {
    if(p == NULL || s == NULL)
        return false;
    s->next = p->next;
    if(p->next != NULL) // 存在 p 结点为双链表最后一个结点的情况
        p->next->prior = s;
    s->prior = p;
    p->next = s;
    return true;
}
```

##### 删除

```c++
// 删除 p 结点之后的结点
bool DeleteNextDNode(DNode *p) {
    if(p == NULL || p->next == NULL)
        return false;
    DNode *q = p->next;
    p->next = q->next;
    if(q->next != NULL) // 存在待删除结点 q 为双链表最后一个结点的情况，p 直接指向 q 后边，释放 q 就完了
        q->next->prior = p;
    free(q);
}

// 销毁一条双链表
void DestroyList(DLinkList &L) {
    while(L->next != NULL) {
        DeleteNextDNode(L); // 只要 L 头结点后边有数据，逐一删除，直到指向 NULL
    }
    free(L); // 释放头结点
    L = NULL; // 头指针指向 NULL
}
```

##### 遍历

```c++
// 后向遍历
while(p != NULL) {
    p = p->next;
}

// 前向遍历
while(p != NULL) {
    p = p->prior;
}

// 前向遍历 - 跳过头结点
while(p->prior != NULL) {
    p = p->prior;
}
```

#### 静态链表

优点：增删操作不需要移动大量元素
缺点：不能随机存取，只能从头结点开始依次往后查找；容量固定不可变。

```c++
#include <stdlib.h>
#define MaxSize 10

typedef struct{
    ELemType data;
    int next;
}SLinkList[MaxSize];
```

# 顺序表与链表的对比

1. 顺序表和链表的逻辑结构都是线性结构，都属于线性表
2. 两者的存储结构不同。
   - 顺序表采用顺序存储，具有随机访问，存储密度高的特点，但是扩展容量、进行插入删除操作均不方便；
   - 链表采用链式存储，对存储空间的分配更加灵活，插入、删除操作方便，但是存储密度低，不支持随机存储。
3. 由于采用不同的存储方式实现，因此基本操作的实现效率也不同。
   - 当初始化时，顺序表需要预分配大片连续空间，若分配空间过小，之后不方便扩容，若分配空间过大，则又浪费内存资源；链表则只需要分配一个头结点，或只分配头指针即可。
   - 当进行插入、删除操作时，顺序表和链表的时间复杂度都是O(n)，顺序表的开销主要是查找后移动元素；而链表的时间开销主要是查找，操作本身只需改变指针；若数据元素很大，则移动元素的时间开销远远大于查找元素的开销。
   - 当进行查找时，顺序表的按位查找时间复杂度为 O(1)，按值查找时间复杂度是 O(n)，若表内元素有序，可在 O(log2n) 时间内找到；链表按位查找和按值查找的时间复杂度都是 O(n)





