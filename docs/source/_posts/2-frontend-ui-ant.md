---
title: 前端 | Ant-UI for Vue 使用体验
date: 2020-7-18 15:20:33
tags: 
- Vue
- Frontend
- Ant-UI
photos:
    - /blog/img/vue.jpg
categories:
- practice
---
<br>
<!-- more -->


# 使用

```bash
# 在 vue 中用脚手架工具创建项目，安装脚手架工具
$ npm install -g @vue/cli
# OR
$ yarn global add @vue/cli

# 创建一个项目
$ vue create antd-demo

# 在项目中安装 ant-design-vue
$ npm i --save ant-design-vue


# 在 App.vue 中完整引入
import Vue from 'vue';
import Antd from 'ant-design-vue';
import App from './App';
import 'ant-design-vue/dist/antd.css';
Vue.config.productionTip = false;

Vue.use(Antd);

/* eslint-disable no-new */
new Vue({
  el: '#app',
  components: { App },
  template: '<App/>',
});
```


# 组件

## menu

> mode 可以确认菜单栏的布局是水平还是竖直排布的，有多中形式供选择。

```html
<a-menu
  class="menu"
  mode="horizontal">
  <a-menu-item key="1">
    <router-link to="/">Home</router-link>
  </a-menu-item>
  <a-menu-item key="2">
    <router-link to="/cart">Cart</router-link>
  </a-menu-item>
  <a-sub-menu key="3">
    <span slot="title">Admin</span>
    <a-menu-item key="3a">
      <router-link to="/admin">查看商品</router-link>
    </a-menu-item>
    <a-menu-item key="3b">
      <router-link to="/admin/new">添加商品</router-link>
    </a-menu-item>
    <a-menu-item key="3c">
      <router-link to="/admin/manufacturers">查看生产商</router-link>
    </a-menu-item>
    <a-menu-item key="3d">
      <router-link to="/admin/manufacturers/new">添加生产商</router-link>
    </a-menu-item>
  </a-sub-menu>
</a-menu>

<router-view/>
```

## breadcrumb

> 可以用来做一个当前的 `pwd` 展示，item 按从上往下依次进入目录，使用路由提供的 `this.$route` 对象，可以获得当前的路由名和路径

```html
<a-breadcrumb class="admin-index">
  <a-breadcrumb-item>
    Admin
  </a-breadcrumb-item>
  <a-breadcrumb-item>
  <router-link :to='this.$route.path'>{{this.$route.name}}</router-link>
  </a-breadcrumb-item>
</a-breadcrumb>

<router-view></router-view>
```

## button

> 使用 block 会使按钮长度充满父级，type 有多种选择，区分颜色可以使用 `:icon=""` 引入图标

```html
<!-- isAdding 作为计算属性返回布尔值，并绑定点击按钮事件 -->
<template>
  <div>
    <a-button
      v-if="isAdding"
      @click="addToCart"
      type="primary"
      block>
      加入购物车
    </a-button>
    <a-button
      v-else
      @click="removeFromCart(id)"
      type="primary"
      block>
      从购物车移除</a-button>
  </div>
</template>
```

## carousel

> 使用 a-carousel 标签实现跑马灯，将其作为组件插到导航下方，在标签内用 v-for 获取 store 里的图片数据

```html
<!-- products 作为计算属性返回 store 里的数据 -->
<template>
  <a-carousel autoplay>
    <template v-for="product in products">
      <router-link
      :to="'/detail/' + product._id"
      :key="product._id"
      class="product-link">
        <img :src="product.image" alt="" class="product__image">
      </router-link>
    </template>
  </a-carousel>
</template>
```

## table

> ant-ui 的 table 将列先定义在 data 里，在标签里引入表头和数据，列通过数据索引直接被提出来，title 表示列名，dataIndex 和数据对象的属性一致，slot-scope 中 text 表示列的数据，单独指定列再用 slot 指明列名，record 表示 整个 data-source，从整个对象中拿出一个其他的给当前列


```html
<a-table
  :columns="columns"
  :data-source="data"
  :pagination="false"
  bordered>
  <template slot="name" slot-scope="text">
    <span class="table-name">{{ text }}</span>
  </template>
  <template slot="operation" slot-scope="text, record">
    <a-button-group>
      <a-button
        type="primary"
        size="small">
        <router-link :to="'/admin/edit/' + record._id" class="edit">
          <a-icon type="form" />
        </router-link>
      </a-button>
      <a-button
        @click="removeProduct(record._id)"
        type="danger"
        size="small">
          <a-icon type="delete" />
      </a-button>
    </a-button-group>
  </template>
</a-table>

data() {
  return {
    columns: [
      {
        title: 'Name',
        dataIndex: 'name',
        scopedSlots: { customRender: 'name' },
      },
      {
        title: 'Price',
        dataIndex: 'price',
      },
      {
        title: 'Manufacturer',
        dataIndex: 'manufacturer.name',
      },
      {
        title: 'Operation',
        dataIndex: 'operation',
        scopedSlots: { customRender: 'operation' },
      },
    ],
    data: this.products,
  };
},
```

## form-model

> form-model 具有双向绑定式的校验功能，通过 `:model` 指定数据源，通过 rules 指定每个表单的输入校验规则；label-col 与 wrapper-col 指定列的位置和大小；每个表单用 label 是输入框的名字，prop 绑定对应的规则名；规则里的 message 相当于 placeholder，required 表示是否必填项，用 `this.$refs.ruleForm.validate` 提交时进行校验

```html
<a-form-model
  ref="ruleForm"
  :model="modelData"
  :rules="rules"
  :label-col="labelCol"
  :wrapper-col="wrapperCol">
  <a-form-model-item
    label="Name"
    prop="name">
    <a-input
      v-model="modelData.name"
      placeholder="please input product name" />
  </a-form-model-item>

  <a-form-model-item :wrapper-col="{ span: 10, offset: 7 }">
    <a-button v-if="isEditing" type="primary" @click="onSubmit">
      Update
    </a-button>
    <a-button v-else type="primary" @click="onSubmit">
      Create
    </a-button>
    <a-button style="margin-left: 10px;" @click="resetForm">
      Reset
    </a-button>
  </a-form-model-item>
</a-form-model>

data() {
  return {
    modelData: {},
    labelCol: { span: 4, offset: 4 },
    wrapperCol: { span: 10 },
    rules: {
      name: [
        {
          required: true,
          message: 'Please input validate name',
          trigger: 'blur',
        },
        {
          min: 2,
          max: 15,
          trigger: 'blur',
          message: 'Length should be 2 to 15',
        },
      ],
    },
  };
},
```

## message

> message 可用于全局消息提示，比如表单点提交按钮后，页面出现提示："提交成功"，有 secccess，loading 等提示，也可以加 then 方法，在上一个提示后接着再出一个提示。

```js
// 在 onSubmit() 方法里
this.$message
  .loading('Action in progress..', 1)
  .then(() => this.$message.success('Loading finished', 2.5))
```





# 参考资料

[Ant Design Vue](https://antdv.com/docs/vue/introduce-cn/)

