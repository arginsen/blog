---
title: 华为机试题
date: 2020-9-12
tags: 
  - interview
  - Frontend
categories: practice
photos:
    - /blog/img/interview.jpg
---

<br>
<!--more-->

> 地址：https://www.nowcoder.com/ta/huawei
> 说明：使用 `readline()` 来获取每行的输入

# 字符串最后一个单词的长度

> 题目描述
> 计算字符串最后一个单词的长度，单词以空格隔开。
> 输入描述:
> 一行字符串，非空，长度小于5000。
> 输出描述:
> 整数N，最后一个单词的长度。

```js
const n = readline();
const arr = n.split(' ');
console.log(arr[arr.length - 1].length)
```

# 计算字符个数

> 题目描述
> 写出一个程序，接受一个由字母和数字组成的字符串，和一个字符，然后输出输入字符串中含有该字符的个数。不区分大小写。
> 输入描述:
> 第一行输入一个有字母和数字以及空格组成的字符串，第二行输入一个字符。
> 输出描述:
> 输出输入字符串中含有该字符的个数。

```js
// 将字母都转换为小写字母，再用letter作为str的分隔符，将str切分为数组
const str = readline().toLowerCase();
const letter = readline().toLowerCase();
console.log(str.split(letter).length -1);
```

# 明明的随即数

题目描述
生成N(N<=1000)个1到1000之间的随机整数，一个数一行，进行去重和排序（从小到大）
输入描述：
输入多行，先输入随机整数的个数，再输入相应个数的整数
输出描述:
返回多行，处理后的结果

```js
// 用 forEach 遍历数组
while(n = Number(readline())) {
    let arr = [];
    while(n--) {
        let index = Number(readline());
        arr[index]=1;
    }
    arr.forEach((el, index) => {
        console.log(index); // 若为空，则不会输出
    });
}

// 使用 for ... in 遍历数组或对象的属性
while(n = readline()) {
    let arr = [];
    while(n--) {
        let index = readline();
        arr[index]=1;
    }
    for(let i in arr) {
        console.log(i);
    }
}
```

# 字符串分隔

> 题目描述
> • 连续输入字符串，请按长度为8拆分每个字符串后输出到新的字符串数组；
> • 长度不是8整数倍的字符串请在后面补数字0，空字符串不处理。
> 输入描述:
> 连续输入字符串(输入2次,每个字符串长度小于100)
> 输出描述:
> 输出到长度为8的新字符串数组

```js
// 循环
// 接收两个字符串，遍历
// 分情况，1.如果字符串长度大于 8，每 8 个长度输出一次，如果长度不是 8 的倍数（整除），
// 再输出余下的几个字符与 '00000000' 从余数位算起长度的 0 拼贴
// 2.如果字符串长度小于等于 8 ，将字符串与 '00000000' 从余数算起长度的 0 拼贴
// ------------------------------
// strOut 对于有余数的字符串都适用：
// 1.l=99，str.substring(96) + '00000000'.substring(3)
// 2.l=3，str.substring(0) + '00000000'.substring(3)
[readline(), readline()].forEach((str) => {
    const l = str.length;
    const n = parseInt(l / 8);
    const y = l % 8;
    const strOut = str.substring(l - y) + '00000000'.substring(y);
    if(l > 8) {
        let strStart = 0;
        for(let i = 1; i <= n; i++) {
            console.log(str.substr(strStart, 8));
            strStart = strStart + 8;
        }
        if(y) console.log(strOut); // 大于 8 无余数的不再处理
    } else {
        if(l) console.log(strOut); // 空字符串不处理
    }
});

// 递归
// 接收字符串，对每个字符串执行 len8 函数
// 分情况：
// 1.长度大于8，打印0-8位，再执行len8，传参从第8位开始的字符串，直到长度小于8进入else
// 2.长度小于8，直接将字符串和需要补的0进行拼贴
[readline(), readline()].forEach(function len8(str) {
    if(str.length > 8) {
        console.log(str.substring(0, 8)); // 不包括第8位
        len8(str.substring(8)); // 当前str去掉前边的8位
    } else {
        console.log(str + '00000000'.substring(str.length));
    }
});
```

# 质数因子

> 题目描述
> 功能:输入一个正整数，按照从小到大的顺序输出它的所有质因子（重复的也要列举）（如180的质因子为2 2 3 3 5 ）；最后一个数后面也要有空格
> 输入描述:
> 输入一个long型整数
> 输出描述:
> 按照从小到大的顺序输出它的所有质数的因子，以空格隔开。最后一个数后面也要有空格。

