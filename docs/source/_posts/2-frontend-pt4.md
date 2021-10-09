---
title: 前端 | 练习四之 js 编写音乐播放器
date: 2020-6-20
tags: 
- JavaScript
- Frontend
categories: practice
photos:
    - /blog/img/javascript.jpg
---
<br>
<!--more-->

# page

- 项目地址: https://github.com/arginsen/Music_player
- 网页展示: https://cd.lixupeng.cn


# code

## html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <title>草东没有派对</title>
    <link rel="icon" href="./image/favicon.ico">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.13.0/css/all.min.css">
    <link rel="stylesheet" type="text/css" href="style.css">
</head>

<body>
    <div class="player">

        <!-- 设定播放曲目详情 -->
        <div class="details">
            <div class="now-playing">PLAYING x OF y</div>
            <div class="track-art"></div>
            <div class="track-name">Track Name</div>
            <div class="track-artist">Track Artist</div>
        </div>

        <!-- 设定播放按钮 -->
        <div class="buttons">
            <div class="prev-track" onclick="prevTrack()">
                <i class="fa fa-step-backward fa-2x"></i>
            </div>
            <div class="playpause-track" onclick="playpauseTrack()">
                <i class="fa fa-play-circle fa-5x"></i>
            </div>
            <div class="next-track" onclick="nextTrack()">
                <i class="fa fa-step-forward fa-2x"></i>
            </div>
        </div>

        <!-- 设定进度调节滑块-->
        <div class="slider_container">
            <div class="current-time">00:00</div>
            <input type="range" min="1" max="100" value="0" class="seek_slider" onchange="seekTo()">
            <div class="total-duration">00:00</div>
        </div>

        <!-- 设定音量调节滑块-->
        <div class="slider_container">
            <i class="fa fa-volume-down"></i>
            <input type="range" min="1" max="100" value="99" class="volume_slider" onchange="setVolume()">
            <i class="fa fa-volume-up"></i>
        </div>
    </div>

    <script src="main.js"></script>
</body>

</html>
```

## css

```css
body {
    background-color: lightgreen;

    /* Smoothly transition the background color */
    transition: background-color 1s;
}

/* Using flex with the column direction to 
   align items in a vertical direction */
.player {
    height: 95vh;
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
}

.details {
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
    margin-top: 25px;
}

.track-art {
    margin: 25px;
    height: 300px;
    width: 300px;
    background-image: URL("https://source.unsplash.com/Qrspubmx6kE/640x360");
    background-size: cover;
    background-position: center;
    border-radius: 15%;
}

/* Changing the font sizes to suitable ones */
.now-playing {
    font-size: 1rem;
}

.track-name {
    font-size: 3rem;
}

.track-artist {
    font-size: 1.2rem;
}

/* Using flex with the row direction to 
   align items in a horizontal direction */
.buttons {
    display: flex;
    flex-direction: row;
    align-items: center;
}

.playpause-track,
.prev-track,
.next-track {
    padding: 25px;
    opacity: 0.8;

    /* Smoothly transition the opacity */
    transition: opacity .2s;
}

/* Change the opacity when mouse is hovered */
.playpause-track:hover,
.prev-track:hover,
.next-track:hover {
    opacity: 1.0;
}

/* Define the slider width so that it scales properly */
.slider_container {
    width: 75%;
    max-width: 400px;
    display: flex;
    justify-content: center;
    align-items: center;
}

/* Modify the appearance of the slider */
.seek_slider,
.volume_slider {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    height: 5px;
    background: black;
    opacity: 0.7;
    -webkit-transition: .2s;
    transition: opacity .2s;
}

/* Modify the appearance of the slider thumb */
.seek_slider::-webkit-slider-thumb,
.volume_slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    width: 15px;
    height: 15px;
    background: white;
    cursor: pointer;
    border-radius: 50%;
}

/* Change the opacity when mouse is hovered */
.seek_slider:hover,
.volume_slider:hover {
    opacity: 1.0;
}

