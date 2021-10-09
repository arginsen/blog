---
title: 数据结构 | C 与 C++ 描述 | 三、栈和队列
date: 2020-10-2
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

# 栈

> 栈（Stack）是只允许在一端进行插入或删除操作的线性表

- 栈只能在栈顶进行插入和删除操作，进栈次序依次为 a1, a2, a3, a4，而出栈次序则依次为 a4, a3, a2, a1。(Last In First Out, LIFO)
- n 个不同元素进栈，则出栈元素不同排列的个数为 `1/(n+1) 乘 C的2n到n`

## 基本操作

- InitStack(&S)：初始化
- StackEmpty(S)：判断栈是否为空
- Push(&S, x)：进栈，栈未满则插入 x 成为新栈顶
- Pop(&S, &x)：出栈，栈非空则弹出栈顶元素，用 x 返回
- GetTop(S, &x)：读栈顶元素，栈非空则用 x 返回栈顶元素
- DestroyStack(&S)：销毁栈，并释放栈 S 占用的空间

## 逻辑结构 - 线性

## 存储结构

> 栈是一种操作受限的线性表，有两种存储方式

### 顺序栈

> 利用一组地址连续的存储单元存放自栈底到栈顶的数据元素，同时附设一个指针（top）指向栈顶元素的地址。

```c++
#define MaxSize 50

// 定义
typedef struct{
    ElemType data[MaxSize];
    int top;
}SqStack;

// 初始化
bool InitStack(SqStack &S) {
    S.top = -1; // 一般令 top 指针为 -1，这样指针总指在当前值；为 0 则是指针指向下一个待定栈顶（还没数据）
    return true;
}

// 判空
bool StackEmpty(SqStack S) {
    if(S.top == -1)
        return true;
    else
        return false;
}

// 进栈
bool Push(SqStack &S, Elemtype x) {
    if(S.top == MaxSize - 1) // 栈满报错
        return false;
    S.data[++S.top] = x; // 从 -1 开始，需先将指针前移一位，再传入值
    return true;
}

// 出栈
bool Pop(SqStack &S, Elemtype &x) {
    if(S.top == -1)
        return false;
    x = S.data[S.top--]; 
    return true;
}

// 读栈顶元素
bool GetTop(SqStack S, Elemtype &x) {
    if(S.top == -1)
        return false;
    x = S.data[S.top]; // 这里如果 top 从 0 开始，就需要指针往后移一位
    return true;
}

// 销毁栈
bool DestroyStack(SqStack &S) {
    S.top == -1;
    return true;
}
```

### 链栈

> 采用链式存储的栈。入栈、出栈的操作都在表头进行。

- 链栈的优点是便于**多个栈**共享存储空间和提高其效率，且不存在栈满上溢的情况。
- 通常链栈**不定义**头结点

```c++
// 定义
typedef struct LNode{
    ElemType data;
    struct LNode *next;
}LNode, *LinkStack;

// 初始化
bool InitStack(LinkStack &S) {
    S = NULL;
    return true;
}

// 判空
bool Empty(LinkStack S) {
    return (S == NULL);
}

// 进栈
bool Push(LinkStack &S, Elemtype x) {
    LNode *q = (LNode*)malloc(sizeof(LNode));
    q->next = S;
    q->data = x;
    S = q;
    return true;
}

// 出栈
bool Pop(LinkStack &S, Elemtype &x) {
    if(S == NULL)
        return false;
    LNode *q = S; // 建立一个中间指针指向头元素，后续释放头元素空间
    x = q->data;
    S = S->next;
    free(q);
    return true;
}

// 读栈顶元素
bool GetTop(SqStack S, Elemtype &x) {
    if(S == NULL)
        return false;
    x = S->data;
    return true;
}
```

# 队列

> 队列（Queue），一种操作受限的线性表，只允许在表的一端进行插入，而在表的另一端进行删除。

- 通常队列都是头部进行删除操作，尾部进行插入操作，因此最早入队的也是最早出队的（First In First Out, FIFO)

## 基本操作

