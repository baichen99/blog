---
title: docker笔记
date: 2021-02-21 22:43:20
tags:
- docker
categories:
- docker

---

## 常用命令

### 后台运行

1. 首先可以`docker run -it <image_id> bash`，在容器运行bash。
2. 如果想退出docker但是不停止容器，可以按ctrl+p+q来退出
3. 如果想回到容器继续执行指令，`docker attatch <container_id>`

### 挂载文件夹

1. 运行之前挂载
	`docker run -v /src/example/:/dest/folder <image> <command>`
2. 通过修改配置来挂载
	```shell
		systemctl stop docker.service
		vim /var/lib/docker/containers/container-ID/config.v2.json
		systemctl start docker.service
		docker start <container-name/ID>
	```


## dockerfile

### 设置时区

```
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone
```
[Dockerfile 时区设置](https://www.cnblogs.com/linjiqin/p/10607800.html)

