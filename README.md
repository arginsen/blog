写在建站之初
===

### 来源

博客是托管在GitHub上的，其实一步步也是自己在探索，前不久简易搭成了自己的博客，但由于自己轻微的强迫症，所以一定要讲学习笔记和心情随笔分开。

自定义主页来自于[Cscoder](https://csdoker.com/)，托管在coding，随后的个人随笔网页也会托管在coding，因为放在国内应该会稍微快些，之前的博客是在hexo上选了一个很好看的h5主题，但是有视频嘛，加载确实慢(用AWS的Cloudfont加速，未用七牛云)，现在也注册了七牛云，认证完成后也可以用来托管网页，或者加速，作图床。

hexo博客主题来源[iTimeTraveler](https://itimetraveler.github.io/hexo-theme-hipaper/)的hipaper主题，第一眼看去真的很森系，很干净精简，适合博客，还有iTimeTraveler的[主页](https://itimetraveler.github.io)也很好看，首页切换的壁纸应该和主题里的[About](https://itimetraveler.github.io/hexo-theme-hipaper/about/)页面的壁纸来源差不多，只不过定时切换，以及加载更快

### 初衷

+ 记学习笔记，学过的操作，知识做笔记很重要，就跟写代码作注释一样

+ 写点随笔，之前有写过一些东西，但终究还是不太正式，而且很凌乱，有些过去的所思所想跟着丢了

+ 实际应用是学习的最好方法 

### 思路

将博客搭建在`arginsen.github.io`的一个项目里;命名**blog**；
生成网址即是`arginsen.github.io/blog/`;
此时在hexo主题中`root`要写为`/blog/`, `url`也要相应更换;
这时可以再关联二级域名`blog.arginsen.com`;
在**blog**仓库根目录添加**CNAME**文件

再可以继续在github中创建自定义主页的代码仓库，名为`arginsen.github.io`；
只有这样命名才能作为个人网站，上一步也是建立在这步的基础上；
进而关联自己的www二级域名，或直接关联主域名;
在**arginsen.github.io**仓库根目录添加CNAME文件

### 所悟

在一个平台上即可以搭建几个页面，这样的平台有：
[GitHub](https://github.com)、[腾讯云开发平台](https://dev.tencent.com/)、[七牛云](https://portal.qiniu.com)等；

我们可以将**username.github.io**或**username.coding.me**作为仓库/项目的名字，以此搭建个人主页(自定义)，将静态html打包进仓库部署，用CAMEN关联个人的主域名或www二级域名，就是自己的主页啦；

接下来继续在平台创建**blog**或**essay**仓库/项目，以此将自己的博客放在平台个站下边，表现为`username.arginsen.io/blog`，这样就不用另找平台建了

最后在自定义主页配置好下一级博客的网址，就算整个网站框架完善了；若是hexo，再在docs文件夹中继续创建博客内的分页**archive**、**about**等

### 语终

自己也确实学到好多东西吧，做喜欢的事真的是很有意思，也会充满干劲哈哈

随后也将会把之前攒的几篇文章搬上来