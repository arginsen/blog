---
title: 前端 | Vue 学习笔记Ⅰ：Vue 核心
date: 2020-6-1
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

# vue 的语法

```html
<!-- 使用 v-model 获取 input 输入的值 
    Vue 的使用也是新 new 一个 Vue 的对象，el 为选择器 -->

<div id="test">
    <div>
        <input type="text" v-model="name">
        <span v-show="name">你的名字是：${ name }</span>
    </div>
    <div>
        <input type="text" v-model="age">
        <span v-show="age">你的年龄是：${ age }</span>
    </div>
    <div>
        <input type="text" v-model="sex">
        <span v-show="sex">你的性别是：${ sex }</span>
    </div>
</div>

<script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.js"></script>
<script>
new Vue({
    delimiters: ['${', '}'],  // 后台模板 swig 也用双括号渲染，因此避免与 vue 冲突可使用 delimiters
    el: '#test',
    data: {
        name: null,
        age: null,
        sex: null,
    }
});
</script>
```

## 样板

<div id="test" class="model">
    <div>
        <input type="text" v-model="name">
        <span v-show="name">你的名字是：${ name }</span>
    </div>
    <div>
        <input type="text" v-model="age">
        <span v-show="age">你的年龄是：${ age }</span>
    </div>
    <div>
        <input type="text" v-model="sex">
        <span v-show="sex">你的性别是：${ sex }</span>
    </div>
</div>

<script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.js"></script>
<script>
var test = new Vue({
    delimiters: ['${', '}'],  // 后台模板 swig 也用双括号渲染，因此避免与 vue 冲突可使用 delimiters
    el: '#test',
    data: {
        name: null,
        age: null,
        sex: null,
    }
});
</script>


# vue 的常用指令

## v-for

```html
<!-- v-for 用来遍历数据 -->

<div id="foodlist" class="model">
    <em>原价：</em>
    <ul>
        <li v-for="food in foodList" :key="food.name">${food.name}：￥ ${food.price}</li>
    </ul>
    <em>有折扣：</em>
    <ul>
        <li v-for="food in foodList" :key="food.name">${food.name}：￥ ${food.discount ? food.price * food.discount : food.price}</li>
    </ul>
</div>

<!-- 采用组件的形式编写，在模板标签上用 v-for 设置遍历 -->
<!-- 使用 v-bind 定义 food 属性为变量传入子组件，其值为 v-for 遍历的单个元素 -->
<!-- 组件内用 props 接收 food 变量，即可在 template 中使用 food  -->

<div id="foodlist2" class="model">
    <em>使用组件实现：</em>
    <ul>
        <food-item v-for="item in foodList2" :food="item" :key="item.name"></food-item>
    </ul>
</div>

<script>
var foodlist = new Vue({
    delimiters: ['${', '}'],
    el: '#foodlist',
    data: {
        foodList: [
            {
                name: '葱',
                price: '10',
                discount: .8
            },
            {
                name: '姜',
                price: '8',
                discount: .6
            },
            {
                name: '蒜',
                price: '8',
                discount: null
            }
        ]
    }
});

var foodlist2 = new Vue({
    el: '#foodlist2',
    components: {
        'food-item': {
        props: ['food'],
        template: '<li>{{food.name}}：￥ {{food.price}}</li>'
        },
    },
    data: {
        foodList2: foodlist.foodList
    }
});
</script>
```

### 样板


<div id="foodlist" class="model">
    <em>原价：</em>
    <ul>
        <li v-for="food in foodList" :key="food.name">${food.name}：￥ ${food.price}</li>
    </ul>
    <em>有折扣：</em>
    <ul>
        <li v-for="food in foodList" :key="food.name">${food.name}：￥ ${food.price * food.discount}</li>
    </ul>
</div>

<script>
var foodlist = new Vue({
    delimiters: ['${', '}'],
    el: '#foodlist',
    data: {
        foodList: [
            {
                name: '葱',
                price: '10',
                discount: .8
            },
            {
                name: '姜',
                price: '8',
                discount: .6
            },
            {
                name: '蒜',
                price: '8',
                discount: 1
            }
        ]
    }
});
</script>

## v-bind

```html
<!-- v-bind 用来给标签属性绑定动态的值
    v-bind 可省略，只留冒号 + 属性 -->

<div id="app">
    <a v-bind:href="url">   <!-- 绑定至 data 内指定的值 -->
        <img v-bind:src="img">  <!-- v-bind 可省略，只留冒号 + 属性 -->
    </a>
</div>

<script>
var app = new Vue({
    el: '#app',
    data: {
        url: 'http://baidu.com',
        img:'https://dummyimage.com/100x100/ffcc00/ffffff'
    }
});
</script>
```

## v-on

