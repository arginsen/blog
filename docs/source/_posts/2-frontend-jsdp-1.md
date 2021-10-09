---
title: 前端 | JS 设计模式 | 观察者模式
date: 2021-7-8
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# 概念

观察者模式(Observer Pattern)：定义对象间的一种「一对多依赖关系」，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式是一种对象行为型模式。

# 模式特点

在观察者模式中，通常包含以下角色：

- 「目标：Subject」
- 「观察目标：ConcreteSubject」
- 「观察者：Observer」
- 「具体观察者：ConcreteObserver」

**优点**

- 观察者模式可以实现「表示层和数据逻辑层的分离」，并「降低观察目标和观察者之间耦合度」；
- 观察者模式支持「简单广播通信」，「自动通知」所有已经订阅过的对象；
- 观察者模式「符合“开闭原则”的要求」；
- 观察目标和观察者之间的抽象耦合关系能够「单独扩展以及重用」。

**缺点**

- 当一个观察目标「有多个直接或间接的观察者」时，通知所有观察者的过程将会花费很多时间；
- 当观察目标和观察者之间存在「循环依赖」时，观察目标会触发它们之间进行循环调用，可能「导致系统崩溃」。
- 观察者模式缺少相应机制，让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

# 使用场景

在以下情况下可以使用观察者模式：

- 在一个抽象模型中，一个对象的行为「依赖于」另一个对象的状态。即当「目标对象」的状态发生改变时，会直接影响到「观察者」的行为；
- 一个对象需要通知其他对象发生反应，但不知道这些对象是谁。
- 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。

# 示例

```ts
// 观察目标接口
interface Subject {
  addObserver: (observer: Observer) => void;
  deleteObserver: (observer: Observer) => void;
  notifyObservers: () => void;
}

// 观察者接口
interface Observer {
  notify: () => void;
}

// 具体观察目标类
class ConcreteSubject implements Subject{
  private observers: Observer[] = [];

  // 添加观察者
  public addObserver(observer: Observer): void {
    console.log(observer, " is pushed~~");
    this.observers.push(observer);
  }

  // 移除观察者
  public deleteObserver(observer: Observer): void {
    console.log(observer, " have deleted~~");
    const idx: number = this.observers.indexOf(observer);
    ~idx && this.observers.splice(idx, 1);
  }

  // 通知观察者
  public notifyObservers(): void {
    console.log("notify all the observers ", this.observers);
    this.observers.forEach(observer => {
      // 调用 notify 方法时可以携带指定参数
      observer.notify();
    });
  }
}

// 具体观察者类
class ConcreteObserver implements Observer{
  constructor(private name: string) {}

  notify(): void {
    // 可以处理其他逻辑
    console.log(`${this.name} has been notified.`);
  }
}

// 测试用例
function useObserver(): void {
  // 生成观察目标实例
  const subject: Subject = new ConcreteSubject();
  // 生成诸多观察者
  const Leo   = new ConcreteObserver("Leo");
  const Robin = new ConcreteObserver("Robin");
  const Pual  = new ConcreteObserver("Pual");
  const Lisa  = new ConcreteObserver("Lisa");

  // 给观察目标实例添加诸多观察者
  subject.addObserver(Leo);
  subject.addObserver(Robin);
  subject.addObserver(Pual);
  subject.addObserver(Lisa);
  // 通知观察者
  subject.notifyObservers();

  // 移除观察者后再通知现有观察者
  subject.deleteObserver(Pual);
  subject.deleteObserver(Lisa);
  subject.notifyObservers();
}

// 执行
useObserver();
```