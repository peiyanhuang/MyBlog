---
layout: post
title: IntersectionObserver 懒加载图片
date: 2020-12-18 19:58:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

1、利用 `getBoundingClientRect` API得到当前元素与视窗的距离来判断

```js
function loadImg(arr) {
  for (var i = 0, len = arr.length; i < len; i++) {
    if (
      arr[i].getBoundingClientRect().top < document.documentElement.clientHeight &&
      !arr[i].isLoad
    ) {
      arr[i].isLoad = true;
      arr[i].style.cssText = "transition: ''; opacity: 0;";
      if (arr[i].dataset) {
        aftLoadImg(arr[i], arr[i].dataset.original);
      } else {
        aftLoadImg(arr[i], arr[i].getAttribute("data-original"));
      }
      (function (i) {
        setTimeout(function () {
          arr[i].style.cssText = "transition: 1s; opacity: 1;";
        }, 16);
      })(i);
    }
  }
}
function aftLoadImg(obj, url) {
  var oImg = new Image();
  oImg.onload = function () {
    obj.src = oImg.src;
  };
  oImg.src = url;
}
```

2、利用h5的新API `IntersectionObserver` 来实现

```js
const io = new IntersectionObserver(callback);
let imgs = document.querySelectorAll('[data-src]');
function callback(entries){
  entries.forEach((item) => {
    if(item.isIntersecting){
      item.target.src = item.target.dataset.src
      io.unobserve(item.target)
    }
  })
}

imgs.forEach((item)=>{
  io.observe(item)
})
```