```html
<!-- 用来绑定事件，方法定义在 methods 里
    v-on: 可以用 @ 代替 -->

<div id="event" class="model">
    <!-- 单个事件用冒号，方法写在引号里 -->
    <button v-on:click="onClick">点击</button>

    <!-- 多个事件直接写对象，属性为事件名，值为函数 -->
    <button v-on="{mouseenter: onEnter, mouseleave: onOut}">滑过</button>

    <!-- .prevent 为阻止浏览器默认行为，.stop 为停止冒泡事件 -->
    <form v-on:submit.prevent="onSubmit">
        <input type="text"><br>
        <button type="submit">提交</button>
    </form>

    <!-- 绑定键盘事件 -->
    <form v-on:keyup="onKeyup">
        <input type="text"><br>
        <button type="submit">提交</button>
    </form>

    <!-- 监听回车，v-on: 可以用 @ 代替 -->
    <form @keyup.enter="onEnter" @submit.prevent="onSubmit">
        <input type="text"><br>
        <button type="submit">提交</button>
    </form>
</div>

<script>
var e = new Vue({
    el: '#event',
    methods: {
        onClick: function() {
            console.log('clicked')
        },
        onIn: function() {
            console.log('mouse enter')
        },
        onOut: function() {
            console.log('mouse leave')
        },
        onSubmit: function() {
            console.log('submitted')
        },
        onKeyup: function() {
            console.log('key pressed')
        },
        onEnter: function() {
            console.log('entered')
        }
    }
});
</script>
```

### 样板

<div id="event" class="model">
    <button v-on:click="onClick">点击</button>
    <button v-on="{mouseenter: onIn, mouseleave: onOut}">滑过</button>
    <form v-on:submit.prevent="onSubmit">
        提交，阻止刷新页面：
        <input type="text"><br>
        <button type="submit">提交</button>
    </form>
    <form v-on:keyup="onKeyup">
        键盘：
        <input type="text">
    </form>
    <form @keyup.enter="onEnter" @submit.prevent="onSubmit">
        监听回车：
        <input type="text"><br>
        <button type="submit">提交</button>
    </form>
</div>

<script>
var e = new Vue({
    el: '#event',
    methods: {
        onClick: function() {
            console.log('clicked')
        },
        onIn: function() {
            console.log('mouse enter')
        },
        onOut: function() {
            console.log('mouse leave')
        },
        onSubmit: function() {
            console.log('submitted')
        },
        onKeyup: function() {
            console.log('key pressed')
        },
        onEnter: function() {
            console.log('entered')
        }
    }
});
</script>

## v-model

```html
<!-- v-model 的多个修饰符 lazy trim number -->
<!-- 在 input label textarea select 输入/选择元素上使用 -->

<div id="input">
    <input type="text" v-model="name">
    <span>同步显示：${name}</span><br>

    <input type="text" v-model.lazy="price">
    <span>惰性显示：${price}</span><br>

    <input type="text" v-model.trim="trim">
    <span>去空格：${trim}</span><br>

    <input type="text" v-model.number="number">
    <span>转为数字：${number}</span><br>

    <label>单选（性别）：
        <span>男</span><input v-model="sex" value="male" type="radio">
        <span>女</span><input v-model="sex" value="female" type="radio">
    </label><br>

    <label>多选（爱好）：
        <span>男</span><input v-model="love" value="male" type="checkbox">
        <span>女</span><input v-model="love" value="female" type="checkbox">
    </label><br>

    <span>多行文字：</span>
    <textarea v-model="article"></textarea><br>
    
    <span>下拉框单选：where are you from?</span>
    <select v-model="from">
        <option value="1">China</option>
        <option value="2">Japan</option>
    </select><br>

    <span>多选：where would you expect going?</span>
    <select v-model="expect" multiple>
        <option value="1">China</option>
        <option value="2">Japan</option>
        <option value="3">Swiss</option>
        <option value="4">France</option>
    </select><br>
</div>

<script>
var w = new Vue({
    delimiters: ['${', '}'],
    el: '#input',
    data: { // 后为默认值
        name: null,
        price: null,
        trim: null,
        number: null,
        sex: null,
        love: [],
        article: 'Symbol 值通过Symbol函数生成。这就是说，对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。',
        from: null,
        expect: []
    }
});
</script>
```


### 样板

<div id="input" class="model">
    <input type="text" v-model="name">
    <span>同步显示：${name}</span><br>
    <input type="text" v-model.lazy="price">
    <span>惰性显示：${price}</span><br>
    <input type="text" v-model.trim="trim">
    <span>去空格：${trim}</span><br>
    <input type="text" v-model.number="number">
    <span>转为数字：${number}</span><br>
    <label>单选（性别）：
        <span>男</span><input v-model="sex" value="male" type="radio">
        <span>女</span><input v-model="sex" value="female" type="radio">
    </label><br>
    <label>多选（爱好）：
        <span>男</span><input v-model="love" value="male" type="checkbox">
        <span>女</span><input v-model="love" value="female" type="checkbox">
    </label><br>
    <span>多行文字：</span>
    <textarea v-model="article"></textarea><br>
    <span>下拉框单选：where are you from?</span>
    <select v-model="from">
        <option value="1">China</option>
        <option value="2">Japan</option>
    </select><br>
    <span>多选：where would you expect going?</span>
    <select v-model="expect" multiple>
        <option value="1">China</option>
        <option value="2">Japan</option>
        <option value="3">Swiss</option>
        <option value="4">France</option>
    </select><br>
</div>

<script>
var w = new Vue({
    delimiters: ['${', '}'],
    el: '#input',
    data: { // 后为默认值
        name: null,
        price: null,
        trim: null,
        number: null,
        sex: null,
        love: [],
        article: 'Symbol 值通过Symbol函数生成。这就是说，对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。',
        from: null,
        expect: []
    }
});
</script>


## v-if

