---
title: yiqiwan.us的诞生记
date: 2015-03-09T09:20:02.322Z
---

yiqiwan.us 是一个很简单的项目，不过从中我学习了:

```
其实不需要任何条件都具备了才开始行动，行动了之后你才真正明白了你需要哪些条件。
```

```
任何工程上面的事情，只有去量化结果，你才知道是否有改进。
```

```
分享让你快乐，分享让你进步。
```

### 起始

v2ex.com 是一个群体讨论质量不错的社区，相对于 Ruby China 来说，这里的讨论更加的多元化一些，这也就是我最近为什么都是
优先在 v2ex.com 发帖的原因，这里有更多的交流和互动。而 yiqiwan.us 的但是就是来源于此的。

[@jianzong](http://www.v2ex.com/member/jianzong) 在 v2ex.com 发帖["点评一个人或者 2-3 人小团队的工作、学习场所"](http://www.v2ex.com/t/172056).
我觉得挺好的想法，特别是对于程序员来说，业余时间在家写代码的效率确实不是很高，有时候会想着出去咖啡店或者图书馆写代码。
但是如何找到一个合适的咖啡店和图书馆，其实不是很容易，有的太吵，有的太贵，有的网络速度不好，有的人多拥挤。所以我就想说，要不咱们就在 Github 上维护
一个 list 吧，专门收藏程序员们自己喜欢的写代码场所，大家一起来添加。

### 诞生

然后一个关于适合程序员写代码的场所收集的表单的 GitHub 仓库就建好了，名约["nice-place-for-coding"](https://github.com/metrue/nice-place-for-coding),
然后不断的有同学 fork 然后添加地点，也有不少的同学只是 star 和收藏，后来有同学发了一个 issue 说，"Don't limit yourself as a coder", 具体信息是说
我们虽然是程序员，但是不要什么事情都 bind 到 Github 上面，还有其他方法来完成这件事情，比如 "街旁", 当时我还觉得 Github 很方便啊，大家添加的时候发一个
PR 就好了。直到后来，我发现为这个 list 添加信息的人越来越少了，我才知道，虽然这是很简单的东西，但是它也是一个产品，作为一个产品，如果不能让用户很简单的使用的话，那么不会有用户喜欢它，更何况这个产品还是需要用户去做贡献的，所以为了让用户可以为这个 list 添加更多的地点，我需要准备一个简单易用的添加页面，
同时为了让用户可以很快看到自己所处的城市有哪些适合写代码的地方，我需要准备一个按城市区分的地点列表页面。所以

### 发展

所以我就开始为这个项目写一个简单的网站。通过简单的分析，我就确定了用 AngularJS + LeanClound 的方式来完成这个小项目，因为: 1. 这是一个简单的单页面应用，
AngularJS 非常适合，其次这是应用的数据非常简单，而且非常结构化，那么没有必要去自己建立一个后代服务器，直接使用 LeanCloud 的存储服务就好了。所以回到上海之后，
快速的用了大约一下午的时间，将网站大体搭建完毕，又花了一晚上的时候完成了前端样式的调整，最终它变成了这个样子 [http://yiqiwan.us](http://yiqiwan.us) .

### 后续

然后我将网站的建设情况发到 v2ex.com，事实证明，简单的使用确实让用户乐于去分享，我们现在有了 上海，杭州，深圳，舟山等城市的适合写代码地点的收集，后续希望有
更多的不同城市的程序员添加他们自己喜欢的地方，为更多的程序员提供便利。

### 最后

我的代码开源在 GitHub 上，欢迎发PR，让这个网站变成你想要的样子。

[https://github.com/metrue/nice-place-for-coding](https://github.com/metrue/nice-place-for-coding)


于此同时可以剧透的是，有同学正在为这个项目写 iOS 的 App， 而另外一个同学正在为其写一个交互式地图的拓展，方便大家对于地点的查询。
也许这就是社区的力量吧，有一句说的很好:

```
if you want to go fast, go alone
if you want to go long, go together
```

