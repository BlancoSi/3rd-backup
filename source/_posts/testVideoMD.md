---
title: testVideoMD
date: 2019-05-07 20:36:01
tags:
---
<div class="aspect-ratio">
  <iframe src="//player.bilibili.com/player.html?aid=13379801&cid=21925894&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>
</div>
 
<style>
/* 这个规则规定了iframe父元素容器的尺寸，我们要去它的宽高比应该是 25:14 */
.aspect-ratio {
  position: relative;
  width: 100%;
  height: 0;
  padding-bottom: 56%; /* 高度应该是宽度的56% */
}
 
/* 设定iframe的宽度和高度，让iframe占满整个父元素容器 */
.aspect-ratio iframe {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
}
</style>

```
上面这种：
<div class="aspect-ratio">
  <iframe src="//player.bilibili.com/player.html?aid=13379801&cid=21925894&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>
</div>
 
<style>
/* 这个规则规定了iframe父元素容器的尺寸，我们要求它的宽高比应该是 25:14 */
.aspect-ratio {
  position: relative;
  width: 100%;
  height: 0;
  padding-bottom: 56%; /* 高度应该是宽度的56% */
}
 
/* 设定iframe的宽度和高度，让iframe占满整个父元素容器 */
.aspect-ratio iframe {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
}
</style>

另外一种：
<iframe id='video' src="//player.bilibili.com/player.html?aid=13379801&cid=21925894&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width=100%> </iframe>
<script type="text/javascript">
document.getElementById("video").style.height=document.getElementById("video").scrollWidth*0.75+"px";
</script>
```
