---
title: 前端 | Vue3 的组合式 API
date: 2021-8-20
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

# 组合式 API 的目的

1. 更好的逻辑复用与代码组织
2. 更好的类型推导

## 代码组织

vue 在构建中小型项目会比较轻松, 但面对大型项目时, 原来的代码组织会使得复杂组件变得难以阅读和理解, 每次看起也只能从 data methods computed 这些选项看起, 并不能较好的读懂该组件到底实现了什么; 

组合式 API 意在将处理一类内容的操作放入一个组合函数, 而 setup 则是简单的作为调用所有组合函数的入口. 那么我们在接触一个陌生的复杂组件时, 便可通过 setup 来认知该组件实现了什么, 同时根据传递的参数了解到各组合函数之间的依赖关系, 最后的 return 语句作为单一的出口确认暴露给模板的内容

## 逻辑提取和复用

组合式 API 的出现将代码组织为处理特定功能的函数, 使得组件之间甚至组件之外逻辑的提取和重用变得更简单. 一个组合函数仅依赖它的参数和 Vue 全局导出的 API , 而不是依赖其微妙的 this 上下文, 这样就可以通过导出整个 setup 函数达到和 extends 等价的效果

# 参考

Vue 组合式 API: https://vue3js.cn/vue-composition/

Vue 组合式 API: https://vue3js.cn/vue-composition-api/

