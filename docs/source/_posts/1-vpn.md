---
title: 使用亚马逊云搭建vpn
tags: vpn
categories: practice
date: 2019-3-23
photos:
    - /blog/img/vpn.jpg
---

<br>
<!-- more -->

# 前 言

+ 亚马逊服务器新用户注册送12个月的免费套餐，一个月提供750小时+30GB(想来平时用用肯定够了)
+ 等到免费期结束后记得一定要关闭实例(服务器)否则就要烧钱了
+ 需要信用卡


# 教 程

### 注册亚马逊

1. 进入官网

![](http://img.xiumi.us/xmi/ua/1O3nK/i/402cf7430dd1a2ac69c56286fea0ff0f-sz_66423.png)

![](http://img.xiumi.us/xmi/ua/1O3nK/i/8c34d5040695e6e7d77227f1b792babf-sz_27239.png)

2. 注册账号

![](http://img.xiumi.us/xmi/ua/1O3nK/i/07c55ef465ea0c3247957b8cb50c6d62-sz_16549.png)

![](http://img.xiumi.us/xmi/ua/1O3nK/i/95f9e60446abd818d91a3f1e17cc1f0b-sz_17358.png)

#按要求填写信息，完成账号注册。

#账号注册完成还需要绑定一张信用卡，之后亚马逊会转给你账户一笔钱，你再用这笔钱支付，验证你的信息。


### 创建EC2实例

1. 登陆刚才注册的账号

![](http://img.xiumi.us/xmi/ua/1O3nK/i/8668322bb9a350b1f7bb0d60854d5320-sz_80596.png)

2. 点击服务进入EC2

![](http://img.xiumi.us/xmi/ua/1O3nK/i/70f97ae114ebe4599620dfaa34636c36-sz_47216.png)

#这里创建实例需要选择地区，意思也就是将你的服务器建在你所选的地区，这里我喜欢东京我就选了东京，网速一般差别不大。

### 配置实例

1. 选择ubuntu server 18.04操作系统

![](http://img.xiumi.us/xmi/ua/1O3nK/i/94cedd98bb0d9db360bbf0ab2880926b-sz_196539.png)

2. 选择实例类型t2.micro（1GB），按推荐即可，后边高端的就要钱了

![](http://img.xiumi.us/xmi/ua/1O3nK/i/17dfb192bff501a76126704cd805776d-sz_136893.png)

3. 启动，安全组名称自定，这里安全组需要下载至本地，之后要用

![](http://img.xiumi.us/xmi/ua/1O3nK/i/b7dec78a069badae3ed55befb2df6d5d-sz_91996.png)

4. 这里建议开启账单警报，超额可是蛮贵

![](http://img.xiumi.us/xmi/ua/1O3nK/i/c67a92ba42d472e8d8a814d38466aea1-sz_32336.png)

5. 返回主页可以看到实例"running"

![](http://img.xiumi.us/xmi/ua/1O3nK/i/fa014a7dd27e443bfe39de9e3d910070-sz_52759.png)

### 配置服务器

1. 下载putty，通过putty连接服务器

![](http://img.xiumi.us/xmi/ua/1O3nK/i/90803c3454a50ac1aaba4ab0f029192e-sz_67729.png)

#如上图，我们需要SSH客户端，这里我们采用putty

2. 百度putty，进官网下载

![](http://img.xiumi.us/xmi/ua/1O3nK/i/e97bd680678626d69e8f67493baff1a1-sz_90781.png)

#按自己电脑下载对应安装包

3. 打开puttygen，创建"钥匙"

![](http://img.xiumi.us/xmi/ua/1O3nK/i/7291d2d650f205ab55468823501edebd-sz_24002.png)

（1）选中菜单栏中的**key**，选中下拉菜单**SSH-2 RSA**。

（2）点击**load**。文件类型选**all files**，找到之前保存的钥匙**arginsen.pem**，打开。

（3）点击**Load**下面的**Save private key**按钮。

（4）弹出的对话框点**yes**，就把它继续命名为**arginsen.ppk**保存

（5）接下来关闭puttygen，打开putty。

4. 通过putty进行对话

![](http://img.xiumi.us/xmi/ua/1O3nK/i/c058298f64ef1471c8ef79b18f54adde-sz_35893.jpg?x-oss-process=style/xmorient)

（1）点击**session**。右**Host Name(or IP address)**这里填写EC2实例连接时的公链，如下:

 >ubuntu@ 分配给你的公有DNS  

![](http://img.xiumi.us/xmi/ua/1O3nK/i/e23ab3986fe8e894fd62cac0d6738ced-sz_165424.png)

（2）端口保持默认的**22**，**Connection type**选择**SSH**。

（3）在左边目录中，展开“Connection”下的**SSH**，找到**Auth**，点击**Auth**，在右边点击**Browse…**，打开我们刚刚配的钥匙**arginsen.ppk**，然后回到刚才的**Session**。

![](http://img.xiumi.us/xmi/ua/1O3nK/i/0fea1d69ec5b16d78c77d6d74b91e1c6-sz_41309.png)

（4）点击右下角的**Open**。

#如果无法正常打开，收到以下错误消息：“Server refused our key”，解决方法参见：https://amazonaws-china.com/cn/premiumsupport/knowledge-center/ec2-server-refused-our-key/

#打开与云服务器对话的窗口如下：

![](http://img.xiumi.us/xmi/ua/1O3nK/i/610c340d9b865eca842c4ff95586b33c-sz_15654.png)

5. 搭建SS服务器端

此时是ubuntu下的终端，命令快捷键参见linux快捷键，依次输入如下命令:

（1）配置环境

```shell
sudo -s

apt-get update

apt-get install python-pip  //等下会提示Y/n，输入Y
```

![](http://img.xiumi.us/xmi/ua/1O3nK/i/bc45d56a029498aacabce8ec45def487-sz_64404.jpg?x-oss-process=style/xmorient)

（2）安装ss 

```shell
pip install shadowsocks   //出现successfully..即成功
```

（3）配置ss

```shell
vim /etc/shadowsocks.json   # 一般之后按两下c，左下显示状态喂inserting，这是可以输入以前信息

{

"server":"自己的公网ip",

"server_port":自己定，如8838,

"local_address":"127.0.0.1",

"local_port":1080,

"password":"自己定",

"timeout":300,

"method":"aes-256-cfb",

"fast_open": false

}
```

#完成后按esc键，键入`:wq`（意为书写并保存）

#公网ip见EC2 实例右下：

![](http://img.xiumi.us/xmi/ua/1O3nK/i/0481781a87a64d3a3927216dada79a3e-sz_18373.png?x-oss-process=image/resize,w_1080/auto-orient,1/crop,x_0,y_0,w_632,h_133)

（4）启动ss

```shell
ssserver -c /etc/shadowsocks.json -d start
```
（5）优化加速（上步已经ok）

```shell
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf

echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf

sysctl -p   //检查
```
![](http://img.xiumi.us/xmi/ua/1O3nK/i/26e99ed3451d3875fbf43fc38b3a5c8f-sz_95436.png)

#一般服务器选择ubuntu18.04，内核版本都会较高，因而不用更新，此外，若以上优化三步不成功便需要自行更换内核，若无此步骤也可使用ss

#若对话有失败的情况，再次进入需要再进入root模式

### 下载SS客户端

下载地址：[https://github.com/shadowsocks](https://github.com/shadowsocks)

#如下可以看见安卓端和windows段

至于ios和mac自行搜取

ios在store里好像有shadowrocket之类。

![](http://img.xiumi.us/xmi/ua/1O3nK/i/a51f390f7cdf6512a146c8e27d395092-sz_43297.png)

![](http://img.xiumi.us/xmi/ua/1O3nK/i/8818175337ec78e5d55767e50daf5c74-sz_44728.png)

win：

![](http://img.xiumi.us/xmi/ua/1O3nK/i/7744d85d90db8d158f5b2684134e281a-sz_17649.png)

#此面板只需填入：

服务器地址（你的公有ip地址）

服务器端口（之前自定）

密码（之前自定）

加密类型（选择aes-256-cfb与之前对应即可）

之后右键点击桌面图标 启用系统代理

![](http://img.xiumi.us/xmi/ua/1O3nK/i/f6684c1a575180d0c5db3c084ed03624-sz_31781.png)


现在就可以开启网上冲浪了

![](http://img.xiumi.us/xmi/ua/1O3nK/i/3d2367b42a2731c06a5caf0e1143683e-sz_781091.png)

好啦，ss的教程就到这里啦！