```js
// 如果一个数字不是素数 那它除了1和他本身一定还有别的约数
// 如果这个数的约数在其开平方的右边，则必然会存在一个约数在其开平方的左边
// 所以判断一个数是否为质数，只需要观察其在 2 到 开平方数中间是否含有约数即可
let n = readline();
let str = '';
for(let i = 2; i < Math.sqrt(n); i++) { // 约数从2开始递增
    while(n % i === 0) { // 连续提出多个i
        str += i + ' ';
        n /= i;
    }
}
if (n > 1) {
    str += n + ' ';
}
console.log(str);
```

# 合并表记录

> 题目描述
> 数据表记录包含表索引和数值（int范围的整数），请对表索引相同的记录进行合并，即将相同索引的数值进行求和运算，输出按照key值升序进行输出。
> 输入描述:
> 先输入键值对的个数
> 然后输入成对的index和value值，以空格隔开
> 输出描述:
> 输出合并后的键值对（多行）

```js
// 使用 num 每读取一行数，用空格分开两个数存入数组 tmp，
// tmp 第一个元素为索引，第二个元素为值
const n = readline();
let arr = [];
while(num = readline()) {
    let tmp = [];
    tmp = num.split(' ');
    if (!arr[Number(tmp[0])])
        arr[Number(tmp[0])] = Number(tmp[1]);
    else
        arr[Number(tmp[0])] += Number(tmp[1]);
}
arr.forEach((el, index) => {
    console.log(index + ' ' + el);
});


// 通过给出的键值对数，每处理一组键值对，就递减 n
// 将一组键值对（字符串）拆分存入数组 arr，

let n = readline();
const obj = {};
while(n > 0){
    let arr = readline().split(' ');
    obj[arr[0]] = (obj[arr[0]] || 0) + Number(arr[1])
    n--;
}
for(let key in obj) {
    console.log(key + ' ' + obj[key])
}
```

# 提取不重复的整数

> 题目描述
> 输入一个int型整数，按照从右向左的阅读顺序，返回一个不含重复数字的新的整数。
> 输入描述:
> 输入一个int型整数
> 输出描述:
> 按照从右向左的阅读顺序，返回一个不含重复数字的新的整数

```js
// 将整数转为字符串str，字符串从后往前一个一个测试是否在res中存在
// 如果不存在，拼贴进res
const str = readline().toString();
let res = '';
for(let i = str.length - 1;i>=0;i--) {
    if(!res.includes(str[i])) {
        res += str[i];
    }
}
console.log(Number(res));
```

# 字符个数统计

> 题目描述
> 编写一个函数，计算字符串中含有的不同字符的个数。字符在ACSII码范围内(0~127)，换行表示结束符，不算在字符里。不在范围内的不作统计。多个相同的字符只计算一次
> 例：输入 abaca   =>  输出 3
> 输入描述:
> 输入N个字符，字符在ACSII码范围内。
> 输出描述:
> 输出范围在(0~127)字符的个数。

```js
// 方法同上，计算去重后的字符串长度
const str = readline().toString();
let res = '';
for(let i = str.length - 1;i>=0;i--) {
    if(!res.includes(str[i])) {
        res += str[i];
    }
}
console.log(res.length);
```

# 数字颠倒

> 题目描述
> 1.输入一个整数，将这个整数以字符串的形式逆序输出
> 2.程序不考虑负数的情况，若数字含有0，则逆序形式也含有0，如输入为100，则输出为001
> 输入描述:
> 输入一个int整数
> 输出描述:
> 将这个整数以字符串的形式逆序输出


```js
const n = readline().toString();
let l = n.length
let str = '';
while(l > 0) {
    str += n[l - 1];
    l--;
}
console.log(str);

// 等同于
const n = readline().toString();
let str = '';
for(let i = n.length - 1; i >= 0; i--) {
    str += n[i];
}
console.log(str);

// 利用现成的
while(readline()) {
    console.log(readline().toString().split("").reverse().join(""))
}
```

# 字符串反转

> 题目描述
> 写出一个程序，接受一个字符串，然后输出该字符串反转后的字符串。（字符串长度不超过1000）
> 输入描述:
> 输入N个字符
> 输出描述:
> 输出该字符串反转后的字符串

```js
const n = readline();
let str = '';
for(let i = n.length - 1; i >= 0; i--) {
    str += n[i];
}
console.log(str);

// 利用现成的
while(readline()) {
    console.log(readline().split("").reverse().join(""))
}
```