```html
<!-- 使用 v-if v-else-if v-else 添加条件语句进行显示 -->
<!-- v-else 和 v-else-if 必须紧跟在带 v-if 或者 v-else-if 的元素之后，
    中间插 <hr> 都不行 -->

<div id="condition" class="model">
    <div v-if="role == 'admin' || role == 'super_admin'">
        管理员你好
    </div>
    <div v-else-if="role == 'hr'">
        待查看简历列表：...
    </div>
    <div v-else>
        您没有权限访问此处
    </div>
</div>

<script>
var condition = new Vue({
    el: '#condition',
    data: {
        role: 'admin'
    }
});
</script>
```

### 样板

<div id="condition" class="model">
    <div v-if="role === 'admin'">管理员你好
    </div><div v-else-if="role === 'hr'">hr你好，待查看简历列表：...
    </div><div v-else>您没有权限访问此处</div>
</div>

<script>
var condition = new Vue({
    el: '#condition',
    data: {
        role: 'hr'
    }
});
</script>


# vue 的计算属性 computed

```html
<!-- 使用 computed 属性动态展示计算结果 -->
<!-- 计算属性是基于函数的响应式依赖进行缓存的 -->

<div id="computed">
    <table>
        <thead>
        <th>学科</th>
        <th>分数</th>
        </thead>
        <tbody>
        <tr>
            <td>数学</td>
            <td><input type="text" v-model="math"></td>
        </tr>
        <tr>
            <td>英语</td>
            <td><input type="text" v-model="english"></td>
        </tr>
        <tr>
            <td>语文</td>
            <td><input type="text" v-model="chinese"></td>
        </tr>
        <tr>
            <td>总分</td>
            <td>${sum}</td>
        </tr>
        <tr>
            <td>平均分</td>
            <td>${average}</td>
        </tr>
        </tbody>
    </table>
</div>

<script>
/* 
 * 计算属性 computed 是基于函数的响应式依赖进行缓存的
 * 只要 data 内的值未发生变化，多次访问 computed 里的函数（sum、average）
 * 会立即返回之前的计算结果，而不必再次执行函数 
 * 而 methods 内的函数并无响应式依赖缓存，每次访问便会执行
 */

var computed = new Vue({
    delimiters: ['${', '}'],
    el: '#computed',
    data: {
        math: 80,
        english: 90.6,
        chinese: 86
    },
    computed: {
        sum: function() {
            return parseFloat(this.math) + parseFloat(this.english) + parseFloat(this.chinese)
        },
        average: function() {
            return Math.round(this.sum / 3)
        }
    }
});
</script>
```

## 样板

<div id="computed">
    <table>
        <thead>
        <th>学科</th>
        <th>分数</th>
        </thead>
        <tbody>
        <tr>
            <td>数学</td>
            <td><input type="text" v-model="math"></td>
        </tr>
        <tr>
            <td>英语</td>
            <td><input type="text" v-model="english"></td>
        </tr>
        <tr>
            <td>语文</td>
            <td><input type="text" v-model="chinese"></td>
        </tr>
        <tr>
            <td>总分</td>
            <td>${sum}</td>
        </tr>
        <tr>
            <td>平均分</td>
            <td>${average}</td>
        </tr>
        </tbody>
    </table>
</div>

<script>
var compute = new Vue({
    delimiters: ['${', '}'],
    el: '#computed',
    data: {
        math: 80,
        english: 90.6,
        chinese: 86
    },
    computed: {
        sum: function() {
            return parseFloat(this.math) + parseFloat(this.english) + parseFloat(this.chinese)
        },
        average: function() {
            return Math.round(this.sum / 3)
        }
    }
});

$('#computed br').remove()
</script>

# vue 的组件 component

## 全局及局部组件

```html
<div style="display: inline-block">
    <div id="seg1">
        <alert></alert>  <!-- 标签名字同摸板定义的名字 -->
    </div>
    <div id="seg2">
        <alert></alert>  <!-- 标签名字同摸板定义的名字 -->
    </div>
    <div id="seg3">
        <alert></alert>  <!-- 标签名字同摸板定义的名字 -->
    </div>
</div>

<script>
// 1.定义全局组件 会泄露
Vue.component('alert', {
    template: '<button @click="on_click">点一下</button>',  // 模板
    methods: {  // 方法
        on_click: function () {
            alert('Yo.')
        }
    }
});

var seg1 = new Vue({
    el: '#seg1'
});

// 2.直接在内部定义组件
var seg2 = new Vue({
    el: '#seg2',
    components: {   // 定义局部组件
        alert: {
            template: '<button @click="on_click">点一下</button>',  // 模板
            methods: {  // 方法
                on_click: function () {
                    alert('Ha.')
                }
            }
        }
    }
});

// 3.将 alert 组件部分封装出去
var Alert = {
    template: '<button @click="on_click">点一下</button>',  // 模板
    methods: {  // 方法
        on_click: function () {
            alert('酱~')
        }
    }
};

var seg3 = new Vue({
    el: '#seg3',
    components: {   // 定义局部组件，引入封装的对象变量
        alert: Alert
    }
});
</script>
```

### 样板

<div class="model">
    <div id="seg1"><alert></alert></div><div id="seg2"><alert></alert></div><div id="seg3"><alert></alert></div>
</div>

<script>
// 1.定义全局组件 会泄露
Vue.component('alert', {
    template: '<button @click="on_click">点一下</button>',  // 模板
    methods: {  // 方法
        on_click: function () {
            alert('Yo.')
        }
    }
});

// 绑定域
var seg1 = new Vue({
    el: '#seg1'
});

