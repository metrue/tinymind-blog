---
title: 闲聊网页 Responsive 设计的一些小点
date: '2016-06-27T09:20:02.322Z
---

科技让我们的生活越来好玩了，信息的载体从原来的书信，报纸，书籍，扩散了电子邮件，社交站点，多媒体网页等，我们习惯了有时候静止的在自己的笔记本上工作，浏览信息，处理邮件，有时候移动的在自己的iPhone, iPhone完成这些事情。

然后信息，也就是文字，图片的元素在各种设备显示的效果就成为了前端工程师们最头疼的问题之一，如何在各种设备显示器都能良好的展示，就是传说中的 Responsive。

## media query

media query 真是一个好东西，简单的语法，好用的功能。

```
@media <media-query-list> {
  <group-rule-body>
}
```

简单来说，media query 是一个css样式覆写功能，当然信息展示的设备满足 media query 的条件时候，就使用增加 media query 大括号中的定义的那些css rules. 比如下面这段代码的意思是：在所有可视宽度小于等于 768px 的设备中使用大括号中的样式,也就是<p>的字体大小变成24px。

```
@media all and (max-width: 768px) {
  p {
    font-size: 24px;
  }
}
```

media query 一共支持如下的media types: all, print, screen, speech. 但是并不是所有的浏览器实现了这些media types，比如此时 Firefox  就只支持了 all 和 screen 类别。

而 media query 支持的查询条件（media conditions）包括width, height, aspect-ratio, color, orientaion 等，可以到[w3网站](http://dev.w3.org/csswg/mediaqueries/#mq-features)上查看.

有了media，很多时候你可以不用写两份 css 文件了。

## 单位

单位真是的一个很头疼的问题，有的人喜欢用px，有的人倾向于em, 有的人狂热 rem，还有的人使用%. 大家都知道他们有这明显的区别，可以具体是什么呢。

* px

px 是一个绝对单位，但是这里的绝对是相对于屏幕尺寸的绝对，所以不同屏幕尺寸下，设置成相同的px值，其实显示的大小也是不同的。 所以我们可以设置 meta 来保证px如我们想要的方式工作。

```
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,uer-scalabe=no"/>
```

* em

em 是一个纯粹的相对单位，font-size 设置为 em 单位的时候表示他相对于父元素的大小，所以当一个元素的font-size改变的时候，各个使用了em的子元素都会进行相应大小变化。

* rem

rem (root em) 是一个CSS3新增的一个相对单位，它和em的区别是，em相对的父元素的大小，而rem则是相对于根元素的大小。所以当根元素的大小发生变化的时候，所有使用rem的元素都会成比例的调整字体大小，避免了字体大小逐层复合的连锁效应。

然而使用哪一种单位并没有优劣之分，选择自己和团队认为合适的然后保持一致，我觉得就算是best practice了。

## 工具

在进行前端开发的过程中，最麻烦的就是多宗设备调试了，没有趁手的设备和工具，调试前端来真是让人发疯啊，所以这里推荐两个工具，硬件和软件各一个。

* browser-sync

这是一个多个设备的工具，使用起来超级简单，而且超级方便，是进行 Responsive 网页设计必不可少的工具。 可以两个命令就可以立即尝试：

```
$ npm install -g browser-sync
$ browser-sync start --server *.css
```

可以去他们官网了解更多： [browser-sync](https://www.browsersync.io/)