.seek_slider {
    width: 60%;
}

.volume_slider {
    width: 30%;
}

.current-time,
.total-duration {
    padding: 10px;
}

i.fa-volume-down,
i.fa-volume-up {
    padding: 10px;
}

/* Change the mouse cursor to a pointer 
   when hovered over */
i.fa-play-circle,
i.fa-pause-circle,
i.fa-step-forward,
i.fa-step-backward {
    cursor: pointer;
}
```

## js

```js
// 定位文档中标签元素 
// 将选中的元素节点赋给变量 
let now_playing = document.querySelector(".now-playing");
let track_art = document.querySelector(".track-art");
let track_name = document.querySelector(".track-name");
let track_artist = document.querySelector(".track-artist");

let playpause_btn = document.querySelector(".playpause-track");
let next_btn = document.querySelector(".next-track");
let prev_btn = document.querySelector(".prev-track");

let seek_slider = document.querySelector(".seek_slider");
let volume_slider = document.querySelector(".volume_slider");
let curr_time = document.querySelector(".current-time");
let total_duration = document.querySelector(".total-duration");

// 指定全局变量 
let track_index = 0;
let isPlaying = false;
let updateTimer;

// 创建 audio 标签元素
let curr_track = document.createElement('audio');

// 传入一个数组，包含所有歌的所有信息 
let track_list = [
    {
        name: "intro",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/1-intro.mp3",
    },
    {
        name: "丑",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/2-chou.mp3",
    },
    {
        name: "烂泥",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/3-lanni.mp3",
    },
    {
        name: "勇敢的人",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/4-yongganderen.mp3",
    },
    {
        name: "大风吹",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/5-dafengchui.mp3"
    },
    {
        name: "埃玛",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/6-aima.mp3",
    },
    {
        name: "等",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/7-deng.mp3",
    },
    {
        name: "鬼",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/8-gui.mp3",
    },
    {
        name: "在",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/9-zai.mp3",
    },
    {
        name: "山海",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/10-shanhai.mp3"
    },
    {
        name: "我们",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/11-women.mp3",
    },
    {
        name: "情歌",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/12-qingge.mp3",
    },
    {
        name: "顶楼",
        artist: "草东没有派对",
        image: "image/chounuer.jpg",
        path: "music/13-dinglou.mp3",
    },
];


// 定义歌曲加载函数
function loadTrack(track_index) {
    // 重置前一首
    clearInterval(updateTimer);
    resetValues();

    // 加载新的轨道，利用 track_index 定位歌曲，在通过 path 获取路径
    curr_track.src = track_list[track_index].path;
    curr_track.load();  // audio 自带的方法

    // 切换一首歌曲，更新其显示信息，封面/歌名/歌手
    track_art.style.backgroundImage = "url(" + track_list[track_index].image + ")";
    track_name.textContent = track_list[track_index].name;
    track_artist.textContent = track_list[track_index].artist;
    now_playing.textContent = "PLAYING " + (track_index + 1) + " OF " + track_list.length;

    // 设定滑块跟着播放进度来移动，每隔 1000 毫秒更新
    updateTimer = setInterval(seekUpdate, 1000);

    // 添加一个事件侦听器，播放结束时，执行 nextTrack 函数
    curr_track.addEventListener("ended", nextTrack);

    // 切换同时更新背景 
    random_bg_color();
}

// 定义生成随机色背景函数
function random_bg_color() {
    // Get a random number between 64 to 256 
    // (for getting lighter colors) 
    let red = Math.floor(Math.random() * 256) + 64;
    let green = Math.floor(Math.random() * 256) + 64;
    let blue = Math.floor(Math.random() * 256) + 64;

    // Construct a color with the given values 
    let bgColor = "rgb(" + red + ", " + green + ", " + blue + ")"; 

    // Set the background to the new color 
    document.body.style.background = bgColor;
}