// 2.直接在内部定义组件
var seg2 = new Vue({
    el: '#seg2',
    components: {   // 定义局部组件
        alert: {
            template: '<button @click="on_click">点一下</button>',  // 模板
            methods: {  // 方法
                on_click: function () {
                    alert('Ha.')
                }
            }
        }
    }
});

// 3.将 alert 组件部分封装出去
var Alert = {
    template: '<button @click="on_click">点一下</button>',  // 模板
    methods: {  // 方法
        on_click: function () {
            alert('酱~')
        }
    }
};

var seg3 = new Vue({
    el: '#seg3',
    components: {   // 定义局部组件，引入封装的对象变量
        alert: Alert
    }
});
</script>

## 配置组件

```html
<!-- 因此博客本身摸板 {{ }} 的使用，导致 Vue 的不能直接使用 -->
<div id="zan">
    <like></like>
</div>

<script>
Vue.component('like', {   // like 为组件名
    template: '<button @click="toggle_like()">👍${like_count}</button>',
    data: function () {   // data 为一个返回值的函数
        return {
            like_count: 10,
            liked: false
        }
    },
    methods: {
        toggle_like: function () {
            if (!this.liked)
                this.like_count++
            else
                this.like_count--
            
            this.liked = !this.liked
        }
    }
});

new Vue({
    delimiters: ['${', '}'],
    el: '#zan'
});
</script>
```

### 样板

<div id="zan">
    <like></like>
</div>

<script>
Vue.component('like', {
    template: '<button @click="toggle_like()">👍${like_count}</button>',
    data: function () {
        return {
            like_count: 10,
            liked: false
        }
    },
    methods: {
        toggle_like: function () {
            if (!this.liked)
                this.like_count++
            else
                this.like_count--
            
            this.liked = !this.liked
        }
    }
});

new Vue({
    delimiters: ['${', '}'],
    el: '#zan'
});
</script>

## 父子组件通信 props

```html
<!-- 父组件向子组件通信 -->
<!-- 通过外部标签以 属性为变量(通过组件 props 获取) 传入文本中 -->
<!-- 因此可以直接在 html 标签指定自定义的属性的值，来改变组件方法的值 -->

<div id="msg">
    <alert msg="Yoooo"></alert>
</div>
<button id="address"><user username="whh"></user></button>

<script>
Vue.component('alert', {
    template: '<button @click="on_click">弹弹弹</button>',
    props: ['msg'],
    methods: {
        on_click: function() {
            alert(this.msg)
        }
    }
});

var msg = new Vue({
    el: '#msg'
});

// 使用局部组件方式，同时 username 作为一个变量，
// 从标签属性获取，由 props 接收，再传入模板中
// 其中 href 使用 v-bind 简写，使子组件动态获取属性值
// 所以 a 的文本是 username 变量对应的值，地址对应的也是 /user/username
var address = new Vue({
    el: '#address',
    components: {
        user: {
            template: '<a :href="\'/user/\' + username">{{username}}</a>',
            props: ['username'],
        }
    }
});
</script>
```

### 样板

<div id="msg">
    <alert msg="Yoooo"></alert>
</div>
<div>
    <button id="address"><user username="whh"></user></button>
</div>

<script>
Vue.component('alert', {
    template: '<button @click="on_click">弹弹弹</button>',
    props: ['msg'],
    methods: {
        on_click: function() {
            alert(this.msg)
        }
    }
});

var msg = new Vue({
    el: '#msg'
});

var address = new Vue({
    delimiters: ['${', '}'],
    el: '#address',
    components: {
        user: {
            template: '<a :href="\'/user/\' + username">${username}</a>',
            props: ['username'],
        }
    }
});
</script>

## 子父组件通信 emit

```html
<div id="yue">
    <balance></balance>
</div>

<script>
// 首先 html 页面插入了父组件定义的组件 balance，因此 balance 生成摸板标签
// 同时 balance 标签内含有子组件 show，此时调用 show 组件
// show 组件绑定了 show-balance 事件，该事件监听函数为 show_balance
// 在 show 标签生成对应的摸板标签 button 时，并绑定 click 事件
// 至此，生成方面结束

// 当外界点击 button 时，触发 click 事件，
// 调用子组件 on_click 方法，触发 show-balance 事件
// 调用父组件 show_balance 方法，指定 show 与 data 中默认的值 false 相反
// 在 balance 模板生成时还有 div 标签，定义条件 show == ture 时显示标签内内容
// 此时，整个流程结束，点击按钮后出现对应内容，再次点击消失(if 为 false)


// 父组件用于接受事件 show-balance，再做出反应
Vue.component('balance', {
    template: `
        <div>
            <show @show-balance="show_balance"></show>
            <div v-if="show">
                您的余额：￥159995
            </div>
        </div> 
    `,
    methods: {
        show_balance: function(data) {
            this.show = !this.show
        }
    },
    data: function() {
        return {
            show: false,
        }
    }
});

// 子组件发出事件
Vue.component('show', {
    template: '<button @click="on_click">显示余额</button>',
    methods: {
        on_click() {
            // $emit 触发事件，传递数据
            this.$emit('show-balance', {a: 1, b: 2})
        }  
    }
});

new Vue({
    el: '#yue'
});
</script>
```

### 样板

<div id="yue">
    <balance></balance>
</div>

<script>
// 父组件用于接受事件 show-balance，再做出反应
Vue.component('balance', {
    template: `
        <div>
            <show @show-balance="show_balance"></show>
            <div v-if="show">
                您的余额：￥159995
            </div>
        </div> 
    `,
    methods: {
        show_balance: function(data) {
            this.show = !this.show
        }
    },
    data: function() {
        return {
            show: false,
        }
    }
});

