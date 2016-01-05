title: 本博客配置汇总
tags: blog
---

本博客是采用 [Hexo][] 搭建的一个纯静态网站。

<!-- more -->

## 托管

博客托管在 [GitHub Pages][] 上。

## 域名

**TODO**
域名购自 [NameCheap][]，域名解析服务使用 [DNSPod 国际版][DNSPod]。

## SSL

**TODO**
SSL 证书申请自 [Let's Encrypt][]。

## CDN

**TODO**

笔者希望自己的博客能够始终使用 SSL 访问，而 [GitHub Pages][] 和 [GitCafe Pages][] 均不支持自定义域名添加 SSL 证书，所以无法靠 Hexo 的多 Repo 部署 + 自定义国内外 DNS 解析地址的方式实现国内访问加速，因此不得不用上全站 CDN。

但是由于笔者懒得备案，所以无法选择国内 CDN 厂商，而国外 CDN 厂商中，[CloudFlare][] 在国内访问速度不佳，[Akamai][] 太贵，最后还是选择了 [KeyCDN][]。

[KeyCDN][] 在亚洲有香港、新加坡和日本节点，为了加速国内用户访问，笔者在 [DNSPod][] 上设置了几条 A 记录，将国内的域名解析指向了 [KeyCDN][] 香港节点。

此外，得益于 [KeyCDN][]，本站还开启了 HTTP/2 支持。

## 主题

博客的主题是我自己写的 [hexo-theme-sleepless][]，设计上主要参考了 [Vue.js 官方博客][] 的样式，不过根据自己的喜好做了一些微调，增加了一些自定义页面。

为了保证国内用户的访问速度，主题里还提供了 Google Fonts 镜像的设置，本博客目前采用的是[中科大的镜像源][USTC Google Fonts Mirror]（[360 的镜像][360CDN]不支持 HTTPS）。

为了完整支持 HTTPS，博客里也提供了替换多说默认的 embed.js 地址的选项（用法见下）。

## 评论系统
    
由于这是个中文博客，主要面向群体还是国人，所以评论系统采用了[多说][]。考虑到多说对 HTTPS 的支持尚不完整，于是参考[这篇文章][让多说支持 HTTPS]，在七牛上搭建了四个 HTTPS 镜像存储用于代理头像资源，然后修改了官方的 embed.js 并托管在自己的七牛空间中。


[Hexo]: https://hexo.io/
[GitHub Pages]: https://pages.github.com/
[NameCheap]: https://www.namecheap.com/
[DNSPod]: https://www.dnspod.com/
[Let's Encrypt]: https://letsencrypt.org/
[GitCafe Pages]: https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9
[CloudFlare]: https://www.cloudflare.com/
[Akamai]: https://www.akamai.com/
[KeyCDN]: https://www.keycdn.com/
[hexo-theme-sleepless]: https://github.com/sodatea/hexo-theme-sleepless
[Vue.js 官方博客]: http://vuejs.org/blog/
[USTC Google Fonts Mirror]: https://lug.ustc.edu.cn/wiki/lug/services/googlefonts
[360CDN]: http://libs.useso.com/
[多说]: http://duoshuo.com/
[让多说支持 HTTPS]: https://quericy.me/blog/788

