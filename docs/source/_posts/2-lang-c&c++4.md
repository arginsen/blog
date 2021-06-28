---
title: 语言 | C 与 C++ 语言基础Ⅳ：指针与数组
date: 2020-9-9 10:25:22
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

# 指针

> 指针保存了变量的地址，指针也可以改变指向变量的值
> 指针必须初始化，`int *p = NULL;`

```c
// 定义变量
int a = 10;
int b = 15;

// 取变量地址
&a // 取变量 a 的地址
&b // 取变量 b 的地址

// 定义指针
int *a_ = &a;
int *b_ = &b;
```

## 指针与函数参数

> 一般通过函数来修改传入值，需将参数 a 定义为引用型 [c++ 语法]

```c++
#include <iostream>
using namespace std;

void f(int &a)
{
    a = 10;
}
int main()
{
    int a = 0;
    f(a);
    cout<<a<<endl;
    return 0;
}
// 0
```

> 通过指针来修改变量的值

```c++
#include <iostream>
using namespace std;

void f(int *a)  // f 函数参数为整型指针型，通过指针取到变量 a 的值进而修改
{
    *a = 10;
}
int main()
{
    int a = 0;
    f(&a);  // &a 获取变量 a 的地址，传给函数 f
    cout<<a<<endl;
    return 0;
}
```

> 纯 c 下改变传入参数的值，实现引用的效果

- c++

```c++
int result = 0;
void getResult(int &r) // 取参数的地址
{
    ++r;
}

// 调用
getResult(result);
```

- c

```c
int result = 0;
void getResult(int *r) // 参数就等于传入的地址 *r=&result
{
    ++(*r); 
} // 用指针操控变量的值递增

// 调用
getResult(&result) // 取result的地址
```

> 纯 c 下改变指针型变量，实现引用的效果

- c++

```c++
int *p = NULL;
void getResult(int *&P)
{
    P = q;
}

// 调用
getResult(p)
```

- c

```c
int *p = NULL; // 整型指针型
void getResult(int **P) // 参数为指向指针的指针型
{
    *P = q; 
} // 将 q 指针赋值给 P 指针

// 调用
getResult(&p); // 得到 p 的地址
```

## 指向函数的指针

> 通过指针来调用选定的函数

```c++
#include <iostream>
using namespace std;

int add(int a, int b) // 定义加法函数
{
    return a + b;
}
int minu(int a, int b) // 定义减法函数
{
    return a - b;
}
int main()
{
    int (*p)(int, int); // 定义指向函数的指针 p，p 指向的函数有两个 int 型的参数，其返回值类型也为 int
    char op = '+';
    if(op == '+')
        p = add; // p 指向 add 函数
    else
        p = minu; // p 指向 minu 函数
    cout<<p(4, 3)<<endl; // 通过 p 指针调用指向的函数
    return 0;
}
```

# 数组

> 数组占据一块连续的存储空间
> `int a[10]` a 为数组的名称，也表示数组中第一个元素的地址

## 初始化数组

```c++
#include <iostream>
using namespace std;

int main()
{
    int a[10] = {0};
    for(int i = 0; i < 10; ++i)
        cout<<a[i]<<" ";
    cout<<endl;

    int b[10] = {1}; // 只给第一个元素赋值
    for(int i = o; i < 10; ++i)
        cout<<b[i]<<" ";
    cout<<endl;

    int c[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    for(int i = 0; i < 10; ++i)
        cout<<c[i]<<" ";
    cout<<endl;
    return 0;
}
// 0 0 0 0 0 0 0 0 0 0
// 1 0 0 0 0 0 0 0 0 0
// 1 2 3 4 5 6 7 8 9 10
```

## 通过指针和数组第一个元素地址访问数组

```c++
#include <iostream>
using namespace std;

int main()
{
    int a[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int *p = a; // 整形指针 p，等于数组 a 第一个元素地址
    for(int i = 0; i < 10; ++i)
        cout<<*(p + i)<<" ";
    cout<<endl;
    return 0;
}
```

## 字符型数组

```c
char s1[] = "I am a string"; // 长度为 14，会在字符串后自动加 \0，以结束字符串
char *s2 = "I am a string";  // 将字符串中第一个字符的地址初始化给 s2，s2 作为字符型指针

*s1 = 's'; // 修改数组 s1，=> "s am a string"
*s2 = 's'; // *s2 取到字符串第一个字符的地址并不能用来修改，*s2 表示常量
```

## 指针型数组

> 指针型数组即存储的元素均为指针
> `int*p[10]`

```c++
#include <iostream>
using namespace std;

int main()
{
    int a[10] = {1, 2, 3, 4, 5};
    int *p[5] = {a, a+1, a+2, a+3, a+4}; // 整型指针型数组 p，第一个元素为数组 a 的第一个元素的地址
    for(int i = 0; i < 5; ++i)
    {
        cout<<*p[i]<<" ";
    }
    cout<<endl;
    return 0;
}
// 1 2 3 4 5
```

> 数组作为函数参数，本身为第一个元素的地址，传入该函数时直接作为指针

```c++
#include <iostream>
using namespace std;

// 数组作为参数不用初始化，只要调用时传入相同类型数组即可
// 同时传入一个整型参数，用以调用函数时指明数组长度
void arrayFun(int a[], int n) // int *a 也可以
{                             
    for(int i = 0; i < n; ++i)
        ++a[i]; // 数组 a 的每个元素自增
    for(int i = 0; i < n; ++i)
        cout<<a[i]<<" "; // 输出数组每个元素
    cout<<endl;
}

int main()
{
    int a[5] = {1, 2, 3, 4, 5};
    arrayFun(a, 5); // 调用 arrayFun 函数时，传入主函数定义的数组 a，而 a 表示数组第一个元素的地址
    return 0;
}
// 2 3 4 5 6
```

## 二维数组

> 定义一个二维数组

```c++
int b[4][3] = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
    {10, 11, 12}
};

b[1][0] // 4
b[3][3] // 9
```

> 二维数组的遍历

```c++
#include <iostream>
using namespace std;

void array2D()
{
    for(int i = 0; i < n; ++i)
    {
        for(int j = 0; j < n; ++j)
            cout<<a[i][j]<<"\t"
        cout<<endl;
    }
}

int main()
{
    int b[4][3] = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
        {10, 11, 12}
    };
    array2D(b, 4);
    return 0;
}
// 1   2   3
// 4   5   6
// 7   8   9
// 10  11  12
```

> 二位字符串数组

```c++
#include <iostream>
using namespace std;

int main()
{
    char *s2D[] = {
        "I",
        "am",
        "a",
        "string"
    };
    for(int i = 0; i < 4; ++i)
        cout<<s2D[i]<<endl;
    return 0;
}
// I
// am
// a
// string
```