// 子组件发出事件
Vue.component('show', {
    template: '<button @click="on_click">显示余额</button>',
    methods: {
        on_click() {
            // $emit 触发事件
            this.$emit('show-balance')
        }  
    }
});

new Vue({
    el: '#yue'
});
</script>


## 任意及平行组件间通信

```html
<!-- 使用事件调度器来使任意组件间通信 -->

<div id="chat" class="model">
<huahua></huahua>
<shuandan></shuandan>
</div>

<script>
// 通过事件调度器，将一个组件的数据传至类一个组件
// 实现不同组件之间通信

// 通过一个组件里的事件监听函数，
// 在其内定义一个触发另一个组件事件的方法，绑定至调度器
// 再在另一个组件，通过钩子初始化，给调度器绑定刚才需要触发的事件
// 该事件接收前一个组件触发时传入的数据，并做出反馈，完成通信

// 输入时 keyup 事件发生，调用监听函数 on_change，
// 通过事件调度器触发 hua_saysomething 事件，也就是要通信传值 hua_said 了
// hua_saysomething 事件是钩子 mounted 一开始就执行绑定上的，
// 此时触发后调用监听函数，传参调度器传过来的值 hua_said，
// 执行后对方组件获得数据，做出反馈


// 事件调度器
var Event_chat = new Vue();

// huahua 组件
Vue.component('huahua', {
    template: `
    <div>
        花花说：<input @keyup="on_change" v-model="hua_said"/>
    </div>
    `,  // 定义一个 keyup 事件，监听函数为 on_change
    methods: {
        on_change: function() {  // $emit 触发一个事件，(事件名, 数据)
            Event_chat.$emit('hua_saysomething', this.hua_said)
        }
    },
    data: function() {
        return {
            hua_said: '',
        }
    }
});

// shuandan 组件
Vue.component('shuandan', {
    template: '<div>拴蛋也说：{{shuan_said}}</div>',
    data: function() {
        return {
            shuan_said: '',
        }
    },
    mounted: function() {  // 钩子 初始化完毕 ，监听 hua_saysomething
        var me = this
        Event_chat.$on('hua_saysomething', function(data) {
            me.shuan_said = data
        })
    }
});


new Vue({
    el: '#chat',
});
</script>
```

### 样板

<div id="chat" class="model">
<huahua></huahua>
<shuandan></shuandan>
</div>

<script>
// 事件调度器
var Event_chat = new Vue();

// huahua 组件
Vue.component('huahua', {
    template: `
    <div>
        花花说：<input @keyup="on_change" v-model="hua_said"/>
    </div>
    `,  // 定义一个 keyup 事件，函数为 on_change
    methods: {
        on_change: function() {  // $emit 触发一个事件，(事件名, 数据)
            Event_chat.$emit('hua_saysomething', this.hua_said)
        }
    },
    data: function() {
        return {
            hua_said: '',
        }
    }
});

// shuandan 组件
Vue.component('shuandan', {
    template: '<div>拴蛋也说：${shuan_said}</div>',  // 这里不能用双括号，与我网站冲突，就很蛋疼
    data: function() {
        return {
            shuan_said: '',
        }
    },
    mounted: function() {  // 钩子 初始化完毕 监听 hua_saysomething
        var me = this
        Event_chat.$on('hua_saysomething', function(data) {
            me.shuan_said = data
        })
    }
});


new Vue({
    delimiters: ['${', '}'],
    el: '#chat',
});
</script>


# 过滤器 filter

```html
<!-- 过滤就是一个数据经过了这个过滤之后出来另一样东西 -->
<!-- 在此将 input 输入的数据，经过过滤之后再展示出来 -->

<!-- 定义了三个例子，分别为无参全局过滤器，有参全局过滤器，
    以及将数据进行处理后展示出来的过滤器 -->

<!-- 通过 html 标签 {{ val | filter }} 传入 val 数据给 filter 过滤器， -->
<!-- val 作为数据传入 filter 第二个参数的函数中，第一个参数为过滤器的名字 -->

<!-- 传参表达形式：
    1.{{ message | filterA | filterB }} 
    message 作为数据传入 filterA，A 的返回值又传入 B，最终显示 B 返回值

    2.{{ message | filterA('arg1', arg2) }}
    filterA 的第一个参数是 message，后边参数接着是 'arg1', arg2

    3.{{ 'a','b' | filterB }} 
    'a' 和 'b' 分别作为参数传给 filterB
    -->

<div id="price" class="model">
    <div><input v-model="price1">
        <div>${price1 | currency1}</div>
    </div>
    <div><input v-model="price2">
        <div>${price2, unit | currency2}</div>
    </div>
    <div><input v-model="length">mm
        <div>${length | meter}</div>
    </div>
</div>

<script>
// 定义有参全局过滤器
Vue.filter('currency1', function(val, unit) {
    val = val || 0
    unit = unit || '元'
    return val + unit
});

Vue.filter('currency2', function(val, unit) {
    val = val || 0
    unit = unit || 'USD'
    return val + unit
});

Vue.filter('meter', function(val, unit){
    val = val || 0
    unit = unit || 'm'
    return (val / 1000).toFixed(2) + unit
});

new Vue({
    delimiters: ['${', '}'],
    el: '#price',
    data: {
        price1: 10,
        price2: 10,
        unit: 'USD',
        length: 20,
    }
});
</script>
```

