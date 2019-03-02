---
title: Floodlight控制器的安装以及SDN环境的配置
date: 2015-12-25 22:28:51
tags: [SDN,Floodlight]
categories: 
- 原创
- SDN
copyright: true
---
虽然网上有好多这种配置教材，但是在配置的过程中还是都会出各种问题，所以我想基于我自己的过程，记录下我的配置过程便于以后少走弯路，也给别人参考参考一下，下面的配置是我每步成功过后就记下来的，可能以后环境不是一模一样的还是会出各种小问题，这也难免。

首先在win7的VMware上安装Ubuntu14.04，并且在Ubuntu里安装一些常用到的软件
```
$sudo apt-get install vim,git
```
然后进入正式安装floodlight的环节：
### 安装java环境以及eclipse
```
$sudo apt-get install build-essentialdefault-jdk ant python-dev eclipse
```
### 下载floodlight源代码以及编译
```
$ git clone git://github.com/floodlight/floodlight.git 
$ cd floodlight 
$ ant; 
$ sudo mkdir /var/lib/floodlight   //同步数据的目录，编译完了floodlight会在这里自动生成一个SyncDB/文件夹，这行不是必需的
$ sudo chmod  /var/lib/floodlight  777
```
### 安装mininet
```
$sudo apt-get install mininet
```
然后可以简单测试下：
```
$sudo mn
```
可以进入mininet的命令行就表示安装成功。
### 运行floodlight：
```
$ cd floodlight
$ java –jar target/floodlight.jar //控制台就打印出debug信息
```
### 运行mininet：
```
$sudo mn --controller=remote,ip=127.0.0.1,port=6653
```
这一步是把在mininet中建立的虚拟网络连接到floodlight控制器上。
### 查看floodlight提供的UI界面
在浏览器中输入：http://127.0.0.1:8080/ui/index.html就可以看到floodlight提供的Web UI界面。在webUI中可以查看交换机，主机，流表以及网络拓扑等信息。
### 配置eclipse
上面已经完成了基本的配置工作，但是为了方便后续的开发，我们还需要配置好eclipse，把floodlight的源代码导入到其中。方便以后给控制器添加应用模块以及查看控制器的各个模块的源代码。
首先需要在floodlight的目录下执行下面这个命令：
```
$ant
```
然后打开eclipse，导入已存在的项目到工作空间，选择根目录为floodlight文件夹。

然后配置eclipse，在eclipse中右键floodlight目录，run as里面的run configurations,新建一个Java Application，name用FloodlightLaunch，project填Floodlight，main填net.floodlightcontroller.core.Main，点应用就OK了。

上面配置好了，就可以运行floodlight控制器了，点工具栏里的三角形按钮或者右键run as a JavaApplication,然后控制台就一直输出调试信息。后面就可以在eclipse中进行模块以及服务的开发。

***