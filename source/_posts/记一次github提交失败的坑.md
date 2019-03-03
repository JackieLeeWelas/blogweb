---
title: 记一次github提交失败的坑
tags:
  - 填坑
categories:
  - 原创
  - Git
copyright: true
abbrlink: f587e199
date: 2016-07-02 16:35:57
---
git push的时候出现错误：
```shell
$ blogweb git:(master) git push
> remote: Permission to XXX/XXX.git denied to XXX.
fatal: unable to access 'https://github.com/XXX/XXX.git/': The requested URL returned error: 403
```
先试着把https方式换成ssh方式
```shell
$ vim .git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
        precomposeunicode = true
[remote "origin"]
        url = git@github.com:XXX/XXX.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```
执行命令：
```shell
$ git remote set-url origin git@github.com:XXX/XXX.git
$ git push
```
又报另一个错：
```shell
> ERROR: Permission to XXX/XXX.git denied to deploy key
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
因为之前在本机连过github，也推过代码，一开始也没想到key的问题

最后还是去github上的设置看了下，发现竟然没加ssh keys，然后想把本地~/.ssh/id_rsa.pub里的公钥加上，结果报Error: Key already in use错误
使用下面命令看看密钥用在哪儿了：
```shell
$ ssh -T -ai ~/.ssh/id_rsa git@github.com
> Hi XXX/XXX! You've successfully authenticated, but GitHub does not provide shell access.
```
发现问题了，用户名竟然是我的github名+仓库名，去github上的这个仓库看了下，还真有个key的配置，删除后，再重新配置全局的ssh keys

再执行以下命令，变正常了，push也成功了
```shell
$ ssh -T -ai ~/.ssh/id_rsa git@github.com
> Hi XXX! You've successfully authenticated, but GitHub does not provide shell access.
```

参考：
https://help.github.com/en/articles/error-permission-denied-publickey
https://help.github.com/en/articles/error-key-already-in-use

-----