## 样板

<div id="price" class="model">
    <div><input v-model="price1">
        <div>${price1 | currency1}</div>
    </div>
    <div><input v-model="price2">
        <div>${price2, unit | currency2}</div>
    </div>
    <div><input v-model="length">mm
        <div>${length | meter}</div>
    </div>
</div>

<script>
Vue.filter('currency1', function(val, unit) {
    val = val || 0
    unit = unit || '元'
    return val + unit
});

Vue.filter('currency2', function(val, unit) {
    val = val || 0
    unit = unit || 'USD'
    return val + unit
});

Vue.filter('meter', function(val, unit){
    val = val || 0
    unit = unit || 'm'
    return (val / 1000).toFixed(2) + unit
});

new Vue({
    delimiters: ['${', '}'],
    el: '#price',
    data: {
        price1: 10,
        price2: 10,
        unit: 'USD',
        length: 20,
    }
});
</script>


# 自定义指令 directives

## 基础配置

```html
<!-- 通过 directive 自定义指令 -->
<!-- 实现在一个按钮卡片定在基于浏览器的固定位置，滑动页面也显示在前端
    且每点一个按钮卡片，下一个就会在前一个基础上往下自动排列，浮于浏览器前端 -->

<div id="card">
    <div v-pin="card1.pinned" class="model">
        <button @click="card1.pinned = !card1.pinned">pin</button>AAA
    </div><br>
    <div v-pin="card2.pinned" class="model">
        <button @click="card2.pinned = !card2.pinned">pin</button>BBB
    </div><br>
    <div v-pin="card3.pinned" class="model">
        <button @click="card3.pinned = !card3.pinned">pin</button>CCC
    </div><br>
    <div v-pin="card4.pinned" class="model">
        <button @click="card4.pinned = !card4.pinned">pin</button>DDD
    </div><br>
    <div v-pin="card5.pinned" class="model">
        <button @click="card5.pinned = !card5.pinned">pin</button>EEE
    </div>
</div>

<script>
// 定义一个指令 pin
// 定义指令绑定一个函数，第一个参数是页面用到该指令的元素，
// 第二个参数是指令传进来的值

new Vue({
    el: '#card',
    data: {
        card1: {
            pinned: false,
        },
        card2: {
            pinned: false,
        },
        card3: {
            pinned: false,
        },
        card4: {
            pinned: false,
        },
        card5: {
            pinned: false,
        }
    },
    directives: {
        'pin': function (el, binding, vnode) {
            // el 是存取有 pin 指令的元素对象
            // vnode 是 Vue 编译生成的虚拟节点
            let isPinned = binding.value  // 获取 pin 的一个绑定对象，里边存储了值
            let _this = vnode.context  // 获取 Vue 的上下文存储 this

            // 判断当 pin 住时，对该元素进行相应改动
            if (isPinned) {
                // 判断目前已经 pin 住的元素里哪些元素已经做了改动
                // 没 fixed 说明没做改动，是新 pin 的元素，不对之前 pin 好的元素做改动
                if (el.style.position != 'fixed') {
                    // 如果是新 pin 的元素，那么就让它位置 fixed，页面样式进行调整
                    el.style.position = 'fixed'
                    el.style.left = '5%'
                    let n = 0

                    // 遍历此 Vue 对象里 $data 的子对象
                    // $data 对象里含有之前 data 对象定义的数据
                    // 从当前数据里获取已经 pin 好的元素的个数，计 n
                    for (let key in _this.$data) {
                        if (_this.$data[key].pinned)
                            n++
                    }

                    // 有几个 pin 好的就将当前正在接受 pin 的元素向下移几个单位
                    el.style.top = 10 * n + '%'
                }

                // 此处仅对缓存中 pin 完成的元素进行判断分离，pin 好的元素不动
            } else {
                // 不 pin 了就把元素位置给还原回去
                el.style.position = 'static'
            }
        }
    }
});
</script>

```


### 样板

<div id="card">
    <div v-pin="card1.pinned" class="model"><button @click="card1.pinned = !card1.pinned">pin</button>AAA</div><br>
    <div v-pin="card2.pinned" class="model"><button @click="card2.pinned = !card2.pinned">pin</button>BBB</div><br>
    <div v-pin="card3.pinned" class="model"><button @click="card3.pinned = !card3.pinned">pin</button>CCC</div><br>
    <div v-pin="card4.pinned" class="model"><button @click="card4.pinned = !card4.pinned">pin</button>DDD</div><br>
    <div v-pin="card5.pinned" class="model"><button @click="card5.pinned = !card5.pinned">pin</button>EEE</div>
</div>

<script>
var card = new Vue({
    el: '#card',
    data: {
        card1: {
            pinned: false,
        },
        card2: {
            pinned: false,
        },
        card3: {
            pinned: false,
        },
        card4: {
            pinned: false,
        },
        card5: {
            pinned: false,
        }
    },
    directives: {
        'pin': function (el, binding, vnode) {
            let isPinned = binding.value
            let _this = vnode.context  // 获取 Vue 的上下文存储 this
            // console.log(binding)

            if (isPinned) {
                if (el.style.position != 'fixed') {
                    el.style.position = 'fixed'  // el 为当前元素
                    el.style.left = '5%'
                    let n = 0

                    for (let key in _this.$data) {
                        if (_this.$data[key].pinned)
                            n++
                    }

                    el.style.top = 10 * n + '%'
                }
            } else {
                el.style.position = 'static'
            }
        }
    }
});
</script>


