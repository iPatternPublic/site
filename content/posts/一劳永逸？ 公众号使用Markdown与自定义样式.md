---
title: "一劳永逸？ 公众号使用Markdown与自定义样式"
author: "iPattern"
date: 2019-04-23T16:44:00+08:00
description: "This is meta description"
categories: ["编程 Programming"]
tags: ["Experiment"]
---

还没写没几篇公众号，已经被微信公众号编辑器和第三方编辑器折磨的无法忍受了！

每次写好的文章都需要去重新排版一次，东托西拽，费时费力，流程割裂。

经过一番搜索与折腾，总算是通过`Markdown`和`CSS`形成了一个初步解决方案，实现了

==禅意写作，一键粘贴，行云流水，一以贯之～==

也许够爽一阵子了，作文测试以志之。



### 根本诉求


> 所写即所得，所见即所得



### 实现途径

+ Markdown
+ CSS

![Markdown-mark.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/4/48/Markdown-mark.svg/64px-Markdown-mark.svg.png)

> **Markdown**是一种轻量级标记语言，创始人为约翰·格鲁伯（英语：John Gruber）。它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML或者HTML文档”。这种语言吸收了很多在电子邮件中已有的纯文本标记的特性。



![CSS-shade.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/9/93/CSS-shade.svg/275px-CSS-shade.svg.png)

> **层叠样式表**（英语：**C**ascading **S**tyle **S**heets，简写**CSS**），又称**串样式列表**、**级联样式表**、**串接样式表**、**阶层式样式表**，一种用来为结构化文档（如HTML文档或XML应用）添加样式（字体、间距和颜色等）的计算机语言，由W3C定义和维护。当前最新版本是CSS2.1，为W3C的推荐标准。CSS3现在已被大部分现代浏览器支持，而下一版的CSS4仍在开发中。



无意间测试了一下==Typora==写作的Markdown文章复制到微信公众号编辑器，发现可以保留部分样式。于是一发不可收拾，又在自定义`css`里面摸索了一阵子，解决了几个主要需求：

+ 自定义字体、字号、颜色
+ 标题
+ 高亮样式
+ 行距
+ 超链接图片

个人比较喜欢`vue`主题，打开`css`一看快800行了，这不是难为懒宅，完全写一个是不可能的了。直接另存一个开改！

![Screen Shot 2019-04-23 at 4.13.12 PM](https://ws1.sinaimg.cn/large/006tNc79gy1g2cn4l3t9lj30u00ven5u.jpg)

如对正文样式的修改：

```css
body {
    font-family: PingFangTC-light, Source Sans;
    color: #34495e;
    font-size: 14px; 
    -webkit-font-smoothing: antialiased;
    line-height: 2; 
    letter-spacing: 0;
    margin: 0;
    overflow-x: hidden;
}
```



折腾了一圈后才搜到早有高手使用==Ulysses app==和==iPic==实现了，难道是一开始搜的关键词不对，好坑。。。不过，好在Typora开源免费对吧。。。

本来还想改的像==Bear==默认主题那样，彩色列表符号之类的一些小细节，可惜`css`功力不足（也可能微信的富文本不兼容），未能实现。挖坑弃之。。。

🔚