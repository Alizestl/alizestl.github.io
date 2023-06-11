---
title: Hexo搭建
date: 2023-05-17 18:06:24
tags: Tech
thumbnail: 
excerpt: "Simply blog by Hexo."
---

# *Blog Frame*

Hexo + nginx

云服务器部署	

![框架](/images/原理图.png)




# *环境*

本地：windows10
服务器：华为云	 centos7.9	CPU 1核  内存 2GiB  系统盘 40GiB  带宽 1Mbit/s




# *本地 Hexo部署*

## 1. 基础环境

### node

```
# 查看版本
node -v
npm -v

# 更改镜像源
npm config set registry https://registry.npm.taobao.org

# 确认镜像源
npm config list 
```

### git

```
# windows安装git
参考：https://blog.csdn.net/mukes/article/details/115693833

# 修改配置文件的信息，作为git操作的标识
git config --global user.name "user"
git config --global user.email user_mail

# 本地生成ssh密钥，默认在C:\Users\User\.ssh生成公钥文件id_rsa.pub和私钥文件id_rsa
ssh-keygen -t rsa -C “your_github_email”（-C commit:注释，可省略）
```



## 2. 创建Hexo框架

```
npm install -g hexo-cli hexo-server
hexo init <folder_name>
cd <folder.name>
npm install
npm install hexo-deployer-git --save
```



## 3. 配置git提交

```
# hexo根目录，配置_config.yml,末尾添加
deploy:
    type: git
    repo:git@github.com:yourname/yourname.github.io.git
    branch:master/main
```

```
# hexo部署命令
hexo s 
hexo clean && hexo g -d 
```

# *服务器端*

## 1. Nginx

### 安装、测试、配置

```
# 安装依赖
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel make

# 下载、解压 Nginx源代码
wget -c http://nginx.org/download/nginx-1.20.2.tar.gz
tar -zxvf nginx-1.20.2.tar.gz -C /usr/local

# 进入源码文件夹，执行配置文件
cd /usr/local/nginx-1.20.2
./configure --with-http_ssl_module 

# 编译安装
make
make install

# 默认安装目录是/usr/local/nginx,执行需要在/usr/local/nginx/sbin目录下

# 为了方便，创建alias
vim ~/.bashrc
alias nginx='/usr/local/nginx/sbin/nginx'
source ~/.bashrc

# 测试
nginx -v
nginx
# 访问 http://ip:80 查看

# nginx常用命令
nginx -s stop     	#关闭nginx
nginx -s reload     #重载nginx
nginx -t  			#检查nginx配置文件的语法是否有错误，常在修改配置文件后执行

# 配置反向代理
vim /usr/local/nginx/conf/nginx.conf

# server语句块
# listen为监听端口，默认80
# server_name 修改为自己注册的域名
# location下的root为部署根目录，即接下来要创建的Hexo部署目录

# 检查语法、重载
nginx -t
nginx -s reload
```



## 2. 创建Hexo部署目录

```
mkdir -p /home/blog/hexo
```



## 3. 创建Git仓库

### 配置git用户

```
# 添加用户
adduesr git
passwd git

# 给git用户root权限
vim /etc/sudoers

# 在 root ALL=(ALL) ALL 语句下添加 git ALL=(ALL) ALL 
```

### 创建仓库并配置自动部署

```
mkdir -p /home/repo
cd /home/repo
git init --bare blog.git
vim /home/repo/blog.git/hooks/post-receive
#输入
#!/bin/sh
git --work-tree=<hexo部署目录> --git-dir=<Git仓库>.git checkout -f
这里的话应该是
git --work-tree=/home/blog/hexo --git-dir=/home/repo/blog.git checkout -f

# 配置权限
chmod +x /home/repo/blog.git/hooks/post-receive     #为钩子文件授予可执行权限（+x）
chown -R git:git /home/repo     					#将仓库目录的所有权移交给git用户
chown -R git:git /home/blog/hexo     				#将hexo部署目录的所有权移交给git用户
```

### 给git用户添加SSH密钥

```
# 将windows本地生成的公钥文件id_rsa.pub内容复制到此处
vim /home/git/.ssh/authorized_keys

# 设置权限
chmod 600 /home/git/.ssh/authorized_keys
chmod 700 /home/git/.ssh

# 移交文件夹所有权
chown -R git:git /home/git/.ssh

# 本地测试
 ssh -v git@server_ip
```







# 参考

[Hexo官方文档](https://hexo.io/zh-cn/)
https://www.glimound.com/build-hexo-blog/
https://juejin.cn/post/6844904131266609165
https://juejin.cn/post/6844904074140205069




























