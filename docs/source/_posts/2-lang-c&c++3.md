---
title: 语言 | C 与 C++ 语言基础Ⅲ：函数
date: 2020-9-8 15:25:22
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

# 函数定义

```c++
返回值类型 函数名(参数列表)
{
    声明和语句;
    return X;
}
```

1. 当返回值类型为 `void`，说明函数不用返回值，使用 `return` 结束函数

```c++
#include <iostream>
using namespace std;

int f1()
{
    return 1;
}
// 1

float f2()
{
    return 0.1f;
}
// 0.1

char f3()
{
    return 'A';
}
// A

void f4()
{
    // ...
    return;
}
```

2. 参数列表，需要定义参数的数据类型

```c++
#include <iostream>
using namespace std;

int add(int a, int b)
{
    int c = a + b;
    return c;
}
```

# 函数调用

```c++
#include <iostream>
using namespace std;

void increase(int x)
{
    for(int i = 1; i <= 5; ++i)
        ++x;
}

int main()
{
    int a = 0;
    increase(a); // 传入参数为变量 a，实际上是 a 赋值给 x，x 自增，a 还是原来的值
    cout<<a;
    return 0;
}
// 0
```

```c++
// &x 为 c++ 的语法，&x 为引用型，直接操作传入的变量 a，且参数传入必须为变量
// c 语言需要通过指针来改变值
#include <iostream>
using namespace std;

void increase(int &x) // 使参数为引用型
{
    for(int i = 1; i <= 5; ++i)
        ++x;
}

int main()
{
    int a = 0;
    increase(a); // 传入变量，如果传入常量则会报错：increase(10)
    cout<<a;
    return 0;
}
// 5
```

# 作用域

```c++
// 函数内部可以使用到外部的变量
#include <iostream>
using namespace std;

int a = 10;
void f()
{
    ++a;
    cout<<a<<endl;
}

int main()
{
    cout<<a<<endl;
    f();
    return 0;
}
// 10
// 11
```


```c++
#include <iostream>
using namespace std;

int a = 10;
void f()
{
    int a = 5;
    cout<<a<<endl; // 此时输出局部变量 a
}

int main()
{
    cout<<a<<endl;
    f();
    return 0;
}
// 10
// 5
```

> 一对花括号 `{}` 包起来的区域也叫语句块，会形成一个作用域

```c++
#include <iostream>
using namespace std;

int main()
{
    {
        int a = 0;
    }
    cout<<a<<endl; // 此时读取不到上边花括号内部的变量 a
    return 0;
}
// 'a' was not defined
```

# 静态变量

```c++
#include <iostream>
using namespace std;

void f()
{
    static int a = 0; // a 被定义为函数 f 的静态变量
    ++a;
    cout<<a<<" ";
}

int main()
{
    for(int i = 0; i <= 5; ++i)
    {
        f(); // 多次执行函数 f，操作的都是同一个 a，a 每次递增后的状态都会更新，如果不是静态变量，则状态在每次调用后都重计，最后输出 6 个 1
    }
    return 0;
}
// 1 2 3 4 5 6
```

# 递归

> 一个递归入口

```c++
#include <iostream>
using namespace std;

int i = 0;
void r()
{
    cout<<i<<" ";
    if(i < 3)
    {
        ++i;
        r();
    }
}

int main()
{
    r();
    return 0;
}
// 0 1 2 3
```

> 两个递归入口

```c++
void r()
{
    static int i = 0;
    if(i < 2)
    {
        ++i;
        r(); // 叠加至 i = 2 退出，执行下个入口

        r(); // 此时 i 已经等于 2，退出
    }
}
```