// 定义歌曲重置参数的函数
function resetValues() {
    curr_time.textContent = "00:00";
    total_duration.textContent = "00:00";
    seek_slider.value = 0;
}


// 定义播放暂停键函数
function playpauseTrack() {
    // 判断当前状态，默认 true 播放，false 暂停
    if (!isPlaying) playTrack();
    else pauseTrack();
}

// 定义播放函数
function playTrack() {
    // 生成的 audio 元素执行 play() 方法
    curr_track.play();
    isPlaying = true;

    // 此时为播放状态，改变按钮图标为暂停，表示当前正在播放
    playpause_btn.innerHTML = '<i class="fa fa-pause-circle fa-5x"></i>';
}

// 定义暂停函数
function pauseTrack() {
    // 生成的 audio 元素执行 pause() 方法
    curr_track.pause();
    isPlaying = false;

    // 此时为暂停状态，改变按钮图标为播放，表示点击后播放歌曲
    playpause_btn.innerHTML = '<i class="fa fa-play-circle fa-5x"></i>';;
}

// 定义播放下一首函数
function nextTrack() {
    // 判断当前播放为第几首，如果当前索引没超出总歌曲数，那么索引递增 1
    // 如果是最后一首，设定 track_index 为 0
    if (track_index < track_list.length - 1)
        track_index += 1;
    else track_index = 0;

    // Load and play the new track 
    loadTrack(track_index);
    playTrack();
}

// 定义播放上一首函数
function prevTrack() {
    // 判断当前播放为第几首，如果当前索引大于 0，也就是不为第一首时，递减 1
    // 如果是第一首（索引为 0，那么就跳转到最后一首，）
    if (track_index > 0)
        track_index -= 1;
    else track_index = track_list.length;

    // Load and play the new track 
    loadTrack(track_index);
    playTrack();
}


// 定义一个播放定位函数
function seekTo() {
    // 滑块的值的百分比乘总的持续时间表示当前当前播放进度
    seekto = curr_track.duration * (seek_slider.value / 100);
    curr_track.currentTime = seekto;
}

// 定义一个音量设定函数
function setVolume() {
    // 根据音量滑块百分比来设定当前的音量 
    curr_track.volume = volume_slider.value / 100;
}

// 定义设定进度跳转的函数
function seekUpdate() {
    let seekPosition = 0;

    // 检查当前歌曲时间是不是一个有效数值
    // 如果是则按百分位置比时长计算当前位置
    if (!isNaN(curr_track.duration)) {
        seekPosition = curr_track.currentTime * (100 / curr_track.duration);
        seek_slider.value = seekPosition;

        // 转化播放的时长和总时长，显示成 00:00 样式
        let currentMinutes = Math.floor(curr_track.currentTime / 60);
        let currentSeconds = Math.floor(curr_track.currentTime - currentMinutes * 60);
        let durationMinutes = Math.floor(curr_track.duration / 60);
        let durationSeconds = Math.floor(curr_track.duration - durationMinutes * 60);

        // 设定页面进度的显示样式，不够十位的补个 0
        if (currentSeconds < 10) {
            currentSeconds = "0" + currentSeconds;
        }
        if (durationSeconds < 10) {
            durationSeconds = "0" + durationSeconds;
        }
        if (currentMinutes < 10) {
            currentMinutes = "0" + currentMinutes;
        }
        if (durationMinutes < 10) {
            durationMinutes = "0" + durationMinutes;
        }

        // 展示当前的进度时长和总时长，00:00 形式
        curr_time.textContent = currentMinutes + ":" + currentSeconds;
        total_duration.textContent = durationMinutes + ":" + durationSeconds;
    }
}


// 加载第一首曲子，函数变量会被提前，每次进入页面时曲子索引都为初始设定的 0
loadTrack(track_index);
```

# link

[JS 之创建音乐播放器](https://mp.weixin.qq.com/s/EkblhT3wAE03aGMSosQR2g)
