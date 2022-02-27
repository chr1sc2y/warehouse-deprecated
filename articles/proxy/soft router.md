# 使用 Mac Mini M1 作为软路由让全家设备出国

[TOC]

本文介绍如何将功耗超低的 Mac Mini M1 改造为软路由，让局域网内连上 Wi-Fi 的所有家庭设备都可以免设置直接出国。

## 代理客户端

首先需要下载一个可以运行在 M1 上的代理客户端，推荐使用 Surge 或者 [ClashX Pro](https://install.appcenter.ms/users/clashx/apps/clashx-pro/distribution_groups/public)，前者买断价格较贵（每个 Mac 设备 $49.99），后者免费，本文以后者为例。

## 配置

一般机场会提供 Clash 的订阅链接，如果没有可以自行搜索 ssr/v2ray 转 clash 的第三方订阅转换服务，生成 Clash 订阅链接。

有了订阅链接之后，点击顶部菜单栏的 Clash 图标 -> Config -> Remote config -> Manage 管理订阅。

![ClashX Pro config manage](https://raw.githubusercontent.com/ZintrulCre/warehouse/main/resources/proxy/ClashX-Pro-config-manage.png)

点击 Add，在 Url 中输入订阅链接，Config Name 可以任意填。

![Add a remote config](https://raw.githubusercontent.com/ZintrulCre/warehouse/main/resources/proxy/Add-a-remote-config.png)

除此之外还要勾选 Clash 图标里的 Set as system proxy 和 Enhanced Mode 两个选项，这样才能保证 Mac Mini M1 能够成为网关。

![Enhanced Mode](https://raw.githubusercontent.com/ZintrulCre/warehouse/main/resources/proxy/Enhanced-Mode.png)

## 设置网关

到此 Mac Mini M1 已经可以成功地出国了，现在将 Mac Mini M1 作为网管，对路由器上的所有流量进行代理。

首先将路由器和 Mac Mini M1 使用网线连接，用 Wi-Fi 连接会不太稳定。

现在打开 System Preference -> Network -> Ethernet，在 Configure IPv4 中关闭 DHCP，选择 Manually 为 Mac Mini M1 设置一个固定 IP，这个 IP 必须和路由器在同一个网段，例如路由器的 IP 是 192.168.0.1，那么 Mac Mini M1 的 IP 就只需要将最后一位改为 2～255 里的数字，并且要避免和其他开启 DHCP 的设备冲突。

![Network](https://raw.githubusercontent.com/ZintrulCre/warehouse/main/resources/proxy/Network.png)

现在打开路由器的设置页面，每个厂商的页面地址都不相同，例如小米路由器的后台 URL 是 miwifi.com，TP-Link 的 URL 是 tplogin.cn，以后者为例。选择路由设置 -> DHCP 服务器，将网关、首选 DNS 服务器、备用 DNS 服务器都修改为刚才设置的 Mac Mini M1 的 IPv4 地址。

![DHCP](https://raw.githubusercontent.com/ZintrulCre/warehouse/main/resources/proxy/DHCP.png)

点击保存之后，局域网 Mesh 下的所有设备都可以成功出国了。

## One more thing

Mac Mini M1 在休眠后，因为路由器网关，整个 Wi-Fi 都会瘫痪，所以要勾选 System Preference -> Energy Saver 里的 Prevent your Mac from automatically sleeping when the display is off。

![Energy Saver](https://raw.githubusercontent.com/ZintrulCre/warehouse/main/resources/proxy/Energy-Saver.png)