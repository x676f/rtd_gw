##############################
搭建网关：透明代理、自动分流
##############################

这是目前我能想到的、正在使用的最优雅的番茄方案……


**********
综述
**********

笔者的部分生活和工作时间在中国大陆，众所周知，当局对国际互联网的访问限制、网络封索是存在已久的特别大的问题，而且预计短期内不会发生变化。

使用一个 **正常** 的国际互联网，是笔者的刚需。


已知协议
==========

已知，相对主流的无特征或有特征伪装的番茄代理的协议：

- Shadowsocks
- ShadowsocksR
- vmess


在用机场
==========

目前，我日常使用最多的是提供了 IPLC 专线的机场，速度飞快，但主要是 `ShadowsocksR`_ 协议，没有 `v2ray`_ 的 `vmess`_ 协议线路；
备用机场是 BGP 多线，提供 `vmess`_ 连入。

两家机场都有提供订阅链接，在 windows 上的 `clash`_ 客户端是惯用的方法。


历史方案的不足之处
==============================

- 机场提供的分流方案不一定适合自己，使用订阅不易个性化修改
- 每个设备都需要安装客户端、配置，不同设备使用的客户端和用法都不尽相同
- 在 Terminal 中使用代理需要单独设置代理，而且不确定命令是否支持，这样在下载和部署一些工作环境的时候，有时会特别缓慢不便
- 电视盒子的代理配置相当繁琐麻烦



综合解决方案：自建网关
==============================

自建网关的好处是，网关配置好后，对设备上运行的任何程序都能无感。

即设备通过 WiFi 或有线连入家庭或办公室网络时，由路由器的 DHCP 服务自动分配指定网关，（当然如果只想少量设备才使用此网关，手工配置也行）


思路框架
==========

- 物理机或虚拟机 1 台
- 安装 ``ubuntu 20.04 LTS`` 系统
- 使用 `v2ray`_ 框架与系统级的 ``iptables`` 配合处理经过网关的数据包


自动分流
----------

自动分流局域网内所有设备的访问流量，包括 PC 电脑、手机、iPad、电视盒子、游戏机等……

- 代理：其它 DNS 解析和访问流量
- 直连：局域网流量
- 直连：墙内 DNS 解析和访问流量
- 直连：BT 流量
- 拦截：广告流量


手工配置
----------

同时提供：本地不同协议和端口执行全代理或分流代理

- ``socks5`` 协议 ``1080`` 端口：全代理
- ``socks5`` 协议 ``1081`` 端口：自动分流
- ``HTTP`` 协议 ``8080`` 端口：全代理
- ``HTTP`` 协议 ``8081`` 端口：自动分流


使 v2ray 兼容 ShadowsocksR
----------------------------------------

想要把临近自己城市的 IPLC 专线作为默认代理，然而 `v2ray`_ 并不直接兼容 `ShadowsocksR`_ 协议，所以需要 `v2ray`_ 和 `ShadowsocksR`_ 协作：

- 独立运行 `ShadowsocksR`_ 实现 ``socks5 代理``，监听在端口 ``1080`` 上
- 然后把 ``1080`` 的 ``socks5`` 提供给 `v2ray`_ 作为一个 ``outbound``，就解决了由 `v2ray`_ 向 `ShadowsocksR`_ 的兼容

如需还需要一个 ``HTTP 代理``，就再引入一个 `privoxy`_ 作辅助，实现 ``HTTP 代理``，监听在端口 ``8080`` 上，转发给 ``1080`` 的 ``socks5``



**********
硬件准备
**********

- 物理机或虚拟机 1 台

家用或小型办公室，用一个树莓派就足够了；虚拟机用 VMWare 或者 PVE 也没问题，这里的目的就是有一个 Linux 环境的网关载体。


**********
搭建记录
**********

简要记录如下


ShadowsocksR
====================

如下


安装 ShadowsocksR
------------------------------

.. code-block:: console

   $ git clone https://github.com/shadowsocksrr/shadowsocksr /path/to/shadowsocksr


配置 ShadowsocksR
------------------------------

.. code-block:: console

   $ sudo nano /etc/shadowsocksr/config.json


使用 IPLC 机场的专线配置，设为系统服务，运行于 ``1080`` 端口，提供 ``socks5 代理``

.. code-block:: json

   {
     "server": "your.proxy.domain.or.ip",
     "server_port": 10000,
     "method": "your-method",
     "password": "your-password",
     "protocol": "your-protocol",
     "protocol_param": "",
     "obfs": "your-obfs",
     "obfs_param": "your-value",
     "local_address": "0.0.0.0",
     "local_port": 1080
   }


设为系统服务
------------------------------

下载并编辑 ShadowsocksR 系统服务配置

.. code-block:: console

   $ sudo wget https://github.com/x676f/rtd_xlog/raw/scripts/proxy/service/shadowsocksr.service /etc/systemd/system/shadowsocksr.service
   $ sudo nano /etc/systemd/system/shadowsocksr.service


内容

.. code-block:: text

   [Unit]
   Description=ShadowsocksR Service
   After=network.target

   [Service]
   ExecStart=/usr/bin/python3 /path/to/shadowsocksr/shadowsocks/local.py -c /etc/shadowsocksr/config.json
   Restart=on-abort

   [Install]
   WantedBy=multi-user.target


启用服务


.. code-block:: console

   $ sudo systemctl enable shadowsocksr



privoxy
==========


安装 privoxy
------------------------------

如题


配置 privoxy
------------------------------

如题



v2ray
==========

如下


安装 v2ray
------------------------------

如题


配置 v2ray
------------------------------

下载 `ToutyRater`_ 提供的 ``ext:h2y.dat`` 域名数据文件

.. code-block:: console

   $ sudo wget https://github.com/ToutyRater/V2Ray-SiteDAT/raw/master/geofiles/h2y.dat /usr/bin/v2ray/h2y.dat


下载并编辑 v2ray 配置文件

- `dokodemo-door`_ 任意门，由 ``tproxy`` 为网关流量提供转发服务
- ``1081`` 端口，提供自动分流的 ``socks5 代理``
- ``8081`` 端口，提供自动分流的 ``HTTP 代理``


.. code-block:: console

   $ sudo wget https://github.com/x676f/rtd_xlog/raw/scripts/proxy/v2ray/config.json /etc/v2ray/config.json
   $ sudo nano /etc/v2ray/config.json



******************************
使用 iptables 设置数据包规则
******************************

如题



******************************
静态 IP 地址与 DHCP 更新
******************************

手工设置静态 IP 地址
==============================

如题


更新路由器的 DHCP 设置中的网关 IP
========================================

如题



.. _Shadowsocks: https://github.com/shadowsocks
.. _ShadowsocksR: https://github.com/shadowsocksrr/shadowsocksr

.. _v2ray: https://www.v2ray.com
.. _dokodemo-door: https://www.v2ray.com/chapter_02/protocols/dokodemo.html
.. _vmess: https://www.v2ray.com/chapter_02/protocols/vmess.html

.. _privoxy: https://www.privoxy.org/
.. _ToutyRater: https://github.com/ToutyRater

.. _clash: https://github.com/Fndroid/clash_for_windows_pkg/releases
