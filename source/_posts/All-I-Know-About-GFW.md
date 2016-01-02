title: GFW 封锁方式以及翻墙手段汇总
date: 2015-12-23 17:19:29
tags:
---

## GFW 的封锁方法
1. 国内 DNS 服务器的缓存污染
2. 发往国外的 DNS 解析请求的拦截或篡改
3. IP 黑名单
4. 端口封锁，针对 OpenVPN，SSH，shadowsocks 等，有用到 DPI，多次换端口后封 IP
5. 关键字封锁，利用深度包检测（DPI），是主要方法。可用全站 https 应对，不过 GFW 因为无法识别 https，所以会针对所有 https 连接进行随机的中断

<!-- more -->

关于 GFW 的更多技术细节可以参见本文的参考链接

## GFW 的设备
大量硬件设备来自 Cisco
1. 入侵检测设备，在北京、上海、广州搭在总交换中心上做旁路监听
2. 动态路由设备，放在 ISP 处

## 翻墙方式
1. hosts，只能翻部分被 DNS 投毒的网站，而且随着 Google IP 被封禁得越来越多，已经很难翻了
2. 第三方 DNS，作用同 hosts，有风险，可能被再劫持
3. HTTP proxy，主要风险是明文传输（试过在海外 VPS 直接搭 HTTP 代理，用来上百度没问题，一打开 Google 马上被封）
4. Tor，P2P 方式，安全性高。但 GFW 会钓鱼，伪造成 Tor 客户端进入 Tor 网络（[obfsproxy](https://www.torproject.org/projects/obfsproxy.html.en) 可以应对）。本身网络传输速度不快，不好用
5. Latern 基本同上，就个人使用体验来说，速度太慢
6. GoAgent，基于 GAE，不过 Google IP 被封得差不多了，所以基本不可用
7. [GoProxy](https://github.com/phuslu/goproxy)，GoAgent 的继任者，用于自己部署在 VPS 上
8. SSH，虽然传输安全，但握手阶段特征太明显，会被监控流量和连接数，所以基本只能用一小会儿，一般需要数小时重连一次。2012 年 GFW 加入 DPI 功能之后被封锁得更为严重了，一旦有 HTTP 流量传输就会被墙
9. VPN，工作在数据链路层，流量特征非常明显，出于商业上的考虑（大量在华跨国公司需要用到）所以才还能存活。但是自建的话，L2TP/PP2P/OpenVPN 基本没办法存活多久，只有 Cisco AnyConnect （服务端用开源的 ocserv）还可以用，但是速度有 2M/s 的上限
10. Shadowsocks，这个名气够大了，不详述
11. ShadowVPN, [GoHop](https://github.com/bigeagle/gohop), [SoftEther VPN](https://www.softether.org/)，都是具有较为强大加密/混淆功能的 VPN 实现，其中 ShadowVPN 因为作者 clowwindy 被请喝茶而删除项目代码，GoHop 功能强大但暂时只支持 Linux，SoftEther VPN 使用不是很方便，所以目前都不是很流行
12. [V2Ray](https://github.com/v2ray/v2ray.github.io/wiki)，新工具，暂不了解
13. IPv6，据说 GFW 暂时还未能有效封禁 IPv6 地址，所以在教育网里还能通过 IPv6 访问 Google/Facebook 等。不过这个应该只是暂时的

## 自己搭建无缝无痛的翻墙服务
1. 翻墙路由器，刷 OpenWrt
    用于自己在家上网。
	1. VPN / shadowsocks + [chnroutes](https://github.com/fivesheep/chnroutes) / [cow](https://github.com/cyfdecyf/cow) / [meow](https://github.com/renzhn/MEOW) 自动分流国内外 IP
	2. [Dnsmasq](http://wiki.openwrt.org/doc/howto/dhcp.dnsmasq) + [pdnsd](http://members.home.nl/p.a.rombouts/pdnsd/) / [ChinaDNS](https://github.com/shadowsocks/ChinaDNS)
	3. 如果用了 Airport Extreme 之类的无法刷系统的路由器的话，可以接两层路由器，第一层用刷过 OpenWrt 的路由先翻一遍墙

2. 翻墙（HTTP）代理
    在命令行终端里只能通过配置 `http_proxy`/`https_proxy` 变量来翻墙，所以需要一个翻墙代理。而且下述 PAC 文件、运营商描述文件都需要有一个代理作为基础。
    1. HTTP 代理只能用国内服务器中转，直接部署在国外肯定被墙
	2. 建议在海外服务器部署 shadowsocks 服务端，国内服务器部署 shadowsocks 客户端
	3. shdaowsocks 代理转 http 代理可用 privoxy / polipo / cow / meow
	4. privoxy / polipo 生成的代理是纯 HTTP 代理，cow / meow 则会自动学习已翻墙/未翻墙网站并更新列表、生成 PAC 等。但是 cow 因为默认认为网站未被墙，所以很多网站会尝试多次连接才能使用代理，速度有一定影响；而 meow 是白名单模式，可能会影响部分国内网站的加载速度

3. 提供远程 PAC 文件
    Windows / Mac / Linux，以及 iOS / Android 5+ 在 WiFi 网络下都可以配置 PAC 代理，Android 4.x 可以用 SmartProxy 这款 App
    1. 如果没什么特殊需求，最简单的是[利用 gfwlist 生成](http://codelife.me/blog/2013/04/06/convert-gfwlist-to-pac/)，然后开个静态文件的 http 服务以便客户端访问
	2. 不过更优的方法是[使用 cow 生成的 PAC](https://github.com/cyfdecyf/cow#user-content-详细使用说明)

4. [制作运营商描述文件](https://velaciela.ms/use_apn_connect)
    用于在 2G/3G/4G 下翻墙。不过不是太建议这个方法，根据个人经验，不论设置文件怎么写，总有些时候会出现莫名其妙的问题，到时候就只能删掉描述文件，在需要翻墙时再加回来，很是麻烦
   而且对于使用了 HttpDNS 服务的各大 App 都无法兼容，包括但不仅限于微信朋友圈小视频、阿里旅行、滴滴出行地图、虾米、淘宝电影选座、网易云音乐等

5. AnyConnect
    未越狱 iOS 设备，除已下架的 Surge 以外，使用 AnyConnect 是最方便的
	1. 服务端使用 ocserv
	2. AnyConnect 用路由表做分流，所以不太精确，而且翻墙后速度最多也只有 2Mbps
	3. ocserv 默认限制路由表最长为 64 条，但其实客户端最长可接受 200 条，所以可以通过修改源代码后编译的方式调整这个上限，参看[这个帖子](https://www.v2ex.com/t/136431)
	4. [这里](https://github.com/travislee8964/Ocserv-install-script-for-CentOS-RHEL-7)有个 CentOS & RHEL7 的安装脚本（已调整过路由表上限），即使不用这个脚本而自行安装，也可以参考其中给出的路由表

6. Surge
   iOS 翻墙首选，支持 shadowsocks 代理，支持类 PAC 的配置，支持路由表，支持根据 IP 地址分流。虽然配置麻烦，但是配置好之后可以说是一劳永逸。
    但是既然已经下架就不多介绍了。

7. 国际网络线路优化
    1. 如果有国内服务器，可以直接用前述 shadowsocks 转 http 代理的方法，也可以直接设置 haproxy 转发 shadowsocks 代理
	2. 如果不想买国内服务器的话，可以使用 [微林的 vxTrans 服务](https://vnet.link/?rc=18139) 将代理进行端口转发，流量转发至电信 CN2 精品网，解决直连海外 VPS 太慢的问题。
    但是这里有个坑：

        > 由于CN2线路是特殊线路，大部分国外ISP运营商由于没有与CN2直接互联，因此为了避免额外的网间结算费用会限制流量而断开链接（特别是国外VPS服务提供商，这种情况很普遍）。

        所以建议使用 AliCloud BGP 线路转发

8. TCP 加速（防丢包）（**不建议使用**）

    1. [net-speeder](https://github.com/snooda/net-speeder)，开源，简单粗暴地通过两倍发包来防止丢包，对丢包严重的网络有一定改善作用，不过有一些缺点：
        1. 双倍发包会造成流量翻倍
        2. net-speeder 会造成 pptpd 等不支持双倍发包的网络软件无法正常使用
        3. 对小文件加速效果不明显
        4. 这种 TCP 优化机制一直存在争议，因为它实际实际上加剧了网络的拥堵，浪费掉了大量没必要的带宽
    2. [锐速](http://serverspeeder.com/)，比较老牌的 TCP 加速服务，闭源，比 net-speeder 智能，但不支持所有 VPS，闭源还要求 root 权限也让人有点不放心。而且，仍然会增加流量消耗，仍然被认为是不道德的，参见 shadowsocks 作者 clowwindy 在 V2EX 上的[评论](http://v2ex.com/t/164883?p=1#r_1742730)

## 收费翻墙服务

**不建议使用免费服务**

1. HTTP 代理以及 AnyConnect
	1. [轻云](http://theqingyun.info/r/2g3wq0)
	   使用两年，换过两次域名，另外还挂过两次，不过恢复时间较快。如果发现网站上不去了可以发任意内容邮件到 theqingyun@gmail.com 获取最新地址
	2. [土行孙](http://itxs.co/s/b6098a9h)
	   使用半年，挂过两次。**文档极为完善**，对于不怎么翻墙、不熟悉翻墙工具的小白用户十分友好
	3. [熊猫翻滚](https://ezcat.xyz/)
	   价格略贵，有[微博客服](http://weibo.com/pandafanorg)，一般没什么问题，但小坑不断，典型的互联网服务。最近因不可抗力挂过一周
2. VPN
	1. [云梯 VPN](https://www.ytvpn.com/)
	2. [Astrill](https://www.astrill.com/)，目前似乎速度已经不怎么样了，不推荐
	3. [VyprVPN](https://www.goldenfrog.com/vyprvpn)
3. [EurekaVPT](https://eurekavpt.com/)
    没用过，口碑不错，价格略贵，需要邀请

## 参考链接
1. [阅后即焚：“GFW”](http://www.chinagfw.org/2009/08/gfw_30.html)
2. [GFW 技术研究和云梯产品故事](http://teahour.fm/2013/07/09/gfw-and-vpncloud.html)
3. [深入理解 GFW：总论](http://gfwrev.blogspot.jp/2009/10/gfw.html)
4. [深入理解 GFW：路由扩散技术](http://gfwrev.blogspot.fr/2009/11/gfw_05.html)
5. [GFW 钓鱼计划](http://gfwrev.blogspot.fr/2009/11/gfw.html)
6. [深入理解 GFW：内部结构](http://gfwrev.blogspot.fr/2010/02/gfw.html)
7. [深入理解 GFW：结论](http://gfwrev.blogspot.fr/2010/03/gfw.html)
8. [GFW 研究与诊断工具](http://gfwrev.blogspot.fr/2009/11/gfw_10.html)
9. [GFW 的工作原理及突破技术](http://m.friendfeed-media.com/4457da9004be475a669d679bec5f17120bdf3d08)
10. [GFW 的原理和绕过](https://raw.githubusercontent.com/shell909090/slides/master/pdf/GFW.pdf)
11. [GFW 的详细分析及翻墙路由器（fqrouter）的原理和实现](https://docs.google.com/document/d/1mmMiMYbviMxJ-DhTyIGdK7OOg581LSD1CZV4XY1OMG8/edit)
12. [How governments have tried to block Tor](https://www.youtube.com/watch?v=GwMr8Xl7JMQ&index=128&list=WL)
13. [Learning more about the GFW's active probing system](https://blog.torproject.org/blog/learning-more-about-gfws-active-probing-system)
14. [道高一尺，牆高一丈：互聯網封鎖是如何升級的](https://theinitium.com/article/20150904-mainland-greatfirewall/)
