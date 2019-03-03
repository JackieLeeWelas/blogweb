---
title: 如何编写Floodlight REST 应用
tags:
  - SDN
  - Floodlight
categories:
  - 原创
  - SDN
copyright: true
abbrlink: b9b7a1c4
date: 2015-12-28 16:41:37
---
可以用任何你喜欢的编程语言编写REST应用
#### 参照步骤
1、确定需求，也就是你编写的REST应用需要哪些网络服务和信息。
2、检查REST API，看看是否有提供你所需的服务。  
* 如果有，了解其RESTAPI的语法，输入的参数以及可得的选项，这样就可以直接拿来用。  
* 如果没有，也可能是你所需的网络服务和资源信息没有提供REST API，但却可以在floodlight模块中可获得这些信息，只是没通过API暴露出来。这种情况，你可以自己实现REST API来提供你所需的服务。  
* 如果既没有REST API，又在floodlight中找不到，那你可以自己开发floodlight Java模块，并且实现自定义的模块的REST API来提供所需的服务。
<!--more-->
3、用所有你需要的REST API方法，设计以及组成你的应用。
4、测试你的应用并且反馈给floodlight。


下面通过在floodlight/apps目录下的 python Circuit Pusher应用说明。  
Curcuit Pusher例子给我们展示了如何创建一个在OpenFlow集群中的两个有IP的主机A和B之间的静态单路径线路。
#### 设计方法 
1、确定所需的网络服务和信息：  
* 主机A和B的接触点，即用（交换机ID，端口）表示的数据实体，代表A和B的物理位置。
* A和B之间接触点的路由，即从A经过哪个交换机和哪个端口到达B的路径
* 在A和B路由上所有交换机安装流量线路的服务

2、从RESTAPI中查到的可提供的信息：
+ 从/wm/device/的GET参数获取设备的接触点信息，比如IP地址
+ 从/wm/topology/route/<switchIdA>/<portA>/<switchIdB>/<portB>/json可以获取A和B接触点之间的路由信息
+ 用/wm/staticflowentrypusher/json的POST方法给指定的交换机安装流表项

3、应用设计：
+ 语言使用Python
+ 使用os.popen方法发送curl 命令来调用REST API的方法(应该还可以使用os.system)
+ 熟悉 /wm/device语法特点，然后在命令返回的结果中解析出A和B接触点的交换机
+ 熟悉 /wm/topology/route的语法，获取交换机和端口用来下发流表项
+ 对于每个交换机和端口对，可以通过/wm/staticflowentrypusher/json下发流表

----