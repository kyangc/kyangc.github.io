title: 老夫终于吃上棒棒糖了哼
date: 2015-01-10 12:30:00
categories: 折腾
tags: [Android,折腾]
description: 三流安卓开发者走在装逼道路上的的心路历程
---

>三流安卓开发者走在装逼道路上的的心路历程。

<!--more-->

觊觎萝莉炮很久了，新Material Design、新的交互动画、新的编译系统，对于用户或者是像我这种三流安卓开发狗而言，真的是吸引力满载！奈何万普拉斯公司实在是效率低下，很长一段时间都只能靠Android 4.4 + Google Now启动器勉力支撑着自己的装逼大业……只能徒有其形的苦楚真的是无处言表……

就在昨天，随手翻了一下万普拉斯论坛，发现CM12竟然已经有Nightly版了，而且最新的一版已经支持OTA更新，真是天空飘来五个字，哈哈哈哈哈，于是趁着周末，一觉睡到11点，起床刷机去~

虽然万普拉斯公司效率奇低，但是不得不说，毕竟销量大，机型单一，开发者还是不少的，必须为万能工具包此等神器怒点一赞，有了它真的省了不少事，忍痛走流量下载了刷机包和工具包还有精简下来的Google服务包，安装，刷机，一气呵成，再点一赞~

只是把鸡鸡刷好只是折腾的开始，开启google now服务才是成功的全部，原本的手机是已经搞定了的，新刷系统之后system分区被格式化，因此需要再一次激活一下google now。

网上有很多所谓的激活教程，其实都没个屌用，下面这个经亲测绝逼好使，为了以后不再在我那天量收藏夹里寻找这个，这里记录如下：

*	保证科学上网环境；英文语言环境；Root环境；GMS环境（暂时不要登录）。
*	依次安装ES文件管理器、BusyBox、Init.d Toggler、Shell。
*	下载模拟Sim文件：[点我下载][1]。
*	打开BusyBox，点击Install，安装BusyBox。
*	打开Init.d Toggler，点击enable Init.d，注意一下提示看是否成功，未成功需要回到上一步重新来一次。
*	用获取Root权限的ES文件管理器，将下载好的Sim文件放到/sysem/etc/init.d目录下，更改Sim文件的权限为755。
*	重启手机，运行Shell，执行getprop命令，检查gsm.sim.operator.iso-country是cn还是us。如果是us则可以打开位置报告。
*	转到google Setting处，在科学上网环境下登陆即可。如果先前已登录，那么请先退出账号，对google服务进行双清，然后再登陆。此时已经可以打开位置报告和Google Now了。

本方法的原理是，利用init.d在开机的时候执行更改Sim卡类型的脚本，将Sim卡类型更改为verison，位置设定在美国，这样就可以绕过中国地Sim卡无法开启位置服务了。

![Hello Lolipop!][2]






[1]:https://www.dropbox.com/s/6e7wvd4us9au5ox/sim?dl=0
[2]:http://ww1.sinaimg.cn/mw690/825558b1gw1eo4f8s5fr8j20j80y6qe0.jpg