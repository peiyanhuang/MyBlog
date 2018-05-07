---
layout: post
title:  滚轮事件(mousewheel/DOMMouseScroll)
date:   2018-05-07 13:08:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

### 1.鼠标滚轮事件

`firefox` 是 `DOMMouseScroll` 事件，对应的滚轮信息（向前滚还是向后滚）存储在 `detail` 属性中，往下滚，这个属性值是 3 的倍数，反之，是 -3 的倍数。(我测试时 detail 出现过 4 )

`firefox` 之外的其他浏览器是 `mousewheel` 事件，对应的滚轮信息存储在 `wheelDelta` 属性中，往下滚，这个属性值是 -120 的倍数，反之， 120 的倍数。(我测试时 Chrome 往下滚出现的是 -136 )

可以这样添加滚轮事件：

```js
if (navigator.userAgent.toUpperCase().indexOf('FIREFOX') === -1) {
    document.body.addEventListener('mousewheel', scrollMouse);
} else {
    document.body.addEventListener('DOMMouseScroll', scrollMouse);
}

function scrollMouse(event) {
    let delta = ((event) => {
        if (event.wheelDelta) {
            return -event.wheelDelta;
        } else {
            // 兼容 firefox
            return event.detail;
        }
    })(event);

    if (delta > 0) {
        // 向下滚动
        console.log('down')
    } else {
        // 向上滚动
        console.log('up')
    }
}
```

### 2.全局滚动

[demo]()看这