## 配置传参及修饰符

```html
<!-- 通过在指令后接修饰符直接改动卡片的位置 -->
<!-- 通过给指令加 : 传参，用来突显当前卡片的不同 -->

<div id="card2">
    <div v-pin.bottom.left="card1.pinned" class="model"><button @click="card1.pinned = !card1.pinned">pin</button>AAA</div><br>
    <div v-pin.top.right="card2.pinned" class="model"><button @click="card2.pinned = !card2.pinned">pin</button>BBB</div><br>
    <div v-pin.top.right="card2.pinned" class="model"><button @click="card2.pinned = !card2.pinned">pin</button>CCC</div>
</div>

<script>
var card2 = new Vue({
    el: '#card2',
    data: {
        card1: {
            pinned: false,
        },
        card2: {
            pinned: false,
        },
        card3: {
            pinned: false,
        },
    },
    directives: {
        'pin': function (el, binding, vnode) {
            // 得到指令对应的值
            let pinned = binding.value
            // 得到指令后的修饰符（平行），如有修饰符则为 true 的一个对象
            let position = binding.modifiers  
            // 得到指令通过冒号传入的参数
            let tip = binding.arg
            
            console.log(binding)

            if (pinned) {
                el.style.position = 'fixed'

                // 遍历包含指定修饰符的对象，对对象内的每个修饰符进行操作
                // 如修饰符为 true，也就是表示添加了这个修饰符在指令上
                // 令当前元素样式的修饰符（对应该元素位置 left、bottom 等）等于指定的样式
                for (let key in position) {
                    if (position[key]) {
                        el.style[key] = '10px'
                    }
                }

                if (tip === 'warning')
                    el.style.background = 'yellow'
            } else {
                el.style.position = 'static'
            }
        }
    }
});
</script>
```

### 样板


<div id="card2">
    <div v-pin.bottom.left="card1.pinned" class="model"><button @click="card1.pinned = !card1.pinned">pin</button>AAA</div><br>
    <div v-pin.top.right="card2.pinned" class="model"><button @click="card2.pinned = !card2.pinned">pin</button>BBB</div><br>
    <div v-pin:warning.bottom.right="card3.pinned" class="model"><button @click="card3.pinned = !card3.pinned">pin</button>CCC</div>
</div>

<script>
var card2 = new Vue({
    el: '#card2',
    data: {
        card1: {
            pinned: false,
        },
        card2: {
            pinned: false,
        },
        card3: {
            pinned: false,
        },
    },
    directives: {
        'pin': function (el, binding, vnode) {
            // 得到指令对应的值
            let pinned = binding.value
            // 得到指令后的修饰符（平行），如有修饰符则为 true 的一个对象
            let position = binding.modifiers  
            // 得到指令通过冒号传入的参数
            let tip = binding.arg
            
            // console.log(binding)

            if (pinned) {
                el.style.position = 'fixed'

                // 遍历包含指定修饰符的对象，对对象内的每个修饰符进行操作
                // 如修饰符为 true，也就是表示添加了这个修饰符在指令上
                // 令当前元素样式的修饰符（对应该元素位置 left、bottom 等）等于指定的样式
                for (let key in position) {
                    if (position[key]) {
                        el.style[key] = '10px'
                    }
                }

                if (tip === 'warning')
                    el.style.background = 'yellow'
            } else {
                el.style.position = 'static'
            }
        }
    }
});
</script>


# 混合 Mixins

```html
<!-- 混合，对不同组件中相同部分的一种封装 -->

<div id="mixins" class="model">
    <popup></popup>
    <tooltip></tooltip>
</div>

<script>
// 建立一个变量对象，保存以下组件共同的部分，
// 再通过 mixins 在组件内调用变量

// 定义变量 base 对象存储共同部分
var base = {
    methods: {
        show: function () {
            this.visible = true
        },
        hide: function () {
            this.visible = false
        },
        toggle: function () {
            this.visible = !this.visible
        },
    },
    data: function () {
        return {
            visible: false,
        }
    }
}

// 定义组件 popup
Vue.component('popup', {
    template: `
    <div>
        <button @click="toggle">点击</button>
        <div v-if="visible">
            <h4>title</h4>
            xxxxxxxxxxxxxx
        </div>
    </div>
    `,
    mixins: [base]
});

// 定义组件 tooltip
Vue.component('tooltip', {
    template: `
    <div>
        <button @mouseenter="show" @mouseleave="hide">滑过</button>
        <div v-if="visible">
            <h4>title</h4>
            xxxxxxxxxxxxxx
        </div>
    </div>
    `,
    mixins: [base]
});

// 创建当前 Vue 对象
new Vue({
    el: '#mixins',
})
</script>
```

## 样板

<div id="mixins" class="model">
    <popup></popup>
    <tooltip></tooltip>
</div>

<script>
var base = {
    methods: {
        show: function () {
            this.visible = true
        },
        hide: function () {
            this.visible = false
        },
        toggle: function () {
            this.visible = !this.visible
        },
    },
    data: function () {
        return {
            visible: false,
        }
    }
}

