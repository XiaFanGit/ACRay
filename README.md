# ACRay

Docs: https://acray.motofans.club

## 前言

通常，我们直接在设备上连接国外 VPS 上搭建的 SS-Server / V2Ray Server 来实现科学上网。

用这样的方法，可能会遇到一些问题，比如 IP 被 BAN 掉，不同线路的访问速度差距较大，客户端配置较为繁琐，在企业环境应用起来尤其如此...

你可能也会直接在 VPS 上架设 VPN...但是这样的办法也会产生问题，比如，路由表的控制难以精确，造成部分站点的无法访问。

另外，无论使用上述哪个办法，DNS 解析全球 CDN 站点时可能将 IP 解析到国外节点这个问题都很难解决，这会造成本土站点访问速度下降，且浪费不必要的流量...

那么有没有在企业环境下比较完美的办法？

## 现成的方案

* [分流中转: https://sosonemo.me/strongswan-to-shadowsocks.html](https://sosonemo.me/strongswan-to-shadowsocks.html)

这种方案是将所有流量统一路由到一台国内的服务器，然后使用 iptables 在服务端完成分流。

好处是逻辑相对简单，无视域名，不需要维护 Pac 列表。最大问题是所有流量都要经过国内服务器（包括国内站点），造成国内节点流量的浪费。

## 我的设想

**使用 Ocserv + Pac + V2Ray-Local 实现智能分流**

### Ocserv

Cisco Anyconnect 是思科推出的一款企业级 VPN。其背后的开源技术是[OpenConnect](http://en.wikipedia.org/wiki/OpenConnect)。

### V2Ray

V2Ray 是一个模块化的代理软件包，它的目标是提供常用的代理软件模块，简化网络代理软件的开发。

![ACRay.png](https://github.com/XiaFanGit/ACRay/raw/master/ACRay.png)

**基于这个方案，我们的浏览器去访问网站时大致是这样一个过程:**

1. Anyconnect 链接 Ocserv 后 ，Ocserv 推送 Route Table，DNS 以及 PAC 到 Client
2. Anyconnect 将 Route Table 中的 IP 访问流量截获到 Ocserv，用于访问 Proxy Server 及其他的内部地址
3. Anyconnect 根据获取的 PAC，对符合规则的流量进行 Proxy，在远程服务器 V2ray Local 完成解析和访问
4. 对于既不存在 Route Table 中，也不符合 PAC 规则的流量，通过 Ocserv 推送的 DNS 在 Ocserv 完成解析，并在 Client 直接访问
5. 对于上述访问流程的控制，Route Table 的优先级高于 PAC
6. Ocserv 推送的 DNS 地址可以是公共 DNS 也可以是私有的

**需要的必要条件:**

* 国内一台公网服务器节点: Ocserv + V2ray-Local
* 国外一台服务器节点: V2ray-Server
 
## 部署 ACRay 服务

本问仅介绍如何在国内节点部署 Ocserv + V2Ray-Local，并不介绍如何在国外 VPS 上部署 V2Ray Server。

由于 Ocserv 的安装较为复杂，为了简化部署，我写了一份 Dockerfile，本文将通过 Docker 完成 ACRay 的部署。

项目地址: https://github.com/XiaFanGit/ACRay
