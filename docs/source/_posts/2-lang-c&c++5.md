---
title: 语言 | C 与 C++ 语言基础Ⅴ：结构体和类
date: 2020-9-9 16:20:22
tags: 
- C
- C++
- 考研
hide: 
categories: notes
photos:
    - /blog/img/c&c++.jpg
---

<br>
<!--more-->

# 类型定义

> 使用 `typedef` 给数据类型命名新的名称

```c
typedef int myint;
myint a = 0;

typedef char mychar;
mychar c = 'c';

printf("%d\n", a); // 0
printf("%c\n", c); // c

// 使用 printf 进行输出，可对输出类型进行说明
// 使用 cout 输出是无视类型的，即任何变量都可以直接输出

// 常用的一些说明符
%a  浮点数
%c  一个字符
%d  有符号的十进制整数
%o  无符号的八进制整数
%s  对应字符串
```

# 结构体

> 结构体用以存放一组不同类型的数据。结构体是一种集合，里边包含了多个变量或数组，它们的类型可相同，也可不同。

```c++
struct {int x; int y;} point; // 给结构体命名 point
point.x = 10;
point.y = 11;
```

> 使用类型定义 typedef 简化结构体的声明

```c++
typedef struct {int x; int y;} Point; // 给结构体数据类型定义新的名称 Point
Point point1; // 用 Point 类型再声明新的变量
point1.x = 10;
point1.y = 11;

Point point2;
```

> 结构体取值

```c++
#include <stdio.h>

typedef struct
{
    int x;
    int y;
}Point;

int main()
{
    Point point;
    point.x = 10;
    point.y = 11;
    printf("%d, %d", point.x, point.y);
    return 0;
}
// 10 11
```

## 指向结构体的指针

```c++
#include <stdio.h>

typedef struct
{
    int x;
    int y;
}Point;

int main()
{
    Point point; // 定义变量 point
    Point *P; // 定义 Point 型的指针
    P = &Point; // 用 P 指向变量 Point
    P->x = 10; // 使用指针赋值
    p->y = 11;
    printf("%d, %d", P -> x, P -> y); // 使用指针取值
    return 0;
}
```

## 自引用结构

> 在结构体内部定义指向自己类型的指针

```c++
#include <stdio.h>

typedef struct Point
{
    int x;
    int y;
    struct Point *next;
}Point;

int main()
{
    Point p1, p2, p3;
    Point *P;
    p1.x = 1; p1.y = 0;
    p2.x = 4; p2.y = 1;
    p3.x = 2; p3.y = 4;
    p1.next = &p2;
    p2.next = &p3;
    p3.next = NULL;

    for(P = &p1; P != NULL; P = P -> next)
        printf("(%d, %d)\n", P -> x, P -> y);
    return 0;
}
// (1, 0)
// (4, 2)
// (2, 4)
```

## c++ 下的结构体

```c++
#include <iostream>
using namespace std;

struct Point
{
    int x;
    int y;
    Point *next; // 定义指针型变量
    Point() // 定义构造函数
    {
        next = NULL; // 对内部变量初始化
        x = 0;
        y = 0;
    }
    void print(); // 结构体内声明函数
};

void Point::print() // 双冒号表示 print 函数属于 Point 结构体，在外部具体定义函数内容
{
    cout<<"("<<this -> x<<","<<this -> y<<")"<<endl; // this 指向结构体的存储空间（结构体变量），通过 this 访问结构体成员变量
}

int main()
{
    Point p1, p2, p3;
    Point *P;
    p1.x = 1; p1.y = 0;
    p2.x = 4; p2.y = 1;
    p3.x = 2; p3.y = 4;
    p1.next = &p2;
    p2.next = &p3;
    p3.next = NULL;

    for(P = &p1; P != NULL; P = P -> next)
        P -> print();
    return 0;
}
// (1, 0)
// (4, 2)
// (2, 4)
```

# 类

```c++
class Point // 用 class 来定义类
{
    public: // 用 public 可被外部取到类中的成员变量
        int x;
        Point() // 构造函数
        {
            //...
        }
        Point *next; // 定义指针型变量
};
```

## 定义一个类

> 通过在类中用 public 和 private 来指明类中能否可被外部访问到的变量

```c++
#include <iostream>
using namespace std;

class Point
{
    public:
        Point()
        {
            x = 0;
            y = 0;
            next = NULL;
        }
    private:
        int x;
        int y;
        Point *next;
};

int main()
{
    Point P;
    P.x;
    return 0;
}
// error: 'int Point::x' is private...
```

> 通过在 public 定义函数来取到 private 中的变量

```c++
#include <iostream>
using namespace std;

class Point
{
    public:
        Point()
        {
            this -> x = 0;
            this -> x = 0;
            this -> next = NULL;
        }
        // 声明各公有函数，实现存值取值
        int getX();
        int getY();
        void inputX(int x); // 存值不需要返回值
        void inputY(int y);
        Point *getNext();
        void inputNext(Point *next);
    private:
        int x;
        int y;
        Point *next;
};

// 分别定义函数具体操作
int Point::getX() {return this -> x;}
int Point::getY() {return this -> y;}
void Point::inputX() {return this -> x = x;}
void Point::inputY() {return this -> y = y;}
Point *Point::getNext() {return this -> next;}
void Point::inputNext() {return this -> next = next;}

int main()
{
    Point p1, p2;
    p1.inputX(10); // 给变量 x 存值
    cout<<p1.getX()<<endl; // 输出变量 p1 的 x
    cout<<p1.getNext()<<endl; // 输出变量 p1 的 next（默认 NULL）

    p1.inputNext(&p2); // p1.next = p2  使 p1 的 next 指针指向 p2 的地址
    cout<<p1.getNext()<<endl;
    return 0;
}
// 10
// 0
// 0x7ffd8ee80600
```

# 结构体与类的空间分配

> 结构体使用 malloc 函数指定空间大小并返回空间地址

```c++
#include <stdio.h>
#include <stdlib.h>

typedef struct Point
{
    int x;
    int y;
    struct Point *next;
}Point;

int main()
{
    // 使用 sizeof 计算结构体空间大小，malloc 返回空间地址，
    // 再进行 Point 类型指针的强制类型转换，再令 P 指针指向该地址
    Point *P = (Point*)malloc(sizeof(Point)); 
    printf("%d, %d", P -> x, P -> y);
    return 0;
}
// 0, 0
```

> 类直接通过 new 动态分配存储空间

```c++
#include <iostream>
using namespace std;

class Point
{
    public:
    Point()
    {
        this -> x = 0;
        this -> y = 0;
        this -> next = NULL;
    }
    int x;
    int y;
    Point *next;
}

int main()
{
    Point *P = new Point; // 通过 new 就进行了空间存储的动态分配，返回空间地址再赋值给指针 P
    cout<<P -> x<<", "<<P -> y<<endl;
    return 0;
}
// 0, 0
```