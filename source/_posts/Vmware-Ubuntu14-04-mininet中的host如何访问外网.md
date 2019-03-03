---
title: Vmware+Ubuntu14.04+mininet中的host如何访问外网
tags:
  - SDN
  - Mininet
  - Linux
categories:
  - 原创
  - SDN
copyright: true
abbrlink: 5bd388e3
date: 2016-01-17 17:32:21
---
最近需要mininet虚拟出的网络拓扑中的host访问外网，搞了几天，总是出些小问题，今天终于可以不出问题的搞定了。在这里总结一下，以防以后再出问题。

`环境：Win7，Vmware workstation 10.0 ，Ubuntu 14.04，mininet 2.2.0`

首先把宿主机win7中的VMnet8设置为自动获取IP地址，然后配置Vmware的Ubuntu，配置两块网卡，都是NAT模式。如下图：
<center>![图1](5bd388e3/1.png)</center>
点击确定后，点击虚拟网络编辑器，配置VMnet8的子网和掩码以及网关：
<center>![图2](5bd388e3/2.png)</center>
我这里设置子网为10.0.0.0，子网掩码为255.255.255.0，当然也可以设置为其他的，因为是NAT模式，所以不影响其连外网。这里主要是方便后面设置主机的IP。
<center>![图3](5bd388e3/3.png)</center>
网关设置为10.0.0.254。
<center>![图4](5bd388e3/4.png)</center>
这里的DHCP地址范围设置随便取一个合适的范围。
 
OK，上面的配置已经为Ubuntu配置好了网络，可以启动Ubuntu了，查看网卡信息：
<center>![图5](5bd388e3/5.png)</center>
这个时候，ping一下，则可以ping通，而且只有通过eth0来ping通，eth1 ping不通。

分别用：`ping -I eth0 baidu.com` 和 `ping –I eth1 baidu.com`测试。

为了后面的需要，我们把eth1的IP设置为：0.0.0.0，这样这个闲置的网卡资源就可以被用来桥接到mininet网络中的交换机上，这个后面会介绍怎样桥接。

利用命令：`sudo ifconfig eth1 0.0.0.0`，查看IP地址时eth1已经看不到IP地址了。
<center>![图6](5bd388e3/6.png)</center>
好了，后面开始重点部分了，先在本地运行floodlight控制器，ip为127.0.0.1，端口为6653。然后编写python脚本创建mininet网络，如下：
``` python
#!/usr/bin/python
import re
import sys
from mininet.cli import CLI
from mininet.log import setLogLevel, info, error
from mininet.net import Mininet
from mininet.link import Intf
from mininet.topolib import TreeTopo
from mininet.util import quietRun
from mininet.node import OVSSwitch, OVSController, Controller, RemoteController
from mininet.topo import Topo
 
class MyTopo( Topo ):
#    "this topo is used for Scheme_1"
    
    def __init__( self ):
        "Create custom topo."
 
        # Initialize topology
        Topo.__init__( self )
 
        # Add hosts 
        h1 = self.addHost( 'h1' , ip="10.0.0.1/24", mac="00:00:00:00:00:01", defaultRoute="via 10.0.0.254")
        h2 = self.addHost( 'h2' , ip="10.0.0.2/24", mac="00:00:00:00:00:02", defaultRoute="via 10.0.0.254")
        h3 = self.addHost( 'h3' , ip="10.0.0.3/24", mac="00:00:00:00:00:03", defaultRoute="via 10.0.0.254")
        h4 = self.addHost( 'h4' , ip="10.0.0.4/24", mac="00:00:00:00:00:04", defaultRoute="via 10.0.0.254")
        
        # Add switches
        s1 = self.addSwitch( 's1' )
        s2 = self.addSwitch( 's2' )
        s3 = self.addSwitch( 's3' )
 
        # Add links
        self.addLink( s1, s2 )
        self.addLink( s1, s3 )
        self.addLink( s2, h1 )
        self.addLink( s2, h2 )
        self.addLink( s3, h3 )
        self.addLink( s3, h4 )
//检查eth1或者其他指定的网卡资源是不是已经被占用
def checkIntf( intf ):
    "Make sure intf exists and is not configured."
    if ( ' %s:' % intf ) not in quietRun( 'ip link show' ):
        error( 'Error:', intf, 'does not exist!\n' )
        exit( 1 )
    ips = re.findall( r'\d+\.\d+\.\d+\.\d+', quietRun( 'ifconfig ' + intf ) )
    if ips:
        error( 'Error:', intf, 'has an IP address,'
               'and is probably in use!\n' )
        exit( 1 )
 
if __name__ == '__main__':
    setLogLevel( 'info' )
 
    # try to get hw intf from the command line; by default, use eth1
    intfName = sys.argv[ 1 ] if len( sys.argv ) > 1 else 'eth1'
    info( '*** Connecting to hw intf: %s' % intfName )
 
    info( '*** Checking', intfName, '\n' )
    checkIntf( intfName )
 
    info( '*** Creating network\n' )
    net = Mininet( topo=MyTopo(),controller=None) //关键函数，创建mininet网络，指定拓扑和控制器。这里的控制器在后面添加进去
    switch = net.switches[ 0 ] //取第一个交换机与eth1桥接
    info( '*** Adding hardware interface', intfName, 'to switch', switch.name, '\n' )
    _intf = Intf( intfName, node=switch ) //最关键的函数，用作把一个网卡与一个交换机桥接
 
    info( '*** Note: you may need to reconfigure the interfaces for '
          'the Mininet hosts:\n', net.hosts, '\n' )
    c0 = RemoteController( 'c0', ip='127.0.0.1', port=6653 )
    net.addController(c0)
    net.start()
    CLI( net )
    net.stop()
```
上面的脚本运行后，在floodlight web UI中可以看到创建了如下拓扑：
<center>![图7](5bd388e3/7.png)</center>
用上面的脚本设置了虚拟网络中的host的IP地址，MAC地址以及默认网关，然后把 Ubuntu的eth1网卡桥接到s1上，这里实现这个桥接功能主要是由Intf函数起作用，可以参看https://github.com/mininet/mininet/blob/master/examples/hwintf.py：

使用 ```sudo python mytopo.py``` 运行脚本，出现mininet命令行。在命令行中使用xterm h1打开h1的独立窗口，再ping一下baidu.com。
<center>![图8](5bd388e3/8.png)</center>
到这里就完成了host访问外网的任务了，而且在Ubuntu和win7中也都可以和host通信（ping通）。
最后我根据自己的理解画了个总体的图，仅作为参考，不对的地方请留言指出，谢谢。。。
<center>![图9](5bd388e3/9.png)</center>

----
参考链接：
1. http://techandtrains.com/2013/11/24/mininet-host-talking-to-internet/
2. http://www.muzixing.com/pages/2013/12/06/yuan-chuang-mininetda-jian-zi-ding-yi-wang-luo-tuo-bu-by-muzi.html