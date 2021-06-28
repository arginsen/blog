---
title: 前端 | 使用 video.js 库
date: 2020-12-15
tags: 
- work
- video.js
- Frontend
categories: notes
photos:
  - /blog/img/frontend.jpg
---

<br>
<!--more-->

> video.js 有更好的兼容性，几乎兼容所有的浏览器，并且优先使用 html5，在不支持的浏览器中，会自动使用flash进行播放。此外也相当于定义了一套视频播放的 UI 组件，更加方便，且界面可以做到定制。

# 文档

官方教程：
  `https://videojs.com/getting-started`
  `https://docs.videojs.com/tutorial-setup.html`
一套 videojs 皮肤：
  `https://github.com/arginsen/blog/doc/lib/videojs`

# videojs 的使用

## 引入

> html 头部引入库

```html
<!-- 路径自定义 -->
<link rel="stylesheet" type="text/css" href="/doc/lib/videojs/videojs-7.10.2.min.css" />
<!-- 一套玩家制作的皮肤 -->
<link rel="stylesheet" type="text/css" href="/doc/lib/videojs/vjs-sublime-skin.min.css">
<script src="/doc/lib/videojs/videojs-7.10.2.min.js"></script>
```


## 编写

### 直接定义

```html
<video id="example_video_1" class="video-js vjs-sublime-skin" controls width="640" height="264">
  <source src="http://vjs.zencdn.net/v/oceans.mp4" type='video/mp4' />
</video> 
```

### 通过 js 插入

#### html 部分

```html
<video id="example_video_2" class="video-js vjs-sublime-skin">
  <p class="vjs-no-js">此视频无法播放</p>
</video>
```

#### js 部分

```js
const playVideo = videojs('example_video_2', {
  controls: false, // 播放窗控件
  autoplay: false, //自动
  loop: true, // 循环
  muted: true, // 静音
  preload: 'auto', // 预加载
  techOrder: ["html5"],
}, () => {
  playVideo.src({type: 'video/mp4', src: 'http://vjs.zencdn.net/v/oceans.mp4'});
});

playVideo.ready(() => {
  playVideo.playbackRate(1); // 播放速度
  playVideo.play()
    .then(_ => {
      // ...
    })
    .catch(err => {
      console.log(err);
    });
});
```

# 加点东西

## 点击弹出在屏幕正中，背景黑幕不可点击

### html

```html
<div class="video"></div>
```

### js 部分

```js
const videoEl = `<div class="black-sheild">
                  <div class="show-video">
                    <video id="now-video" class="video-js vjs-sublime-skin">
                      <p class="vjs-no-js">此视频无法播放</p>
                    </video> 
                  </div>
                </div> `;
$('.video').after(videoEl);

$('.video').click(function() {
  playVideo.src({type: 'video/mp4', src: nowClassLink});
  playVideo.ready(function() {
    playVideo.play();
  });

  // 展示
  $(".black-sheild").show();
  $('.show-video').fadeIn(150);
    // 添加关闭按钮
  const closeButton = '<a id="close-video" class="vjs-control-bar" href="javascript:;">×</a>';
  $('#now-video').append(closeButton);

  // 点击按钮关闭视频
  $('#close-video').click(function() {
    $(".black-sheild").fadeOut(100, function() {
      playVideo.ready(function() {
        playVideo.pause();
      });
    });
  });
});
```

### css 部分

```css
.black-sheild {
  display: none;
  width: 100%;
  height: 100%;
  background-color: rgba(10,10,10,0.4);
  position: fixed;
  top: 0;
  left: 0;
}

.show-video {
  width: 1024px;
  height: 576px;
  position: absolute;
  top: 50%;
  left: 50%;
  margin-left: -512px;
  margin-top: -288px;
  display: none;
}

#close-video {
  display: block;
  width: 40px;
  height: 30px;
  font-size: 30px;
  position: relative;
  float: right;
  text-align: center;
  padding-top: 1px;
  border-radius: 0 5px 0 0;
  transition: background .2s ease-out;
}

#close-video:hover {
  background: rgba(255, 255, 255, 0.3);
}

.video-js {
  width: 100%;
  height: 100%;
  box-shadow: 0 2px 15px 1.5px rgb(39 39 39 / 50%);
  border-radius: 5px;
}

.video-js .vjs-picture-in-picture-control {
  display: none;
}
```

## 动态弹出

> 在上边代码基础上，视频从上方掉出

```css
/* css 设置初始 top 为 0 */
.show-video {
  top: 0;
}
```

```js
// js 给视频容器加点动画
$(".black-sheild").show();
$(".show-video").slideDown(50)
                .animate({top: 0}, 50)
                .animate({top: '56%'}, 160)
                .animate({top: '48%'}, 180)
                .animate({top: '51%'}, 150)
                .animate({top: '50%'}, 100);
```

效果见：http://www.aiipu.com/xserver/

# 参考链接

https://docs.videojs.com/
https://www.cnblogs.com/Renyi-Fan/p/9539663.html
