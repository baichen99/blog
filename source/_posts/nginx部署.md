---
title: flask+gunicorn+nginx部署
date: 2019-12-29 10:43:00
tags:
- flask
- vue
- nginx
- 部署
categories:
- 运维
preview: 100
---




1. 买服务器
2. ubuntu系统
3. 开启80 8080端口 [教程](https://www.cnblogs.com/codeman-hf/p/10535923.html)


1. 安装nginx

   ```shell
   apt update
   apt upgrade
   sudo apt install nginx
   ```

2. 测试nginx

   ```shell
   sudo service nginx restart
   ```

   然后访问ip地址 [如何查看公网ip](https://blog.csdn.net/ssssSFN/article/details/89501469)

3. 安装docker [docker安装教程](https://www.runoob.com/docker/ubuntu-docker-install.html)

4. 编写dockerfile文件

   ```dockerfile
   FROM python:3.8
   RUN mkdir /home/web && cd /home/web
   WORKDIR /home/web
   COPY ./backend/* /home/web/
   RUN pip install -r ./requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/  \
       && pip install gunicorn
   
   EXPOSE 8080
   CMD [ "gunicorn", "-w", "5",  "api:app"]
   
   ```

5. 

   ```shell
   docker build . flask:0.1
   docker save [image_id] > docker.tar
   ```

6. 在服务器

   ```
   docker load -i docker.tar
   ```

7. `npm run build && scp dist/* root@ip:path/`

8. 编写nginx配置

   ```
   
   server {
     listen       80;
   	server_name  ipAddr;
   
   	location / {
   		root /root/app;
   		index index.html;
   	}
   }
   
   server {
   	listen       8081;
   	server_name  ipAddr;
   
   	location / {
   		proxy_pass  http://127.0.0.1:8080
   		proxy_set_header Host $host;
   		proxy_set_header X-Forwarded-For
   		$proxy_add_x_forwarded_for;
   }
   ```

9. `docker run -d -p 8080:8080 image_id gunicorn -w 4 api:app`

   报错说明端口占用

   `ps aux | grep 8080`

   `ps -ef | grep 8080`

## 参考内容
1. [Vue+Flask部署到阿里云服务器](https://www.cnblogs.com/doocool/p/8847288.html)
