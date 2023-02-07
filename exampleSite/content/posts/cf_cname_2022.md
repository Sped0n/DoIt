---
title: "通过Cloudflare SaaS实现域名CNAME接入"
description: "Access domain name with CNAME through Cloudflare SaaS."
date: 2022-09-13T08:00:00+08:00
lastmod: 2022-09-13T08:00:00+08:00
url: "/dev/notageek/cf_cname_2022/"
tags: ["server","guide"]

categories: ["折腾"]
authors: ["spedon"]
draft: false
toc:
    enable: true
    auto: true
math:
    enable: false
share:
    enable: true
comment:
    enable: true
sponsor:
    enable: false


rssFullText: true
twemoji: false
fontawesome: true
lightgallery: true
ruby: false
fraction: false

linkToMarkdown: false
linkToSource: false
linkToEdit: false
linkToReport: false

---

2022年通过SaaS将域名通过CNAME方式接入Cloudflare的方法记录

<!--more-->

## 为什么要使用CNAME接入?

因为国内外的网络环境不同，很多人都希望可以在保留目前国内域名DNS服务器不改变的情况下，还可以接入国外的DNS/CDN服务商，或者实现国内/国外DNS分流，令国内外都能有比较好的访问体验。

## 目前通过CNAME接入CloudFlare的现状

### 前情提要

先前通过`CloudFlare Partner`接入的方法已经失效，很不幸我们错过了`CloudFlare Partner`能薅的羊毛，但官方的`CloudFlare for SaaS`也为我们提供了一种更加灵活的接入方式，并且自从2022年3月开始，CloudFlare调整了SaaS的免费额度，详见[官方博客](https://blog.cloudflare.com/zh-cn/waf-for-saas-zh-cn/)，过去是每个月会向SaaS下每个域名收取 <u>2USD</u> 的费用，现在不仅提供<u>100个</u>**免费域名额度**，并且超额以后只会按照每个域名 <u>0.1 USD/月</u> 收取费用，这才是良心大厂啊。

### CloudFlare for SaaS是什么

CloudFlare 中一个完全接入的域名即为一个 `zone`，点进去包括套餐、安全等等都是针对这一主域名配置的。官方 SaaS 功能针对的是你服务的客户，开放这项功能允许使用他们自己的域名直接附加在你的 `zone` 上，享受你 `zone` 中配置的安全、加速选项。

> 假设`a.com`通过`NS`接入了CloudFlare，`b.com` 未接入 CF；可以通过 SaaS 功能实现 `1.b.com`/`2.b.com` 等直接附加在 `a.com` 上，通过 CNAME 指向 CF 的节点，并享受`a.com`中设置的WAF规则，页面规则，加速设置等等。

> 不包括以下功能：
>
> **DNS范域名**/**Spectrum流量分析**/**Argo**/**Page Shield**
>
> 详见[官方博客](https://developers.cloudflare.com/cloudflare-for-saas/)

## 配置CNAME接入

### 需要准备

* 外币信用卡一张/PayPal账户 (可使用绑定国内银联卡的账户)
* 两个域名 (一个设置`CloudFlare for SaaS`，我们称为`承载域名`，一个是你需要通过`CNAME`接入的`主域名`)

### 承载域名接入

> 先自行遵循 CloudFlare 的指示通过NS方式接入你的`承载域名`，接入成功以后我们就可以开始订阅`CloudFlare for SaaS`

打开你接入好的`承载域名`，选择`SSL/TLS`下的`自定义主机名`，点击启用`CloudFlare for SaaS`，之后会让你绑定支付方式，不用担心扣费，里面是用个 0USD 的方案的，连验证扣费都没有，确认以后就可以开始下一步了。

![image-20220913004449926](https://img.sped0nwen.com/image/2022/09/13/t9rhq-0.webp "我的已经设置过了，就大概截个图展示一下位置")

### 设置源站

先前往`承载域名`的DNS处添加一条解析记录，把一个子域名指向你服务器的IP，比如我这里就是`cdn.xxx.xxx`

![image-20220913005627246](https://img.sped0nwen.com/image/2022/09/13/xiwb3-0.webp)

然后前往之前的自定义主机名处，添加回退源，就是<u>你刚刚设置的子域名</u>，添加后自行刷新或**等待一会**，直到下方的回退源状态变为**有效**，这个回退源会**同步**这个子域名设置的<u>源站/源IP</u>作为后续在此接入域名的<u>源站/源IP</u>。

![image-20220913010044185](https://img.sped0nwen.com/image/2022/09/13/1ntj2c-0.webp)

> 目前**免费**/**专业**/**商业**版本用户都无法为每个接入SaaS的域名设置单独的回退源，只有**企业**版本可以自由选择回退源，如果你要接入的网站有不同的<u>源站/源IP</u>，你就需要添加多几个`承载域名`的zone来设置，比较麻烦。
>
> ![image-20220913011011105](https://img.sped0nwen.com/image/2022/09/13/1tkkwg-0.webp)

### 添加自定义主机名

后续的工作就很简单了，点击`添加自定义主机名`，输入你要添加的未在 CF 接入的`主域名`的子域名，选择TXT验证。

![image-20220913011832843](https://img.sped0nwen.com/image/2022/09/13/1yh0qh-0.webp)

### 验证域名所有权

添加完成后，按要求在你DNS服务商处添加解析证书和主机名两个 TXT 记录，解析生效后等待一段时间即可验证通过，到此这个`主域名`的子域名就正确的添加到了你`承载域名`的 zone 中并接入了 CloudFlare 的CDN网络

![image-20220913012010432](https://img.sped0nwen.com/image/2022/09/13/1zj3kl-0.webp)

> 特别提醒，如图这里 CloudFlare 给出的验证 TXT 名称是应完整域名的解析记录，所以在自己的第三方 DNS 配置的时候，**填入的主机名应当是** `_acme-challenge.abc` 和 `_cf-custom-hostname.example.abc`，如果**直接复制**框内的内容把根域名 `def.com` 填进了主机名全域就变成了 `_acme-challenge.abc.def.com.def.com` 了，解析出来**肯定是错的**，想验证自己的域名解析 TXT 记录是否匹配，可以前往[这里](https://coding.tools/tw/nslookup)测试查看，如果没有问题就慢慢等 CloudFlare这边生效吧

> 在<u>证书状态</u>和<u>主机名状态</u>两个未显示<u>有效全绿</u>时**不可添加**CNAME记录！！！

### 在DNS服务商通过CNAME接入你的主域名下的子域名

选择解析类型为`CNAME`，输入你之前设置的回退源

> 对于防火墙规则、页面规则，直接将添加进的域名输入规则中即可
