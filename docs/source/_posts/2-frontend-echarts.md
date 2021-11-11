---
title: 前端 | echarts 学习
date: 2021-9-20 18:45:00
tags: 
- Vue
- Frontend
categories: notes
photos:
  - /blog/img/echarts.jpg
---

<br>
<!--more-->

echarts 是一个基于 JavaScript 的可视化图标库

> 介绍: https://echarts.apache.org/handbook/zh/get-started

可以用来生成折线图、柱状图、饼图、散点图、地理坐标、地图、热力图、关系图、树图、仪表盘、日历坐标、以及3D等图

> 例子: https://echarts.apache.org/examples/zh/

# 使用

直接使用 init 方法来注册图标容器, 其内参数为 dom 节点;
可以对图标进行定制样式, 配置 option
最后再用 setOption 设定

```ts
const myChart = echarts.init(document.getElementById('main'));

let option = {
  title,
  tooltip,
  legend,
  grid,
  toolbox,
  xAxis,
  yAxis,
  series: []
}

option && myChart.setOption(option)
```

# 配置

> 配置: https://echarts.apache.org/zh/option.html

## title

标题组件

```ts
let option = {
  title: {
    id: '',
    show: true,
    text: '',
    link: '',
    target: 'blank',
    textStyle: {},
    subtext: '',
    sublink: '',
    subtarget: 'self',
    subtextStyle: {},
    textAlign: 'center',
    textVerticalAlign: 'top',
    triggerEvent: false,
    padding: 5,
    left: 'center'
  }
}
```

## legend

图例组件。展示不同系列的标记，颜色和名字

```ts
let option = {
  title: 'price'
  legend: {
    type: 'plain', // scrll 图例较多时可翻页
    id: '',
    show: true,
    left: 'center', // 20 20% left right 距离容器左侧的距离
    width: 'auto',
    orient: 'horizontal', // 'vertical' 布局朝向
    align: 'auto', // 图例标记和文本的对齐
    padding: 5,
    data: [], // 图例的数据数组 字符串
    tooltip: {}
  }
}
```

## grid

直角坐标系内绘图网格，单个 grid 内最多可以放置上下两个 X 轴，左右两个 Y 轴

```ts
let option = {
  grid: {
    id: '',
    show: true,
    left: '10%',
    width: 'auto',
    containLabel: true, // 防止标签溢出
  }
}
```

## xAxis/yAxis

type 类型可以选取坐标轴类型, 'value' 为数值轴，适用于连续数据; 'category' 类目轴，适用于离散的类目数据; 'time' 时间轴，适用于连续的时序数据; 'log' 对数轴。适用于对数数据。

data 是类目数据, 仅在类目轴中有效

```ts
let option = {
  grid: {
    containLabel: true
  },
  xAxis: {
    type: 'category',
    data: ['wanke', 'pingan']
  },
  yAxis: {
    type: 'value'
  }
}
```

## tooltip

提示框组件. 可以设在全局, 可以设置再坐标系中, 可以设置在 series 中, 可以设置在每项数据中

触发类型 trigger 有两种形式的配置, 'item' 为数据项图形触发, 主要用于散点图, 饼图等无类目轴的图表; 'axis' 为坐标轴触发, 主要在柱状图, 折线图等会使用类目轴的图表

```ts
let option = {
  tooltip: {
    trigger: 'item'
  }
}
```

## dataset

数据集（dataset）组件用于单独的数据集声明，从而数据可以单独管理，被多个组件复用，并且可以自由指定数据到视觉的映射

source 是必要的, 用以下这种形式来表示二维数据, 第一行给出维度名, 如下第一列表示一组数据中同时比较三个年份, 具体数据从第二行开始记起

```ts
let option = {
  legend: {},
  tooltip: {},
  dataset: {
    source: [
      ['product', '2015', '2016', '2017'],
      ['Matcha Latte', 43.3, 85.8, 93.7],
      ['Milk Tea', 83.1, 73.4, 55.1],
      ['Cheese Cocoa', 86.4, 65.2, 82.5],
      ['Walnut Brownie', 72.4, 53.9, 39.1]
    ]
  },
  // 声明一个 X 轴，类目轴（category）。默认情况下，类目轴对应到 dataset 第一列。
  xAxis: { type: 'category' },
  // 声明一个 Y 轴，数值轴。
  yAxis: {},
  // 声明多个 bar 系列，默认情况下，每个系列会自动对应到 dataset 的每一列。
  series: [{ type: 'bar' }, { type: 'bar' }, { type: 'bar' }]
}
```

