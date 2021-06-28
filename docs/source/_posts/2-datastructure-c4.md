---
title: 数据结构 | C 与 C++ 描述 | 四、串
date: 2020-10-10
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

> 串（String）是由零个或多个字符组成的有限序列。无字符时成为空串。串中任意个连续的字符组成的子序列成为该串的子串，包含子串的串相应地称为主串。子串在主串中的位置用子串的第一个字符在主串中的位置来表示。

# 串的基本操作

- StrAssign(&T, chars)：赋值。将串 T 赋值给 chars
- StrCopy(&T, S)：复制。由串 S 复制得串 T
- StrEmpty(S)：判空。
- StrCompare(S, T)：比较。若 `S>T`，则返回值 `>0`，若 S=T，则返回值 `=0`，若 `S<T`，则返回值 `<0`
- StrLength(S)：求串长。
- SubString(&Sub, S, pos, len)：求子串。用 Sub 返回串 S 的第 pos 个字符起长度为 len 的子串。
- Concat(&T, S1, S2)：联接串。用 T 返回 S1 和 S2 联接而成的新串。
- Index(S, T)：定位操作。若主串 S 中存在与串 T 相同的子串，返回其在主串 S 中第一次出现的位置，否则返回 0。
- ClearString(&S)：清空串 S。
- DestroyString(&S)：销毁串 S。

# 串的存储结构

## 定长顺序存储

> 用一组地址连续的存储单元存储串值的字符序列。

```c++
#define MAXLEN 255

typedef struct{
    char ch[MAXLEN]; // 串值序列长度超出 MAXLEN，多出的字符会被截断
    int length;
}SString;
```

## 堆分配存储

> 以一组地址连续的存储单元存放串值的字符序列，但其存储空间是动态分配的。

```c++
#include <stdlib.h>

typedef struct{
    char *ch;
    int length;
}HString;
```

## 块链存储

> 用链式存储结构存储串值，由于串的每个元素只有一个字符，因此每个结点可以存放一个字符，也可以存放多个字符（每个元素同时）。每个结点称为块，整个链表称为块链结构。

# 串的模式匹配

> 子串的定位操作通常称为串的模式匹配，目的是求子串（模式串）在主串中的位置。一般有暴力匹配算法，KMP 算法，KMP 优化的算法

## 暴力匹配算法

```c++
int Index(SString S, SString T) {
    int i = 1, j = 1;
    while(i <= S.length && j <= T.length) {
        if(S.ch[i] == T.ch[j]) {
			j++; i++;
		} else {
			i = i - j + 2; j = 1; // 或者可添加一个辅助定位的 k 记位
		}
	}
    if(j > T.length)
		return i - T.length;
	else
		return 0;
}
```

> 代码实现：串的定义，初始化赋值，及简单模式匹配

```c++
#include <stdio.h>
using namespace std;
#define MAXLEN 255

typedef struct{
	char ch[MAXLEN];
	int length;
}SString;

void InitString(char *s, SString &S) {
	int i = 0;
	char *p = S.ch; // 用指针指向串用以修改数据
	*p = ' '; // 令自定义的串结构第 0 位为空
    // S 从第 1 位开始逐位被 s 赋值
	while(*++p = *s++) {
		i++;
	}
	S.length = i;
}

int Index(SString S, SString T) {
	int i = 1, j = 1;
	while(i <= S.length && j <= T.length) {
		printf("%c, %c\n", S.ch[i], T.ch[j]);
		if(S.ch[i] == T.ch[j]) {
			++j; ++i;
		} else {
			i = i - j + 2; j = 1;
		}
	}
	printf("%d, %d \n", i, j); 
	if(j > T.length)
		return i - T.length;
	else
		return 0;
}

int main() {
	SString S, T;
	char t[] = "defg", s[] = "abcdefg";
	printf("【定义字符串：char[]】%s, %s \n", s, t);

	InitString(s, S);
	printf("【初始化主串：SString】%s ：%d \n", S.ch, S.length);
	InitString(t, T);
	printf("【初始化模式串：SString】%s ：%d \n", T.ch, T.length);

	printf("'%s' starts at position %d of '%s'", T.ch, Index(S, T), S.ch);
	return 0;
}
}
```

## KMP 算法

> 得到子串的 next 数组，通过 next 数组进行串的模式匹配

主串："abcabcabcacaba"
子串："abcac"

```c++
// 得到 next 数组
void get_next(SString T, int next[]) {
    int i = 1, j = 0;
    next[1] = 0;
    while(i < T.length) {
        if(j == 0 || T.ch[i] == T.ch[j]) {
            i++; j++;
            next[i] = j;
        } else {
            j = next[j];
        }
    }
}

// 模式匹配 ：S 为主串，T 为子串
int Index_KMP(SString S, SString T, int next[]) {
    int i = 1, j = 1;
    get_next(T, next);
    while(i <= S.length && j <= T.length) {
        if(j == 0 || S.ch[i] == T.ch[j]) {
            i++; j++;
        } else {
            j = next[j];
        }
    }
    if(j > T.length)
        return i - T.length;
    else
        return 0;
}
```

## KMP 算法的优化

> 对 next 数组进行改进，相比 next 数组，nextval 是在两指针（i, j）在串中指向的值相等并递增（i, j）后继续判断在串中是否相等，如果相等，就令 `nextval[i] = nextval[j]`（当前的 i 指向的值等于前边 j 指向的值），如果不等，就原按照 next 处理方法，将 `nextval[j] = j`

```c++
// 得到 nextval 数组
void get_nextval(SString T, int next[]) {
    int i = 1, j = 0;
    next[1] = 0;
    while(i < T.length) {
        if(j == 0 || T.ch[i] == T.ch[j]) {
            i++; j++;
            if(T.ch[i] == T.ch[j]) // 判断递增后两者是否相等
                nextval[i] = nextval[j]; // 如果相等则给数组 i 位赋上 j 位的值
            else
                nextval[i] = j; // 如果不等则依然保留 i 位的值等于当前的 j - 相当于原 next 数组
        } else {
            j = nextval[j];
        }
    }
}

```


