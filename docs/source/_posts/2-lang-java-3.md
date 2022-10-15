---
title: 语言 | Java 基础Ⅲ：数组
date: 2021-12-16
tags: 
- Java
- Backend
categories: notes
photos:
    - /blog/img/java.jpg
---

<br>
<!--more-->

# 定义

和单个基本类型变量不同，数组变量初始化必须使用 `new int[5]` 表示创建一个可容纳5个 int 元素的数组。

- 数组所有元素初始化为默认值，整型都是0，浮点型是0.0，布尔型是false；
- 数组一旦创建后，大小就不可改变。
- 数组是引用类型，在使用索引访问数组元素时，如果索引超出范围，运行时将报错

也可以定义数组时直接指定初始化的元素

```java
public class Main {
    public static void main(String[] args) {
        int[] ns = new int[5];
        ns[0] = 1
        System.out.println(ns.length); // 5
        System.out.println(ns[5]); // 报错
    }
}

// 指定初始化
public class Main {
    public static void main(String[] args) {
        int[] ns = new int[] { 68, 79, 91, 85, 62 };
        int[] ns2 = { 68, 79, 91, 85, 62 }; // 简写
        System.out.println(ns.length);
    }
}
```

数组元素不是基本类型, 而是引用类型字符串, 下边代码 s 获得的是字符串 "XYZ", 而通过 `names[1]` 修改的是 names 数组的值; 变量 s 的指向并没有变动

```java
public class Main {
    public static void main(String[] args) {
        String[] names = {"ABC", "XYZ", "zoo"};
        String s = names[1];
        names[1] = "cat";
        System.out.println(s); // "XYZ"
    }
}
```

# 遍历数组

for 循环

```java
public class Main {
    public static void main(String[] args) {
        int[] ns = { 1, 4, 9, 16, 25 };
        for (int i=0; i<ns.length; i++) {
            int n = ns[i];
            System.out.println(n);
        }
    }
}
```

for each 循环, 直接拿到数组的元素

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

# 数组排序

## 冒泡排序

冒泡排序的特点是，每一轮循环后，最大的一个数被交换到末尾，因此，下一轮循环就可以“刨除”最后的数，每一轮循环都比上一轮循环的结束位置靠前一位。

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] ns = { 28, 12, 89, 73, 65, 18, 96, 50, 8, 36 };
        System.out.println(Arrays.toString(ns));

        for (int i = 0; i < ns.length - 1; i++) {
            for (int j = 0; j < ns.length - i - 1; j++) {
                if (ns[j] > ns[j + 1]) {
                    int tmp = ns[j];
                    ns[j] = ns[j + 1];
                    ns[j + 1] = tmp;
                }
            }
        }

        System.out.println(Arrays.toString(ns));
    }
}
```

# 二维数组

```java
public class Main {
    public static void main(String[] args) {
        int[][] ns = {
            { 1, 2, 3, 4 },
            { 5, 6, 7, 8 },
            { 9, 10, 11, 12 }
        };
        System.out.println(ns.length); // 3
        System.out.println(ns[0].length); // 4
        System.out.println(ns[1][2]); // 7

        // 打印一个二维数组
        System.out.println(Arrays.deepToString(ns));
    }
}
```