- InitQueue(&Q)：初始化
- QueueEmpty(Q)：判队列空
- EnQueue(&Q, x)：入队，队列未满则插入 x 作为新队尾
- DeQueue(&Q, &x)：出队，队列非空则删除对头元素，用 x 返回
- GetHead(Q, &x)：读队头元素，若队列非空，用 x 返回队头元素

## 逻辑结构 - 线性

## 存储结构

### 顺序存储

> 队列的顺序存储是分配一块连续的存储单元存放队列中的元素，并附设两个指针，`front` 指针指向队头的位置，`rear` 指针指向队尾元素的下一个位置（也就是待插入元素的空位置）

> 传统顺序存储实现的顺序链表，在判断队满/对空的时候会存在问题，当指针 `Q.front = Q.rear = 0`，可以作为判空的条件，而 `Q.rear = MaxSize` 则不一定为满，有可能为空（刚插满又删完，两指针都移动到刚好超出 MaxSize 的位置，此时为“假溢出”）。因此使用循环队列的概念。

#### 循环队列

> 循环链表，即把存储队列元素的表从逻辑上视为一个环，当队首指针 `Q.front = MaxSize - 1`（即连续空间中最后一个位置），再往前进一个位置就又从 0 开始，利用取余运算实现：`Q.front = (Q.front + 1) % MaxSize`，同时新元素入队 rear 指针也会按顺时针方向向前进一。

> - 循环链表的队空为 `Q.front == Q.rear`，但等到队满后头尾指针也会相遇，因此使用三种方法用以区分：
  1. **牺牲一个单元**来区分队空和队满，**入队**时少用一个队列单元，约定队头指针在队尾指针的下一位只作为队满的标志：`(Q.rear + 1) % MaxSize == Q.front` ，而队空条件依旧为两指针相同，此时队列中的元素个数为 `(Q.rear - Q.front + MaxSize) % MaxSize`；
  2. 类型中增设表示元素个数的数据成员，队空的条件为：`Q.size == 0`，队满的条件为：`Q.size == MaxSize`；
  3. 类型中增设 tag 数据成员，用以标记上一步操作是入队还是出队，判断队空的条件就为：`Q.front == Q.rear && Q.tag == 0`，队满的条件是：`Q.front == Q.rear && Q.tag == 1`。

#### 牺牲一个单元

```c++
#define MaxSize 50

// 定义
typedef struct{
    ElemType data[MaxSize];
    int front, rear;
}SqQueue;

// 初始化
bool InitQueue(SqQueue &Q) {
    Q.front = Q.rear = 0;
    return true;
}

// 判空
bool QueueEmpty(SqQueue Q) {
    if(Q.front == Q.rear)
        return true;
    else
        return false;
}

// 入队 - 判断队满
bool EnQueue(SqQueue &Q, ElemType x) {
    if((Q.rear + 1) % MaxSize == Q.front) // rear 再往后移一位是 front，就不添加元素了，rear 当前指的这个空单元就被废弃了
        return false;
    Q.data[Q.rear] = x;
    Q.rear = (Q.rear + 1) % MaxSize; // rear 加 1 取余，往前移一位
    return true;
}

// 出队 - 判断队空
bool DeQueue(SqQueue &Q, ElemType &x) {
    if(Q.front == Q.rear)
        return false;
    x = Q.data[Q.front];
    Q.front = (Q.front + 1) % MaxSize; // front 加 1 取余，往前移一位
    return true;
}

// 读队头元素 - 判断队空
bool GetHead(SqQueue Q, ElemType &x) {
    if(Q.front == Q.rear)
        return false;
    x = Q.data[Q.front];
    return true;
}
```

#### 增设 size

