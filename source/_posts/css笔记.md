---
title: css笔记
date: 2019-02-27 09:43:11
tags: 
    - css
categories: css
---
### 快速导航  
[1、隐藏滚动条](#mark1)  
[2、div居中](#mark2)  
[3、元素动画(左右移动)](#mark3)  
[4、选中元素内指定子元素](#mark4)  
[5、颜色渐变](#mark5)
<!--more-->

<a id="mark1"></a>

#### 1、隐藏滚动条
```css
.inner-container {
    position: absolute;
    left: 0;
    top: 0;
    bottom: 60px;
    overflow-x: hidden;
    overflow-y: scroll;
    scrollbar-width: none;
}

/* for Chrome 只针对谷歌浏览器*/
.inner-container::-webkit-scrollbar {
    display: none;
}
```

<a id="mark2"></a>

#### 2、div居中
```css
#testDiv {
    width: 300px;
    margin: 0 auto;
  }
```
<a id="mark3"></a>

#### 3、元素动画(左右移动)
```css
#enterLeft {
    position: relative;
    -webkit-animation: moveLeft 0.6s infinite alternate;
    animation: moveLeft 0.6s infinite alternate;

  }

@-webkit-keyframes moveLeft {
    0% { left: 0px; }
    100% { left: -10px; }
  }

@keyframes moveLeft {
    0% { left: 0px; }
    100% { left: -10px; }
  }

```
<a id="mark4"></a>

#### 4、选中元素内指定子元素
```css
/* 第一个li */ 
#testDiv > li:first-child{}  

/* 最后一个li */
#testDiv > li:last-child{} 

/* 第五个   */
#testDiv > li:nth-child(5){} 
```

<a id="mark5"></a>

#### 5、颜色渐变
```css
/* 双色 */
/* 方向可以是 left、right、bottom left等等 */
/* 也可以指定角度 90deg */
#testDiv1 {
    width: 500px;
    height: 2.5px;
    float: left;
    background: -webkit-linear-gradient(left, rgba(198, 198, 198, 0), rgba(198, 198, 198, 1));
    /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(right, rgba(198, 198, 198, 0), rgba(198, 198, 198, 1));
    /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(right, rgba(198, 198, 198, 0), rgba(198, 198, 198, 1));
    /* Firefox 3.6 - 15 */
    background: linear-gradient(to right, rgba(198, 198, 198, 0), rgba(198, 198, 198, 1));
    /* 标准的语法（必须放在最后） */
}

/* 3种颜色均匀分布 */
#testDiv2 {
    height: 200px;
    background: -webkit-linear-gradient(red, green, blue); /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(red, green, blue); /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(red, green, blue); /* Firefox 3.6 - 15 */
    background: linear-gradient(red, green, blue); /* 标准的语法（必须放在最后） */
}

/* 7种颜色均匀分布 */
#testDiv3 {
    height: 200px;
    background: -webkit-linear-gradient(red, orange, yellow, green, blue, indigo, violet); /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(red, orange, yellow, green, blue, indigo, violet); /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(red, orange, yellow, green, blue, indigo, violet); /* Firefox 3.6 - 15 */
    background: linear-gradient(red, orange, yellow, green, blue, indigo, violet); /* 标准的语法（必须放在最后） */
}

/* 颜色分布指定比例 */
#testDiv4 {
    height: 200px;
    background: -webkit-linear-gradient(red 10%, yellow 50%, blue 90%); /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(red 10%, yellow 50%, blue 90%); /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(red 10%, yellow 50%, blue 90%); /* Firefox 3.6 - 15 */
    background: linear-gradient(red 10%, yellow 50%, blue 90%); /* 标准的语法（必须放在最后） */
}
```


