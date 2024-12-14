---
title: 家庭网关折腾小记
date: 2023-02-06T05:35:07.322Z
---

关于路由器这块，其实我也没有深入研究过，各种专有词汇, 什么软软路由，旁路由等我也没有深究过，到底是什么意思，但是呢，我的初心很简单，那就是我要配置好了我的家庭路由，任何通过我的无线路由器进行网络连接的设备，都可以
直接享受自由网络，无需任何额外操作.

由于家里已经有了一个普通的路由器，而且之前因为可能是受到某些软文的推荐买了 GLiNet mt2500, 关于 GLiNet mt2500 的详细信息，你可以到官网查询，不过简单来说就是它是一款运行 OpenWrt 的迷你网关. 所以我们只需要在 GLiNet
mt2500 进行科学上网配置，然后将路由器连接到 GLiNet mt2500 即可, 家里的设备还是通过路由器就行网络连接.

```
以太网 ----> GLiNet mt2500 ----> 路由器
```

### 迷你网关和路由器连接

GLiNet 的官方文档上有非常详细的步骤介绍，如果想看视频，他们的 B 站上也有很详细的操作视频. 因为我的网络拓扑非常简单，所以操作起来也很容易，网线连接到 GLiNet mt2500 的 wan ，然后将 GLiNet mt2500 的 lan 和你的路由器
连接，然后设备通电即可，指示灯显示正常之后，你的电脑通过原来的路由器连接网络，然后在浏览器中输入 `http://192.168.8.1/` 进入 GLiNet mt2500 的管理面板. 

### OpenClash

当然上面的完成了之后只是说迷你网关工作了，要支持自由自在的网络体验，那么还需要一点额外的工作，这也是我们选购了支持
OpenWrt 的设备的原因，因为这样我们就可以在上面跑一些我们需要的网络软件了，而 [OpenClash](https://github.com/vernesong/OpenClash/wiki) 就是其中一种，当然你也可以选择别的比如 Shadowsocks.
在 GLiNet mt2500 运行的 OpenClash 有两种方式，一种在 GLiNet mt2500 的软件包里面去搜索安装，一种是直接下载社区编译好的固件，然后升级整个 GLiNet mt2500 的固件，我选择第二种，我选择是社区中编译好的[固件](https://forum.gl-inet.cn/forum.php?mod=viewthread&tid=967). 
升级好了之后就可以在[系统->高级设置]里面的 LuCI 页面进入到 OpenWrt 的管理页面，可以在 [Services] 中进入到 OpenClash 的管理和配置页面.
看着着实有些复杂，但是其实最重要的就一步, 那就是在 [Config Update] 中进行配置增加.

![](/assets/blog/homelab/openclash.png)

一切配置妥当之后，你可以在 [Overviews -> OPENPANNEL] 进入到 OpenClash 的监控面板.

![](/assets/blog/homelab/openclash_monitor.png)

### 结论

如果你已经订阅比较稳定的机场，那么在你的 OpenWrt 设备中使用 OpenClash 来进行家庭网络自由化建设还是一个很不错的方案.

### 参考

* [https://world.crisp.help/zh/article/openwrt-openclash-1s6td6i/](https://world.crisp.help/zh/article/openwrt-openclash-1s6td6i/)
* [https://github.com/vernesong/OpenClash/wiki](https://github.com/vernesong/OpenClash/wiki)
