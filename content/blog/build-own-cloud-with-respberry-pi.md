---
title: 通过 Cludflare 和 Raspberry Pi 构建一个简单的云
date: 2023-01-04
---

家里的两块 Respberry Pi 已经机积灰了好几年了，最近正好是圣诞休假，是时候构建自己的小机房了.

### 目标

* 将 blog.minghe.me 运行在 Respberry Pi 上, 并且公网可以访问.
* 通过 GitHub Actions 来进行 CI/CD

### 准备工作

* 一个 Cloudflare 账号
* 一个 代理在 Cloudflare 的域名
* 一块 Respberry Pi 4
* 一块 SD 卡

### 具体步骤

##### 系统安装

我选择是官方的 [Raspberry Pi Imager](https://www.raspberrypi.com/news/raspberry-pi-imager-imaging-utility/) 来进行系统的安装，你选择自己喜欢的系统进行安装， 然后选择自己写入的磁盘，也就是你的 SD 卡. 最后配点击设置去设置 WiFi, SSH 登陆用户.

* 配置 WiFi, 准备号你的WiFi名称和密码，按提示输入即可.
* 配置 SSH, 选择用户名和密码登陆. (比如, 用户名pi, 密码xxxx)

#### 启动并登陆 Respberry Pi

将 SD 卡插入到你的 Raspberry
Pi，然后通电，等几秒钟钟待其启动完毕，然后在浏览器中输入你的路由器地址(一般是:
http://192.168.0.1), 然后查看你的 Raspberry Pi 的 IP  地址.

```
ssh pi@<IP>
```

然后按提示输入你的密码，完成了启动并登陆.

#### 创建 SSH tunnel

因为我们需要可以通过公网访问到我们运行在 Pi 上的博客服务, 而我们又没有公网
IP，所以我们需要借助 Cloudflare 的 tunnel 来进行访问.

* 安装必要的软件

登陆到你的 Respberry Pi, 然后进行更新和安装必要的软件.

```
ssh pi@<IP>
sudo apt update
sudo apt upgrade
sudo apt install curl lsb-release
```

为了安装 cloudflared, 我们需要增加 cloudflared 的软件源.

```
curl -L https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-archive-keyring.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/cloudflare-archive-keyring.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee  /etc/apt/sources.list.d/cloudflared.list
```

然后在进行一次更新软件源信息，

```
sudo apt update
```

最后安装一下 cloudflared
```
sudo apt install cloudflared
cloudflared -v
```


* 创建和配置 tunnel

登陆 cloudflared 授权 tunnel，

```
cloudflared tunnel login
```

这一步会产生一个 URL , 你应该复制 URL 到浏览器中，然后进行授权操作，在完成了授权之后，相应的密钥会保持到你的 `~/.cloudflared/` 目录下,

```
/home/pi/.cloudflared/cert.pem
```

现在你就可以创建一个 tunnel 了.

```
cloudflared tunnel create <TUNNELNAME>
```

创建好了 tunnel 之后，你的 `~/.cloudflared/` 目录就有多了 `xxxx-xxxx-xxx-xxx.json` 这样的文件, 里面其实就是这个 tunnel 的 secrets
信息. 有了 tunnel 之后，我们来将域名路由到这个 tunnel 吧.

```
cloudflared tunnel route dns <TUNNELNAME> <DOMAINNAME>
```

然后让我们来进行 ssh tunnel 的配置吧.

```
vim ~/.cloudflared/config.yml
```

根据你的信息输入即可. 详细配置方式可以参考[这里](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/ingress/#requirements)

```
tunnel: <tunnel_id>
credentials-file: <xxx.json>

ingress:
  - hostname: blog.minghe.me
    service: http://localhost:5000 # 假设我们即将把 blog 服务在 5000 端口.
  - hostname: <hostname> # eg. pi.minghe.me
    service: ssh://localhost:22
  - service: http_status:404
```

* 使用 systemctl 来管理 cloudflared

将 cloudflared 设置成systemctl服务,

```
sudo cloudflared --config ~/.cloudflared/config.yml service install
sudo systemctl enable cloudflared
```

启动 cloudflared 服务

```
sudo systemctl start cloudflared
```

你可以到 cloudflare 的 [tunnel 页面](https://one.dash.cloudflare.com/b86d6c03c736d44c2f76489f9c750ebf/access/tunnels)查看你的tunnel状态. 如果你看到 `Healthy` 就说明所有的设置妥当了. 那么就尝试一下通过域名来进行 SSH
登陆吧.

我们需要在我们的本地电脑也进行一点配置才能通过 tunnel 进行 SSH 登陆. 首先还是需要先在本地也安装一下 `cloudflared`

```
brew install cloudflared
```

然后配置一下 `~/.ssh/config`, 增加如下配置.

```
Host <domain>
  ProxyCommand /opt/homebrew/bin/cloudflared access ssh --hostname %h
```

配置妥当之后，就可以通过域名来进行一个 SSH 登陆了.

```
ssh pi@pi.minghe.me

Linux pi0 5.15.76-v8+ #1597 SMP PREEMPT Fri Nov 4 12:16:41 GMT 2022 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Jan  3 12:15:51 2023 from ::1
```

#### 安装 Docker

是否需要使用 Docker 可以根据自己的需要来决定，我自己比较喜欢用 Docker
来部署服务，当然在 Pi 上安装 Docker 也很简单.

```sh
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo bash get-docker.sh
$ sudo usermod -aG docker $(whoami)
$ sudo apt install python3-pip -y   
$ sudo pip3 install docker-compose
$ sudo reboot # to make it effective
```

#### 运行博客服务

我这里使用的是 [next.js](https://github.com/vercel/next.js/tree/canary/examples/with-docker) 来构建自己的博客.

```sh
$ docker build -t blog .
$ docker run -p 5000:5000 blog
```

#### 通过 tunnel 访问博客

增加 ingress rules 来支持博客的公网访问, 在你的 `/etc/cloudflared/config.yml`
文件里面增加访问博客的规则.

```
    - hostname: blog.minghe.me
      service: http://localhost:5000
```

然后让这样就可以尝试来重启 `cloudflared` 服务，

```sh
$ systemctl restart cloudflared
```

然后就可以通过 [https://blog.minghe.me](https://blog.minghe.me) 来访问我们的博客了.

### CI/CD on GitHub Actions 

用通过 Github Actions 来部署博客，本质上就是如何在 Github Action
里面远程运行部署命令. 我在前面的步骤已经配置好了通过 Cludflare Tunnel 来 ssh
登录到 Pi 了，那么我们只需要在 Github Action 里面也通过 ssh
来远程运行部署脚本即可.

```yml
name: blog-deployment

on:
  push:
    branches:
      - master

jobs:
  deployment:
    runs-on: ubuntu-latest
    steps:
      - name: check out
        uses: actions/checkout@master
      - name: install cloudflared
        run: |
          curl -L https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-archive-keyring.gpg >/dev/null
          echo "deb [signed-by=/usr/share/keyrings/cloudflare-archive-keyring.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee  /etc/apt/sources.list.d/cloudflared.list
          sudo apt update
          sudo apt-get install cloudflared
          cloudflared --version
      - name: install-ssh-key
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          name: id_rsa
          known_hosts: ${{ secrets.KNOWN_HOSTS }}
          if_key_exists: fail
      - name: deploy blog
        run:
          ssh -o ProxyCommand="cloudflared access ssh --hostname pi1.minghe.me" pi@pi1.minghe.me < ./scripts/deploy_blog.sh
```

可以看到上面的配置用到了 ssh 私钥，这是为了在 Github Actions
里面进行无密码登录，然后也可以看到我们使用 `-o ProxyCommand` 来使得 ssh 通过
cloudflared 来走 tunnel 访问 Pi, 然后运行我们的部署脚本.

### 最后

至此，积灰许久的 Respberry Pi 终于有点发挥价值了.
