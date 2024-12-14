---
title: Slack: 借IRC的刀，走在杀死Email的道路上
date: 2015-05-05T09:20:02.322Z
---

本着不是一篇安利文的原则写这篇文章.

Slack 作为一款从 side-project 成长而来的协作工具，以 email killer 的定位在 2014 年 2 月推出当月就有超过 8000  家公司注册，之后立即成为了独角兽俱乐部成员。第一使用了 Slack 之后，立即觉得这货似乎和 IRC 差不多啊，那到底是什么让 Slack 成为企业和团队协作的宠儿呢。

### Slack vs IRC vs Email

Slack 实际上原原本本的继承了 IRC 的概念，Slack 有着和 IRC 一样的 Channel， 可以自己创建Channel，也可以加入到某个 Channel 参与讨论。和 IRC 一样也可以和 Channel 中的人进行直接的对话。然而 IRC 依然还是只是 geek 界的工具，Slack 却成为了无数团队的协作工具。

Email 自 1970 诞生以来，已经成为了人们交流的最基本的手段之一，虽然现在有各种各样的及时通信工具，仍然没有谁能够取代 Email 在个人生活以及企业沟通所扮演的角色，Email 仍然是那些比较正式的信息传送的最主要载体。而 Email 的主要问题，很难找到一种完美的方式去判断信息的重要程度以及邮件数量中断导致的某些重要邮件被忽略的问题，于此同时，基于邮件的讨论效率极低。

Slack 实际上是借用了 IRC 的理念去解决传统 Email 无法解决的问题，与此同时，Slack 还在第三方服务即成方面做的非常优秀，真正做到了让团队的信息流统一在一个地方，防止团队沟通的碎片化。

Slack 的具体功能:

* Team

每一个使用 Slack 的团队都会有一个独立的 Team Domain， 比如 https://sswwee.skack.com, 所有的沟通和交流都会发生在这个 Team Domain 里. 每个人都可以创建和加入多个 Team.

* Channel

IRC 中的 Channel 更多是一个主题的沟通群。所以你会看到有很多类似 #RubyOnRails, #Django, #Dancer 这样的Channel。Slack 对于 Channel 没有做具体定义，你可以为一个组, 一个项目，一个讨论主题，一个事件处理流创建一个 open 的 Channel，然后整个团队可以参与讨论，文件分享等。

* Group

Channel 都是 open 的，也就说任何人都可以加入参与沟通。如果要进行一些小范围的沟通和讨论，则可以在 Private Group 中进行。在使用上，Group 和 Channel 唯一的区别是, Group 的讨论内容只有 Group 中的人才能访问，而 Channel 则是团队中所有人都可以参与。

* Notification

无论是QQ 还是 Wechat, 他们对于使用者而言都是强干扰的，比如某一个群，有了新消息就会弹出来，如果设置了免打扰，有非常有可能错过重要内容，而在 Slack 中你可以设置只有别人提到我或者我关注的关键词出现的时候才弹出提示。

* Integration

和第三方服务的集成绝对是令 Slack 成为炙手可热的工具的功臣之一，和 Twitter 等社交网络的集成使得团队所有团队成员第一时间获得客户对于团队的反馈，然后做出反应。和 Dropbox，Google Drive 的集成使得文件的分享变成异常的简单而且成为了信息流中的一部分. 和 Github, Bitbucket 等的集成让团队的协作开发更加方便，当然其他的集成，比如 Jenkins CI， New Relic 同样可以帮支持团队更好的做监控和支持。

* API

Slack 的开发 API 可以让你很轻易的继承自己服务，丰富的 Incoming Webhooks 和 Outgoing Webhooks 基本上可以让 Slack 成为团队的信息中心， 所以完成一个监控报警服务基于 Slack 绝对是分分钟的事情啊。当然 Web API 可以让用户去 build 自己的 App 和 Slack 交互，而 Real Time Messaging API 也是开放的.

* Search

每一个 Channel 或者 Group 的沟通 Session 都支持特别方便的 Seach。

* All Platforms

Web, Windows, Linux, Mac, iOS, Android, Windows Phone,支持所有平台，而且同步功能完美。

### 可是

对于国内的企业或者团队对使用 Slack 来说有这样一些顾虑: Slack 是否也会某天就被墙呢，而且目前 Slack 现成的第三方集成大部分都是国外的产品，速度是否稳定呢。 这确实是一个比较关心的问题，可是如果作为技术团队的话，网络访问不应该是问题，对吧。

### 最后

如果对于希望使用 Slack 之类的服务，但是又不想依然于 Slack 平台，那么可以考虑使用 Open Source 的方案 [lets-chat](https://github.com/sdelements/lets-chat).

最后推荐一个视频，基本可以大致了解 Slack 的日常使用了。

<embed src="http://player.youku.com/player.php/sid/XOTQ5NTQ5NTYw/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash">