## series

series 系列用来设定数据, 指明图标的展示类型, 有 line/bar/pie/tree 等等; 也可以将 series 中的数据交由 dataset 数据集管理

如上 dataset 中的配置, 转换成 series 表示就成了将数据拆成单个的对象, 塞入 series 数组中, 每项依然会配置 type, 而 dataset 中则是将 name 和 series 提出, 将数据统一管理

```ts
option = {
  xAxis: {
    type: 'category',
    data: ['Matcha Latte', 'Milk Tea', 'Cheese Cocoa', 'Walnut Brownie']
  },
  yAxis: {},
  series: [
    {
      type: 'bar',
      name: '2015',
      data: [89.3, 92.1, 94.4, 85.4]
    },
    {
      type: 'bar',
      name: '2016',
      data: [95.8, 89.4, 91.2, 76.9]
    },
    {
      type: 'bar',
      name: '2017',
      data: [97.7, 83.1, 92.5, 78.1]
    }
  ]
};
```

# 例子

## 折线图

```ts
let option = {
  title: {
    text: 'Stacked Line'
  },
  tooltip: {
    trigger: 'axis'
  },
  legend: {
    data: ['Email', 'Union Ads', 'Video Ads', 'Direct', 'Search Engine'],
    tooltip: {
      show: true
    }
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true
  },
  toolbox: {
    feature: {
      saveAsImage: {} // 保存图片
    }
  },
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
  },
  yAxis: {
    type: 'value'
  },
  series: [
    {
      name: 'Email',
      type: 'line',
      stack: 'Total',
      data: [120, 132, 101, 134, 90, 230, 210]
    },
    {
      name: 'Union Ads',
      type: 'line',
      stack: 'Total',
      data: [220, 182, 191, 234, 290, 330, 310]
    },
    {
      name: 'Video Ads',
      type: 'line',
      stack: 'Total',
      data: [150, 232, 201, 154, 190, 330, 410]
    },
    {
      name: 'Direct',
      type: 'line',
      stack: 'Total',
      data: [320, 332, 301, 334, 390, 330, 320]
    },
    {
      name: 'Search Engine',
      type: 'line',
      stack: 'Total',
      data: [820, 932, 901, 934, 1290, 1330, 1320]
    }
  ]
};
```

## 柱状图

主要需要在 series 里描述展示数据的类型为 bar

```ts
option = {
  xAxis: {
    type: 'category',
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
  },
  yAxis: {
    type: 'value'
  },
  series: [
    {
      data: [120, 200, 150, 80, 70, 110, 130],
      type: 'bar'
    }
  ]
};
```

用数据集 dataset 来表示柱状图

```ts
option = {
  legend: {},
  tooltip: {},
  dataset: {
    source: [
      ['product', '2015', '2016', '2017'],
      ['Matcha Latte', 43.3, 85.8, 93.7],
      ['Milk Tea', 83.1, 73.4, 55.1],
      ['Cheese Cocoa', 86.4, 65.2, 82.5],
      ['Walnut Brownie', 72.4, 53.9, 39.1]
    ]
  },
  xAxis: { type: 'category' },
  yAxis: {},
  // Declare several bar series, each will be mapped
  // to a column of dataset.source by default.
  series: [{ type: 'bar' }, { type: 'bar' }, { type: 'bar' }]
};
```

## 饼状图

饼状图的特性是需要在 series 中设定 type 为 pie

```ts
option = {
  title: {
    text: 'Referer of a Website',
    subtext: 'Fake Data',
    left: 'center'
  },
  tooltip: {
    trigger: 'item'
  },
  legend: {
    orient: 'vertical',
    left: 'left'
  },
  series: [
    {
      name: 'Access From',
      type: 'pie',
      radius: '50%',
      data: [
        { value: 1048, name: 'Search Engine' },
        { value: 735, name: 'Direct' },
        { value: 580, name: 'Email' },
        { value: 484, name: 'Union Ads' },
        { value: 300, name: 'Video Ads' }
      ],
      emphasis: {
        itemStyle: {
          shadowBlur: 10,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0, 0, 0, 0.5)'
        }
      }
    }
  ]
};
```