Vue.component('popup', {
    template: `
    <div>
        <button @click="toggle">点击</button>
            <div v-if="visible">
                <h4>title</h4>
                知而不行，谓之不诚。
                行而不成，谓之不能。
                知而行，是赤诚之心，
                行而能，是贯彻到底，
                已经很难被其他人和事影响了。
                而知行合一的前提是，格物致知，
                将一件事研究到极致，变成自己的知识。
                比如做菜，将做菜这门手艺做到尽善尽美，色香味无可挑剔；
                比如建筑，将楼宇亭台建得坚固美观使用，美轮美奂，风雨不倒地震不塌；
                比如造船，将造船原理吃透，用料坚固，风浪不沉。
                这便是格物致知，将一件东西一件事情研究到极点，
                明白其中所有的道理。
                做到格物致知，方能知行合一。
            </div>
    </div>
    `,
    mixins: [base]
});

Vue.component('tooltip', {
    template: `
    <div>
        <button @mouseenter="show" @mouseleave="hide">滑过</button>
        <div v-if="visible">
            <h4>title</h4>
            知而不行，谓之不诚。
            行而不成，谓之不能。
            知而行，是赤诚之心，
            行而能，是贯彻到底，
            已经很难被其他人和事影响了。
            而知行合一的前提是，格物致知，
            将一件事研究到极致，变成自己的知识。
            比如做菜，将做菜这门手艺做到尽善尽美，色香味无可挑剔；
            比如建筑，将楼宇亭台建得坚固美观使用，美轮美奂，风雨不倒地震不塌；
            比如造船，将造船原理吃透，用料坚固，风浪不沉。
            这便是格物致知，将一件东西一件事情研究到极点，
            明白其中所有的道理。
            做到格物致知，方能知行合一。
        </div>
    </div>
    `,
    mixins: [base]
});

new Vue({
    el: '#mixins',
})
</script>


# 插槽 Slot

```html
<!-- slot 用于创造一套摸板的基础上，使摸板数据动态化，样式统一，只需关注内容创作 -->

<!-- 定义一个组件 panel，使其在 Vue 追踪的 el 中建立，
    组件的摸板采用分离式写法，摸板写在 html 页面 el 元素的下面，
    Vue 的摸板中就只写 html 中 <template> 的定位选择器即可 -->

<!-- slot 工作原理 -->
<!-- 在 template 摸板标签里将每部分的内容用 slot 标签替换
    slot 标签的 name 属性标记当前部分
    在 panel 生成标签里再定义标签通过 slot 属性可以给对应部分传入内容，
    如不写某部分，某部分将会显示 tamplate 模板里的默认内容 -->

<div id="slot">
    <!-- 创建一个公告板，其内包含有三部分，每部分均可自定义内容 -->
    <panel>
        <div slot="title">新闻</div>
        <div slot="content">新华社北京7月17日电 中共中央政治局常务委员会7月17日召开会议，研究部署防汛救灾工作。中共中央总书记习近平主持会议并发表重要讲话。</div>
        <div slot="footer">详情</div>
    </panel>
    <panel>
        <div slot="title">快讯</div>
        <div slot="content">据央视曝光，金浩阳毛巾，永亮毛巾等用便宜的纱线加工，达不到国家标准。用来制作毛巾的纱线都是用再生棉制成的。</div>
    </panel>
</div>
<template id="panel-tpl">
    <div class="panel">
        <div class="title">
            <slot name="title"></slot>
        </div>
        <div class="content">
            <slot name="content"></slot>
        </div>
        <div class="footer">
            <slot name="footer">更多信息</slot>  
        </div>
    </div>
</template>

<script>
Vue.component('panel', {    // panel 是组件名，用于生成摸板指定内容
    template: '#panel-tpl',  // panel-tpl 是摸板名，用于定位
});

new Vue({
    el: '#slot',
})
</script>
```


## 样板


<div id="slot"><panel><div slot="title">新闻</div><div slot="content">新华社北京7月17日电 中共中央政治局常务委员会7月17日召开会议，研究部署防汛救灾工作。中共中央总书记习近平主持会议并发表重要讲话。</div><div slot="footer">详情</div></panel><panel><div slot="title">快讯</div><div slot="content">据央视曝光，金浩阳毛巾，永亮毛巾等用便宜的纱线加工，达不到国家标准。用来制作毛巾的纱线都是用再生棉制成的。</div></panel></div>

<template id="panel-tpl"><div class="panel"><div class="title"><slot name="title"></slot></div><div class="content"><slot name="content"></slot></div><div class="footer"><slot name="footer">更多信息</slot></div></div></template>

<script>
Vue.component('panel', {    // panel 是组件名，用于生成摸板指定内容
    template: '#panel-tpl',  // panel-tpl 是摸板名，用于定位
});

new Vue({
    el: '#slot',
})
</script>



# 参考资料

[表严肃 - Vue 精讲](https://biaoyansu.com/18.1)
[Vue.js 中文教程](https://cn.vuejs.org/v2/guide/index.html)






<script>
// 定义背景摸板 model
$('.model').css({
    background: '#f5f5f5',
    display: 'inline-block',
    padding: '0 20px',
    'border-radius': '7px'
});

$('.model ul').css('margin', '0 20px');

// 定义 panel 摸板
$('.panel').css({
    background: '#f5f5f5',
    'max-width': '300px',
    border: '1px solid #999',
    'margin-bottom': '15px',
});

$('.panel > *').css({
    padding: '15px',
});

$('.panel .title').css({
    'border-bottom': '1px solid #999',
});

$('.panel .footer').css({
    'border-top': '1px solid #999',
});
</script>