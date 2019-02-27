---
title: jquery笔记
date: 2019-02-27 09:43:11
tags: 
    - jquery
# 赞赏    
reward: false
# 目录   
toc: true 
---
### 快速导航  
[1、监听浏览器窗口变化](#mark1)  
[2、添加\移除class](#mark2)  
[3、设置css](#mark3)  


<a id="mark1"></a>

#### 1、监听浏览器窗口变化
```js
$(window).resize(function () {          //当浏览器大小变化时
    //浏览器时下窗口可视区域高度
    console.log($(window).height()); 

    //浏览器时下窗口文档的高度         
    console.log($(document).height());   

    //浏览器时下窗口文档body的高度     
    console.log($(document.body).height());  

    //浏览器时下窗口文档body的总高度 包括border padding margin 
    console.log($(document.body).outerHeight(true)); 
});

```
<a id="mark2"></a>

#### 2、添加\移除class
```js
$("#test").addClass("testClass");
$("#test").removeClass("testClass");
```

<a id="mark3"></a>

#### 3、设置css
```js
$("#testDiv").css("height", 100);
```
