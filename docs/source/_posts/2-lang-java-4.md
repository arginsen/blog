---
title: 语言 | Java 基础Ⅳ：面向对象编程
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

# 方法

一个class可以包含多个 field, 为了避免外部代码直接去访问field，我们可以用private修饰field，拒绝外部访问; 使用方法（method）来让外部代码可以间接修改field

```java
public class Main {
    public static void main(String[] args) {
        Person ming = new Person();
        ming.setName("Xiao Ming"); // 设置name
        ming.setAge(12); // 设置age
        System.out.println(ming.getName() + ", " + ming.getAge());
    }
}

class Person {
    private String name;
    private int age;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return this.age;
    }

    public void setAge(int age) {
        if (age < 0 || age > 100) {
            throw new IllegalArgumentException("invalid age value");
        }
        this.age = age;
    }
}
```

## private 方法

private 方法是保证只在内部可以调用的方法

```java
class Person {
    private String name;
    private int birth;

    public void setBirth(int birth) {
        this.birth = birth;
    }

    public int getAge() {
        return calcAge(2019); // 调用private方法
    }

    // private方法:
    private int calcAge(int currentYear) {
        return currentYear - this.birth;
    }
}
```

## this 变量

在方法内部, 可以使用一个隐含的变量 this，它始终指向当前实例。

```java
// 如果没有命名冲突，可以省略this。例如：
class Person {
    private String name;

    public String getName() {
        return name; // 相当于this.name
    }
}

// 但是，如果有局部变量和字段重名，那么局部变量优先级更高，就必须加上this：
class Person {
    private String name;

    public void setName(String name) {
        this.name = name; // 前面的this不可少，少了就变成局部变量name了
    }
}
```

## 方法参数

```java
class Person {
    ...
    public void setNameAndAge(String name, int age) {
        ...
    }
}

// 调用这个 setNameAndAge() 方法时，必须有两个参数，
// 且第一个参数必须为String，第二个参数必须为int：
Person ming = new Person();
ming.setNameAndAge("Xiao Ming"); // 编译错误：参数个数不对
ming.setNameAndAge(12, "Xiao Ming"); // 编译错误：参数类型不对
```

## 可变参数

可变参数用类型...定义，可变参数相当于数组类型; 当然也可以直接将参数指定未 `String[]` 类型, 但是调用方就需要先构造 `String[]`, 此外调用方也可以传入 null

```java
class Group {
    private String[] names;

    public void setNames(String... names) {
        this.names = names;
    }
}

Group g = new Group();
g.setNames("Xiao Ming", "Xiao Hong", "Xiao Jun"); // 传入3个String
g.setNames("Xiao Ming", "Xiao Hong"); // 传入2个String
g.setNames("Xiao Ming"); // 传入1个String
g.setNames(); // 传入0个String
```

# 构造方法

与类同名的方法, 接收参数用于创建实例时初始化

任何类 class 都有构造方法, 如果一个类没有定义构造方法, 编译器会自动为我们生成一个默认的不带参数的构造方法; 

如果我们定义了构造方法, 则编译器不会再添加了; 所以既要能使用带参数的构造方法，又想保留不带参数的构造方法，那么只能把两个构造方法都定义出来;

可以同时定义多个构造方法, 在通过 new 操作符调用时, 编译器会根据构造方法的参数数量、位置和类型自动区分

```java
public class Main {
    public static void main(String[] args) {
        Person p = new Person("Xiao Ming", 15);
        System.out.println(p.getName());
        System.out.println(p.getAge());
    }
}

class Person {
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return this.name;
    }

    public int getAge() {
        return this.age;
    }
}
```

# 方法重载

在一个类中，我们可以定义多个方法。如果有一系列方法，它们的功能都是类似的，只有参数有所不同，那么，可以把这一组方法名做成同名方法, 这种方法名相同，但各自的参数不同，称为方法重载（Overload）。

方法重载的返回值类型通常都是相同的。
方法重载的目的是，功能类似的方法使用同一名字，更容易记住，因此，调用起来更简单。

# 继承

继承是面向对象编程中非常强大的一种机制，它首先可以复用代码。一个类可以从另一个类继承属性和方法, 新的类只需要编写新的方法.

Java只允许一个class继承自一个类，因此，一个类有且仅有一个父类

继承有个特点，就是子类无法访问父类的private字段或者private方法。

```java
class Person {
    private String name;
    private int age;

    public String getName() {...}
    public void setName(String name) {...}
    public int getAge() {...}
    public void setAge(int age) {...}
}

class Student extends Person {
    // 不要重复name和age字段/方法,
    // 只需要定义新增score字段/方法:
    private int score;

    public int getScore() { … }
    public void setScore(int score) { … }
}
```

## protected 

protected 关键字可以把字段和方法的访问权限控制在继承树内部，一个 protected 字段和方法可以被其子类，以及子类的子类所访问

```java
class Person {
    protected String name;
    protected int age;
}

class Student extends Person {
    public String hello() {
        return "Hello, " + name; // OK!
    }
}
```

## super

一般如果没有明确地调用父类的构造方法，编译器会帮我们自动加一句super(), 而此处的 super() 则是执行无参数的构造方法

如果父类没有默认的构造方法，子类就必须显式调用super()并给出参数以便让编译器定位到父类的一个合适的构造方法。

```java
public class Main {
    public static void main(String[] args) {
        Student s = new Student("Xiao Ming", 12, 89);
    }
}

class Person {
    protected String name;
    protected int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

class Student extends Person {
    protected int score;

    public Student(String name, int age, int score) {
        super(name, age); // 调用父类的构造方法Person(String, int)
        this.score = score;
    }
}
```