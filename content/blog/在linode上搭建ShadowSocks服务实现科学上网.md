---
title: 在linode上搭建ShadowSocks服务实现科学上网
date: 2015-08-01T09:20:02.322Z
---


越强大，越开放，越落后，越封闭。然后恶性循环。

生活在我大党国的程序员真是苦经磨难，每年花了钱买了一个vpn之后，服务商又被搞了，好吧，好在我一直都有备用的云主机，以前用的亚马逊的 EC2, 最近改用了 Linode 的 VPS，速度挺好的，那就自己建立一个 ShadowSocks 来科学上网吧。没有 Google 对于一个程序员来说，或者说对于大多人来说，真是不知如何是好啊。

#### 服务器端

* 安装 nodejs

```
  sudo apt-get install nodejs
```

* 建立 ShadowSocks server

```
 git clone git://github.com/clowwindy/shadowsocks-nodejs.git
 cd shadowsocks-nodejs
```

配置文件，替换成你的 Server IP 和你想要的密码.
```
 vim config.json
     {
         "server":"200.201.202.203",
         "server_port":8899,
         "local_port":4131,
         "password":"password",
         "timeout":300,
         "method":"aes-256-cfb"
     }
```

然后运行 ShadowSocks 服务

```
 nohup /usr/local/node/bin/node server.js > log &
```

### 本地客户端

在 OSX 平台上，可以下载 ShadowSocks 客户端

```
  https://github.com/shadowsocks/shadowsocks-iOS
```

安装之后启动，输入你的server信息即可，需要注意的是你的加密方式的选择要和你配置在 ShadowSocks Server 上一致，比如本文中我们选择的是 aes-256-cfb. 默认客户端的模式的自动代理，你也可以手动选择全局代理。然后你就可以畅游天下了.
