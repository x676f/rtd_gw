搭建网关：透明代理、自动分流
==================================

这是目前我能想到的、正在使用的最优雅的番茄方案……


预期效果
----------

自动分流局域网内所有设备的访问流量，包括 PC 电脑、手机、iPad、电视盒子、游戏机等……

设备通过 WiFi 或有线连入家庭或办公室网络时，由路由器的 DHCP 服务自动分配指定网关，（当然如果只想少量设备才使用此网关，手工配置也行）

自动分流规则：

- 代理：其它 DNS 解析和访问流量
- 直连：局域网流量
- 直连：墙内 DNS 解析和访问流量
- 直连：BT 流量
- 拦截：广告流量


适用范围
----------

中国大陆墙内


代理形式
----------

使用网关的设备，自动分流。

单独经过网关指定协议和端口的流量：

- ``socks5`` 协议 ``1080`` 端口：全部代理
- ``socks5`` 协议 ``1081`` 端口：自动分流
- ``HTTP`` 协议 ``1080`` 端口：全部代理
- ``HTTP`` 协议 ``1081`` 端口：自动分流



Lorem Ipsum
--------------------

Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.


静态路由
----------

非必需，写在这里是个人需求的备忘

.. code-block:: console

   $ route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.2.1
