---
title: zero-tier使用
date: 2021-02-06 20:35:36
tags:
- zero tier
- 内网穿透
- 远程桌面
categories:
- tools

---

## 我的情况

设备：PC, mbp, vps

需求：想通过PC直接远程mbp

## 流程



## 准备工作

官网注册，并且创建一个网络，这个网络有个ID，

### 创建moon节点
[参考这篇文章](https://www.cnblogs.com/Yogile/p/12642423.html)

1. 安装zerotier

```shell
curl -s https://install.zerotier.com/ | sudo bash
```

加入网络，`zerotier-cli join <network id>`

2. 生成moon模板

```shell
cd /var/lib/zerotier-one
zerotier-idtool initmoon identity.public > moon.json
```

3. 修改moon.json 的endpoint选项`"stableEndpoints": [ "x.x.x.x/9993" ]` 9993是zerotier端口，记得在vps开放udp端口

4. 生成签名文件 `zerotier-idtool genmoon moon.json
  `  之后会生成一个000000xxxx.moon 文件。

5. **将 moon 节点加入网络**

  在 VPS 的 Zerotier 安装目录下（/var/lib/zerotier-one）建立文件夹 moons.d，将生成的 .moon文件拷贝进去。`cp 0000006xxxxxxxxx.moon moons.d/
  `

  重启 zerotier，重启服务。`/etc/init.d/zerotier-one restart
  `至此，VPS 上（moon 服务器）配置完成。 记住moon.json中的id

  

### 设备端

[参考这篇文章](https://www.cnblogs.com/Yogile/p/12502311.html)

不同系统都不一样

#### MAC

- 加入moon节点  `sudo zeroiter-one orbit <id> <repeart_id>` id是moon.json中的
- 检验是否成功`sudo zeroiter-one listpeers`
- 

## 文件共享

### pc -> mac

首先在PC上开启共享功能，以及选择共享文件夹 [参考此文章](https://zhuanlan.zhihu.com/p/110788184)

在zerotier官网的网络中可以看leaf的局域网ip，这个ip是固定的

之后到finder中按下command+k输入cifs://<ip>

然后对话框中输入pc的登录账户和密码就可以连接了



### mac -> pc

[参考这篇文章](http://www.xitongcheng.com/jiaocheng/dnrj_article_48868.html)

在mac上设置好共享文件夹后，直接打开pc的网络就能看到共享文件夹

## 远程桌面









