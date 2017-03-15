title: Digital Ocean + Shadowsocks + ipv6，免流量科学上网
date: 2015-01-01 12:30:00
categories: 折腾
tags: [Shadowsocks,VPS,DigitalOcean] 
description: 写个小白教程骗骗粉。
---

校园网收费，真的断我等屌丝的活路，不过好在ipv6足够坚挺，Github的教育网大礼包足够屌，那么这里就简单介绍个利用[Shadowsocks][3]和[Digital Ocean][4]自建SS服务器免流量上(fan)网(qiang)的方法。

<!--more-->

# 准备工作

## 准备无线ipv6环境

OK，既然是打算免流量，那么当然包括我所有的设备都必须要免流量，对吧？有线连接的设备和无线连接的设备都得保证能够免流量，那么原有的路由器工作模式就不适用了，这里我们可以将无线路由器改成交换机模式，让无线设备也可以使用ipv6，具体步骤如下：

1.进入路由器的管理页面，关闭DHCP功能。

2.将入口网线从WAN端口转移到交换机端口上。

3.重启一下路由器，这个时候也是用原有的用户名密码登陆路由器，只不过这个时候就相当于连接到了一台交换机上，而且你也没办法再通过192.168.1.1进入管理界面了。但是这个时候你会发现可以直接连上BT了，这就说明ipv6在无线网络环境中启用了。

## 准备VPS

VPS有很多家了……Linode、ramnode、DigitalOcean之类的，很多，blabla，这里只需要保证VPS有ipv6链接即可，并且最好是服务器架设在美国，因为某些众所周知的原因，出口的ipv6流量现在基本都会从那边绕回来，所以就不要选台湾香港日本的服务器了~

这里推荐一下前文提到的也是现在在用的Digital Ocean，做的真的很不错，而且用github的教育礼包真的优惠力度十足，特别适合我等穷学生，而且就算花钱，DO的VPS也是性价比相当高。

如何申请VPS这里略过，但是有一点需要注意，最好有一张Visa的信用卡，不然可能会比较麻烦，在选取VPS之后有的时候需要手动开启ipv6功能~

# 搭建Shadowsocks

>Shadowsocks的作者[Clowwindy][2]于2015年8月20日成功被TG约谈，因此Shadowsocks版本最终锁定在了2.8.2。不知道将来Shadowsocks会不会由后来人继续更新，无论如何，向Clowwindy致敬，向开源致敬，向自由致敬。

本文最早介绍安装Shadowsocks的时候还很复杂，随着作者不断地迭代更新，也是在不断地将安装简化，在一些特定的系统中，我们只需要很简单的几个步骤就可以完成Shadowsocks服务器的搭建和优化，向这些充满理想的程序员致敬。

官方推荐 Ubuntu 14.04 LTS 作为服务器以便使用 TCP Fast Open。服务器端的安装非常简单。

*Debian / Ubuntu:*

```bash
apt-get install python-pip
pip install shadowsocks
```

*CentOS*

```bash
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

运行的时候也很简单，首先建议建立一个独立的文件夹存放`config.json`文件：

```bash
{
    "server":"::",//双栈
    "server_port":7777,
    "local_address": "127.0.0.1",//设置该参数和下面的参数将指定客户端的本地代理地址和端口
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"rc4-md5"//该加密方法安全性、性能较为均衡
}
```

然后建立两个sh文件作为启动脚本：

*Start*

```bash
ssserver -c path-to-config.json --fast-open -d start #以config.json为配置文件、开启fast-open、静默后台启动
```

*Stop*

```bash
ssserver -d stop #停止服务
```

至此Shadowsocks服务就已经跑起来了，是不是很方便？

## 优化性能

网上关于性能调优的文章蛮多，但是经测试都没什么屌用（汗），这里主要推荐使用[锐速][5]，效果拔群。

锐速是一款非常不错的TCP底层加速软件，可以非常方便快速地完成服务器网络的优化，配合 Shadowsocks效果奇佳。目前锐速官方也出了永久免费版本，适用带宽20M、3000加速连接，个人使用是足够了。

使用时，首先需要在锐速官网注册账户，然后确定自己的内核是否在锐速的支持列表里，如果不在，请先更换内核，如果确定自己的内核版本在支持列表里，就可以使用以下命令快速安装了：

```bash
wget http://my.serverspeeder.com/d/ls/serverSpeederInstaller.tar.gz
tar xzvf serverSpeederInstaller.tar.gz
bash serverSpeederInstaller.sh
```

输入在官网注册的账号密码进行安装，参数设置直接回车默认即可，到这里还没结束，我们还要修改锐速的3个参数：

```bash
vi /serverspeeder/etc/config
```

修改以下几个参数：

```bash
rsc="1" #RSC网卡驱动模式  
advinacc="1" #流量方向加速  
maxmode="1" #最大传输模式

#DO的网卡支持rsc和gso高级算法:
rsc="1"
gso="1"
```

然后重启下就好啦：

```bash
service serverSpeeder restart
```

# 使用Shadowsocks

使用的话，就是各个平台上的软件啦，这方面的事情不需要多说，自己翻Github，小白的一逼。

## 自定义代理域名

ipv6好是好，但是地址确实太长了，别说告诉别人了，你自己都记不住，那可如何是好？ok，你需要一个你自己的域名。

买域名的话，狗爹godaddy.com和name.com都是很好的选择，但是有一点，他们都是美元支付，而且都需要绑定信用卡或者paypal，如果没有的话会稍微蛋疼一点，但也不至于没办法，这个就靠大家的智慧啦~

买到域名之后，DO的做法是需要将域名的nameserver替换为DO自己的nameserver作为域名解析，估计别的vps站也是大同小异，就不具体叙述啦。我定义了两个服务器，分别接受ipv4访问和ipv6访问。

	ss4.kyangcis.me
	ss6.kyangcis.me
	
也有人说可以怎么怎么样一个网址双栈访问，的确是可以，但是为了在学校的时候不误走v4流量，我还是分开来了。我现在的服务器虽然只有我和我的几个朋友在用，但是为了安全就不对外开放啦。

## 使用情况

就上张Speedtest的图好了：

![speedtest][6]

实际使用中，一般的国外网址打开都没什么压力，有的国内的网站可能是因为DNS的原因，第一次打开还是会有点慢，不过问题不大。

[2]:https://twitter.com/clowwindy
[3]:https://github.com/shadowsocks/shadowsocks
[4]:https://www.digitalocean.com/?refcode=b3afbdd6f9b9
[5]:http://www.serverspeeder.com/
[6]:http://ww2.sinaimg.cn/mw690/825558b1gw1evdzgnw49sj214c04otak.jpg