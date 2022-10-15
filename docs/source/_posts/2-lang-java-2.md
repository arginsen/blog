---
title: 语言 | Java 基础Ⅱ：流程控制
date: 2021-12-15
tags: 
- Java
- Backend
categories: notes
photos:
    - /blog/img/java.jpg
---

<br>
<!--more-->

# 输入和输出

## 输出

println 是 print line 的缩写, 意为输出并换行
print 则是普通的输出

```java
public class Main {
    public static void main(String[] args) {
        System.out.print("A,");
        System.out.print("B,");
        System.out.print("C.");
        System.out.println();
        System.out.println("END");
    }
}
```

## 格式化输出

将输出的数据转换为期望的格式, 使用 printf 配合占位符

```java
public class Main {
    public static void main(String[] args) {
        double d = 3.1415926;
        System.out.printf("%.2f\n", d); // 显示两位小数3.14
        System.out.printf("%.4f\n", d); // 显示4位小数3.1416
    }
}
```

占位符	说明
%d	格式化输出整数
%x	格式化输出十六进制整数
%f	格式化输出浮点数
%e	格式化输出科学计数法表示的浮点数
%s	格式化字符串

## 输入

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in); // 创建Scanner对象
        System.out.print("Input your name: "); // 打印提示
        String name = scanner.nextLine(); // 读取一行输入并获取字符串
        System.out.print("Input your age: "); // 打印提示
        int age = scanner.nextInt(); // 读取一行输入并获取整数
        System.out.printf("Hi, %s, you are %d\n", name, age); // 格式化输出
    }
}
```

# 条件判断

java 中的判断使用 if , else if, else, 和 JavaScript 的语法相同;

因为 java 定义变量会有类型定义, 所以判断条件是否相等使用 ==, 不等使用 !==

对于引用类型的数据, 判断需要使用 equals 方法, 且调用该方法的变量为 null 会报错, 因此可以使用 短路运算符&&, 前边条件为 true 再执行右边的语句

```java
public class Main {
    public static void main(String[] args) {
        String s1 = null;
        if (s1 != null && s1.equals("hello")) {
            System.out.println("hello");
        }
    }
}
```

# switch 多重选择

switch 用法与 JavaScript 一致
default 用于当没有匹配到任何 case 时的通道
break 用于匹配到 case 执行语句后的终止

```java
public class Main {
    public static void main(String[] args) {
        int option = 1;
        switch (option) {
        case 1:
            System.out.println("Selected 1");
            break;
        case 2:
            System.out.println("Selected 2");
            break;
        case 3:
            System.out.println("Selected 3");
            break;
        default:
            System.out.println("Not selected");
            break;
        }
    }
}
```

如果有几个case语句执行的是同一组语句块，可以将 case 写在一起; 下边代码表示 case 2 与 case 3 执行相同的语句

```java
public class Main {
    public static void main(String[] args) {
        int option = 2;
        switch (option) {
        case 1:
            System.out.println("Selected 1");
            break;
        case 2:
        case 3:
            System.out.println("Selected 2, 3");
            break;
        default:
            System.out.println("Not selected");
            break;
        }
    }
}
```

# while 循环

while 循环首先会判断条件是否成立, 成立再执行语句, 否则直接跳过循环

```java
public class Main {
    public static void main(String[] args) {
        int sum = 0; // 累加的和，初始化为0
        int n = 1;
        while (n <= 100) { // 循环条件是n <= 100
            sum = sum + n; // 把n累加到sum中
            n ++; // n自身加1
        }
        System.out.println(sum); // 5050
    }
}
```

# for 循环

```java
public class Main {
    public static void main(String[] args) {
        int[] ns = { 1, 4, 9, 16, 25 };
        int sum = 0;
        for (int i=0; i<ns.length; i++) {
            System.out.println("i = " + i + ", ns[i] = " + ns[i]);
            sum = sum + ns[i];
        }
        System.out.println("sum = " + sum);
    }
}
```

# for each 循环

for each 循环用于遍历数组, n 就为 ns 数组中的元素

```java
public class Main {
    public static void main(String[] args) {
        int[] ns = { 1, 4, 9, 16, 25 };
        for (int n : ns) {
            System.out.println(n);
        }
    }
}
```

# break 与 continue

在循环过程中，可以使用 break 语句跳出当前循环。break 语句通常都是配合 if 语句使用; 且 break 只能跳出一层循环

continue 是提前结束本次循环，直接继续执行下次循环。一般情况下也是配合 if 语句, 如果满足条件则直接跳过本次循环的其他语句