# 句子逆序

> 题目描述
> 1.将一个英文语句以单词为单位逆序排放。例如“I am a boy”，逆序排放后为“boy a am I”
> 2.所有单词之间用一个空格隔开，语句中除了英文字母外，不再包含其他字符
> 输入描述:
> 将一个英文语句以单词为单位逆序排放。
> 输出描述:
> 得到逆序的句子

```js
const arr = readline().split(" ");
let str = '';
for(let i = arr.length - 1; i >= 0; i -= 1) {
    str += arr[i] + ' ';
}
console.log(str);
```

# 字符串的连接最长路径查找

> 题目描述
> 给定n个字符串，请对n个字符串按照字典序排列。
> 输入描述:
> 输入第一行为一个正整数n(1≤n≤1000),下面n行为n个字符串(字符串长度≤100),字符串中只含有大小写字母。
> 输出描述:
> 数据输出n行，输出结果为按照字典序排列的字符串。

```js
let n = readline();
let arr = [];
while(n > 0) {
    let num = readline();
    arr.push(num);
    n--;
}
arr.sort();
for (let i of arr) {
    console.log(i);
}
```

# 求int型数据在内存中存储时1的个数

> 题目描述
> 输入一个int型的正整数，计算出该int型数据在内存中存储时1的个数。
> 输入描述:
>  输入一个整数（int类型）
> 输出描述:
>  这个数转换成2进制后，输出1的个数

```js
// 转为二进制全为 0 1，字符串相减可自动转换
const str = Number(readline()).toString(2);
let len = 0;
for(let i = 0; i < str.length; i++) {
    len -= str[i];
}
console.log(-len);

// 判断当前字符是否为 '1'
const str = Number(readline()).toString(2);
let len = 0;
for(let i = 0; i < str.length; i++) {
    if(str[i] === '1') {
        len += 1;
    }
}
console.log(len);
```

# 坐标移动

> 题目描述
> 开发一个坐标计算工具， A表示向左移动，D表示向右移动，W表示向上移动，S表示向下移动。从（0,0）点> 开始移动，从输入字符串里面读取一些坐标，并将最终输入结果输出到输出文件里面。
> 输入：
> 合法坐标为A(或者D或者W或者S) + 数字（两位以内）；坐标之间以 `;` 分隔。非法坐标点需要进行丢弃。> 如 `AA10;`  `A1A;`  `$%$;`  `YAD;` 等。
> 
> 输入描述:
> 多组输入输出，每次输入一行字符串
> 输出描述:
> 最终坐标，以,分隔

