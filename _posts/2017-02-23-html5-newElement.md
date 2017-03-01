---
layout: post
title:  Html5 新增标签及属性
date:   2017-02-23 15:58:00 +0800
categories: HTML
tag: HTML5
---

* content
{:toc}


[HTML 中的表单新属性](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Forms_in_HTML)

[HTML5 标签列表](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/HTML5/HTML5_element_list)

-`<section>` 定义文档中的一个章节。

-`<nav>` 定义只包含导航链接的章节。

-`<article>` 定义可以独立于内容其余部分的完整独立内容块。

-`<aside>` 定义和页面内容关联度较低的内容——如果被删除，剩下的内容仍然很合理。

-`<header>` 定义页面或章节的头部。它经常包含 logo、页面标题和导航性的目录。

-`<footer>` 定义页面或章节的尾部。它经常包含版权信息、法律信息链接和反馈建议用的地址。

-`<main>` 定义文档中主要或重要的内容。

-`<figure>` 代表一个和文档有关的图例。
-`<figcaption>` 代表一个图例的说明。

```
<figure>
	<img src="..." alt="picture">	
	<figcaption>Fig1. picture</figcaption>
</figure>
```

-`<mark>` 代表一段需要被高亮的引用 文字。

-`<ruby>` 代表被ruby 注释 标记的文本，如中文汉字和它的拼音。
-`<rt>` 代表ruby 注释 ，如中文拼音。
-`<rp>` 代表 ruby 注释两边的额外插入文本 ，用于在不支持 ruby 注释显示的浏览器中提供友好的注释显示。

```
<ruby>
	明日 <rp>(</rp><rt>Ashita</rt><rp>)</rp>
</ruby>
```

-`<bdi>` 代表需要脱离 父元素文本方向的一段文本。它允许嵌入一段不同或未知文本方向格式的文本。

-`<wbr>` 代表建议换行 (Word Break Opportunity) ，当文本太长需要换行时将会在此处添加换行符。

-`<embed>` 代表一个嵌入 的外部资源，如应用程序或交互内容

-`<video>` 代表一段视频 及其视频文件和字幕，并提供了播放视频的用户界面。

-`<audio>` 代表一段声音 ，或音频流 。

-`<source>` 为 `<video>` 或 `<audio>` 这类媒体元素指定媒体源 。

-`<track>` 为 `<video>` 或 `<audio>` 这类媒体元素指定文本轨道（字幕） 。

-`<canvas>` 代表位图区域 ，可以通过脚本在它上面实时呈现图形，如图表、游戏绘图等。

-`<svg>` 定义一个嵌入式矢量图 。

-`<math>` 定义一段数学公式 。

-`<datalist>` 代表提供给其他控件的一组预定义选项 。

```
<label>Choose a browser from this list:
<input list="browsers" name="myBrowser" /></label>
<datalist id="browsers">
  <option value="Chrome">
  <option value="Firefox">
  <option value="Internet Explorer">
  <option value="Opera">
  <option value="Safari">
</datalist>
```

-`<keygen>` 代表一个密钥对生成器 控件。

-`<output>` 代表计算值 。

-`<progress>` 代表进度条 。

-`<meter>` 代表滑动条 。

-`<details>` 代表一个用户可以(点击)获取额外信息或控件的小部件 。
-`<summary>` 代表 `<details>` 元素的综述 或标题 。

```
<details>
  <summary>Some details</summary>
  <p>More info about the details.</p>
</details>
```

-`<menuitem>` 代表一个用户可以点击的菜单项。
-`<menu>` 代表菜单。