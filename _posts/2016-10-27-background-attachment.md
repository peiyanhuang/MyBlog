---
layout: post
title:  视差滚动效果
date:   2016-10-27 19:58:00 +0800
categories: JS
tag: JS
---

* content
{:toc}



所谓的视差滚动，就是在页面滚动过程中，多层次的元素进行不同程度的位移，带来立体的视差效果。还有很多的奇思妙想的展现方式，都是滚动页面触发的，也可称为视差滚动。视差滚动里面最基础的就是切换背景，这点其实一个CSS就满足了  

### 视差滚动原理

	background-attachment: fixed || scroll || local

默认是scroll，内容跟着背景走，而视差滚动页面里用 **fixed** ，背景相对页面固定而不跟内容滚动。  

### 添加事件

在scroll事件上添加一些动画事件  

	window.addEventListener('scroll',function(e){
        var scrollTop = window.scrollY;
        if(scrollTop > 0 && scrollTop < articleHeight){
            title1.classList.add('title-anim');
            content1.classList.add('content-anim');
        }else if(scrollTop >= articleHeight && scrollTop < articleHeight*2){
            title2.classList.add('title-anim');
            content2.classList.add('content-anim');
        }else if(scrollTop >= articleHeight*2 && scrollTop < articleHeight*3){
            title3.classList.add('title-anim2');
            content3.classList.add('content-anim');
        }
    })

原文：[视差滚动的爱情故事](http://www.alloyteam.com/2014/01/parallax-scrolling-love-story/)  
	[视差滚动的爱情故事之优化篇](http://www.alloyteam.com/2014/02/optimized-articles-of-parallax-scrolling-love-story/)