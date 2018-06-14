---
title: html5
comments: true
tags:
  - html5
  - egret
date: 2018-06-01 10:45:45
---

###简介

在html5平台上,视频,音频，图像，动画以及同电脑的交互都被标准化。新特性：
- 绘画增加了canvas元素
- 用于媒介回放的video和audio元素
- 对本地离线存储的更好的支持
- 新的特殊元素，比如 
afafaf
##### 视频

##div division section 一片

<!DOCTYPE html>
<html>
<body>
    <div style="text-align: center;">
        <button onclick="playPause()"> 播放/暂停 </button>
        <button onclick="makeBig()"> 大 </button>
        <button onclick="makeSmall()"> 小</button>
        <br />
        <video id='video1' width="320" height="240" style="margin-top: 15px;">
            <source src="http://video.699pic.com/videos/43/51/00/N5D6dJTy80vD1527435100_10s.mp4" type="video/mp4">
                your browser does not support html5 vidoe.
        </video>
    </div>
    
    <script type="text/javascript">
        var myVedio=document.getElementById("video1");

        function playPause()
        {
            if (myVideo.paused) 
              myVideo.play(); 
            else 
              myVideo.pause();
        }

        function makeBig()
        { 
        myVideo.width=560; 
        } 

        function makeSmall()
        { 
        myVideo.width=320; 
        } 

        function makeNormal()
        { 
        myVideo.width=420; 
        } 

    </script>

</body>
</html>


##拖放


<!DOCTYPE HTML>
<html>
<head>
<style type="text/css">
#div1 {width:198px; height:66px;padding:10px;border:1px solid #aaaaaa;}
</style>
<script type="text/javascript">
function allowDrop(ev)
{
ev.preventDefault();
}

function drag(ev)
{
ev.dataTransfer.setData("Text",ev.target.id);
}

function drop(ev)
{
ev.preventDefault();
var data=ev.dataTransfer.getData("Text");
ev.target.appendChild(document.getElementById(data));
}
</script>
</head>
<body>

<p>请把 W3School 的图片拖放到矩形中：</p>

<div id="div1" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
<br />
<img id="drag1" src="http://www.w3school.com.cn/i/eg_dragdrop_w3school.gif" draggable="true" ondragstart="drag(event)" />

</body>
</html>

