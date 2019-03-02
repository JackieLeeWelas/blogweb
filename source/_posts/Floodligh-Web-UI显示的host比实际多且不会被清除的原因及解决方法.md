---
title: Floodligh Web UI显示的host比实际多且不会被清除的原因及解决方法
date: 2016-01-09 22:24:24
tags: [SDN,Floodlight]
categories:
- 原创
- SDN
copyright: true
---
每次启动完floodlight控制器，在http://127.0.0.1:8080/ui/index.html 中打开floodlight的Web UI界面后，发现host总是会比我定义的多，打开拓扑图界面也很混乱。网上查了下，说是因为OVS的一个local port会去发现外部网络的拓扑，只要禁用OVS的这个端口就可以了，有兴趣的可以试试那个方法。下面是我的实验过程及解决方法。

**命令：**
```
sudo mn --controller=remote,ip=127.0.0.1,port=6653 --topo=tree,2
```
**如下图：**
<center>![图1](1.png)</center>
<center>![图2](2.png)</center>
这样都分不清哪个host是我定义的，交换机倒还好，都很清晰。

所以我在用mininet创建网络拓扑的时候使用了如下命令：
```
sudo mn --controller=remote,ip=127.0.0.1,port=6653 --mac --topo=tree,2
```
多加了个mac参数，表示自动设置host的mac，会使我们的host的mac很有规律，如下图：
<center>![图3](3.png)</center>
虽然这样还没解决主机多出来几个的问题，但至少我们能很快分清哪几个使我们的host，这时的host的mac地址，会从00:00:00:00:00:01开始分配。

**1. 方法一：**   
最后我解决host多于实际的方法是先启动mininet，再启动floodlight，因为交换机启动时，链路需要协商，如果先启动floodlight，就会把这些数据包也记录下来。所以先启动mininet，等OVS稳定下来，再启动floodlight控制器，这样就不会把OVS协商链路时发现的一些主机也记录进去。
<center>![图4](4.png)</center>
<center>![图5](5.png)</center>
虽然一开始，启动mininet时，不能连接上控制器，但在启动控制器后，mininet会主动与控制器连接。

还有一个问题就是，当退出mininet后，UI上的交换机会立马没了，而主机还在。
<center>![图6](6.png)</center>
然后再用mininet创建网络，UI上的原来的host不变，host在这基础上又会增加几个，每次退出再创建都会多几个host。
<center>![图7](7.png)</center>
上面是我重复三次这样的过程后生成的host，本来只会生成四个host，如今已越来越多。这样很烦，解决的方法是，退出mininet后，就刷新一遍网页，注意不是直接按F5刷新，这样会出错，是再输一次：http://127.0.0.1:8080/ui/index.html 按回车。这时host才会从网页中清除：
<center>![图8](8.png)</center>
后面再创建拓扑的时候就重复上面的过程，先启动mininet，再启动floodlight，每次退出mininet，就刷新一遍网页。

**2. 方法二：**
Google查了下发现了其他的几种方法，试了下面的方法，感觉比之前的好了，但还是有点小问题，可能只是我机子的问题。仅作参考。
在floodlight的日志输出里有很多IPv6的信息。所以这个解决方法是禁用IPv6。
用命令：`sudo vim /etc/sysctl.conf` ，然后在最后添加下面三行：
```
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
    net.ipv6.conf.lo.disable_ipv6 = 1
```
保存后重启电脑或者运行：`sudo sysctl –p`
上面那种禁用IPv6的方法不一定都适用，使用其他禁用方法也可以。

**3. 方法三：**
上面的方法二有时候不怎么好，后面我又找到了另外一种更彻底的方法：
我的OVS版本是 2.0.1：
<center>![图9](9.png)</center>
Ubuntu版本是3.13:
<center>![图10](10.png)</center>
Google上说是OVS版本和Ubuntu的问题，OVS 2.0.1版本支持Ubuntu 2.6.32 到 3.10，所以我的问题出在OVS版本太低，或者Ubuntu版本过高。解决方法是升级OVS或者降低Ubuntu版本。

运行如下命令：
```
sudoapt-get install openvswitch-controller openvswitch-switchopenvswitch-datapath-source
```
把OVS更新到2.0.2：
<center>![图11](11.png)</center>
然后问题就解决了，至少目前是解决了，彻不彻底后面再看。

----