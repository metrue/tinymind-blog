---
title: 苹果 APP 很安全? XCodeGhost事件始末
date: 2015-09-19T09:20:02.322Z
---

这两天，整个iOS开发者圈子都被 'XCodeGhost' 事件刷屏了，各大安全团队也都对事件，从各种角度进行分析，给我们安全外行人士还原了整个事情的前因后果，甚至已经人肉到了 'XCodeGhost' 的作者信息，苹果也下架不少的受到 'XCodeGhost' 感染的 APP. 本文我们就来梳理一下这件事情的来龙去脉吧，最后作为开发者或者普通的iOS用户，我们应该如何应对吧。

### XCodeGhost 事件

XCode 作为苹果 APP 的标准开发工具, 几乎每一位开发者或者团队都使用 XCode 去开发 APP，XCode 通过编译，链接，打包将开发者的代码变成一个个运行在我们手机中的APP, 那么如果开发者所使用的 XCode 不是苹果官方提供的版本，而是随意在某些网盘随意下载的 XCode 的话，就很难保证开发的 APP 除了正常的代码之外是否被恶意的 XCode 加入非正常的编译选项，然后植入恶意代码模块，后果将不堪设想。

XCodeGhost 事件就是这么一个事件，由于网络原因，国内不少开发者没有下载使用苹果官方提供的 XCode 开发工具，而是图下载速度快，在某网盘，或者迅雷上下载 XCode, 更加让人不可思议的事，XCodeChost 作者似乎很懂 SEO，通过百度搜索 XCode 出来的页面，除了指向苹果 AppStore 的那几个链接之外，都是指向各种不同 ID 的百度网盘上，这些网盘的地址在各个开发社区，人气社区，下载站点上帖子均有存在，而根据腾讯安全应急响应中心的报告，从这些网盘上下载的 XCode, 在路径 Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/Library/Frameworks/CoreServices.framework/下，存在一个名为CoreServices.framework的 "系统组件" , 而从苹果官方下载的xcode安装包，却不存在这些目录和系统组件。也就是说，从这些网盘上下载的 XCode 安装包中，被别有用心的人植入了远程控制模块，通过修改 XCode 的编译参数，将这个恶意模块自动的部署到任何通过这个 XCode 编译的苹果 APP 中。

### XCodeGhost 的危害

被植入这些恶意模块的 APP 将成为了黑客控制用户的工具，在 APP 受到污染之后.

* 启动、后台、恢复、结束时上报信息至黑客控制的服务器, 上报的信息包括：APP版本、APP名称、本地语言、iOS版本、设备类型、国家码等设备信息，能精准的区分每一台iOS设备。上报的域名是icloud-analysis.com，同时腾讯安全应急中心还发现了攻击者的其他三个尚未使用的域名。

* 通过 openURL 这个API, 黑客可以下发伪协议命令在受感染的iPhone中执行, 黑客能够通过上报的信息区分每一台iOS设备，然后如同已经上线的肉鸡一般，随时、随地、给任何人下发伪协议指令，不仅能够在受感染的iPhone中完成打开网页、发短信、打电话等常规手机行为，甚至还可以操作具备伪协议能力的大量第三方APP。实际上，iPhone上的APP如果被感染，完全可以理解为黑客已经基本控制了你的手机！

* 黑客可以在受感染的 iPhone 中弹出内容由服务器控制的对话框窗口, 和远程执行指令类似，黑客也可以远程控制弹出任何对话框窗口。至于用途，将机器硬件数据上报、远程执行伪协议命令、远程弹窗这几个关键词连起来，反正我们是能够通过这几个功能，用一点点社工和诱导的方式，在受感染的iPhone中安装企业证书APP。

* 远程控制模块协议存在漏洞，可被中间人攻击, 在进行样本分析的同时，腾讯安全应急中心还发现这个恶意模块的网络协议加密存问题，可以被轻易暴力破解。我们尝试了中间人攻击，验证确实可以代替服务器下发伪协议指令到手机，成为这些肉鸡的新主人。

植入的远程控制模块并不只一个版本。而现已公开的分析中，都未指出模块具备远程控制能力和自定义弹窗能力，而远程控制模块本身还存在漏洞可被中间人攻击，组合利用的威力可想而知。这个事件的危害其实被大大的低估了。

### 有哪些已知的 APP 受到污染呢

目前发现的受到污染的 APP 名单如下。

* 网易云音乐
* 滴滴出行
* 12306
* 中国联通手机营业厅
* 高德地图
* 简书
* 豌豆荚的开眼
* 网易公开课
* 下厨房
* 51 卡保险箱
* 同花顺
* 中信银行动卡空间
* 微信
* 口袋记账
* 滴滴打车
* 喜马拉雅
* 夫妻床头话
* 我叫 MT

### 我们该怎么办呢

作为开发者，当然是确定自己 XCode 没有包含上述所说的模块，本人使用的任何软件基本是都是在官方网站上下载的，基本上没有什么问题。

作为普通的 iOS 用户，应该第一时间更改自己的 APP ID 的密码，如果担心自己在 iOS 设备输入的其他的密码有被盗的可能，那么也十分有必要进行修改.

### 最后

XCodeGhost 作者 @XcodeGhostSource 最后还将代码开发到 GitHub 上面: https://github.com/XcodeGhostSource/XcodeGhost, 并且声明说所谓的XcodeGhost实际是苦逼iOS开发者的一次意外发现：修改Xcode编译配置文本可以加载指定的代码文件，于是我写下上述附件中的代码去尝试，并上传到自己的网盘中... , 不知道各位客官怎么看，反正我是不相信啊，正如 @onevcat 所说的: 算个账，微信用户总数5亿日活70%。每天每人就算5个POST请求，每个请求300Byte，日流入流量就接近500G，以及17.5亿次请求。据说服务器扔在亚马逊，那么资费算一下每个月应该是存储$450，请求$260K。这还只是单单一个微信，再算上网易云音乐等等，每月四五十万刀仅仅是苦逼iOS开发者的个人实验？
