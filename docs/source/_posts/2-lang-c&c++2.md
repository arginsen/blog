---
title: 语言 | C 与 C++ 语言基础Ⅱ：控制流
date: 2020-9-8 9:25:22
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

# 条件语句

## if else 语句

```c++
if(表达式)
    语句块1;
else
    语句块2;
```

## else if 语句

```c++
if(表达式1)
    语句块1;
else if(表达式2)
    语句块2;
else if(表达式3)
    语句块3;
else
    语句块4;
```

## switch 语句

```c++
switch(表达式)
{
    case 常量表达式1 : 语句系列1; break;
    case 常量表达式2 : 语句系列2; break;
    case 常量表达式3 : 语句系列3; break;
    default : 语句序列4;
}
```

# 循环语句

## while 循环

```c
while(表达式)
    语句块
```

> 表达式值为真，则循环执行语句块中的内容，直到表达式值为假为止
> 循环体内可以用 `break` 语句配合 if 语句来适时结束循环
> 循环体内可以用 `continue` 语句配合 if 语句来适时跳过当前循环

```c++
// break 语句
#include <iostream>
using namespace std;

int main()
{
    int i = 1;
    while(true)
    {
        cout<<i<<" ";
        if(i == 10)
            break;
        i++;
    }
    return 0;
}
// 1 2 3 4 5 6 7 8 9 10

// continue 语句
#include <iostream>
using namespace std;

int main()
{
    int i = 0;
    while(i <= 5)
    {
        i++;
        if(i == 3)
            continue;
        cout<<i<<" ";
    }
    return 0;
}
// 1 2 4 5 6
```

## for 循环

```c++
for(int i = 0; i <= 3; i++)
    cout<<i<<"";
return 0;

// 无限循环
while(true)
for(;;)
```

## do-while 循环

> 先做，至少执行一次

```c++
do
{
    语句块;
}
while(表达式);

// break 必须在循环体内，以此可以终止 do-while 中的循环
do
{
    语句块1;
    语句块2;
    if(表达式)
        break;
    语句块3;
    语句块4;
}
while(false)
```