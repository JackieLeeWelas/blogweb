---
title: 简单配置bind9 的master 和slave
date: 2016-01-02 20:55:21
tags: [bind9]
categories:
- 原创
- 网络
copyright: true
---
**系统**：两台FreeBSD 10.1
**部署**：一台做master，一台做slave

**具体步骤如下：**
1. 首先是安装bind9，我是用的ansible远程安装的，暂时还没有把主从两个安装和配置分开，所以一开始在两台FreeBSD上安装的是一样的bind9，包括named.conf和zone文件都是一样，后面再分开配置的。

2. 安装的过程就不赘述了，网上有很多资料，安装完后，就该分别配置两台主机使它们分别作为主从域名服务器了，其实基本配置差不多，比如options里的参数就差不多，只需要改变zone的配置。  

**在master中：**
```
zone"XXX.com" IN {
            type master;
            file "XXX.com.zone";
            allow-update { none; };
            allow-transfer { <slave的IP地址>; };  //允许被哪台slave复制数据过去
};
```
  **在slave中：**
  ```
  zone "XXX.com" IN {
             type slave;
             file "slaves/XXX.com.zone"; //自动创建并从master复制内容
             masters { <master的IP地址>; };  //指明那台是master，可以有多台，指定多台的时候，multi-master设置为yes
  };
  ```
 
3. Zone对应的资源文件只需要在master里编写和修改就可以了，配置好了后，分别重启服务：service named restart ，就可以看到在slave中原本没有资源文件，现在自动从master中同步过来了。当master中的zone设置了allow-transfer，且资源文件里的Serial有改变时，就会通知slave同步masters里对应地址的主域名服务器的数据。

----