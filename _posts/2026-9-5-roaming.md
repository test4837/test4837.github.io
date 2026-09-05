---
title: 国际漫游在中国也有墙,我如何看.
date: 2026-09-05
---
![Roaming](/images/roaming.jpg)

在中文互联网的认知里，长期存在一个“技术避风港”：外国SIM卡。无论是新加坡的 Singtel，还是香港的 CMHK 或 CSL，只要手机开启漫游，跳出“境外 IP”的那一刻，用户往往觉得自己已经物理性地越过了防火墙。
然而，最近有用户发现，这种“特权”正在缩水。当你拿着 Singtel 的新加坡 IP，却在访问某些特定新闻网站或国际组织网页时看到熟悉的 ERR_CONNECTION_RESET，那种荒诞感——或者说用网友的话讲，“太抽象了”——便油然而生。

## 一、现象：被选择性干预的漫游流量

根据目前的实测反馈，测试涉及 Singtel、CMHK、CSL 等主流国际漫游服务。一个比较明显的现象是：**漫游流量并没有因为使用境外 SIM 卡而完全避开中国网络侧的流量干预。**

**大众平台：** Google、YouTube、Twitter、Facebook 等服务在部分测试中仍然可以正常访问。

**特定目标：** 一些特定的政治敏感新闻媒体、组织及网站则出现连接重置、无法建立连接等现象，例如万维、博讯、国际特赦组织以及部分深度政治网站。

因此，**国际漫游并不能简单理解为“进入中国以后就完全不受中国网络侧的干预”。**不同目标之间存在明显的差异化表现。

---

## 二、技术解释：为什么 IP 在境外，连接仍然可能被重置？

很多人可能会产生一个疑问：

> 既然使用的是新加坡、香港等地的 SIM 卡，手机获得的出口 IP 也可能属于境外运营商，那么流量是不是应该直接从境外出去？

这里需要区分两个概念：

**IP 的归属地 ≠ 数据包进入互联网之前所经过的网络路径。**

### 1. 漫游接入仍然发生在中国境内

使用国际 SIM 卡在中国大陆进行 4G/5G NSA 数据漫游时，手机首先仍然需要接入中国境内的无线接入网络，例如中国移动、中国联通等运营商的基站。

对于通常的蜂窝数据漫游，在采用**归属网路由（Home Routed）**的情况下，用户数据会从访问地网络进入漫游互联网络，再传送到归属运营商的核心网。

简化来看，可以理解为：

```text
境外 SIM / 漫游用户
        ↓
中国境内基站
        ↓
中国访问网核心网络
        ↓
漫游互联 / GTP 用户面 (中国侧即监管)
        ↓
国际漫游互联网络
        ↓
归属运营商核心网
        ↓
Internet
```

因此，**手机虽然最终可能使用境外运营商的 IP 出口，但数据流量在离开中国境内网络之前，仍然会经过中国境内的运营商网络和相关国际互联设施。**

对于 Home Routed 漫游来说，流量通常需要经过中国骨干网的国际出口网关或相关国际互联链路；但具体路径会受到运营商漫游架构、互联方式以及是否采用 Local Breakout 等因素影响。

需要特别说明的是，这里讨论的是**普通 4G/5G 蜂窝数据漫游**，并非 Wi-Fi Calling 等基于非 3GPP 接入的场景。

---

### 2. SNI：加密连接中的“域名线索”

HTTPS 可以加密网页正文、Cookie、账号密码等大量内容，但传统 TLS 连接建立过程中，客户端通常仍会提供服务器名称指示（SNI）。

例如访问：

```text
https://example.com
```

在传统 TLS 连接中，连接建立阶段可能出现：

```text
ClientHello
    └── SNI: example.com
```

因此，网络中间设备即使无法看到 HTTPS 加密后的网页内容，也可能根据 TLS ClientHello 中的 SNI 判断客户端正在尝试访问哪个域名。

需要注意的是，**这并不意味着所有 HTTPS 连接都必然以明文形式暴露域名。**例如 TLS 1.3 的 ECH（Encrypted Client Hello）可以进一步隐藏部分 ClientHello 信息。

---

### 3. DPI：在连接建立阶段进行识别

如果网络侧设备能够看到 TLS ClientHello，那么就可能根据其中的 SNI、IP 地址、TLS 特征等信息进行流量分类。

一种可能的处理过程是：

```text
客户端
  ↓
TLS ClientHello
  ↓
网络侧设备识别目标
  ↓
命中某种过滤规则
  ↓
连接被主动终止
```

在 TCP 场景下，外部设备可以通过注入 TCP RST 等方式主动终止连接，于是客户端最终可能看到：

> `ERR_CONNECTION_RESET`

因此，**“境外 IP + 中国境内漫游接入”并不能单独证明连接一定绕过了中国网络侧的流量检测。**

---

## 三、真正值得注意的地方

这也是国际漫游与普通境外代理/VPN之间一个很重要的区别：

> **漫游的“出口 IP”属于谁，与漫游数据在到达这个出口之前经过哪些网络，并不是同一个问题。**

如果采用 Home Routed 漫游：

```text
          中国境内
┌───────────────────────┐
│ 手机 → 基站 → 访问网   │
└──────────┬────────────┘
           │
           │ 漫游互联
           ↓
      归属运营商核心网
           │
           ↓
       Internet
           │
           ↓
     境外出口 IP
```

(离开中国的最后一跳即监管)

因此，即使最终网站看到的是一个新加坡、香港等地的 IP，**也不能据此推断从手机到境外出口之间的整个路径都处于中国网络侧的“不可见区域”。**

这也能够解释为什么某些国际漫游卡在中国大陆使用时，仍然可能出现与境内网络类似的连接重置现象。

(使用手段有:SNI DPI,DNS 污染,SNI RESET)

被屏蔽的网站有
法轮功系列,博讯网,国际特赦组织等.

---------------------

境外卡漫游到中国带墙的测试
（以下测试是CSL/Singtel 漫游到中国联通/电信的测试。）

curl -Iv --http2 --doh-url https://1.1.1.1/dns-query https://www.epochtimes.com
* Host www.epochtimes.com:443 was resolved.
* IPv6: (none)
* IPv4: 130.211.7.151
*   Trying 130.211.7.151:443...
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* Recv failure: Connection reset by peer
* TLS connect error: error:00000000:lib(0)::reason(0)
* OpenSSL SSL_connect: Connection reset by peer in connection to www.epochtimes.com:443
* closing connection #0
curl: (35) Recv failure: Connection reset by peer

~ $ curl -Iv --http3-only --doh-url https://1.1.1.1/dns-query https://www.epochtimes.com
* Host www.epochtimes.com:443 was resolved.
* IPv6: (none)
* IPv4: 130.211.7.151
*   Trying 130.211.7.151:443..
> HEAD / HTTP/3
> Host: www.epochtimes.com
HTTP/3 200

--------------------

部分灵感引用来源
V2EX: https://v2ex.com/t/832129
Twitter: https://x.com/realNyarime/status/2034692544358228138?lang=zh

--------------------
维基百科的"中华人民共和国被封锁网站列表"也注明了
"现时，中国之外的SIM卡漫游到中国会采取不同的封锁策略。会屏蔽一些有关法轮功网站和部分新闻网站。"