> 本道题：[地址](https://www.nowcoder.com/practice/119bcca3befb405fbe58abe9c532eb29?tpId=37&tags=&title=&diffculty=0&judgeStatus=0&rp=1) 其中通过的 JS 代码中，基本是只验证第一位是不是 `ASWD`，对后边也没有约束就直接开始加了，遇到 `'AA20'` 这种肯定跑不过了，使用 `'AA20'.substring(1)` 就成了 `NaN`

```js
// 使用正则匹配输入字符串每个小节是否满足要求
while(n = readline()) {
    const arr = n.split(';');
    let x = 0, y = 0;
    for(let v of arr) {
        if(/^[ADSW]\d{1,2}$/.test(v)) {
            let num = Number(v.substring(1));
            if(v[0]==='A') { x -= num; }
            if(v[0]==='D') { x += num; }
            if(v[0]==='S') { y -= num; }
            if(v[0]==='W') { y += num; }
        }
    }
    console.log(x + ',' + y);
}
```

# 简单错误记录

> 题目描述
> 开发一个简单错误记录功能小模块，能够记录出错的代码所在的文件名称和行号。
> 处理：
> 1、 记录最多8条错误记录，循环记录（或者说最后只输出最后出现的八条错误记录），对相同的错误记录（净文件名（保留最后16位）称和行号完全匹配）只记录一条，错误计数增加；
> 2、 超过16个字符的文件名称，只记录文件的最后有效16个字符；
> 3、 输入的文件可能带路径，记录文件名称不能带路径。
>
> 输入描述:
> 一行或多行字符串。每行包括带路径文件名称，行号，以空格隔开。
> 输出描述:
> 将所有的记录统计并将结果输出，格式：文件名 代码行数 数目，一个空格隔开

> 字符串中的单斜杠是不能识别的，路径显示单斜杠，实际应该是'E:\\V1R2\\product\\fpgadrive.c'

```js
const obj = {};
while(str = readline()) {
    const inputArr = str.split(' ');
    const dirArr = inputArr[0].split('\\'); 
    const fileName = dirArr[dirArr.length - 1].substr(-16); // 剪裁字符串后16个字符
    const key = fileName + ' ' + inputArr[1];
    obj[key] = obj[key] ? obj[key] + 1 : 1;
}
const keys = Object.keys(obj).slice(-8); // 剪裁数组后8个元素
for(let v of keys) {
    console.log(v + ' ' + obj[v]);
}
```

# 密码验证合格程序

> 题目描述
> 密码要求:
> 1.长度超过8位
> 2.包括大小写字母.数字.其它符号,以上四种至少三种
> 3.不能有相同长度大于2的子串重复
> 输入描述:
> 一组或多组长度超过2的子符串。每组占一行
> 输出描述:
> 如果符合要求输出：OK，否则输出NG


```js
while(str = readline()) {
    check(str);
}

function check(str) {
    // 第一个条件
    if(str.length <= 8) {
        console.log('NG');
        return;
    }
    // 第二个条件
    let point = 0;
    if(/[A-Z]/.test(str)) { point++; }
    if(/[a-z]/.test(str)) { point++; }
    if(/\d/.test(str)) { point++; }
    if(/\W/.test(str)) { point++; }
    if(point < 3){
       console.log('NG');
       return;
    }
    
    // 第三个条件
    // 验证长度大于2的子串，子串从str第0位记起始往后移动，长度记为l
    // 第一层循环为起始位置的推移，在判断长度为3的子串是否在后边有存在，无论长度再长，都满足长度3的重复
    for(let i = 0; i < str.length - 3; i++) {
        const tempStr = str.substr(i, 3);
        if(str.lastIndexOf(tempStr) > i) {
            console.log('NG');
            return;
        }
    }
    console.log('OK');
};
```

# 简单密码

> 规则：
> 1.`1--1`, `abc--2`, `def--3`, `ghi--4`, `jkl--5`, `mno--6`, `pqrs--7`, `tuv--8 wxyz--9`, `0--0`
> 2.小写字母都变成对应的数字，大写字母则变成小写之后往后移一位，Z往后移是a，数字和其他的符号都不做变换
> 输入描述:
> 输入包括多个测试数据。输入是一个明文，密码长度不超过100个字符，输入直到文件结尾
> 输出描述:
> 输出渊子真正的密文


```js
while(n = readline()) {
    encrypt(n);
}

function encrypt(str) {
    const prev = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
    const curr = 'bcdefghijklmnopqrstuvwxyza22233344455566677778889999';
    let newStr = '';
    // 遍历待加密的字符串
    for(let i = 0; i < str.length; i++) {
        let tmp = str[i];
        // 使用正则仅对字母做转换
        if(/[A-Za-z]/.test(tmp)) {
            tmp = curr[prev.indexOf(tmp)];
        }
        newStr += tmp;
    }
    console.log(newStr);
}
```

# 汽水瓶

> 题目描述
> 有这样一道智力题：“某商店规定：三个空汽水瓶可以换一瓶汽水。小张手上有十个空汽水瓶，她最多可以换多少瓶汽水喝？”答案是5瓶，方法如下：先用9个空瓶子换3瓶汽水，喝掉3瓶满的，喝完以后4个空瓶子，用3个再换一瓶，喝掉这瓶满的，这时候剩2个空瓶子。然后你让老板先借给你一瓶汽水，喝掉这瓶满的，喝完以后用3个空瓶子换一瓶满的还给老板。如果小张手上有n个空汽水瓶，最多可以换多少瓶汽水喝？
> 输入描述:
> 输入文件最多包含10组测试数据，每个数据占一行，仅包含一个正整数n（1<=n<=100），表示小张手上的空汽水瓶数。n=0表示输入结束，你的程序不应当处理这一行。
> 输出描述:
> 对于每组测试数据，输出一行，表示最多可以喝的汽水瓶数。如果一瓶也喝不到，输出0。

```js
// 按步骤写...
while(n = Number(readline())) {
    console.log(check(n));
}

function check(n) {
    if (n === 1) {
        return 0;
    } else if (n === 2 || n === 3) {
        return 1;
    } else {
        const num = Math.floor(n / 3);
        let y = n % 3;
        let k = num + y;
        
        k += check(k);
        return k - y;
    }
}

// 数学的方法思考...
let n;
while (n = parseInt(readline())) {
    if (n >= 1){
        console.log(parseInt(n/2))
    }
}
```