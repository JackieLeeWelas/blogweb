---
title: Floodlight控制器创建一个模块的简单过程
date: 2015-12-24 17:07:18
tags: [SDN,Floodlight]
categories: 
- 原创
- SDN
copyright: true
---
假设floodlight和eclipse的安装以及配置已经完成，如果还没有，请参考：
https://floodlight.atlassian.net/wiki/display/floodlightcontroller/Installation+Guide

很简单的过程，大神就不用看了，主要是记下来方便自己以后用，也给需要的人参考，以下过程全部在eclipse中操作完成
1. 在floodlight项目的src/main/java包上右键新建Java类，填上包路径和Java类名以及继承的类（继承的类一般都包括"IOFMessageListener" 和 "IFloodlightModule"），然后就会自动生成一些需要重写的函数。

2. 为了使我们新建的这个类监听到OpenFlow消息，需要在FloodlightProvider （一个IFloodlightProviderService类）注册我们的类。

3. 我们需要修改getModuleDependencies()函数，用来告诉模块装载器我们依赖它。getModuleDependencies()函数是第一步添加父类后自动生成的函数。

4. 接着编写init方法，init方法在控制器启动的时候就会调用，用来加载依赖模块和初始化数据结构。

5. 然后实现基本的监听器，在startUP方法中注册PACKET_IN消息

6. 为OFMessage监听器加上一个ID，这步在getName()中实现

7. 关键的一步，定义接收到PACKET_IN消息后的行为，在receive()中实现，返回Command.CONTINUE以允许这个消息继续被其他的消息处理模块接收到。

8. 我们还需要为之前我们创建的模块注册，这样floodlight启动的时候就可以加载我们的模块，在这一步，首先我们得告诉加载器我们的模块的存在，这可以在src/main/resources/META-INF/services/net.floodlightcontroller.core.module.IFloodlightModule文件里添加我们的模块的类

9. 最后，我们还必须在floodlight模块配置文件中添加我们创建的模块，这个是在src/main/resources/floodlightdefault.properties文件里的floodlight.modules里添加我们的包和类的全路径。
-----