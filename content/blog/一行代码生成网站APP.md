---
title: 2017 年终总结
date: 2016-01-03T09:20:02.322Z
---


在很久很久，我们开发一个互联网产品，我们只需要完成一个桌面 PC 版的网站，后来移动设备开始出现，很多闲有余力的人开始开发
wap 站点，再后来，移动端的发展越来越快，流量来源的比例越占越高，而
Android 和 iOS 在移动领域的战争中成为目前的赢家，而 PC
端的比重仍然不可小觑，所以很多的互联网产品不仅仅需要一个适应于 PC
的站点，而且还配套有相应的 Android 和 iOS App,
于此同时还维护着适用于小屏幕浏览器访问的移动版网站。地球上所有的开发者都厌倦了一套业务却要维护多个平台的代码这种事情，所以出现了HTML
5, HTML5 + Native (Hybrid: PhoneGap)，以及今年特别火的 React
Native，React 的野心很大，我这几天一直在玩，觉得 React
确实配得上它的野心，然而目前来看，没有一种方案可以统一整个前端
(泛指: 移动应用，网站前端) 的开发。

## 思考

其实现在大多数网站都很不错的自适应 (Responsive)，
无论是在桌面访问还是移动设备访问都有很不错的用户体验，但是他们依然需要一个移动
App，因为他们想让自己的服务常住到用户的屏幕上(然而我们都知道，其实除了首屏有意义之外，没有任何实质上的作用)，
但是他们没有 Android 的开发人员，也没有 iOS
的开发人员，或者虽然有开发者，但是却没有精力去分散在那么多的平台上。

他们仅仅是想要用户可以快速的点击即可使用自己的服务而已，难道真的需要从头开始开发
iOS 和 Android 的 APP
吗，更何况他们的网站在移动设备上也有良好的用户体验。也许我们不需要那么麻烦，因为无论是
iOS 还是 Android 都有一个东西, 那就是 WebView,
我们何不把我们的网站放倒 WebView 里面去呢？

是的，我们可以那么做，剩下我们事情就是，找出那些需要 Native API
支持的功能，使用 Native API 的实现即可，咿，不对，这不又回到了
Hibrid 的模式了，我们不想去写任何的 Objective C/Swift 或者 Java
代码，那么到底可不可以呢?


## 实验

我们都知道，在 iOS 开发中，Objective C 可以通过以下方法来运行
JavaScript 代码.

```
[jsContext evaluateScript:@"javaScriptCodeString"]

or

[webView stringByEvaluatingJavaScriptFromString:@"javaScriptCodeString"];
```

虽然 JavaScript 不能直接调用Objective C
代码，但是我们却是可以通过以下方法来 hack 以下。

* 首先试图去访问一个 fake url

```
window.location.href = "MYAPP://something/something/..."
```

* 然后在 Objective C 中，通过判断 url 形式来判断需要做出什么动作.

```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
  NSString *urlString = request.URL.absoluteString;
  if (urlString 'match something') {
    /*
      do something;
    */
    return YES;
  } else {
    /*
     do something;
    */
    return NO;
  }
}

```

有了这两个东西，我们就可以往我们的网页中 inject 一些 hook
了，然后来实现一下 Web API 到 Native API 的连接了,
那么我们就有可能实现一个快速将我们网站变成一个 iOS 的工具了。


<strong> Take is cheap, show me the code </strong>

```
 URL -> iOS APP
```

这两天，我实现了一个叫做 Applize 的项目，这是一个将网站变成 iOS APP
的一个工具项目，只需要两步:

* 输入你的网站 URL

* 编译项目

你就可以得到一个属于你网站的 iOS APP
喽，当然，这只是一个简单的原型，不过，输入你的个人网站 URL，试试吧。

查阅代码，你可以了解更多哦。

[https://github.com/metrue/Applize](https://github.com/metrue/Applize)

