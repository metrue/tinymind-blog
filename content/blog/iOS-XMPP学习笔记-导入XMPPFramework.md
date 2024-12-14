---
title: XMPP学习笔记:导入XMPPFramework
date: 2015-04-09T09:20:02.322Z
---

XMPP（Extensible Messaging and Presence Protocol，前称Jabber[1]）是一种以XML为基础的开放式实时通信协议，是经由互联网工程工作小组（IETF）通过的互联网标准。XMPP因为被Google Talk应用而被广大网民所接触, 现在Gtalk已经变成了Hangout，不再支持 XMPP，不过XMPP协议自由、开放、公开的特点仍然有很多的项目在使用它。

[XMPPFramework](https://github.com/robbiehanson/XMPPFramework) 是Objective C实现的XMPP框架，为iOS/Mac中的IM工具开发听过了非常大的便利. 本文将介绍如何将XMPPFramework导入Xcode项目中.

### 开始吧

####. 建立一个Single-PageView的项目
  不再多说

####. 从GitHub上clone XMPPFramework

```
  git clone https://github.com/robbiehanson/XMPPFramework
```

#### 导入XMPPFramwork.

* CocoaLumberjack: 日志框架

* CocoaAsyncSocket: 底层网络框架

```
  依赖 CFNetwork, Security
```

* KissXML

```
  libxml2.dylib
```

于此同时，再Project level 编辑添加

```
  OTHER_LDFAGES = -lxml2
  HEADER_SEARCH_PATHS = /usr/include/libxml2
```

* libidn

* 导入 XMPPFramwork 核心模块

```
  Authentication, Categories, Core, Utilities
```

以及其拓展

```
  Extensions
```
他们需要依赖

```
  libresolv.dylib
```

每一步添加完了之后都要 command + B 来编译一下确认结果, 最后添加 Extensions 的时候还出现 UIImage expect a type 的错误，解决方法是在 XMPP.h 中加入

```
#import <UIKit/UIKit.h>
```