```c++
#define MaxSize 50

// 定义
typedef struct{
    ElemType data[MaxSize];
    int front, rear, size;
}SqQueue;

// 初始化
bool InitQueue(SqQueue &Q) {
    Q.front = Q.rear = 0;
    Q.size = 0;
    return true;
}

// 判空
bool QueueEmpty(SqQueue Q) {
    if(Q.size == 0)
        return true;
    else
        return false;
}

// 入队
bool EnQueue(SqQueue &Q, ElemType x) {
    if(Q.size == MaxSize)
        return false;
    Q.data[Q.rear] = x;
    Q.rear = (Q.rear + 1) % MaxSize;
    Q.size++;
    return true;
}

// 出队
bool DeQueue(SqQueue &Q, ElemType &x){
    if(Q.size == 0)
        return false;
    x = Q.data[Q.front];
    Q.front = (Q.front + 1) % MaxSize;
    Q.size--;
    return true;
}
```

#### 增设 tag

```c++
#define MaxSize 50

// 定义
typedef struct{
    ElemType data[MaxSize];
    int front, rear, tag;
}SqQueue;

// 初始化
bool InitQueue(SqQueue &Q) {
    Q.front = Q.rear = 0;
    Q.tag = 0;
    return true;
}

// 判空
bool QueueEmpty(SqQueue Q) {
    if(Q.front == Q.rear && Q.tag == 0)
        return true;
    else
        return false;
}

// 入队
bool EnQueue(SqQueue &Q, ElemType x) {
    if(Q.front == Q.rear && Q.tag = 1)
        return false;
    Q.data[Q.rear] = x;
    Q.rear = (Q.rear + 1) % MaxSize;
    Q.tag = 1; // 有新元素入队，改变 tag 的状态为 1
    return true;
}

// 出队
bool DeQueue(SqQueue &Q, ElemType &x){
    if(Q.front == Q.rear && Q.tag = 0)
        return false;
    x = Q.data[Q.front];
    Q.front = (Q.front + 1) % MaxSize;
    Q.tag = 0; // 有元素出队，改变 tag 的状态为 0
    return true;
}
```

### 链队列

> 采用链式存储的队列。同时带有队头指针和队尾指针的单链表，`front` 指向队头结点（一般为头结点），`rear` 指向队尾结点；用头指针进行删除操作，尾指针进行插入操作。

- 用单链表表示的链式队列特别适用于数据元素变动比较大的情形，且不存在队列满、产生溢出的问题；
- 使用多个队列时，最好使用链队，存储分配会更合理，也不会存在溢出问题；
- 通常链队会定义**头结点**，这样插入、删除操作会统一。

```c++
// 定义
typedef struct LNode{
    ElemType data;
    struct LNode *next;
}LNode;

typedef struct{
    LNode *front, *rear;
}LinkQueue;

// 初始化
bool InitQueue(LinkQueue &Q) {
    Q.front = Q.rear = (LNode*)malloc(sizeof(LNode));
    Q.front->next = NULL; // Q.front 指向头结点，进行插入操作
    return true;
}

// 判空
bool QueueEmpty(SqQueue Q) {
    if(Q.front == Q.rear)
        return true;
    else
        return false;
}

// 入队
bool EnQueue(LinkQueue &Q, ElemType x) {
    LNode *s = (LNode*)malloc(sizeof(LNode));
    s->data = x; s->next = NULL;
    Q.rear->next = s;
    Q.rear = s;
    return true;
}

// 出队
bool DeQueue(LinkQueue &Q, ElemType &x) {
    if(Q.front == Q.rear)
        return false;
    LNode *s = Q.front->next;
    x = s->data; // 将待删元素返回
    Q.front->next = s->next;
    if(Q.rear = s) // 当只有一个元素，此时 rear 指针需要回到头结点，不然就被释放掉了
        Q.rear = Q.front;
    free(s);
    return true;
}

// 读队头元素
bool GetHead(SqQueue Q, ElemType &x) {
    if(Q.front == Q.rear)
        return false;
    x = Q.front->next->data;
    return true;
}
```

### 双端队列

> 允许两端都可以进行入队和出队操作的队列。由此可以推广出输出受限的双端队列，即一端允许插入、删除，另一端只允许插入；与输入受限的双端队列，即一端允许插入、删除，另一端只允许删除。同栈一样，一般会考察输入输出次序的问题

